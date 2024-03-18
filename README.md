# DataLogger
Código C/C++ usado no Arduino IDE
### **Projeto de Sistemas Embarcados**

Projeto de Sistemas Embarcados realizado em grupo por:

- Aparecida
- Eduardo Accacio Spedine Toledo, RA: 081220018
- Kauê de Souza Silva, RA: 081220003
- Guilherme Rodrigues dos Santos, RA: 0812200
- Marcos Felipe Correa dos Santos, RA: 081220020

🎥 Vídeo do Projeto no _Youtube_: 

**Descrição:**

- Construção de um Data Logger para monitoramento ambiental de temperatura, umidade relativa do ar e luminosidade.

Objetivos:

- Desenvolver um PoC (Proof of Concept) baseado na plataforma Arduino UNO R3 que permita medir os valores médios de luminosidade, temperatura e umidade de um ambiente industrial.

- Calcular a média ponderada das leituras a cada minuto de operação, além de apresentá-las no display de cristal líquido (LCD) e realizar o backup na memória (EEPROM - data logger) com o timestamp e o valor da média quando os parâmetros ficarem fora dos níveis de referência.

- Soar um alerta sonoro e luminoso.

- Abertura gráfica com o logo ou slogan da empresa.

**Níveis dos Triggers (Gatilhos):**

- Faixa de Temperatura: 15 < t < 25 ºC

- Faixa de Luminosidade: 0 < l < 30%

- Faixa de Umidade: 30% < u < 50%

**Lista de Materiais e Componentes:**

- 1 MCU (Atmega 328P) - Arduino Uno R3
- 1 LDR + Resistor 10KOhm
- 1 DHT-11 (Sensor de temperatura e umidade)
- 1 LCD 16x2 - I2C
- 1 Bateria de 9V + suporte para bateria
- 1 RTC (Real Time Clock)
- 1 protoboard
- Jumpers
- LEDs
- Resistores

**Observações:**

- O projeto deve ser apresentado na forma de um hands-on no dia 21/3 (5 minutos / equipe).
- Atenção às unidades de medida e precisão dos sensores (estas informações devem estar descritas na documentação).
- Entregar o link do GitHub (público) com o arquivo readme.md contendo a documentação com as especificações técnicas do produto, diagrama elétrico e manual de operação, além do código fonte devidamente comentado.
- Será necessário gerar evidências através de um vídeo demonstrativo com 3 minutos (time-lapse) apresentando as funcionalidades do produto.
- O vídeo deve estar no YouTube (link público).
- 🔗 Forms para Entrega: [Link do Formulário de Entrega](https://forms.office.com/r/wLZ3Uu3L33)

Os grupos devem ser os mesmos do PBL (Project-based Learning).
