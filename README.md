#include <ESP8266HTTPClient.h>
#include <ESP8266WiFi.h>
const char* ssid = "RI";
const char* pass = "12345678";
String url ;
//siapkan wifi client
WiFiClient client;
//tempat sampah........................
#include <Servo.h>
Servo tutup;
const int trigPin = 4;
const int echoPin = 5;
int pirPin = 16;                           
int sensorValue = LOW; 
int Buzzer = 13;
int duration, distance;
void setup() {
  Serial.begin (115200);
  WiFi.hostname ("Smart_Trash");
  WiFi.begin (ssid, pass);
  //Tong Sampah........................
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(Buzzer, OUTPUT);
  pinMode(pirPin, INPUT);    
  tutup.attach (14);
  tutup.write (135);
  while (WiFi.status() != WL_CONNECTED) {
        digitalWrite (Buzzer, LOW);
    Serial.print("....");
    delay (500);
  }
    digitalWrite (Buzzer, HIGH);
    delay (100);
    digitalWrite (Buzzer, LOW);
  Serial.print("\n");
  Serial.print("IP address : ");
  Serial.print(WiFi.localIP());
  Serial.print("\n");
  Serial.print("MAC : ");
  Serial.println(WiFi.macAddress());
  Serial.println("");
  Serial.print("Terhubung dengan : ");
  Serial.println(ssid);
}
void loop() {
  sensorValue = digitalRead(pirPin);         //Mebaca input pin sensor pir
  Serial.println (sensorValue);
  digitalWrite(trigPin, LOW);
  delayMicroseconds(3);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  distance = (duration / 2) / 29.1;
  Serial.println(distance);
  
  if (distance < 25 )
  {
    tutup.write (0);
    tone(Buzzer, 1023);
    delay (1000);
    digitalWrite(Buzzer,LOW);
    
  }
  if (distance == 0 )
  {
    digitalWrite (Buzzer, LOW);
    tutup.write (135);
  }
  if ( sensorValue == HIGH) {    
    Serial.println("Penuh");
    send_wa ("Warning!!! Sampah Penuh");
    tone(Buzzer, 1000);         
    delay(500);                             
    noTone(Buzzer);            
    delay(100);     
    tone(Buzzer, 1000);         
    delay(500);                             
    noTone(Buzzer);            
    delay(100);     
    tone(Buzzer, 1000);         
    delay(500);                             
    noTone(Buzzer);            
    delay(100);            
  }
else {
  digitalWrite (Buzzer, LOW);
    tutup.write (135);
}
}

//logika untuk pengiriman pesan.......
void send_wa (String pesan) {
  url = "http://api.callmebot.com/whatsapp.php?phone=6285894748805&text=" + urlencode(pesan) + "&apikey=6869828";
  //kirim pesan
  postData ();
}
void postData () {
  int httpCode;
  HTTPClient http;
  http.begin (client, url);
  httpCode = http.POST(url);
  if (httpCode == 200) {
    Serial.println ("Notifikasi WA berhasil terkirim");
  }
  else {
    Serial.println ("Notifikasi WA gagal terkirim");
  }
  http.end();
}

String urlencode (String str) {
  String encodedString = "";
  char c;
  char code0, code1, code2;
  for (int i = 0; i < str.length (); i++) {
    c = str.charAt (i);
    //mengubah sepasi yang dikirim menjadi tanda +
    if (c == ' ') {
      encodedString += '+' ;
    }
    else if (isalnum (c)) {
      encodedString += c;
    }
    else {
      code1 = (c & 0xf) + '0';
      if ((c & 0xf) > 9) {
        code1 = (c & 0xf) - 10 + 'A';
      }
      c = (c >> 4) & 0xf;
      code0 = c + '0';
      if (c > 9) {
        code0 = c - 10 + 'A';
      }
      code2 = '\0';
      encodedString += '%';
      encodedString += code0 ;
      encodedString += code1 ;
    }
    yield ();
  }
  Serial.println (encodedString);
  return encodedString;
}
