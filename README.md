# Configuração da Tela Nextion (IHM da Empilhadeira)

Este diretório contém os arquivos de design da tela **Nextion Intelligent Display**:
- **`nextion-file.HMI`**: Arquivo de projeto do Nextion Editor (código fonte da tela).
- **`nextion-file.tft`**: Arquivo compilado pronto para gravação física via cartão SD.

Para que a tela exiba corretamente os novos campos de telemetria transmitidos pelo ESP32 via UART, certifique-se de que os seguintes componentes e variáveis estejam criados no projeto dentro do Nextion Editor:

---

## 🛠️ Campos de Telemetria Necessários

### 1. Valores Numéricos (Componentes do Tipo `Variable` ou propriedades `.val`)
Estes campos devem ser criados no projeto (geralmente como variáveis globais ou componentes de valor/número) para receber os dados inteiros do ESP32:

| Nome do Componente | Tipo de Componente | Significado / Valores |
| :--- | :---: | :--- |
| **`steer`** | Variable / Number | Ângulo de esterço em graus (inteiro, ex: `-45` a `90`) |
| **`reverse`** | Variable / Number | Sentido de translação (0 = Frente / Neutro, 1 = Trás/Marcha Ré) |
| **`throttle`** | Variable / Number | Aceleração em porcentagem (inteiro, `0` a `100`) |
| **`soc`** | Variable / Number | Porcentagem de carga da bateria (inteiro, `0` a `100`) |
| **`hours`** | Variable / Number | Horímetro acumulado da empilhadeira (inteiro, ex: `7471`) |
| **`speed1`** | Variable / Number | RPM do Motor de Tração 1 |
| **`speed2`** | Variable / Number | RPM do Motor de Tração 2 / Hidráulico |
| **`kmh`** | Variable / Number | Velocidade linear multiplicada por 10 (ex: `52` representa `5.2 km/h`) |
| **`turtle`** | Variable / Number | Status do limitador de velocidade Lento/Tartaruga (`0` ou `1`) |
| **`deadman`** | Variable / Number | Presença do operador no banco / Deadman Switch (`0` ou `1`) |
| **`pedal`** | Variable / Number | Interruptor físico do pedal do acelerador / Parking (`0` ou `1`) |
| **`emul_tx`** | Variable / Number | Status de emulação de Heartbeats ativo no ESP32 (`0` ou `1`) |
| **`error`** | Variable / Number | Indicador geral de falha ativa na empilhadeira (`0` = Sem Erros, `1` = Erro Ativo) |
| **`manut`** | Variable / Number | Indicador de necessidade de manutenção/calibração (`0` = OK, `1` = Manutenção) |
| **`temp`** | Variable / Number | Indicador de alta temperatura nos motores/inversores (`0` = Normal, `1` = Alta) |

### 2. Textos Formatados (Componentes do Tipo `Text` ou propriedades `.txt`)
Estes componentes são caixas de texto que exibirão os valores formatados com unidades:

| Nome do Componente | Tipo | Exemplo de Formato Enviado pelo ESP32 |
| :--- | :---: | :--- |
| **`tSteer`** | Text | `"-12.5"` (Exibe o ângulo com precisão de uma casa decimal) |
| **`tThrottle`** | Text | `"45.2%"` (Exibe o percentual do acelerador) |
| **`tSoC`** | Text | `"82.0%"` (Exibe o nível de bateria) |
| **`tVolts`** | Text | `"48.0 V"` (Exibe a tensão nominal ou calculada do barramento) |
| **`tHours`** | Text | `"7471 h"` (Exibe o horômetro formatado com a unidade) |
| **`tSpeed1`** | Text | `"1500 RPM"` (Rotação do motor de tração) |
| **`tSpeed2`** | Text | `"1200 RPM"` (Rotação do motor hidráulico/direção) |
| **`tKmh`** | Text | `"6.4 km/h"` (Velocidade translacional calculada) |
| **`tTurtle`** | Text | `"ACTIVE"` ou `"INACTIVE"` (Status legível do modo lento) |
| **`tDeadman`** | Text | `"OCCUPIED"` ou `"VACANT"` (Status do banco do operador) |
| **`tPedal`** | Text | `"PRESSED"` ou `"RELEASED"` (Status do botão inicial do acelerador) |
| **`tEmulTx`** | Text | `"EMULATING"` ou `"MONITOR"` (Status de emulação do ESP32) |
| **`tError`** | Text | `"132, 564"` (Códigos de alarme ativos separados por vírgula. Exibe `"999"` em perda de sinal) |

### 3. Componentes de Imagem (Componentes do Tipo `Picture` ou propriedades `.pic`)
Estes componentes recebem o ID numérico da imagem cadastrada no Nextion Editor:

| Nome do Componente | Tipo | Significado / IDs no ESP32 |
| :--- | :---: | :--- |
| **`batteryPic`** / **`batPic`** | Picture | Ícone booleano de Bateria (`< 30%` = Alerta/Baixa, `>= 30%` = Normal) |
| **`socPic`** | Picture | Barra de nível de carga da Bateria (0 a 10 níveis, base ID configurável) |
| **`throttlePic`** | Picture | Barra de nível do acelerador (0 a 8 níveis, base ID configurável) |
| **`slowPic`** / **`turtlePic`** | Picture | Ícone de Modo Tartaruga / Lento |
| **`reversePic`** | Picture | Ícone de Marcha Ré ativa |
| **`deadmanPic`** | Picture | Ícone de Operador Presente / Ausente |
| **`pedalPic`** | Picture | Ícone de Pedal Acelerador |
| **`breakPic`** / **`brakePic`** | Picture | Ícone de Freio acionado (Entrada física E2) |
| **`errorPic`** | Picture | Ícone de Falha / Alarme Ativo |
| **`manutPic`** | Picture | Ícone de Manutenção |
| **`tempPic`** | Picture | Ícone de Alerta de Alta Temperatura |
| **`emulPic`** | Picture | Ícone de Modo Emulação CAN Ativo |

---

## 🎨 Sugestões de Implementação Visual no Nextion Editor

1.  **Lógica Visual de Erros (`error.val` e `tError`):**
    - Associe a variável `error.val` a um ícone visual de exclamação vermelho (`vis errorIcon,error.val` no evento de atualização da tela).
    - Exiba a caixa de texto `tError` apenas se `error.val == 1`.
2.  **Indicador de Sentido de Direção (`reverse.val`):**
    - Crie setas na tela (Frente e Trás).
    - Se `reverse.val == 0`, ligue a seta de Frente e desligue a de Trás.
    - Se `reverse.val == 1`, ligue a seta de Trás e desligue a de Frente.
3.  **Indicador de Temperatura Alta (`temp.val`):**
    - Vincule a variável `temp.val` a um ícone de termômetro vermelho que pisca ou fica visível apenas quando o valor for `1`.
4.  **Alerta de Manutenção (`manut.val`):**
    - Vincule `manut.val` a um ícone de chave de boca amarelo.

---

## 🔌 Protocolo de Comunicação Serial
O ESP32 atualiza esses campos enviando comandos UART no formato padrão da Nextion:
```
<nome_do_campo>.<propriedade>=<valor><0xFF><0xFF><0xFF>
```
*   **Campos Numéricos**: `error.val=1`
*   **Campos de Texto**: `tError.txt="82, 85"`

*Nota: O ESP32 envia a lista de códigos de erros tanto para a propriedade `tError.txt` quanto para `tError.val` por compatibilidade com qualquer nomenclatura de atributo escolhida no seu design Nextion.*
