# DewSync Sensor

## **DewSync Sensor - DataLogger**

Um dispositivo poderoso e versátil projetado para facilitar a coleta precisa e confiável de dados em uma variedade de contextos e aplicações. Nosso datalogger combina a robustez e a flexibilidade da plataforma Arduino com recursos avançados de registro de dados, oferecendo uma solução completa para monitoramento e análise em tempo real.

Desenvolvido com a finalidade de atender às demandas de diversos setores, desde a pesquisa científica e o monitoramento ambiental até o controle de processos industriais e a automação residencial, nosso datalogger se destaca pela sua facilidade de uso, confiabilidade e escalabilidade.

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
6. **Alertas**: Se os parâmetros ficarem fora dos níveis de referência, o sistema emitirá um alerta sonoro e luminoso.
7. **Backup de Dados**: As leituras e timestamps serão armazenados na memória EEPROM do Arduino.

## **Diagrama Elétrico**

![Captura de tela 2024-03-18 221043](https://github.com/ConfuseKarma/DataLogger/assets/145780136/1cf86f0d-dddc-4afa-b60b-ad0ef2a1e1bb)

## Código Fonte

O código fonte está disponível no repositório público do GitHub neste [link](https://github.com/ConfuseKarma/DataLogger/blob/main/codigo-fonte.md). Certifique-se de ler e compreender os comentários no código para uma melhor compreensão do funcionamento.

