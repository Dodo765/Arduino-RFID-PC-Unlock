/*
 * 10 - SDA b
 * 14 - MISO g
 * 15 - SCK y
 * 16 - MOSI o
 * 
 * Created by Dominik Kawalec
 */

#include <SPI.h>
#include <MFRC522.h>\


int relay1Pin = 5; //pc
int relay2Pin = 8; //pendrive
bool skan = false;
int timek = 60; //czas pendrive w sec
constexpr uint8_t RST_PIN = 9;
constexpr uint8_t SS_PIN = 10;
MFRC522 mfrc522(SS_PIN, RST_PIN);
int RXLED = 17;

void setup() {
  pinMode(RXLED, OUTPUT); 
  Serial.begin(9600);
  pinMode(relay1Pin, OUTPUT);
  pinMode(relay2Pin, OUTPUT);
  digitalWrite(relay1Pin, LOW);
  digitalWrite(relay2Pin, LOW);
  timek = timek*1000;
  SPI.begin();
  mfrc522.PCD_Init();
  mfrc522.PCD_DumpVersionToSerial();
}

void readHex(byte *buffer, byte bufferSize) {
  for (byte i = 0; i < bufferSize; i++) {
    Serial.print(buffer[i] < 0x10 ? " 0" : " ");
    Serial.print(buffer[i], HEX);
  }
}
void loop() {
  if ( ! mfrc522.PICC_IsNewCardPresent()) {
    digitalWrite(RXLED, HIGH); 
    return;
  }

  if ( ! mfrc522.PICC_ReadCardSerial()) {
    return;
  }

  digitalWrite(RXLED, LOW);
  Serial.print("RFID UID: ");
  readHex(mfrc522.uid.uidByte, mfrc522.uid.size);
  Serial.println();
  skan = false;

  if (mfrc522.uid.uidByte[0] == 0x04 && //UID mojego zegarka
      mfrc522.uid.uidByte[1] == 0x17 &&
      mfrc522.uid.uidByte[2] == 0x2D &&
      mfrc522.uid.uidByte[3] == 0x13 &&
      mfrc522.uid.uidByte[4] == 0x9A &&
      mfrc522.uid.uidByte[5] == 0x57 &&
      mfrc522.uid.uidByte[6] == 0x80

) {
    Serial.println("Poprawne UID");
      
    digitalWrite (relay1Pin, HIGH);
    delay(300);
    digitalWrite (relay1Pin, LOW);
    Serial.println ("PC włączony");
  
    digitalWrite (relay2Pin, HIGH);
    Serial.println ("Pendrive włączony");
    delay(timek); 
    digitalWrite (relay2Pin, LOW);
    Serial.println ("Pendrive wyłączony");
    skan = true;
  }
  
  if (mfrc522.uid.uidByte[0] == 0x9A && //UID breloka
      mfrc522.uid.uidByte[1] == 0x0D &&
      mfrc522.uid.uidByte[2] == 0x8B &&
      mfrc522.uid.uidByte[3] == 0x80 
) {
    Serial.println("Poprawne UID");
      
    digitalWrite (relay1Pin, HIGH);
    delay(300);
    digitalWrite (relay1Pin, LOW);
    Serial.println ("PC włączony");
  
    digitalWrite (relay2Pin, HIGH);
    Serial.println ("Pendrive włączony");
    delay(timek); //30 sec.
    digitalWrite (relay2Pin, LOW);
    Serial.println ("Pendrive wyłączony");
    skan = true;
  }
  if(skan==false) {
    Serial.println("Nie wykryto UID lub jest ono niezgodne");
    delay(500);
    return;
  }
  
  digitalWrite(RXLED, HIGH);
  delay(500);   
}
