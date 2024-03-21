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

**• 'unsigned long intervalLeituras', 'ultimoMillisModoAtual' e 'ultimoMillisBotao':** Essas variáveis são utilizadas para armazenar o tempo em milissegundos da última execução de diferentes tarefas relacionadas às leituras dos sensores, atualização do modo de operação e detecção do botão.

**• 'const long ultimoMillisLeituras', 'intervalModo' e 'intervalBotao':** Essas constantes definem os intervalos de tempo em milissegundos para realizar as diferentes tarefas no programa. Por exemplo, intervalLeituras define o intervalo de tempo para as leituras dos sensores.

**• 'int modoIDGlobal':** Esta variável inteira mantém o modo atual de operação do sistema. Ela é incrementada conforme o botão é pressionado e utilizada para determinar qual modo de operação está ativo.

**• 'const int botaoPin':** Define o pino digital utilizado para a detecção do botão. Neste caso, o botão está conectado ao pino digital 2 do Arduino.

**• 'int ultimoEstadoBotao':** Armazena o estado anterior do botão para comparar com o estado atual e determinar se o botão foi pressionado.

**• 'struct Anomalia { ... };':** É uma estrutura de dados (struct) utilizada para armazenar informações sobre anomalias detectadas de data, temperatura, umidade e luminosidade no momento da ocorrência.
‎‎‎‎‎‎‎‎ㅤ

**Seção DHT:**

DHTPIN e DHTTYPE: Essas macros são utilizadas para definir o pino ao qual o sensor DHT está conectado (DHTPIN) e o tipo de sensor DHT (DHTTYPE). No caso, o sensor está conectado ao pino analógico A1 e é do tipo DHT22. A escolha entre DHT11 e DHT22 pode ser alternada através das definições.
