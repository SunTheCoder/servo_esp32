#include <WiFi.h>
#include <WebServer.h>
#include <ESP32Servo.h>

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// === OLED Setup ===
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 32
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// === Servo Setup ===
Servo servo;
int angle = 90;  // Start at middle
const int servoPin = 13;

// === Web Server Setup ===
WebServer server(80);

const char* ssid = "ESP32-Robot";
const char* password = "servo1234";

// === HTML UI ===
const char HTML[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head><title>ESP32 Servo</title></head>
<body style="text-align:center; font-family:sans-serif;">
  <h2>Web Controlled Servo</h2>
  <input type="range" min="0" max="180" value="90" id="slider" oninput="send(this.value)">
  <p>Angle: <span id="val">90</span>°</p>

<script>
function send(val) {
  document.getElementById("val").innerText = val;
  fetch("/servo?angle=" + val);
}
</script>
</body>
</html>
)rawliteral";

void setup() {
  Serial.begin(115200);

  // Servo
  servo.attach(servoPin);
  servo.write(angle);

  // OLED
  Wire.begin(21, 22); // SDA, SCL
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("ESP32 Servo Ready");
  display.display();

  // Wi-Fi AP
  WiFi.softAP(ssid, password);
  IPAddress IP = WiFi.softAPIP();
  Serial.print("AP IP address: "); Serial.println(IP);

  // Web server routes
  server.on("/", []() {
    server.send_P(200, "text/html", HTML);
  });

  server.on("/servo", []() {
    if (server.hasArg("angle")) {
      angle = server.arg("angle").toInt();
      angle = constrain(angle, 0, 180);
      servo.write(angle);

      // Update OLED
      display.clearDisplay();
      display.setCursor(0, 0);
      display.print("Angle: ");
      display.println(angle);
      display.display();

      Serial.print("Servo angle set to: ");
      Serial.println(angle);
      server.send(200, "text/plain", "OK");
    }
  });

  server.begin();
}

void loop() {
  server.handleClient();
}
