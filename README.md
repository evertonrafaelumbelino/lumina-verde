# Lumina Verde 🌿
## Biophilic Smart Mini-Terrarium with Sustainable Automation

O **Lumina Verde** é um protótipo educacional de mini terrário biofílico automatizado, desenvolvido no contexto da robótica educacional. O projeto integra programação, sensoriamento ambiental, automação e sustentabilidade utilizando **Arduino Uno**.

O sistema monitora temperatura, umidade relativa do ar e luminosidade, utilizando as informações dos sensores para controlar automaticamente a iluminação do protótipo de acordo com limiares previamente definidos no programa. Além disso, o projeto utiliza materiais reutilizados em sua estrutura física, relacionando automação e tecnologia ao reaproveitamento criativo.

---

## 🛠️ Componentes

| Componente | Conexão / Pino | Função |
| :--- | :--- | :--- |
| **Arduino Uno** | — | Controle e processamento do sistema |
| **DHT11** | Pino digital 2 | Medição de temperatura e umidade |
| **BH1750** | I²C (SDA / SCL) | Medição da luminosidade em lux |
| **Fita NeoPixel** | Pino digital 6 | Iluminação automatizada |
| **NeoPixel** | 30 LEDs | Sistema de iluminação |
| **Breadboard** | — | Montagem e organização do circuito |
| **Jumpers** | — | Conexões elétricas |
| **Cabo USB** | USB | Alimentação, programação e comunicação serial |

---

## 📚 Bibliotecas Utilizadas

O código utiliza as seguintes bibliotecas que devem ser instaladas pelo **Library Manager** da Arduino IDE antes da compilação:

```cpp
#include <Wire.h>
#include <BH1750.h>
#include <DHT.h>
#include <Adafruit_NeoPixel.h>
```

### Função das bibliotecas:
* **`Wire.h`** — Comunicação I²C com o sensor BH1750.
* **`BH1750.h`** — Leitura da intensidade luminosa em lux.
* **`DHT.h`** — Leitura de temperatura e umidade pelo sensor DHT11.
* **`Adafruit_NeoPixel.h`** — Controle individual e em lote dos LEDs da fita NeoPixel.

---

## 🔌 Esquema Básico de Conexões

```text
Arduino Uno
│
├── DHT11
│   └── DATA → Digital 2
│
├── BH1750
│   └── I²C → SDA / SCL
│
└── NeoPixel (30 LEDs)
    └── DATA → Digital 6
```

> **Nota:** As conexões de alimentação (`VCC`/`5V` e `GND`) devem seguir rigorosamente as especificações dos componentes utilizados na montagem.

---

## ⚙️ Lógica de Funcionamento

O sistema realiza leituras periódicas dos sensores a cada **3 segundos** e utiliza valores predefinidos para determinar o comportamento da iluminação:

| Condição Medida | Comportamento do Sistema |
| :--- | :--- |
| **Luminosidade < 100 lux** | Ativação do modo de iluminação para as plantas (roxo/violeta) |
| **100 a 200 lux** | Ativação do modo de iluminação de conforto (suave) |
| **Luminosidade > 200 lux** | Iluminação desligada para economizar energia |
| **Temperatura ≥ 30 °C** | Iluminação desligada como condição adicional de segurança |

> ⚠️ **Prioridade de Segurança:** O limite de temperatura possui prioridade absoluta sobre o controle baseado na luminosidade.

---

## ♻️ Estrutura Física e Sustentabilidade

A estrutura do Lumina Verde foi construída predominantemente com **materiais reutilizados**:
* **Parte superior da luminária:** Recipiente de grande porte originalmente utilizado para armazenar gelo e resfriar bebidas, reutilizado de forma invertida.
* **Terrário:** Pote grande de vidro para conserva, anteriormente utilizado em uma pizzaria.
* **Suporte:** Haste metálica reutilizada com aproximadamente 30 cm.
* **Elemento de sustentação:** Concha metálica reutilizada e posicionada de forma invertida.

---

## 📊 Monitoramento

Durante os testes, o sistema foi utilizado para acompanhar em tempo real:
* Temperatura ($\text{^\circ C}$)
* Umidade relativa ($\%$)
* Luminosidade ($\text{lx}$)

O **Serial Monitor** e o **Serial Plotter** da Arduino IDE permitem visualizar facilmente os valores obtidos pelos sensores durante o funcionamento do protótipo.

---

## 🧑‍💻 Código-Fonte (`lumina_verde.ino`)

```cpp
#include <Wire.h>
#include <BH1750.h>
#include <DHT.h>
#include <Adafruit_NeoPixel.h>

// -------- PINOS --------
#define PINO_DHT 2
#define TIPO_DHT DHT11

#define PINO_LED 6
#define NUM_LEDS 30   // Altere para a quantidade de LEDs da sua fita

// -------- OBJETOS --------
DHT dht(PINO_DHT, TIPO_DHT);
BH1750 sensorLuz;
Adafruit_NeoPixel fita(NUM_LEDS, PINO_LED, NEO_GRB + NEO_KHZ800);

// -------- VALORES DE CONTROLE --------
int limitePoucaLuz = 100;   // Abaixo disso liga a fita
int limiteLuzBoa = 200;     // Acima disso desliga
float limiteTemperatura = 30.0;

void setup() {
  Serial.begin(9600);

  dht.begin();

  Wire.begin();
  sensorLuz.begin();

  fita.begin();
  fita.show();

  Serial.println("Terrario inteligente iniciado!");
}

void loop() {
  float temperatura = dht.readTemperature();
  float umidade = dht.readHumidity();
  float luminosidade = sensorLuz.readLightLevel();

  Serial.print("Temperatura: ");
  Serial.print(temperatura);
  Serial.print(" °C | Umidade: ");
  Serial.print(umidade);
  Serial.print(" % | Luz: ");
  Serial.print(luminosidade);
  Serial.println(" lx");

  if (isnan(temperatura) || isnan(umidade)) {
    Serial.println("Erro ao ler o DHT11");
    delay(2000);
    return;
  }

  // Segurança: se estiver muito quente, desliga os LEDs
  if (temperatura >= limiteTemperatura) {
    desligarLED();
    Serial.println("Temperatura alta: LED desligado.");
  }
  else {
    // Pouca luz: liga LED para ajudar as plantas
    if (luminosidade < limitePoucaLuz) {
      ligarModoPlanta();
      Serial.println("Pouca luz: modo planta ligado.");
    }

    // Luz suficiente: desliga para economizar energia
    else if (luminosidade > limiteLuzBoa) {
      desligarLED();
      Serial.println("Luz suficiente: LED desligado.");
    }

    // Luz intermediária: modo suave para conforto humano
    else {
      ligarModoConforto();
      Serial.println("Luz intermediaria: modo conforto ligado.");
    }
  }

  delay(3000);
}

// -------- FUNÇÕES DA FITA --------

void ligarModoPlanta() {
  for (int i = 0; i < NUM_LEDS; i++) {
    fita.setPixelColor(i, fita.Color(120, 0, 180)); 
  }
  fita.setBrightness(120);
  fita.show();
}

void ligarModoConforto() {
  for (int i = 0; i < NUM_LEDS; i++) {
    fita.setPixelColor(i, fita.Color(255, 160, 60)); 
  }
  fita.setBrightness(60);
  fita.show();
}

void desligarLED() {
  for (int i = 0; i < NUM_LEDS; i++) {
    fita.setPixelColor(i, 0);
  }
  fita.show();
}
```

---

## 🏫 Contexto Educacional

* **Público-alvo:** Quatro estudantes entre 13 e 15 anos integrantes da equipe de robótica escolar.
* **Período e Carga Horária:** Desenvolvido no primeiro semestre de 2026, totalizando aproximadamente 20 horas de atividades.
* **Metodologia:** A professora atuou como mediadora e observadora, enquanto os estudantes lideraram o planejamento, montagem, programação, testes, calibração dos sensores e análise dos dados. Erros de programação e dificuldades de calibração foram abordados como oportunidades investigativas seguindo a lógica de testar, observar, identificar problemas, modificar e testar novamente.