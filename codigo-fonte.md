### Link Wokwi (Simulador): https://wokwi.com/projects/392373342094474241


# **Código Fonte Arduino**

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "RTClib.h" //INCLUSÃO DA BIBLIOTECA
#include <DHT.h>


//Seçao DHT
#define DHTPIN A1    // Pino de dados do sensor DHT11 conectado ao pino 2
#define DHTTYPE DHT22
//#define DHTTYPE DHT11   // Tipo do sensor DHT (DHT11 ou DHT22)
DHT dht(DHTPIN, DHTTYPE);


#define NUM_VALORES 5  // Número de valores a considerar
int valoresTemperatura[NUM_VALORES] = {0};  // Array para armazenar os últimos valores de temperatura
int valoresUmidade[NUM_VALORES] = {0};  // Array para armazenar os últimos valores de umidade
int pesos[NUM_VALORES] = {1, 2, 3, 4, 5};  // Pesos decrescentes


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
 
  lcd.begin(16, 2); //Inicia o Display LCD
 
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
  FuncionamentoLDR();
  FuncionamentoDHT();
  delay(2000);
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


void FuncionamentoDHT()
{
  // Leitura da temperatura e umidade
  float temperatura = dht.readTemperature();  // Leitura da temperatura
  float umidade = dht.readHumidity();  // Leitura da umidade


  // Cálculo das médias ponderadas
  float mediaTemperatura = mediaMovelPonderada(temperatura, valoresTemperatura, NUM_VALORES);
  float mediaUmidade = mediaMovelPonderada(umidade, valoresUmidade, NUM_VALORES);


  // Utilize as médias para o que for necessário (por exemplo, verificar se está fora do padrão)
  // Neste exemplo, apenas exibiremos as médias no Monitor Serial
  Serial.print("Média de Temperatura: ");
  Serial.print(mediaTemperatura);
  Serial.println(" °C");


  Serial.print("Média de Umidade: ");
  Serial.print(mediaUmidade);
  Serial.println(" %");
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

```
