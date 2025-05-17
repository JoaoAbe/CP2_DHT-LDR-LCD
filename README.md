# CP01 - Edge Computing & Computer Systems  
## 🍷 Arduino, LDR, DHT11 e LCD da Vinheria Agnello

A Vinheria Agnello sempre se destacou pela experiência personalizada oferecida aos seus clientes. Com a chegada da pandemia, o negócio foi impactado pela ausência de frequentadores. Para se reinventar, o Sr. Agnello decidiu investir em uma **plataforma de vendas digital**, mantendo o atendimento e a qualidade da vinheria.

---

## 📝 Descrição do Projeto

O armazenamento correto de vinhos depende de condições ambientais controladas, como **luminosidade, temperatura e umidade**. Pensando nisso, este projeto implementa:

- Um **sensor LDR** para detecção de **excesso de luz**, com alertas visuais e sonoros;
- Um **sensor DHT11** para monitorar **temperatura e umidade** do ambiente;
- Um **display LCD I2C** para exibir os dados coletados em tempo real.

---

## ⚠️ Problemas Causados por Condições Inadequadas

### 🌞 Excesso de Luz
A luz, especialmente a radiação ultravioleta, pode degradar compostos orgânicos dos vinhos, prejudicando seu sabor e aroma — especialmente vinhos brancos e espumantes.

### 💧 Excesso de Umidade
Altos níveis de umidade favorecem o surgimento de mofo, danificam rótulos e afetam a rolha, comprometendo a qualidade do vinho.

### 🔥 Excesso de Temperatura
Temperaturas elevadas aceleram o envelhecimento do vinho, alteram seu sabor e provocam evaporação, podendo levar ao ressecamento e contaminação pela rolha.

---

## 🧪 Testes e Simulações

O sistema foi:

- **Simulado no Wokwi**
- **Testado fisicamente em um Arduino Uno real**

---

## 👨‍💻 Integrantes do Grupo

- Eduardo Santiago Bassan — RM: 561474  
- Vitor Fernandes dos Santos — RM: 566275  
- Henry Andrade Browne — RM: 562622  
- João Victor de Souza Abe — RM: 561446  

---

## 🧰 Tecnologias Utilizadas

- Arduino IDE  
- Wokwi  
- Tinkercad  

---

## 🎥 Link para o Pitch

> (https://youtu.be/JhC-pSsHGn8?si=iPF_4Z8nHxC7qdK_)

---

## 🔧 Materiais Utilizados

- 1 Arduino Uno  
- 1 Buzzer  
- 3 LEDs (verde, amarelo, vermelho)  
- 3 Resistores de 220Ω  
- 2 Resistores de 10kΩ  
- 1 Sensor LDR  
- 1 Sensor DHT11  
- 1 Tela LCD I2C  
- Jumpers  
- 1 Protoboard  

---

## 🛠️ Como Montar o Projeto

1. **Sensor LDR**:
   - Uma perna no **VCC**
   - Outra no **A0**
   - Em paralelo: resistor de **10kΩ** entre A0 e GND

2. **LEDs com resistores de 220Ω**:
   - Cátodo → GND  
   - Ânodo →  
     - Verde: **Pino 10**  
     - Amarelo: **Pino 9**  
     - Vermelho: **Pino 8**

3. **Buzzer**:
   - Positivo → **Pino 11**  
   - Negativo → **GND**

4. **DHT11**:
   - 1ª perna → VCC  
   - 2ª perna → Pino digital (ex: 7)  
   - 3ª perna → Não usada  
   - 4ª perna → GND  
   - Resistor de 10kΩ entre 1ª e 2ª pernas

5. **LCD I2C**:
   - GND → GND  
   - VCC → 5V  
   - SDA → A4  
   - SCL → A5

6. **Bibliotecas necessárias**:
   - `DHT.h`  
   - `Wire.h`  
   - `LiquidCrystal_I2C.h`

---

## 💻 Código Arduino

```cpp
#include <Adafruit_Sensor.h>            
#include <DHT.h>
#include <DHT_U.h>
#include <LiquidCrystal.h>

LiquidCrystal lcd(13, 12, 11, 10, 9, 8);
#define DHTTYPE DHT22
#define DHTPIN 6
DHT HT(DHTPIN, DHTTYPE);

#define LDR A0
#define BUZZER 5
#define LED_VERDE 4
#define LED_AMARELO 3
#define LED_VERMELHO 2

void setup() {
  HT.begin();
  lcd.begin(16, 2);
  pinMode(BUZZER, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_AMARELO, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);
  Serial.begin(9600);
}

float mediaUmidade() {
  float soma = 0;
  int vezes = 0;
  for (int i = 0; i < 5; i++) {
    float leitura = HT.readHumidity();
    if (!isnan(leitura)) {
      soma += leitura;
      vezes++;
    }
    delay(200);
  }
  if (vezes == 0) return 0;
  return soma / vezes;
}

float mediaTemperatura() {
  float soma = 0;
  int vezes = 0;
  for (int i = 0; i < 5; i++) {
    float leitura = HT.readTemperature();
    if (!isnan(leitura)) {
      soma += leitura;
      vezes++;
    }
    delay(200);
  }
  if (vezes == 0) return 0;
  return soma / vezes;
}

void loop() {
  int valor_ldr = analogRead(LDR);
  valor_ldr = map(valor_ldr, 0, 1015, 0, 100);

  float mediaUmidadeValor = mediaUmidade();
  float mediaTemperaturaValor = mediaTemperatura();

  Serial.print("LDR: "); Serial.println(valor_ldr);
  Serial.print("Umidade: "); Serial.println(mediaUmidadeValor);
  Serial.print("Temperatura: "); Serial.println(mediaTemperaturaValor);

  if (valor_ldr <= 40) { 
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_AMARELO, LOW);
    digitalWrite(LED_VERMELHO, HIGH);
    tone(BUZZER, 262);
    lcd.setCursor(0,0);
    lcd.print("Ambiente muito  ");
    lcd.setCursor(0,1);
    lcd.print("claro           ");
  } else if (valor_ldr > 40 && valor_ldr <= 80) { 
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_AMARELO, HIGH);
    noTone(BUZZER);
    lcd.setCursor(0,0);
    lcd.print("Ambiente a meia ");
    lcd.setCursor(0,1);
    lcd.print("luz             ");
  } else { 
    lcd.setCursor(0,0);
    lcd.print("Ambiente escuro ");
    lcd.setCursor(0,1);
    lcd.print("                ");
    }
    if(((mediaUmidadeValor < 70) && (mediaUmidadeValor > 50)) && ((mediaTemperaturaValor < 15) && (mediaTemperaturaValor > 10))){
    digitalWrite(LED_AMARELO, LOW);
    digitalWrite(LED_VERMELHO, LOW);
    digitalWrite(LED_VERDE, HIGH);

    noTone(BUZZER);
   
  }

  delay(4000);

  if (mediaTemperaturaValor < 10) {
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_AMARELO, HIGH);
    tone(BUZZER, 262);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Temp. Baixa     ");
    lcd.setCursor(0,1);
    lcd.print("Temp = ");
    lcd.print((int)mediaTemperaturaValor);
    lcd.print("C   ");
  } else if (mediaTemperaturaValor > 15) {
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_AMARELO, HIGH);
    tone(BUZZER, 262);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Temp. Alta      ");
    lcd.setCursor(0,1);
    lcd.print("Temp = ");
    lcd.print((int)mediaTemperaturaValor);
    lcd.print("C   ");
  } else {
    if (valor_ldr <= 40 || valor_ldr > 80) {
      digitalWrite(LED_AMARELO, LOW);
    }
    digitalWrite(LED_AMARELO, LOW);
    if (valor_ldr > 40) noTone(BUZZER);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Temperatura OK  ");
    lcd.setCursor(0,1);
    lcd.print("Temp = ");
    lcd.print((int)mediaTemperaturaValor);
    lcd.print("C   ");
  }

  delay(4000);

  if (mediaUmidadeValor < 50) {
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_VERMELHO, HIGH);
    tone(BUZZER, 262);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Umidade Baixa   ");
    lcd.setCursor(0,1);
    lcd.print("Umid = ");
    lcd.print((int)mediaUmidadeValor);
    lcd.print("%   ");
  } else if (mediaUmidadeValor > 70) {
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_VERMELHO, HIGH);
    tone(BUZZER, 262);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Umidade Alta    ");
    lcd.setCursor(0,1);
    lcd.print("Umid = ");
    lcd.print((int)mediaUmidadeValor);
    lcd.print("%   ");
  } else {
    if (valor_ldr > 40 && mediaTemperaturaValor >= 10 && mediaTemperaturaValor <= 15) {
      digitalWrite(LED_VERMELHO, LOW);
      noTone(BUZZER);
    }
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Umidade OK      ");
    lcd.setCursor(0,1);
    lcd.print("Umid = ");
    lcd.print((int)mediaUmidadeValor);
    lcd.print("%   ");
  }

  delay(4000);
}