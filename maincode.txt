// Informasi Blynk
#define BLYNK_TEMPLATE_ID "TMPL6ENfzfbtc"
#define BLYNK_TEMPLATE_NAME "sensor 2"
#define BLYNK_AUTH_TOKEN "paYTmX4FFBGaU7OawaUBHlc7iiEenF1f"

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>
#include <Adafruit_INA219.h>
#include <time.h>  
#include "esp_task_wdt.h"  // Watchdog Timer ESP32
#include "FS.h"
#include "SD.h"
#include "SPI.h"

#define CS_PIN 5  // Pin Chip Select untuk microSD

// Informasi WiFi
const char* ssid = "POCO X3 NFC";              
const char* password = "bima1234";   

// Konfigurasi DHT22
#define DHTPIN 4                        
#define DHTTYPE DHT22                   
DHT dht(DHTPIN, DHTTYPE);

// Konfigurasi LCD I2C & INA219 (SDA 21, SCL 22)
#define SDA_PIN 21
#define SCL_PIN 22
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_INA219 ina219;

// **Deklarasi Pin LED**
#define LED_PIN 12  

// Konfigurasi NTP Server
const char* ntpServer = "pool.ntp.org";
const long gmtOffset_sec = 7 * 3600;   
const int daylightOffset_sec = 0;       
struct tm timeinfo;

// **Konfigurasi Watchdog Timer**
#define WDT_TIMEOUT 10  // Timeout dalam detik

void setup() {
  Serial.begin(115200);
  
    if (!SD.begin(CS_PIN)) {
        Serial.println("Gagal inisialisasi SD Card!");
    } else {
        Serial.println("SD Card siap digunakan.");

        // Cek apakah file datalog.csv ada, kalau belum buat baru
        File file = SD.open("/datalog.csv", FILE_READ); /////////////////////////////////////////////////////////////////////////////////////////////////////////////
        if (!file) {
            Serial.println("File datalog.csv tidak ditemukan, membuat baru...");
            file = SD.open("/datalog.csv", FILE_WRITE);
            if (file) {
                file.println("Waktu, Suhu, Kelembaban"); // Header CSV
                file.close();
                Serial.println("File datalog.csv berhasil dibuat!");
            } else {
                Serial.println("Gagal membuat file datalog.csv!");
           }
        } else {
            Serial.println("File datalog.csv ditemukan!");
            file.close();
       }
    }

  // Inisialisasi I2C untuk LCD dan INA219
  Wire.begin(SDA_PIN, SCL_PIN);
  lcd.init();
  lcd.noBacklight();
  lcd.setCursor(0, 0);
  lcd.print("Starting...");

  dht.begin();
  unsigned long wifiStartTime = millis();

  // Koneksi ke WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
    lcd.setCursor(0, 1);
    lcd.print("Connecting WiFi.. ");
    
    if (millis() - wifiStartTime > 10000) {
        Serial.println("WiFi failed! Restarting ESP32...");
        lcd.clear();
        lcd.print("WiFi Failed!");
        lcd.setCursor(0, 1);
        lcd.print("Restarting...");
        delay(2000);
        ESP.restart();
    }
  }

  Serial.println("Connected to WiFi");
  Serial.print("Server IP Address: ");
  Serial.println(WiFi.localIP());

  lcd.clear();
  lcd.print("WiFi Connected!");
  lcd.setCursor(0, 1);
  lcd.print(WiFi.localIP());

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, password);

  // Inisialisasi INA219
  if (!ina219.begin()) {
    Serial.println("Gagal mendeteksi INA219. Periksa koneksi!");
    while (1);
  }
  Serial.println("INA219 Terdeteksi!");

  // **Inisialisasi LED sebagai output**
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);

  // **Inisialisasi Watchdog Timer**
  esp_task_wdt_config_t wdtConfig = {
      .timeout_ms = WDT_TIMEOUT * 1000,
      .idle_core_mask = (1 << 0) | (1 << 1),
      .trigger_panic = true 
  };
  esp_task_wdt_init(&wdtConfig);
  esp_task_wdt_add(NULL); 

  // Sinkronisasi waktu NTP
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  Serial.println("Menunggu sinkronisasi waktu NTP...");

  if (!getLocalTime(&timeinfo)) {
      Serial.println("Gagal mendapatkan waktu NTP");
  } else {
      Serial.println("Waktu NTP berhasil diperoleh!");
  }
}

void loop() {
  // **Reset Watchdog Timer**
  esp_task_wdt_reset();

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("WiFi Lost! Restarting ESP32...");
    lcd.clear();
    lcd.print("WiFi Lost!");
    lcd.setCursor(0, 1);
    lcd.print("Restarting...");
    delay(2000);
    ESP.restart();
  }

  // Kirim data sensor ke Blynk & LCD
  sendSensorData();

  // Cetak waktu ke Serial Monitor
  Serial.println("Waktu sekarang: " + getFormattedTime());

  delay(1000);
}

// Fungsi untuk mendapatkan waktu dalam format HH:MM:SS DD/MM/YYYY
String getFormattedTime() {
    if (!getLocalTime(&timeinfo)) {
        Serial.println("Gagal mendapatkan waktu!");
        return "00:00:00 01/01/1970";
    }

    char buffer[30];
    strftime(buffer, sizeof(buffer), "%H:%M:%S %d/%m/%Y", &timeinfo);
    return String(buffer);
}

void writeToSDCard(float temperature, float humidity) {
    if (!SD.begin(CS_PIN)) { // Cek apakah microSD masih terdeteksi
        Serial.println("MicroSD tidak terdeteksi! Pastikan kartu terpasang.");
        return;
    }

    File file = SD.open("/serverdatalog.csv", FILE_APPEND);
    if (file) {
        String logData = getFormattedTime() + "," + String(temperature, 2) + "," + String(humidity, 2);
        file.println(logData);
        file.close();
        Serial.println("Data berhasil ditulis ke SD Card: " + logData);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("writing");
    } else {
        Serial.println("Gagal membuka file di SD Card!");
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("SD card error");
    }
}

unsigned long lastSDWriteTime = 0;  // Variabel untuk menyimpan waktu terakhir pencatatan SD Card
const unsigned long sdWriteInterval = 1 * 60 * 1000;  // 5 menit dalam milidetik

void sendSensorData() {
  float h = dht.readHumidity();    
  float t = dht.readTemperature(); 
  String sensor1Data;

  if (isnan(h) || isnan(t)) {
    sensor1Data = "Sensor Error";
  } else {
    sensor1Data = "T1:" + String(t) + "C H1:" + String(h) + "%";
  }

  Blynk.virtualWrite(V0, t);
  Blynk.virtualWrite(V1, h);
  Serial.println("Server Sensor Sent: " + sensor1Data);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Temp: " + String(t) + "C");
  lcd.setCursor(0, 1);
  lcd.print("Hum: " + String(h) + "%");

  float busVoltage = ina219.getBusVoltage_V();
  Serial.print("Tegangan Bus: "); Serial.print(busVoltage); Serial.println(" V");

  if (busVoltage < 6.5) {
    digitalWrite(LED_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
  }

  // Cek apakah sudah 5 menit sejak terakhir menyimpan data ke SD card
  if (millis() - lastSDWriteTime >= sdWriteInterval) {
    writeToSDCard(t, h);  // Simpan data ke microSD
    lastSDWriteTime = millis();  // Update waktu terakhir pencatatan
  }

  delay(2000);
}
