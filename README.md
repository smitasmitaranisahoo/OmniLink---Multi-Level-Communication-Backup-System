**📡 OmniLink – Multi-Level Communication Backup System**

**🌟 Project Overview**

OmniLink is a smart multi-level communication backup system developed using ESP32, GSM, and LoRa technologies.
The main objective of this project is to maintain communication reliability even when one communication medium fails.
The system first uses Wi-Fi/mobile internet through ESP32 and Telegram Bot communication.
If internet communication fails, the system automatically switches to GSM communication.
If GSM communication also fails, the system finally uses LoRa communication for long-range message transmission.
This project ensures uninterrupted communication using automatic fallback mechanisms.

**🖼️ Project Images**
Hardware Setup
🚀 Features
📶 ESP32 Wi-Fi Communication
🤖 Telegram Bot Notification System
📱 GSM Based Backup Messaging
📡 Long Range LoRa Communication
🔄 Automatic Communication Fallback
🔋 Portable Battery Powered System
📟 LCD Status Display
🌐 IoT Based Smart Communication
⚡ Real-Time Alert Transmission
📍 Reliable Long Distance Connectivity
🧠 Working Principle

**Step 1** – System Startup
The power switch is turned ON.
ESP32 becomes active.
LCD display initializes.

**Step 2** – Primary Communication (Internet)
ESP32 connects to mobile hotspot/Wi-Fi.
Telegram Bot becomes active.
Messages are sent through internet communication.

**Step 3** – GSM Backup Communication
If ESP32 internet connection fails,
System automatically switches to GSM module.
GSM uses mobile network to send messages/SMS alerts.

**Step 4** – LoRa Backup Communication
If GSM communication also fails,
System activates LoRa communication.
LoRa sends messages over long distance wirelessly.

**Step 5** – Continuous Monitoring
LCD continuously displays:
Active communication mode
RSSI value
Temperature/status
LoRa fallback status

**🔄 Communication Flow**
Plain text
Power ON
   ↓
ESP32 + Internet Active
   ↓
Telegram Bot Communication
   ↓
If Internet Fails
   ↓
Switch to GSM Communication
   ↓
If GSM Fails
   ↓
Activate LoRa Communication
   ↓
Long Distance Backup Messaging

**🧰 Components Uses**
Component
Purpose
ESP32
Main controller and internet communication
LoRa Module
Long distance wireless communication
GSM Module
Backup communication using mobile network
20x4 LCD Display
Display system status and messages
I2C Module
Reduces LCD wiring connections
Antenna
Improves LoRa signal range
18650 Batteries
Portable power supply
Battery Holder
Holds Li-ion batteries
Buck Converter
Voltage regulation
Switch
Power ON/OFF control
PCB/Perfboard
Circuit mounting
Jumper Wires
Electrical connections

**🔍 Component Description**
**🔹 ESP32**
ESP32 is the brain of the project.
It handles:
Wi-Fi connection
Telegram communication
Data processing
Fallback control logic

**🔹 GSM Module**
The GSM module sends alerts/messages using cellular network when internet is unavailable.

**🔹 LoRa Module**
LoRa provides very long-range communication with low power consumption.
It acts as the final backup communication method.

**🔹 LCD Display**
The LCD displays:
Communication mode
Signal strength
System status
Error/fallback information

**🔹 Buck Converter**
Converts battery voltage into stable output voltage required for modules.

**🔹 18650 Battery******
Provides portable rechargeable power supply to the system.

**⚙️ Technologies Used**
Embedded Systems
IoT (Internet of Things)
Wireless Communication
LoRa Technology
GSM Communication
Wi-Fi Networking
Telegram Bot API
Arduino IDE
C/C++ Programming

**💻 Software Requirements**
Arduino IDE
ESP32 Board Package
LoRa Library
WiFi Library
Universal Telegram Bot Library
TinyGSM Library
LiquidCrystal I2C Library

**📂 Files in This Repository
Plain text**
📁 OmniLink

**Transmitter codde()**
 #include <WiFi.h>
#include <HTTPClient.h>
#include <HardwareSerial.h>
#include <SPI.h>
#include <LoRa.h>

// ================= CONFIGURATION =================
// WiFi
const char* ssid     = "IOT";
const char* password = "1234567890";

// Telegram
const String botToken = "7998457978:AAHq1AaIllRgB0HXQ-OiEOHUukCEHuCGMJ4";
const String chatID   = "7394829757";

// GSM (A7670C) - tera working configuration
HardwareSerial sim(2);
const int gsmTx = 17;   // ESP32 TX → A7670C RX
const int gsmRx = 16;   // ESP32 RX → A7670C TX
const String smsNumber = "+918093344602";

// LoRa (RA-02 / SX1278) – 433 MHz
#define LORA_SS   5
#define LORA_RST  14
#define LORA_DIO0 2

// Sending interval (milliseconds)
const unsigned long sendInterval = 30000;   // 30 seconds

// ================= GLOBAL =================
bool gsmReady = false;
bool loraReady = false;

// ================= FUNCTION PROTOTYPES =================
// Tere chhote code ke GSM helpers (exact copy)
void sendCmd(String cmd);
String readResponse();
bool waitFor(String target, int timeoutMs);

// Baaki functions
void initGSM();
bool sendViaGSM(String message);
bool sendViaTelegram(String message);
String urlEncode(String str);
void connectWiFi();
void initLoRa();
bool sendViaLoRa(String message);
String sendMessageWithFallback(String data, String time);

// ================= SETUP =================
void setup() {
  Serial.begin(115200);
  sim.begin(115200, SERIAL_8N1, gsmRx, gsmTx);   // Tera working baud rate

  Serial.println("\n=== OMNIEYE TX (WiFi + GSM + LoRa 433MHz) ===");

  // 1. WiFi
  connectWiFi();

  // 2. GSM (using your proven init)
  initGSM();

  // 3. LoRa
  initLoRa();

  Serial.println("System ready.");
}

// ================= MAIN LOOP =================
void loop() {
  static unsigned long lastSend = 0;
  if (millis() - lastSend >= sendInterval) {
    lastSend = millis();
    
    // Build a generic data message
    String sensorData = "Temperature: 24.5C, Humidity: 60%";
    String timestamp = String(millis() / 1000);
    
    String status = sendMessageWithFallback(sensorData, timestamp);
    Serial.println("Final status: " + status);
  }

  // Keep WiFi alive
  if (WiFi.status() != WL_CONNECTED) {
    connectWiFi();
  }

  delay(100);
}

// ================= CORE SEND LOGIC (WiFi → GSM → LoRa) =================
String sendMessageWithFallback(String data, String time) {
  Serial.println("\n--- Sending data: " + data + " (time: " + time + ") ---");

  // 1) Try Telegram via WiFi
  if (WiFi.status() == WL_CONNECTED) {
    String msg = "[✅ WiFi/Telegram] " + data + " | Time: " + time;
    if (sendViaTelegram(msg)) {
      Serial.println("✅ Sent via Telegram (WiFi)");
      return "WiFi/Telegram OK";
    }
  }

  // 2) Fallback to GSM SMS (using your working sendViaGSM)
  if (gsmReady) {
    String msg = "[⚠️ GSM/SMS Fallback] " + data + " | Time: " + time;
    if (sendViaGSM(msg)) {
      Serial.println("✅ Sent via SMS (GSM fallback)");
      return "GSM/SMS OK";
    } else {
      // Retry after delay as in your proven logic
      delay(3000);
      if (sendViaGSM(msg)) {
        Serial.println("✅ Sent via SMS (GSM retry)");
        return "GSM/SMS retry OK";
      }
    }
  }

  // 3) Last resort: LoRa transmission
  if (loraReady) {
    String msg = "[📡 LoRa Fallback] " + data + " | Time: " + time;
    if (sendViaLoRa(msg)) {
      Serial.println("✅ Sent via LoRa (backup link)");
      return "LoRa OK";
    }
  }

  Serial.println("❌ FAILED: all communication channels down");
  return "ALL FAILED";
}

// ================= TELEGRAM (WiFi) =================
bool sendViaTelegram(String message) {
  HTTPClient http;
  String url = "https://api.telegram.org/bot" + botToken +
               "/sendMessage?chat_id=" + chatID +
               "&text=" + urlEncode(message);

  http.begin(url);
  http.setTimeout(5000);
  int httpCode = http.GET();
  http.end();

  if (httpCode == 200) return true;

  Serial.println("Telegram failed, code: " + String(httpCode));
  return false;
}

String urlEncode(String str) {
  String encoded = "";
  char c;
  for (int i = 0; i < str.length(); i++) {
    c = str.charAt(i);
    if (isalnum(c) || c == '-' || c == '_' || c == '.' || c == '~')
      encoded += c;
    else if (c == ' ')
      encoded += "%20";
    else {
      encoded += '%';
      char hex[3];
      sprintf(hex, "%02X", c);
      encoded += hex;
    }
  }
  return encoded;
}

// ================= GSM SMS (A7670C) – EXACTLY YOUR WORKING CODE =================
bool sendViaGSM(String message) {
  Serial.println("Sending via GSM...");

  sim.println("AT+CMGF=1");
  delay(200);

  sim.print("AT+CMGS=\"");
  sim.print(smsNumber);
  sim.println("\"");

  if (!waitFor(">", 5000)) {
    Serial.println("No SMS prompt");
    return false;
  }

  sim.print(message);
  delay(300);
  sim.write(26);                 // CTRL+Z

  if (waitFor("+CMGS", 15000)) {
    Serial.println("GSM sent + confirmed");
    return true;
  }

  Serial.println("GSM send failed");
  return false;
}

// ================= GSM INITIALISATION – EXACTLY YOUR WORKING CODE =================
void initGSM() {
  Serial.println("Initialising GSM...");

  sendCmd("AT");
  sendCmd("ATE0");
  sendCmd("AT+CMGF=1");

  // Wait for network registration
  bool registered = false;
  for (int i = 0; i < 20; i++) {   // try up to 20 sec
    sim.println("AT+CREG?");
    delay(1000);
    String resp = readResponse();
    if (resp.indexOf("0,1") != -1 || resp.indexOf("0,5") != -1) {
      registered = true;
      break;
    }
    Serial.print(".");
  }

  if (registered) {
    gsmReady = true;
    Serial.println("\nGSM ready ✅");
  } else {
    Serial.println("\nGSM not registered (SMS may fail)");
  }
}

// ================= GSM HELPERS (teri working file se exactly copy) =================
void sendCmd(String cmd) {
  sim.println(cmd);
  delay(500);
  readResponse();   // discard
}

String readResponse() {
  String res = "";
  long start = millis();
  while (millis() - start < 2000) {
    while (sim.available()) {
      res += (char)sim.read();
    }
  }
  return res;
}

bool waitFor(String target, int timeoutMs) {
  String response = "";
  long start = millis();
  while (millis() - start < timeoutMs) {
    while (sim.available()) {
      char c = sim.read();
      response += c;
      if (response.indexOf(target) != -1) return true;
    }
  }
  return false;
}

// ================= LoRa INIT & SEND (433 MHz) =================
void initLoRa() {
  Serial.println("Initialising LoRa at 433 MHz...");
  SPI.begin(18, 19, 23, LORA_SS);  // SCK=18, MISO=19, MOSI=23, SS=5
  LoRa.setPins(LORA_SS, LORA_RST, LORA_DIO0);

  if (!LoRa.begin(433E6)) {          // 433 MHz
    Serial.println("LoRa init failed!");
    loraReady = false;
    return;
  }

  // Optional settings
  LoRa.setSpreadingFactor(12);
  LoRa.setTxPower(20);               // max power
  LoRa.enableCrc();

  loraReady = true;
  Serial.println("LoRa ready ✅");
}

bool sendViaLoRa(String message) {
  Serial.println("Sending via LoRa...");
  LoRa.beginPacket();
  LoRa.print(message);
  LoRa.endPacket();
  return true;
}

// ================= WIFI MANAGEMENT =================
void connectWiFi() {
  if (WiFi.status() == WL_CONNECTED) return;

  Serial.print("Connecting to WiFi");
  WiFi.begin(ssid, password);

  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWiFi connected ✅ IP: " + WiFi.localIP().toString());
  } else {
    Serial.println("\nWiFi failed ❌ (GSM / LoRa will be used)");
  }
}
 **receiever code**()
 #include <SPI.h>
#include <LoRa.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define LORA_SS   5
#define LORA_RST  14
#define LORA_DIO0 2

LiquidCrystal_I2C lcd(0x27, 20, 4);

String incomingMessage = "";
bool newMessage = false;

unsigned long lastPacketTime = 0;
const unsigned long timeoutInterval = 5000;
bool showingNoData = false;

void setup() {
  Serial.begin(115200);
  Serial.println("\n=== OMNIEYE RECEIVER (LoRa 433MHz + 20x4 LCD) ===");

  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("LoRa Receiver");
  lcd.setCursor(0, 1);
  lcd.print("433 MHz");
  lcd.setCursor(0, 2);
  lcd.print("Waiting...");

  SPI.begin(18, 19, 23, LORA_SS);
  LoRa.setPins(LORA_SS, LORA_RST, LORA_DIO0);

  if (!LoRa.begin(433E6)) {
    Serial.println("LoRa init failed!");
    lcd.setCursor(0, 3);
    lcd.print("LoRa ERROR!    ");
    while (1);
  }

  LoRa.setSpreadingFactor(12);
  LoRa.enableCrc();
  Serial.println("LoRa ready, listening on 433 MHz...");
  lcd.setCursor(0, 3);
  lcd.print("LoRa OK        ");
  delay(2000);
  lcd.clear();

  lastPacketTime = millis();
  showingNoData = false;
  showNoDataScreen();
}

void loop() {
  int packetSize = LoRa.parsePacket();
  if (packetSize) {
    incomingMessage = "";
    while (LoRa.available()) {
      incomingMessage += (char)LoRa.read();
    }

    int rssi = LoRa.packetRssi();
    Serial.print("Received: ");
    Serial.println(incomingMessage);
    Serial.print("RSSI: ");
    Serial.println(rssi);

    newMessage = true;
    lastPacketTime = millis();
    if (showingNoData) {
      showingNoData = false;
    }
  }

  if (newMessage) {
    displayOnLCD(incomingMessage);
    newMessage = false;
  }

  if (!showingNoData && (millis() - lastPacketTime > timeoutInterval)) {
    showNoDataScreen();
    showingNoData = true;
  }

  delay(50);
}

void displayOnLCD(String msg) {
  lcd.clear();

  lcd.setCursor(0, 0);
  if (msg.length() >= 20) {
    lcd.print(msg.substring(0, 20));
  } else {
    lcd.print(msg);
    for (int i = msg.length(); i < 20; i++) lcd.print(" ");
  }

  lcd.setCursor(0, 1);
  if (msg.length() > 20) {
    int len = msg.length() - 20;
    if (len > 20) len = 20;
    lcd.print(msg.substring(20, 20 + len));
  } else {
    lcd.print("                    ");
  }

  lcd.setCursor(0, 2);
  lcd.print("RSSI: ");
  lcd.print(LoRa.packetRssi());
  lcd.print(" dBm   ");

  lcd.setCursor(0, 3);
  if (msg.indexOf("WiFi/Telegram") != -1) {
    lcd.print("Via: Telegram   ");
  } else if (msg.indexOf("GSM/SMS") != -1) {
    lcd.print("Via: SMS        ");
  } else if (msg.indexOf("LoRa") != -1) {
    lcd.print("Via: LoRa (backup)");
  } else {
    lcd.print("Unknown source ");
  }
}

void showNoDataScreen() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("No LoRa data   ");
  lcd.setCursor(0, 1);
  lcd.print("Using: SMS/GSM ");
  lcd.setCursor(0, 2);
  lcd.print("or WiFi        ");
  lcd.setCursor(0, 3);
  lcd.print("(waiting for   ");
  lcd.setCursor(12, 3);
  lcd.print("LoRa)");
}

**1️⃣ Clone Repository**

**2️⃣ Open Arduino IDE****
**Install ESP32 board support.
Install required libraries.

**3️⃣ Connect Hardware**
Connect ESP32, GSM, LoRa, and LCD modules properly.
**4️⃣ Upload Code**
Upload transmitter code to transmitter ESP32.
Upload receiver code to receiver ESP32.
**5️⃣ Power ON**
Turn ON the switch.
System starts automatic communication monitoring.

**📊 Expected Output**
Telegram message during normal internet condition
GSM message during internet failure
LoRa communication during GSM failure
LCD status monitoring

**📈 Advantages**
Reliable communication system
Automatic backup switching
Long distance communication
Low power consumption
Portable and compact
Real-time monitoring
Emergency communication support

**🌍 Applications**
Disaster management systems
Military communication
Remote monitoring systems
Industrial alert systems
Smart agriculture
Emergency communication networks
Rural communication systems
IoT safety systems

**🔮 Future Improvements**
GPS integration
Mobile application support
Cloud data storage
AI-based network prediction
Solar power integration
AES encrypted communication
Sensor integration

**📌 Conclusion**
OmniLink is a smart and reliable communication backup system that ensures continuous communication using multiple technologies like Wi-Fi, GSM, and LoRa.
The project automatically switches between communication methods whenever failure occurs. This makes the system highly reliable for emergency and remote communication applications.
The integration of IoT, GSM, and LoRa technologies creates a robust and efficient real-time communication platform.

**👩‍💻 Author**
Smitarani Sahoo
Electronics & Telecommunication Engineering
📜 License
This project is developed for educational and research purposes.
⭐ GitHub Repository Tags
Plain text
ESP32
IoT
LoRa
GSM
TelegramBot
WirelessCommunication
EmbeddedSystems
Arduino
FallbackCommunication
EmergencyCommunication# OmniLink---Multi-Level-Communication-Backup-System
