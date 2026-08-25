```
#include <Wire.h>
#include <MPU6050.h>
#include <TinyGPS++.h>
#include <WiFi.h>
#include <HTTPClient.h>

// Twilio Credentials
const char* account_sid = "AC3ca9c4989c4f32d8d48547f2ea3a05cd";
const char* auth_token = "65bd1217356453dbc139c999cf9bc84f";
const char* to_phone_number = "+919923250304";
const char* twilio_phone_number = "+19255267173";

// WiFi Credentials
const char* ssid = "Airtel-MyWiFi-AMF-311WW-86BD";
const char* password = "211ce5fd";

// MPU6050 and GPS setup
MPU6050 mpu;
TinyGPSPlus gps;
HardwareSerial ss(1);  // GPS Serial communication on UART1

// Variables for accelerometer data
int16_t ax, ay, az;
float accelX, accelY, accelZ;
const float impactThreshold = 2.0;  // Adjust as needed

void setup() {
  Serial.begin(115200);
  ss.begin(9600, SERIAL_8N1, 16, 17);  // GPS on TX=16, RX=17
  Wire.begin();
  mpu.initialize();

  // Check for MPU6050 connection
  if (!mpu.testConnection()) {
    Serial.println("MPU6050 connection failed!");
    while (1);  // Halt on failure
  }

  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi!");
}

void loop() {
  mpu.getAcceleration(&ax, &ay, &az);
  accelX = ax / 16384.0;
  accelY = ay / 16384.0;
  accelZ = az / 16384.0;

  float totalAcceleration = sqrt(accelX * accelX + accelY * accelY + accelZ * accelZ);

  if (totalAcceleration > impactThreshold) {
    Serial.println("Accident Detected!");

    // Read GPS location
    while (ss.available() > 0) {
      gps.encode(ss.read());
    }

    if (gps.location.isValid()) {
      String latitude = String(gps.location.lat(), 6);
      String longitude = String(gps.location.lng(), 6);
      sendSMS(latitude, longitude);
    } else {
      Serial.println("Waiting for valid GPS data...");
    }
  }
  delay(1000);
}

void sendSMS(String latitude, String longitude) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String message = "Accident Detected! Location: http://maps.google.com/?q=" + latitude + "," + longitude;

    // Twilio API endpoint
    String url = "https://api.twilio.com/2010-04-01/Accounts/" + String(account_sid) + "/Messages.json";
    
    // Create a URL encoded message
    String data = "To=" + String(to_phone_number) + "&From=" + String(twilio_phone_number) + "&Body=" + message;
    
    http.begin(url);
    http.setAuthorization(account_sid, auth_token);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");
    
    int httpResponseCode = http.POST(data);
    if (httpResponseCode > 0) {
      Serial.println("Message Sent Successfully!");
    } else {
      Serial.println("Error Sending Message: " + String(httpResponseCode));
    }
    
    http.end();
  }
}
```
