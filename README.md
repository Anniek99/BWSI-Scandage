# BWSI-Scandage
A wearable bandage wrap with sensors for post-operative abdominal area wound monitoring.

## Description

This project is designed to monitor the temperature of two areas on the skin using thermistors: the periwound area (around a wound) and the contralateral area (an area of skin away from the wound). By comparing these temperatures, the device can help identify potential signs of infection. If the temperature difference exceeds a certain threshold, the device will alert the user.

## Components

- Arduino Circuit Playground 
- Thermistors (e.g., MA300TA502B)
- Fixed resistors (4.7kΩ)
- Conductive thread
- Elastic band for wearable attachment

## Circuit Diagram

![Circuit Diagram]([https://github.com/yourusername/arduino-temperature-monitor/raw/main/path_to_your_circuit_diagram_image.png](https://github.com/Anniek99/BWSI-Scandage/blob/main/Circuit%20Diagram.png
))

## Code

The Arduino code collects temperature data from the thermistors, calculates the average temperature for the differnet areas, and determines if there is a significant temperature difference that could indicate an infection (2 °F).

### Arduino Code

```cpp
#include <math.h>

// Thermistor parameters
const float Beta = 3892;   // Beta parameter for the MA300TA502B thermistor
const float T0 = 298.15;   // Reference temperature in Kelvin (25°C)
const int R0 = 5000;       // Resistance at reference temperature (25°C)
const int fixedResistor = 4700; // 4.7kΩ fixed resistor

const int periwoundPins[] = {A3, A6};
const int contralateralPins[] = {A4, A5};
const int numReadings = 30; // Collect data for 30 seconds

void setup() {
  Serial.begin(115200);
  // Wait for Serial to initialize
  while (!Serial) {
    ; // wait for serial port to connect
  }
  for (int i = 0; i < 2; i++) {
    pinMode(periwoundPins[i], INPUT);
    pinMode(contralateralPins[i], INPUT);
  }
  Serial.println("System ready. Collecting data for 30 seconds...");
  collectAndProcessData();
}

void loop() {
  // 
}

void collectAndProcessData() {
  float periwoundSum = 0;
  float contralateralSum = 0;
  int readings = 0;

  unsigned long startTime = millis();
  while (millis() - startTime < 30000) { // Collect data for 30 seconds
    float periwoundAvg = 0;
    float contralateralAvg = 0;

    for (int i = 0; i < 2; i++) {
      periwoundAvg += calculateTemperature(calculateResistance(analogRead(periwoundPins[i])));
      contralateralAvg += calculateTemperature(calculateResistance(analogRead(contralateralPins[i])));
    }
    periwoundAvg /= 2;
    contralateralAvg /= 2;

    periwoundSum += periwoundAvg;
    contralateralSum += contralateralAvg;
    readings++;

    int secondsRemaining = 30 - (millis() - startTime) / 1000;
    Serial.print("Collecting data. Time remaining: ");
    Serial.print(secondsRemaining);
    Serial.println(" seconds");

    delay(1000); // Delay between readings
  }

  float finalPeriwoundAvg = periwoundSum / readings;
  float finalContralateralAvg = contralateralSum / readings;
  float tempDifference = finalPeriwoundAvg - finalContralateralAvg;

  Serial.print("Periwound: ");
  Serial.print(finalPeriwoundAvg);
  Serial.print(", Contralateral: ");
  Serial.print(finalContralateralAvg);
  Serial.print(", Difference: ");
  Serial.println(tempDifference);

  if (tempDifference >= 2.0) {
    Serial.println("You are showing signs of an infection. Check with your doctor.");
  } else if (tempDifference >= 1.7) {
    Serial.println("You are showing borderline signs of an infection. Monitor your symptoms closely.");
  } else {
    Serial.println("You are not displaying signs of infection. Continue to monitor your symptoms.");
  }
}

float calculateResistance(int analogValue) {
  float voltage = analogValue * (3.3 / 1023.0);
  return (fixedResistor * (3.3 - voltage)) / voltage;
}

float calculateTemperature(float resistance) {
  float temperatureK = 1.0 / ((1.0 / T0) + (1.0 / Beta) * log(resistance / R0));
  return temperatureK * 9.0 / 5.0 - 459.67; // Convert Kelvin to Fahrenheit
}
