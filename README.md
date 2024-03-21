# DewSync Sensor

## **DewSync Sensor - DataLogger**

Um dispositivo eficiente e versátil projetado para facilitar a coleta precisa de dados em uma variedade de contextos e aplicações. Nosso datalogger combina a robustez e a flexibilidade da plataforma Arduino com recursos avançados de registro de dados, oferecendo uma solução completa para monitoramento e análise em tempo real.

Desenvolvido com a finalidade de atender às demandas de diversos setores, desde a pesquisa científica e o monitoramento ambiental até o controle de processos industriais e a automação residencial, nosso datalogger se destaca pela sua facilidade de uso, confiabilidade e escalabilidade.

🎥 Vídeo do Projeto no _Youtube_: 

## Especificações Técnicas

### Sensores Utilizados

- Sensor de Luminosidade: LDR (Light Dependent Resistor) + Resistor 10KOhm
- Sensor de Temperatura e Umidade: DHT-11

### Componentes Adicionais

- MCU (Microcontrolador): Atmega 328P (Arduino Uno R3)
- Display de Cristal Líquido (LCD): LCD 16x2 - I2C
- Bateria de Alimentação: 9V + Suporte para Bateria
- RTC (Real Time Clock)
- Protoboard
- Jumpers
- LEDs
- Resistores


## Manual de Operação

1. **Montagem do Circuito**: Monte o circuito conforme o diagrama elétrico fornecido.
2. **Upload do Código**: Faça o upload do código fonte para o Arduino UNO R3.
3. **Alimentação**: Conecte o Arduino UNO R3 a uma fonte de alimentação adequada.
4. **Monitoramento**: O sistema começará a monitorar os valores de luminosidade, temperatura e umidade do ambiente.
5. **Visualização de Dados**: Os valores médios serão apresentados no display de cristal líquido (LCD).
6. **Botão Interativo**: Utilize o botão de seleção para escolher o dado que deseja visualizar (temperatura, umidade, luminosidade).
7. **Alertas**: Se os parâmetros ficarem fora dos níveis de referência, o sistema emitirá um alerta sonoro e luminoso.
8. **Backup de Dados**: As leituras e timestamps serão armazenados na memória EEPROM do Arduino.

## **Diagrama Elétrico**

<img src="https://github.com/ConfuseKarma/DataLogger/assets/145780136/3f22056d-c2c8-41d3-9c4f-ab6cc2ee0b94" alt="arduino-circuito" width="75%">

https://wokwi.com/projects/392907885243715585


## Código Fonte

O código fonte está disponível no repositório público do GitHub neste [link](https://github.com/ConfuseKarma/DataLogger/blob/main/codigo-fonte.md). Certifique-se de ler e compreender os comentários no código para uma melhor compreensão do funcionamento.


## Lógica do Código Fonte

### Inclusão de Bibliotecas e Definição de Constantes e Variáveis Globais:

O código começa incluindo diversas bibliotecas necessárias para o funcionamento do programa, como **Wire** (para comunicação I2C), **LiquidCrystal_I2C** (para controle de um display LCD), **RTClib** (para manipulação do RTC), **DHT** (para o sensor de temperatura e umidade) e **EEPROM** (para acessar a memória EEPROM do Arduino). Essas bibliotecas fornecem funções e métodos que facilitam o acesso e o controle dos dispositivos e sensores conectados ao Arduino.

Em seguida, são definidas constantes e variáveis globais que serão utilizadas ao longo do programa:

**SEÇÃO GERAL:**
- **'unsigned long intervalLeituras', 'ultimoMillisModoAtual' e 'ultimoMillisBotao':** Essas variáveis são utilizadas para armazenar o tempo em milissegundos da última execução de diferentes tarefas relacionadas às leituras dos sensores, atualização do modo de operação e detecção do botão.
- **'const long ultimoMillisLeituras', 'intervalModo' e 'intervalBotao':** Essas constantes definem os intervalos de tempo em milissegundos para realizar as diferentes tarefas no programa. Por exemplo, intervalLeituras define o intervalo de tempo para as leituras dos sensores.
- **'int modoIDGlobal':** Esta variável inteira mantém o modo atual de operação do sistema. Ela é incrementada conforme o botão é pressionado e utilizada para determinar qual modo de operação está ativo.
- **'const int botaoPin':** Define o pino digital utilizado para a detecção do botão. Neste caso, o botão está conectado ao pino digital 2 do Arduino.
- **'int ultimoEstadoBotao':** Armazena o estado anterior do botão para comparar com o estado atual e determinar se o botão foi pressionado.
- **'struct Anomalia { ... };':** É uma estrutura de dados (struct) utilizada para armazenar informações sobre anomalias detectadas de data, temperatura, umidade e luminosidade no momento da ocorrência.‎‎‎‎‎‎‎‎

**SEÇÃO DHT:**
- **'#define DHTPIN' e 'DHTTYPE':** Essas macros são utilizadas para definir o pino ao qual o sensor DHT está conectado (DHTPIN) e o tipo de sensor DHT (DHTTYPE). No caso, o sensor está conectado ao pino analógico A1 e é do tipo DHT22. A escolha entre DHT11 e DHT22 pode ser alternada através das definições.
- **'DHT dht(DHTPIN, DHTTYPE)':** Aqui, é criado um objeto da classe DHT utilizando as informações definidas anteriormente. Esse objeto será usado para interagir com o sensor DHT e realizar as leituras de temperatura e umidade.

**CONSTANTES E VARIÁVEIS PARA ARMAZENAMENTO DE VALORES DOS SENSORES:**
- **'#define NUM_VALORES':** Define o número de valores a serem considerados para cálculos de médias.
- **'int valoresTemperatura[]', 'valoresUmidade[]', 'valoresLuminosidade[]':** São arrays utilizados para armazenar os últimos valores lidos dos sensores de temperatura, umidade e luminosidade, respectivamente.
- **'int pesos[]':** Define os pesos a serem utilizados no cálculo da média ponderada dos valores lidos. Neste caso, são pesos decrescentes que favorecem os valores mais recentes.

**SEÇÃO MODO PÂNICO:**
- **'int redLedPin' e 'buzzerPin':** Definem os pinos aos quais o LED vermelho e o buzzer estão conectados, respectivamente.
- **'float sinVal' e 'int toneVal':** Variáveis utilizadas para controle da sirene, emitindo um som de acordo com os valores definidos.

**SEÇÃO LCD:**
- **'LiquidCrystal_I2C lcd(0x27, 16, 2)':** Aqui é criado um objeto da classe LiquidCrystal_I2C para controlar um display LCD de 16 colunas por 2 linhas, com endereço I2C 0x27. Esse objeto será usado para exibir informações no display LCD.

**SEÇÃO INTRO/LOGO:**
- São definidos arrays de bytes (**'name0x13'**, **'name0x14'**, **'name0x15'**, **'name1x13'**, **'name1x14'**, **'name1x15'**) que representam caracteres personalizados a serem exibidos no display LCD. Esses caracteres são utilizados para formar um logo ou uma introdução visual no display.

**SEÇÃO LDR:**
- **'ValorLDR:'** Variável utilizada para armazenar a leitura do sensor LDR, que mede a luminosidade.
- **'IntensidadeLuz:'** Variável que armazena a intensidade de luz transformada em uma escala de 0 a 100 para exibição no display LCD.
- **'pinoLDR:'** Define o pino analógico ao qual o sensor LDR está conectado para realizar a leitura da luminosidade.

**SEÇÃO DO RTC:**
- **'RTC_DS3231 rtc':** Cria um objeto do tipo RTC_DS3231 para interagir com o módulo RTC DS3231. Esse objeto permite obter a data e hora atuais do relógio em tempo real.

**DECLARAÇÃO DOS DIAS DA SEMANA:**
- **'char daysOfTheWeek[]':** Array de strings utilizado para armazenar os nomes dos dias da semana. Essa informação pode ser útil para exibir a data no formato completo com o dia da semana.

### Setup():

**1. INICIALIZAÇÃO DE PINOS**
 - **'pinMode(botaoPin, INPUT_PULLUP)':** Define o pino do botão como entrada com resistor pull-up interno ativado. Isso significa que o botão é conectado entre o pino **'botaoPin'** e o GND, e o resistor pull-up interno ajuda a garantir um estado lógico alto quando o botão não está pressionado.

**2. CONFIGURAÇÃO DO SENSOR LDR**
- **'Serial.begin(9600)':** Inicializa a comunicação serial a uma taxa de transmissão de 9600 bps, permitindo a comunicação com o monitor serial para depuração e exibição de informações.
- **'dht.begin()':** Inicializa o sensor DHT para leitura de temperatura e umidade.

**3. INICIALIZAÇÃO DO RTC DS3231**
- **'rtc.begin()':** Inicializa o RTC DS3231. Se não for possível inicializar o RTC, uma mensagem será exibida no monitor serial informando que o DS3231 não foi encontrado.
- **'if (rtc.lostPower()) { ... }':** Verifica se o RTC foi ligado pela primeira vez, se ficou sem energia ou se a bateria foi esgotada. Se sim, uma mensagem é exibida no monitor serial informando que o DS3231 está OK e, opcionalmente, a data e hora são ajustadas para a data e hora em que o código foi compilado **('__DATE__ e __TIME__')**, ou outra data e hora especificada.

**4. CONFIGURAÇÃO DO PINO DO LDR:**
- **'pinMode(pinoLDR, INPUT)':** Define o pino ao qual o sensor LDR está conectado como entrada, para realizar a leitura do sensor LDR posteriormente.

**5. CONFIGURAÇÃO DOS PINOS DO LED E BUZZER:**
- **'pinMode(redLedPin, OUTPUT)':** Define o pino ao qual o LED vermelho está conectado como saída, para controlar o estado do LED (ligado/desligado).
- **'pinMode(buzzerPin, OUTPUT)':** Define o pino ao qual o buzzer está conectado como saída, para controlar o som emitido pelo buzzer.

**6. INICIALIZAÇÃO DO LCD:**
- **'lcd.init()':** Inicializa o display LCD com os parâmetros especificados (endereço I2C, número de colunas e linhas).
- **'lcd.backlight()':** Ativa a luz de fundo do LCD.

**7. APRESENTAÇÃO NO LCD:**
- São exibidos o nome "DewSync" e o slogan "it just works" no LCD, seguidos de uma animação do logo em formato floco de neve.

### Loop():

**1. Leitura do Estado do Botão:**
- **'int estadoBotao = digitalRead(botaoPin)':** Lê o estado atual do botão conectado ao pino botaoPin.
- **'unsigned long currentMillis = millis()':** Obtém o tempo atual em milissegundos desde o início do programa.

**2. Verificação de Pressionamento do Botão:**
- **'if (estadoBotao == LOW && ultimoEstadoBotao == HIGH) { ... }':** Verifica se o botão foi pressionado, comparando seu estado atual com o estado anterior. Isso evita múltiplas detecções de um único pressionamento.
- **'if(currentMillis - ultimoMillisBotao >= intervalBotao) { ... }':** Verifica se passou o tempo mínimo (intervalBotao) desde o último pressionamento do botão.

**3. Alteração de Modo:**
- **'ultimoMillisBotao = currentMillis':** Atualiza o tempo da última execução da verificação do botão.
- **'modoIDGlobal++'**: Incrementa o modo de operação global.
- **'if (modoIDGlobal > 3) { modoIDGlobal = 0; }'**: Se o modo atual ultrapassar o último modo, volta ao primeiro modo.


