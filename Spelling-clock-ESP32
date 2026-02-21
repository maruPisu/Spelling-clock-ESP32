#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_NeoPixel.h>
#include <RTClib.h>

// Configuration
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define LED_PIN 4
#define NUM_LEDS 110
#define LED_TYPE WS2812
#define MAX_TIME 46799
#define ONE_O_CLOCK 3600
#define MAX_HOUR_LETTERS 13
#define MAX_MINS_LETTERS 16
#define BUTTON 0
#define LED 2
#define OLED_ADDR 0x3C 

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
Adafruit_NeoPixel strip(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800);
RTC_DS3231 rtc;

const unsigned long debounceDelay = 50;
const unsigned long doubleClickDelay = 400;
unsigned long lastDebounceTime = 0;
unsigned long lastClickTime = 0;
int clickCount = 0;
unsigned long previousMillis = 0;
const long interval = 100;

short arrowPosition = 0;
bool lastButtonState = HIGH;
bool buttonState;
int ledState = LOW;
char timeBuf[9];  // "HH:MM:SS" + '\0'

const int hours[][MAX_HOUR_LETTERS] = {
  { 0, 1, 5, 6, 8, 9, 10, -1 },                      // 1
  { 1, 2, 3, 5, 6, 7, 11, 12, 13, -1 },              // 2
  { 1, 2, 3, 5, 6, 7, 46, 47, 48, 49, -1 },          // 3
  { 1, 2, 3, 5, 6, 7, 22, 23, 24, 25, 26, 27, -1 },  // 4
  { 1, 2, 3, 5, 6, 7, 34, 35, 36, 37, 38, -1 },      // 5
  { 1, 2, 3, 5, 6, 7, 13, 14, 15, 16, -1 },          // 6
  { 1, 2, 3, 5, 6, 7, 49, 50, 51, 52, 53, -1 },      // 7
  { 1, 2, 3, 5, 6, 7, 38, 39, 40, 41, -1 },          // 8
  { 1, 2, 3, 5, 6, 7, 55, 56, 57, 58, 59, -1 },      // 9
  { 1, 2, 3, 5, 6, 7, 17, 18, 19, 20, -1 },          // 10
  { 1, 2, 3, 5, 6, 7, 27, 28, 29, 30, -1 },          // 11
  { 1, 2, 3, 5, 6, 7, 61, 62, 63, 64, -1 }           // 12
};

const int mins[][MAX_MINS_LETTERS] = {
  { -1 },                                                                        // 0
  { 66, 77, 78, 79, 80, 81, -1 },                                                // 5
  { 66, 73, 74, 75, 76, -1 },                                                    // 10
  { 66, 88, 89, 90, 91, 92, 93, -1 },                                            // 15
  { 66, 82, 83, 84, 85, 86, 87, -1 },                                            // 20
  { 66, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, -1 },                  // 25
  { 66, 94, 95, 96, 97, 98, -1 },                                                // 30
  { 67, 68, 69, 70, 71, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, -1 },  // 35
  { 67, 68, 69, 70, 71, 82, 83, 84, 85, 86, 87, -1 },                            // 40
  { 67, 68, 69, 70, 71, 88, 89, 90, 91, 92, 93, -1 },                            // 45
  { 67, 68, 69, 70, 71, 73, 74, 75, 76, -1 },                                    // 50
  { 67, 68, 69, 70, 71, 77, 78, 79, 80, 81, -1 },                                // 55
};

const int strip_map[] = {  // 110 elements on 11 columns
  0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
  21, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11,
  22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32,
  43, 42, 41, 40, 39, 38, 37, 36, 35, 34, 33,
  44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54,
  65, 64, 63, 62, 61, 60, 59, 58, 57, 56, 55,
  66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76,
  87, 86, 85, 84, 83, 82, 81, 80, 79, 78, 77,
  88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98,
  109, 108, 107, 106, 105, 104, 103, 102, 101, 100, 99
};

const char letters[] = {
  'E', 'S', 'O', 'N', 'W', 'L', 'A', 'S', 'U', 'N', 'A',  // 0
  'D', 'O', 'S', 'E', 'I', 'S', 'D', 'I', 'E', 'Z', 'S',  // 11
  'C', 'U', 'A', 'T', 'R', 'O', 'N', 'C', 'E', 'X', 'T',  // 22
  'E', 'C', 'I', 'N', 'C', 'O', 'C', 'H', 'O', 'R', 'H',  // 33
  'F', 'Y', 'T', 'R', 'E', 'S', 'I', 'E', 'T', 'E', 'P',  // 44
  'N', 'U', 'E', 'V', 'E', 'G', 'D', 'O', 'C', 'E', 'J',  // 55
  'Y', 'M', 'E', 'N', 'O', 'S', 'K', 'D', 'I', 'E', 'Z',  // 66
  'C', 'I', 'N', 'C', 'O', 'V', 'E', 'I', 'N', 'T', 'E',  // 77
  'C', 'U', 'A', 'R', 'T', 'O', 'M', 'E', 'D', 'I', 'A',  // 88
  'U', 'V', 'E', 'N', 'T', 'I', 'C', 'I', 'N', 'C', 'O'   // 99
};

int convertTime(int in_time) {
  int h = in_time / 60;
  int m = in_time - (h * 60);

  return h * 100 + m;
}

int getHour() {
  return rtc.now().hour();
}

int getMins() {
  return rtc.now().minute();
}

int getRoundedMins() {
  float tmp = getMins() / 5.0;
  return round(tmp) * 5;
}

int getUsableMins() {
  int rounded = getRoundedMins();
  if (rounded == 60)
    return 0;
  return rounded;
}

int getUsableHour() {
  int rounded = getRoundedMins();
  if (rounded > 30)
    return (getHour() + 1) % 12;
  return getHour();
}

void incrementMinutes(short amount) {
  DateTime now = rtc.now();

  // Add 60 seconds to the current time
  DateTime newTime = DateTime(
    now.year(),
    now.month(),
    now.day(),
    now.hour(),
    now.minute() + amount,  // Add 1 minute
    now.second());

  rtc.adjust(newTime);

  Serial.println("Minutes incremented by 1");
}

// Function to increment hours by 1
void incrementHours(short amount) {
  DateTime now = rtc.now();

  // Add 1 hour to the current time
  DateTime newTime = DateTime(
    now.year(),
    now.month(),
    now.day(),
    now.hour() + amount,  // Add 1 hour
    now.minute(),
    now.second());

  rtc.adjust(newTime);

  Serial.println("Hours incremented by 1");
}

// print the strip_map. Is better to hardcode the result, for better performance
void printMap(int rowLen) {
  int index = 0;
  bool backwards = false;
  while (index < NUM_LEDS) {
    for (int i = 0; i < rowLen; i++) {
      if (backwards) {
        Serial.print(index + (rowLen - i) - 1);
      } else {
        Serial.print(index + i);
      }
      Serial.print(",");
    }
    backwards = !backwards;
    index += rowLen;
  }
}

void drawArrow() {
  if (arrowPosition == 0) {
    return;
  }
  switch (arrowPosition) {
    case 0:
      break;
    case 1:
    case 2:
      display.setCursor((arrowPosition - 1) * 36, 20);
      display.print("^^");
      break;
    case 3:
    case 4:
      display.setCursor((arrowPosition - 3) * 36, 16);
      display.print("vv");
      break;
  }
}

void handleClick() {
  clickCount++;
  lastClickTime = millis();

  if (clickCount == 2) {
    clickCount = 0;
    arrowPosition = (arrowPosition + 1) % 5;
    Serial.printf("arrow %d", arrowPosition);
  }
}

void initStrip() {
  strip.begin();
  for (int i = 0; i < NUM_LEDS; i++) {
    strip.setPixelColor(i, strip.Color(0, 0, 0));
  }
  strip.show();  // Initialize all pixels to 'off'
}

void setup() {
  Serial.begin(115200);
  delay(1000);  // Important: Give serial time to start

  Wire.begin(21, 22);
  delay(100);
  
  Serial.println("Starting devices...");
  
  // Initialize OLED display
  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    Serial.println("OLED not found at 0x3C!");
  } else {
    Serial.println("OLED initialized");
    display.clearDisplay();
    display.setTextSize(1);
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 0);
    display.println("Display OK");
    display.display();
  }
  
  // Initialize RTC
  if (!rtc.begin()) {
    Serial.println("RTC not found!");
  } else {
    Serial.println("RTC initialized");
  }
  pinMode(LED, OUTPUT);
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);

  initStrip();
}

void loop() {
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    /*
    if (ledState == LOW) {
      ledState = HIGH;
    } else {
      ledState = LOW;
    }
    // set the LED with the ledState of the variable:
    digitalWrite(LED, ledState); */

    Serial.println();

    int led_index = 0;
    strip.clear();

    // Hour leds
    int my_hour = getUsableHour() - 1;
    for (int i = 0; i < MAX_HOUR_LETTERS; i++) {
      led_index = hours[my_hour][i];
      if (led_index == -1) break;
      strip.setPixelColor(strip_map[led_index], 255, 255, 255);
      Serial.print(letters[led_index]);
    }

    // Minutes leds
    int my_mins = getUsableMins() / 5;
    for (int i = 0; i < MAX_MINS_LETTERS; i++) {
      led_index = mins[my_mins][i];
      if (led_index == -1) break;
      strip.setPixelColor(strip_map[led_index], 255, 255, 255);
      Serial.print(letters[led_index]);
    }

    strip.show();
    // RTC read
    DateTime now = rtc.now();
    snprintf(
      timeBuf,
      sizeof(timeBuf),
      "%02d:%02d:%02d",
      now.hour(),
      now.minute(),
      now.second());

    // Refresh LCD
    display.clearDisplay();
    display.setCursor(1, 1);
    display.println(timeBuf);
    drawArrow();
    display.display();
  }

  //********** HANDLE CLICK / DOUBLE CLICK begin
  bool reading = digitalRead(BUTTON);

  // Debounce
  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != buttonState) {
      buttonState = reading;

      // Detect press (LOW)
      if (buttonState == LOW) {
        handleClick();
      }
    }
  }

  lastButtonState = reading;

  // Reset click count if too much time passed
  if (clickCount > 0 && (millis() - lastClickTime > doubleClickDelay)) {
    if (clickCount == 1) {
      // single click. Increment time
      switch (arrowPosition) {
        case 0:
          break;

        case 1:
          incrementHours(1);
          break;
        case 2:
          incrementMinutes(1);
          break;
        case 3:
          incrementHours(-1);
          break;
        case 4:
          incrementMinutes(-1);
          break;
      }
    }
    clickCount = 0;
  }
  //********** HANDLE CLICK / DOUBLE CLICK end
}