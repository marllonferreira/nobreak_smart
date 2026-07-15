# Projeto Nobreak Smart

Bem-vindo à documentação oficial do projeto **Nobreak Smart**. 
O objetivo deste projeto é documentar e executar a transformação de um nobreak comum em um dispositivo inteligente (smart), com foco em **monitoramento passivo** avançado e integração com automações residenciais/locais. Não haverá controle invasivo da placa (botões ou relés), preservando o funcionamento original do aparelho.

## 📚 Índice
- [Visão Geral](#visão-geral)
- [Arquitetura e Hardware](#arquitetura-e-hardware)
- [Software e Firmware](#software-e-firmware)
- [Checklist e Passo a Passo](CHECKLIST.md)
- [Guia de Montagem e Configuração](MONTAGEM_E_CONFIGURACAO.md)

## 🎯 Visão Geral
O projeto consiste em pegar um nobreak comum (pequeno porte, em torno de 600 VA, entrada 220V e saída 110V) e embarcar eletrônica inteligente. O objetivo principal é torná-lo capaz de fornecer informações operacionais de telemetria em tempo real, monitorando tanto a rede elétrica externa (AC) quanto seu circuito interno de baterias (DC). Dessa forma, o nobreak se torna um monitor geral para identificar quedas de rede e também um analista de autonomia, estimando quanto tempo de bateria ainda resta durante um blecaute.

## 🏗️ Arquitetura e Hardware
O hardware principal da modificação consistirá em:

- **Microcontrolador (Cérebro):**
  Utilizaremos a placa **ESP32 DevKit V1 (38 pinos)**. Ela lerá os sensores e conectará o sistema ao Wi-Fi.
- **Alimentação Estabilizada:**
  Para manter o ESP32 e os sensores ligados, usaremos um módulo Buck Converter **LM2596 DC-DC**. Ele será ligado diretamente nos terminais da bateria do nobreak. Dessa forma, mesmo que a bateria sofra queda de tensão ao descarregar durante o uso, o LM2596 garantirá os 5V/3.3V constantes para o circuito de controle.

- **Monitoramento AC (Entrada Externa):** 
  Utilização do sensor **HLW8032** (ou equivalente) para monitorar a rede da concessionária. Ele medirá a presença de energia, a tensão da rede, a corrente e a potência que o nobreak está consumindo da tomada.
- **Monitoramento DC (Bateria Interna):** 
  Utilização do sensor **INA226** (ou família similar) conectado à bateria e ao circuito interno. Como o resistor original do módulo INA não suporta a alta corrente do nobreak, faremos uma **adaptação utilizando um resistor shunt externo (75 mV / 15 A)** em substituição ao original. O sensor será responsável por ler a tensão da bateria, a corrente e o consumo interno DC (equivalente ao consumo AC), possibilitando calcular a estimativa real de tempo de vida e descarga da bateria.
- **Monitoramento Térmico (Interno):**
  Adição do sensor de temperatura e umidade **DHT11**. Apesar de ser um sensor simples, ele é perfeitamente adequado para monitorar o calor dentro da carcaça do nobreak, prevenindo superaquecimento na bateria e no circuito inversor.

## 💻 Software e Firmware
O sistema operacional (firmware) escolhido para o ESP32 é o **Tasmota**.
Isso vai requerer atenção em dois pontos cruciais:
1. **Compilação Customizada (TasmoCompiler):** A versão padrão (binário genérico) do Tasmota não ativa todos os sensores nativamente para poupar espaço. Ao utilizar o **TasmoCompiler**, atente-se aos seguintes detalhes:
   - Comunicação **MQTT**: Necessária caso queira integrar o nobreak ao Home Assistant (opcional, o Tasmota também opera 100% de forma standalone/independente).
   - Driver do sensor **HLW8032 / CSE7766** (AC): **(Já é nativo)** O Tasmota base já traz suporte padrão a ele (lido sob o driver do chip `CSE7766 Rx`).
   - Driver do sensor **INA226** (DC): **(Requer ativação explícita)** Não vem por padrão na versão base. É estritamente necessário marcá-lo/incluí-lo na customização.
   - Driver do sensor **DHT11** (Temp/Umid): **(Requer ativação explícita)** Também precisa ser incluído no pacote da compilação.
2. **Mapeamento Seguro de Pinos (GPIO):** Como o ESP32 possui pinos sensíveis no boot (strapping pins), definimos a seguinte pinagem segura para evitar falhas de inicialização:
   - **Sensor INA226 (I2C):** **GPIO 21** (SDA) e **GPIO 22** (SCL). Pinos padrão I2C do ESP32, totalmente seguros.
   - **Sensor HLW8032 (UART/Serial):** **GPIO 16** (RX2). O HLW apenas envia dados, e o GPIO 16 é seguro como entrada sem interferir no boot.
   - **Sensor DHT11 (Digital 1-Wire):** **GPIO 4**. Pino genérico de I/O, muito estável e seguro para leitura digital sem atrapalhar a inicialização.

## 🚀 Etapas de Desenvolvimento
O processo de transformação será dividido em etapas estruturadas e documentadas passo a passo. 
- Para acompanhar todas as tarefas de forma iterativa, acesse nosso arquivo de [Checklist](CHECKLIST.md).
- Para o esquema elétrico detalhado, mapeamento na interface Tasmota e comandos de calibração, acesse o [Guia de Montagem e Configuração](MONTAGEM_E_CONFIGURACAO.md).
