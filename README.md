# ESP_TEMHUforFAN
โค้ดใช้ในการแก้ไขโปรแกรมสำหรับเครื่องวัดอุณหภูมิความชื้นและระบายอากาศแจ้งเตือนผ่านไลน์


#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "DHT.h"
#include "Ticker.h"
#include <TridentTD_LineNotify.h>
#define DHTPIN D4 // ขาของ DHT
#define Fan D6 // ขาของ Relay

#define DHTTYPE DHT11 
#define LINE_TOKEN  "5o90fAI640ZF9Mrmns6Rd2HpRFqWr1jzPb3uGwrKDoU" // ไลน์ Token Notify


#define TEST  1 // 0 = อ่านค่าจากเซ็นเซอร์  1 = จำลองค่าจากการพิมพ์ในมอนิเตอร์

int SP = 30 ; // อุณหภูมิที่ไม่ต้องการให้เกิน
int SET_Time_M = 1 ; // ตั้งเวลาที่พัดลมจะทำงานต่อหลังจากอุณหภูมิเกิน



float h = 0; 
float t = 0;
float f = 0;

void READ_TEMP() ;
void PRINT() ;

int ST =  0 ;

int H = 0, M = 0, S = 0, i = 0;

const unsigned long eventInterval = 250;
unsigned long previousTime = 0;

DHT dht(DHTPIN, DHTTYPE);
Ticker  count_time1;
LiquidCrystal_I2C lcd(0x27, 16, 2);


String a;


void setup() {
  Serial.begin(115200);
  Serial.println(LINE.getVersion());

  WiFi.begin("WA0852777338_2.4GHz", "0617131914"); // ไวไฟ และ รหัสไวไฟ
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  pinMode(Fan, OUTPUT);

  digitalWrite(Fan, HIGH);

  lcd.begin();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("LCD SERVER 1");
  lcd.setCursor(0, 1);
  lcd.print("--Online--");
  lcd.clear();

  delay(3000);

  dht.begin();
  count_time1.attach(0.1, c_time);
  LINE.setToken(LINE_TOKEN);

}

void loop() {

  if (TEST == 1) {


    while (Serial.available()) {

      a = Serial.readString();

      Serial.println(a.toInt());
      t = a.toInt();
    }
  } else {
    READ_TEMP();
  }
  
  unsigned long currentTime = millis();
  if (currentTime - previousTime >= eventInterval) {
    previousTime = currentTime ;


    PRINT() ;

    lcd.setCursor(0, 0);
    lcd.print("T: " + String(t) + " H: " + String(h));

    lcd.setCursor(0, 1);
    lcd.print("Timer : " + String(H) + ":" + String(M) + ":" + String(S) + "    ");

  }

  switch (ST) {
    case 1:
      digitalWrite(Fan, LOW);

      if (M >= SET_Time_M && t <= SP ) {
        LINE.notify("FAN_OFF");
        ST = 0 ;
        H = 0;
        M = 0;
        S = 0;
        i = 0;
      }

      if (M >= SET_Time_M && t > SP  ) {
        LINE.notify("อุณหภูมิ เกินกำหนด");
        ST = 2 ;
        H = 0;
        M = 0;
        S = 0;
        i = 0;
      }

      break;
    case 2:
      if (t > SP  ) {
        LINE.notify("FAN_OFF");
        ST = 0 ;
        H = 0;
        M = 0;
        S = 0;
        i = 0;
      }

      break;
    default:

      digitalWrite(Fan, HIGH);
      if (t >= SP ) {
        LINE.notify("FAN_ON");
        ST = 1 ;
        H = 0;
        M = 0;
        S = 0;
        i = 0;
      }

      break;
  }


  //  digitalWrite(Fan, HIGH);
  //  delay(500);
  //  digitalWrite(Fan, LOW);
  //  delay(500);

}

void PRINT() {

  Serial.print(" อุณหภูมิ Server1 "); Serial.print(t);
  Serial.print(" TIMER : ");
  Serial.print(H); Serial.print(":");
  Serial.print(M); Serial.print(":");
  Serial.print(S);

  Serial.println(" ");
}

void READ_TEMP() {

  h = dht.readHumidity();
  t = dht.readTemperature();
  f = dht.readTemperature(true);

  if (isnan(h) || isnan(t) || isnan(f)) {
    Serial.println(F("Failed to read from DHT sensor!"));
    return;
  }
  float hif = dht.computeHeatIndex(f, h);
  float hic = dht.computeHeatIndex(t, h, false);
}
void c_time() {
  if (ST == 1 ) {
    i++;
    if (i >= 10) {
      S++;
      i = 0;
    }
    if (S >= 60) {
      M++;
      S = 0;
    }
    if (M >= 60) {
      H++;
      M = 0;
    }
  }
}
