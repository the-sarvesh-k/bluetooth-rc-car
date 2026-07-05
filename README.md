




# 🚗 Bluetooth RC Car — Arduino

A 4-wheel Bluetooth controlled RC car built with Arduino Uno, 
L298N motor driver, and HC-05 Bluetooth module.
Supports forward, backward, left, right, horn, and headlight toggle.

---

## 📹 Demo Video
[Click here to watch demo](https://github.com/user-attachments/assets/406715a0-3537-48f9-9bed-ba2dcacfbc78)

---

## ⚙️ Components Used

| Component | Quantity |
|---|---|
| Arduino Uno | 1 |
| L298N Motor Driver | 1 |
| HC-05 Bluetooth Module | 1 |
| DC Motors (6V–12V) | 4 |
| Buzzer | 1 |
| LED (Headlight) | 1 |
| 18650 Battery (2S/3S) | 2–3 |
| Capacitors (100nF, 100µF, 470µF) | Multiple |
| Jumper Wires | As needed |
| Car Chassis | 1 |

---

## 📌 Pin Connections

### L298N → Arduino
| L298N Pin | Arduino Pin |
|---|---|
| IN1 | D4 |
| IN2 | D5 |
| IN3 | D6 |
| IN4 | D7 |
| GND | GND |

### HC-05 → Arduino
| HC-05 Pin | Arduino Pin |
|---|---|
| RX | D2 |
| TX | D3 |
| VCC | 5V |
| GND | GND |

### Other Components
| Component | Arduino Pin |
|---|---|
| Buzzer | D8 |
| LED (Headlight) | D9 |

---

## 📱 Bluetooth App Commands

| Button | Command Sent | Action |
|---|---|---|
| Forward | F | Move forward |
| Backward | B | Move backward |
| Left | L | Turn left |
| Right | R | Turn right |
| Stop | S | Stop car |
| Horn | Y | Beep horn |
| Headlight | U | Toggle LED on/off |

**Recommended App:** Bluetooth RC Controller (available on Play Store)

---

## 💡 Capacitor Connections (Noise Fix)

| Capacitor | Connection |
|---|---|
| 100nF ceramic | Across OUT1–OUT2 on L298N |
| 100µF electrolytic | Across OUT1–OUT2 on L298N |
| 100nF ceramic | Across OUT3–OUT4 on L298N |
| 100µF electrolytic | Across OUT3–OUT4 on L298N |
| 470µF electrolytic | L298N 12V pin → GND pin |
| 10µF electrolytic | Arduino 5V → RESET pin |

---

## 🔥 Known Issues & Fixes

### 1. Motor Timing Problem
**Cause:** Low battery voltage + L298N internal voltage drop  
**Fix:** Use 3S 18650 battery pack (11.1V nominal)

### 2. Bluetooth Stuck Command
**Cause:** Serial buffer overflow when button held  
**Fix:** Command timeout implemented in code (auto-stop after 300ms)

### 3. L298N Thermal Shutdown
**Cause:** L298N overheats running 4 motors  
**Fix:** Add heatsink + thermal paste on L298N chip. Upgrade to TB6612FNG recommended for permanent fix.

---

## 📄 Code

```cpp
#include <SoftwareSerial.h>

SoftwareSerial BT(2, 3);

#define IN1 4
#define IN2 5
#define IN3 6
#define IN4 7
#define BUZZER 8
#define LED 9

unsigned long lastCommandTime = 0;
const unsigned long TIMEOUT = 300;

bool hornActive = false;
int hornStep = 0;
unsigned long hornTimer = 0;
bool headlightOn = false;

const int hornPattern[][2] = {
  {HIGH, 200},
  {LOW, 80},
  {HIGH, 300},
  {LOW, 0}
};

void setup() {
  BT.begin(9600);
  Serial.begin(9600);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  pinMode(LED, OUTPUT);
  digitalWrite(LED, LOW);
  stopCar();
}

void loop() {
  handleHorn();
  if (millis() - lastCommandTime > TIMEOUT) {
    stopCar();
  }
  if (BT.available()) {
    char cmd = BT.read();
    Serial.println(cmd);
    if (cmd != 'Y' && cmd != 'U') {
      lastCommandTime = millis();
    }
    switch (cmd) {
      case 'F': forward(); digitalWrite(BUZZER, LOW); break;
      case 'B': backward(); digitalWrite(BUZZER, HIGH); break;
      case 'L': left(); digitalWrite(BUZZER, LOW); break;
      case 'R': right(); digitalWrite(BUZZER, LOW); break;
      case 'S': stopCar(); lastCommandTime = 0; break;
      case 'Y':
        hornStep = 0;
        hornActive = true;
        hornTimer = millis();
        break;
      case 'U':
        headlightOn = !headlightOn;
        digitalWrite(LED, headlightOn);
        break;
    }
  }
}

void handleHorn() {
  if (!hornActive) return;
  if (hornPattern[hornStep][1] == 0) {
    hornActive = false;
    digitalWrite(BUZZER, LOW);
    return;
  }
  if (millis() - hornTimer >= hornPattern[hornStep][1]) {
    hornStep++;
    digitalWrite(BUZZER, hornPattern[hornStep][0]);
    hornTimer = millis();
  }
}

void forward() {
  digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
}

void backward() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
}

void left() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
}

void right() {
  digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
}

void stopCar() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW); digitalWrite(IN4, LOW);
  digitalWrite(BUZZER, LOW);
}
```

---

## 🛠️ How To Upload Code

1. Install [Arduino IDE](https://www.arduino.cc/en/software)
2. Connect Arduino Uno via USB
3. Select board: **Tools → Board → Arduino Uno**
4. Select port: **Tools → Port → COMX**
5. Copy paste the code above
6. Click **Upload**

---

## 📝 License
MIT License — free to use and modify

---

## 👤 Author
**the-sarvesh-k**  
GitHub: [github.com/the-sarvesh-k](https://github.com/the-sarvesh-k)
