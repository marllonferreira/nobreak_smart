# Checklist de Transformação: Nobreak Smart

Este documento contém a listagem de todas as etapas necessárias para a modificação do nobreak. 
Ele será estruturado e detalhado conforme formos alinhando cada parte do projeto.

## 🛠️ 1. Planejamento e Lista de Materiais
- [ ] Definir o modelo exato do Nobreak (base: ~600 VA, entrada 220V / saída 110V).
- [ ] Obter microcontrolador principal: **ESP32 DevKit V1 (38 pinos)**.
- [ ] Obter Módulo Regulador de Tensão: **LM2596 (Buck Converter DC-DC)** para alimentar o ESP32 a partir da bateria do nobreak.
- [ ] Obter Sensor de Rede AC: **HLW8032** (ou equivalente) para leitura de tensão, corrente e potência de entrada.
- [ ] Obter Sensor de Bateria DC: **INA226** (ou família INA) para leitura da bateria (tensão, corrente, consumo interno).
- [ ] Obter **Resistor Shunt Externo (75 mV / 15 A)** para substituir o resistor interno de baixa capacidade do módulo INA226.
- [ ] Obter Sensor de Temperatura/Umidade: **DHT11** (para monitorar o calor interno da carcaça do nobreak).
- [ ] Listar as ferramentas adicionais necessárias para a modificação (ferro de solda, multímetro, chaves, fios).
- [ ] Adquirir e separar todos os materiais para o início da parte prática.

## 🔬 2. Análise e Mapeamento (Fiações de Intervenção)
- [ ] Desmontar o nobreak com segurança (⚠️ **Atenção aos riscos elétricos da bateria mesmo fora da tomada!**).
- [ ] Identificar a **Fiação de Entrada AC** (após o cabo de força) para inserir/derivar a ligação do sensor HLW8032.
- [ ] Identificar a **Fiação da Bateria DC** para interceptar o polo com o Resistor Shunt e ligar o módulo LM2596.
- [ ] Definir o local de fixação do **DHT11** no interior do nobreak (preferencialmente próximo à bateria, mas sem tocar em dissipadores de calor extremos).
- [ ] *Nota Técnica: A intervenção no hardware original é mínima. Não faremos soldas na placa-mãe nem tentaremos controlar o liga/desliga ou os relés. É um sistema puramente focado em monitoramento passivo.*

## 💻 3. Firmware e Configuração (Tasmota)
- [ ] Utilizar o **TasmoCompiler** (local via Docker/CasaOS ou nuvem via GitHub Codespaces) para gerar o binário leve.
- [ ] Configurar os parâmetros do TasmoCompiler:
  - **Tipo de ESP32:** Generic.
  - **Recursos:** Desmarcar absolutamente tudo, mantendo ativada apenas a **Interface WEB** (e MQTT opcionalmente se for integrar ao Home Assistant).
  - **Custom Parameters:** Adicionar o código `#define USE_DHT`, `#define USE_INA226`, `#define USE_INA219`.
- [ ] Flashear o firmware Tasmota customizado no ESP32.
- [ ] Mapear os pinos de forma segura na tela do Tasmota:
  - Configurar **GPIO 21 (SDA)** e **GPIO 22 (SCL)** para o INA226 (I2C).
  - Configurar **GPIO 16** como `CSE7766 Rx` (para leitura serial do sensor AC).
  - Configurar **GPIO 4** para o sensor DHT11 (Dado Digital).
- [ ] Configurar conexão Wi-Fi na interface do Tasmota.
- [ ] Calibrar a medição de corrente/potência DC no Console do Tasmota executando o comando `Sensor54 1, 5`.

## 🔌 4. Montagem e Integração Eletrônica
- [ ] Remover o resistor shunt interno R100 do módulo INA226.
- [ ] Soldar os fios finos de sinal do Shunt externo de 15A nos pinos pequenos **IN+** e **IN-** da fileira inferior do INA226 (ignorando os furos superiores).
- [ ] Conectar o pino **VBS** (ou VBUS) do INA226 ao polo **POSITIVO (+12V)** da bateria para leitura da tensão DC.
- [ ] Ligar o **LM2596** nos terminais da bateria e regular a tensão de saída (trimpot) para exatamente **5.0 Volts** com um multímetro antes de conectá-lo ao ESP32.
- [ ] Montar o circuito de teste e validar a leitura correta dos sensores no painel Tasmota.
- [ ] Soldar os componentes em uma placa definitiva, acomodar e isolar eletricamente o circuito dentro do nobreak.

## 📱 5. Central de Visualização e Dashboard
- [ ] **Modo Standalone (Padrão/Independente):** Monitorar a telemetria do nobreak diretamente pela tela inicial do Tasmota (salvando nos favoritos do celular ou computador).
- [ ] **Modo Home Assistant (Opcional):** Configurar o MQTT no Tasmota para habilitar a auto-descoberta plug-and-play do dispositivo "Nobreak Smart".
- [ ] Configurar o painel Lovelace no Dashboard do Home Assistant utilizando as entidades geradas automaticamente (como `cse7766` para rede e `ina226` para bateria).

## ✅ 6. Testes Finais e Validação de Segurança
- [ ] Realizar teste forçado de queda de energia (tirando da tomada com carga conectada) para simular o blecaute.
- [ ] Verificar a precisão das leituras de corrente e tensão DC na interface web ou no dashboard.
- [ ] Validar a reconexão automática ao Wi-Fi após restabelecimento do sinal ou queda de energia.

---
*Nota: Este é o esqueleto inicial do checklist. Cada etapa será expandida e receberá novas sub-tarefas assim que alinharmos as especificidades técnicas.*
