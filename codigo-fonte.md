### Link Wokwi (Simulador): https://wokwi.com/projects/392373342094474241


# **Código Fonte Arduino**

```cpp

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "RTClib.h" //INCLUSÃO DA BIBLIOTECA
#include <DHT.h>
#include <EEPROM.h> // Biblioteca para acessar a memória EEPROM

/*Ao invés de usarmos delays, adotaremos o método de multitarefa cooperativa,
aonde através de millis, mediremos o tempo de cada execução, executando as 
baseando se no momento da última execução de cada tarefa
*/

unsigned long ultimoMillisLeituras = 0;  
unsigned long ultimoMillisModoAtual = 0;
unsigned long ultimoMillisBotao = 0; 
const long intervalLeituras = 12000;
const long intervalModo = 1000;
const long intervalBotao = 500;


/*Implementaremos também três modos de uso: Modo DHT (exibe termometro e humidade),
Modo Relógio(Exibe Data e hora) e modo Luxômetro (exibe a intensidade de luz percentual).
Independentemente do modo de operação, ocorrerão leituras dos valores capturados pelos 
sensores, que quando fora dos parâmetros especificados, registrarão uma ocorrência na EEPROM
do arduìno e ativarão o modo Pânico (indicação sonora e visual da irregularidade);
*/

int modoIDGlobal = 0;
const int botaoPin = 2;
int ultimoEstadoBotao = LOW; 

struct Anomalia {
  String anomaliaDate;
  float temperatura;
  float umidade;
  int luminosidade;
};

//Seçao DHT
#define DHTPIN A1    // Pino de dados do sensor DHT11 conectado ao pino 2
#define DHTTYPE DHT22 //Mudar para dht11 no modelo físico
//#define DHTTYPE DHT11   // Tipo do sensor DHT (DHT11 ou DHT22)
DHT dht(DHTPIN, DHTTYPE);

#define NUM_VALORES 5  // Número de valores a considerar
int valoresTemperatura[NUM_VALORES] = {0};  // Array para armazenar os últimos valores de temperatura
int valoresUmidade[NUM_VALORES] = {0};  // Array para armazenar os últimos valores de umidade
int valoresLuminosidade[NUM_VALORES] = {0};
int pesos[NUM_VALORES] = {1, 2, 3, 4, 5};  // Pesos decrescentes

//seção modo pânico
int redLedPin = 8;
int buzzerPin = 5;
float sinVal; //coisa da sirene
int toneVal;

//Seção LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);  // Set the LCD address to 0x27 for a 16x2 display

//Seção Intro/LOGO
byte name0x13[] = { B00000, B00000, B01010, B00100, B01010, B00001, B00010, B10100 };
byte name0x14[] = { B00000, B01010, B00100, B01110, B10101, B00100, B10101, B01010 };
byte name0x15[] = { B00000, B00000, B01010, B00100, B01010, B10000, B01000, B00101 };
byte name1x13[] = { B01111, B10100, B00010, B00001, B01010, B00100, B01010, B00000 };
byte name1x14[] = { B10101, B01010, B10101, B00100, B10101, B01110, B00100, B01010 };
byte name1x15[] = { B11110, B00101, B01000, B10000, B01010, B00100, B01010, B00000 };

//Seção LDR
int ValorLDR; //Armazenar a leitura do sensor LDR
int IntensidadeLuz; //Transforma a leitura em uma escala de 0 a 100
float pinoLDR = A0; //PINO ANALÓGICO utilizado para ler o LDR

//Seção do RTC
RTC_DS3231 rtc; //OBJETO DO TIPO RTC_DS3231

//DECLARAÇÃO DOS DIAS DA SEMANA
char daysOfTheWeek[7][12] = {"Domingo", "Segunda", "Terça", "Quarta", 
"Quinta", "Sexta", "Sábado"};

void setup() {
  pinMode(botaoPin, INPUT_PULLUP);

  //Setup LDR
  Serial.begin(9600); //Define a velocidade do monitor serial
  dht.begin();

  if(! rtc.begin()) { // SE O RTC NÃO FOR INICIALIZADO, FAZ
    Serial.println("DS3231 não encontrado"); //IMPRIME O TEXTO NO MONITOR SERIAL
    while(1); //SEMPRE ENTRE NO LOOP
  }

  if(rtc.lostPower()){ //SE RTC FOI LIGADO PELA PRIMEIRA VEZ / FICOU SEM ENERGIA / ESGOTOU A BATERIA, FAZ
    Serial.println("DS3231 OK!"); //IMPRIME O TEXTO NO MONITOR SERIAL
    //REMOVA O COMENTÁRIO DE UMA DAS LINHAS ABAIXO PARA INSERIR AS INFORMAÇÕES ATUALIZADAS EM SEU RTC
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__))); //CAPTURA A DATA E HORA EM QUE O SKETCH É COMPILADO
    //rtc.adjust(DateTime(2018, 9, 29, 15, 00, 45)); //(ANO), (MÊS), (DIA), (HORA), (MINUTOS), (SEGUNDOS)
  }

  delay(1000);

  pinMode(pinoLDR, INPUT); //Define o pino que o LDR está ligado como entrada de dados
  
  pinMode(redLedPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  
  lcd.init(); //Inicia o Display LCD
  lcd.backlight();

  //Logo e slogan
  lcd.setCursor(0,0);
  lcd.print("DewSync");

  delay(500);
  lcd.setCursor(0,1);
  lcd.print("it just works");

  lcd.createChar(0, name0x13);
  lcd.setCursor(13, 0);
  lcd.write(0);
  
  lcd.createChar(1, name0x14);
  lcd.setCursor(14, 0);
  lcd.write(1);
  
  lcd.createChar(2, name0x15);
  lcd.setCursor(15, 0);
  lcd.write(2);
  
  lcd.createChar(3, name1x13);
  lcd.setCursor(13, 1);
  lcd.write(3);
  
  lcd.createChar(4, name1x14);
  lcd.setCursor(14, 1);
  lcd.write(4);
  
  lcd.createChar(5, name1x15);
  lcd.setCursor(15, 1);
  lcd.write(5);
  delay(2000);

  lcd.clear();
}

void loop(){
  int estadoBotao = digitalRead(botaoPin);
  unsigned long currentMillis = millis();

  // Verifica se o botão foi pressionado
  if (estadoBotao == LOW && ultimoEstadoBotao == HIGH) {
    
    // Incrementa o modo
    if(currentMillis - ultimoMillisBotao >= intervalBotao){

      ultimoMillisBotao = currentMillis;
      
      modoIDGlobal++;

      if (modoIDGlobal > 3) { // Se atingir o último modo, retorna ao primeiro
        modoIDGlobal = 0;
      }

      // Mostra o modo atual
      Serial.print("Modo atual: ");
      Serial.println(modoIDGlobal);
    }
  }
  
  ultimoEstadoBotao = estadoBotao;
  //Serial.print(estadoBotao);
  //Serial.print(ultimoEstadoBotao);
  

  if (currentMillis - ultimoMillisModoAtual >= intervalModo) {
    
    ultimoMillisModoAtual = currentMillis;

    if(modoIDGlobal == 0){
      FuncionamentoRTC();
    }
    if(modoIDGlobal == 1){
      FuncionamentoLDR();
    }
    if(modoIDGlobal == 2){
      FuncionamentoDHT();
    }
    if(modoIDGlobal == 3){
      ExibeEEPROM();
    }
  }

  if (currentMillis - ultimoMillisLeituras >= intervalLeituras) {
    // Atualiza o tempo da última execução da função 2
    ultimoMillisLeituras = currentMillis;

    Leituras();
  }
}

void FuncionamentoLDR(){
  ValorLDR = analogRead(pinoLDR); //Faz a leitura do sensor, em um valor que pode variar de 0 a 1023
  IntensidadeLuz = map(ValorLDR, 0, 1023, 1, 100); //Converte o valor para uma escala de 0 a 100
  
  
  lcd.clear();

  lcd.setCursor(0,0);
  lcd.print("Intensidade = "); //Imprime texto “Intensidade de Luz = ” na tela
  lcd.setCursor(0,1);
  lcd.print(IntensidadeLuz); //Imprime o valor lido na tela após o sinal de igual do texto acima
}

void FuncionamentoDHT(){
  // Leitura da temperatura e umidade
  float temperatura = dht.readTemperature();  // Leitura da temperatura
  float umidade = dht.readHumidity();  // Leitura da umidade

  // Utilize as médias para o que for necessário (por exemplo, verificar se está fora do padrão)
  // Neste exemplo, apenas exibiremos as médias no Monitor Serial
  lcd.clear();
  
  lcd.setCursor(0,0);
  lcd.print("Temp.: ");
  lcd.print(temperatura);
  lcd.println(" C");

  lcd.setCursor(0,1);
  lcd.print("Umid.: ");
  lcd.print(umidade);
  lcd.print(" %"); 
}

void FuncionamentoRTC(){
  DateTime now = rtc.now(); //CHAMADA DE FUNÇÃO

  lcd.clear();
  lcd.setCursor(0,0);

  lcd.print(now.day(), DEC); //IMPRIME NO MONITOR SERIAL O DIA
  lcd.print('/'); //IMPRIME O CARACTERE NO MONITOR SERIAL
  lcd.print(now.month(), DEC); //IMPRIME NO MONITOR SERIAL O MÊS
  lcd.print('/'); //IMPRIME O CARACTERE NO MONITOR SERIAL
  lcd.print(now.year(), DEC); //IMPRIME NO MONITOR SERIAL O ANO
  lcd.print(" quinta"); //IMPRIME NO MONITOR SERIAL O DIA

  lcd.setCursor(0,1);
  lcd.print("Horas: "); //IMPRIME O TEXTO NA SERIAL
  lcd.print(now.hour(), DEC); //IMPRIME NO MONITOR SERIAL A HORA
  lcd.print(':'); //IMPRIME O CARACTERE NO MONITOR SERIAL
  lcd.print(now.minute(), DEC); //IMPRIME NO MONITOR SERIAL OS MINUTOS
  lcd.print(':'); //IMPRIME O CARACTERE NO MONITOR SERIAL
  lcd.print(now.second(), DEC); //IMPRIME NO MONITOR SERIAL OS SEGUNDOS
}

// Função para calcular a média ponderada
float mediaMovelPonderada(int novoValor, int valores[], int numValores) {
    float soma = 0;
    float somaPesos = 0;
    // Shift nos valores antigos e inserção do novo valor
    for (int i = numValores - 1; i > 0; i--) {
        valores[i] = valores[i - 1];
    }
    valores[0] = novoValor;

    // Cálculo da média ponderada
    for (int i = 0; i < numValores; i++) {
        soma += valores[i] * pesos[i];
        somaPesos += pesos[i];
    }
    return soma / somaPesos;
}

void Leituras(){
  // Leitura da temperatura e umidade
  float temperatura = dht.readTemperature();  // Leitura da temperatura
  float umidade = dht.readHumidity();  // Leitura da umidade
  ValorLDR = analogRead(pinoLDR);
  IntensidadeLuz = map(ValorLDR, 0, 1023, 1, 100);
  
  // Cálculo das médias ponderadas
  float mediaTemperatura = mediaMovelPonderada(temperatura, valoresTemperatura, NUM_VALORES);
  float mediaUmidade = mediaMovelPonderada(umidade, valoresUmidade, NUM_VALORES);
  float mediaLuminosidade = mediaMovelPonderada(IntensidadeLuz, valoresLuminosidade, NUM_VALORES);

  if(mediaLuminosidade<25||mediaTemperatura>25||mediaUmidade>50)
  {
    String horario = retornarDataHora(rtc.now());
    salvarAnomaliaNaEEPROM(horario, mediaTemperatura, mediaUmidade, mediaLuminosidade);
    ModoPanico();
  }

  // Utilize as médias para o que for necessário (por exemplo, verificar se está fora do padrão)
  // Neste exemplo, apenas exibiremos as médias no Monitor Serial
  /*
  Serial.print("Média de Temperatura: ");
  Serial.print(mediaTemperatura);
  Serial.println(" °C");

  Serial.print("Média de Umidade: ");
  Serial.print(mediaUmidade);
  Serial.println(" %");

  Serial.print("Média de Luminosidade: ");
  Serial.print(mediaLuminosidade);
  Serial.println(" %");
  */
}

void ModoPanico(){
  digitalWrite(redLedPin, HIGH);
  Sirene();
}

void Sirene(){
  tone(5,1000);
  delay(300);
  noTone(buzzerPin);
}

void salvarAnomaliaNaEEPROM(String data, float temperatura, float umidade, float luminosidade) {
  Anomalia novaAnomalia;
  novaAnomalia.anomaliaDate = data;
  novaAnomalia.temperatura = temperatura;
  novaAnomalia.umidade = umidade;
  novaAnomalia.luminosidade = luminosidade;

  // Escrever na EEPROM a partir do endereço 0
  escreverEEPROM(0, novaAnomalia);
}

void escreverEEPROM(int endereco, Anomalia dados) {
  EEPROM.put(endereco, dados);

  Serial.print(dados.anomaliaDate); //IMPRIME NO MONITOR SERIAL O DIA
  Serial.print('/'); //IMPRIME O CARACTERE NO MONITOR SERIAL
  Serial.print(dados.luminosidade);
}

Anomalia lerEEPROM(int endereco) {
  Anomalia dados;
  EEPROM.get(endereco, dados);

  Serial.print(dados.anomaliaDate); //IMPRIME NO MONITOR SERIAL O DIA
  Serial.print('/'); //IMPRIME O CARACTERE NO MONITOR SERIAL
  Serial.print(dados.luminosidade); 

  return dados;
}

void ExibeEEPROM(){
  Anomalia a = lerEEPROM(0);

  lcd.clear();
  lcd.setCursor(0,0);  
  
  lcd.print(a.temperatura);
  lcd.print('/');
  lcd.print(a.umidade);
  lcd.print('/');
  lcd.print(a.luminosidade);
}

String retornarDataHora(DateTime now) {
  // Cria uma string para armazenar a data e hora
  String dataHora = "";

  // Concatena os componentes da data e hora na string
  dataHora += String(now.year()) + "-";
  dataHora += String(now.month()) + "-";
  dataHora += String(now.day()) + " ";
  dataHora += String(now.hour()) + ":";
  dataHora += String(now.minute()) + ":";
  dataHora += String(now.second());

  // Imprime a data e hora atual no Serial Monitor
  return dataHora;
}



```
