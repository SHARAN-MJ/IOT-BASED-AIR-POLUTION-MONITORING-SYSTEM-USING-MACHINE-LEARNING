## Title of the Project
IoT Based Air Pollution Monitoring System

This IoT-based air pollution monitoring system provides real-time tracking of air quality, enabling proactive responses to pollution through accessible data and alerts.

## About
<!--Detailed Description about the project-->
The IoT-based air pollution monitoring system is designed to provide real-time tracking and analysis of air quality by measuring key pollutants such as carbon monoxide (CO), particulate matter (PM2.5), and other harmful gases. Using a network of sensors connected to a microcontroller, the system continuously collects environmental data, which is then transmitted to an IoT platform for visualization and analysis. Users can monitor pollution levels through an intuitive dashboard and receive notifications if air quality exceeds safe thresholds, enabling timely responses to environmental risks. With added predictive capabilities, the system can also forecast pollution trends, offering a proactive solution for maintaining healthier air quality in urban and industrial areas.

## Features
<!--List the features of the project as shown below-->
- Recieves Tempareture and Humidity data from DHT11 Senosr.
-  Recieves Gas Threshold Value From MQ135 Sensor.
-  The values are pushed via virtual pins to an iot application called BLynk.
-   Continous Monitoring of the Gases Threshold Level and Weather.
-   Future upgradable.

## Requirements
<!--List the requirements of the project as shown below-->
1. Hardware Requirements
   
Sensors:
MQ135 (for gases like CO₂, ammonia, NO₂)

DHT11 or DHT22 (for temperature and humidity)
   
ESP32 (to collect and process sensor data)

2. Software Requirements

Arduino IDE or PlatformIO (for coding and uploading firmware to the microcontroller)

Blynk, ThingsBoard, or Adafruit IO (for real-time data visualization and alerts)

Firebase, AWS IoT, or ThingSpeak (to store and manage historical data)

Blynk app or custom mobile app for real-time monitoring and notifications


## System Architecture
<!--Embed the system architecture diagram as shown below-->
![upgrade_architect](https://github.com/user-attachments/assets/9d60784f-ec3e-4b1f-9685-6e438772397a)

## Program
## Arduino Code
```
#include <Arduino.h>
#include <ESP32WiFi.h>
#include <ESP32HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>
#include <DHT.h>
#include <BlynkSimpleEsp8266.h>

#define BLYNK_TEMPLATE_ID "TMPL3NLw3Kr-5"
#define BLYNK_TEMPLATE_NAME "AIR Quality"
#define BLYNK_AUTH_TOKEN "Z29qqK8mbQH9sf2iGEojdfhLvmAg6AFN"

const char* ssid = "TRYME";
const char* password = "12345678";
const char* gasIFTTTKey = "bm3rXYTDiawPYVrzmRvBmY";

#define DHTPIN 2    // DHT11 data pin is connected to GPIO 2
#define DHTTYPE DHT11 // DHT type is DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);

  // Connect to Wi-Fi
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }

  // Initialize Blynk
  Blynk.begin(BLYNK_AUTH_TOKEN);
  dht.begin();
}

void loop() {
  Blynk.run();

  // Check gas value
  int gasValue = analogRead(A0); // Assuming MQ135 sensor is connected to analog pin A0

  if (gasValue > 200) {
    Serial.println("Gas value exceeds threshold. Sending HTTPS request to IFTTT...");

    // Trigger IFTTT event
    sendIFTTTEvent();
  }

  // Read DHT11 values
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  // Send DHT11 data to Blynk
  Blynk.virtualWrite(V0, temperature);
  Blynk.virtualWrite(V1, humidity);

  // Wait for the next round
  delay(120000);
}

void sendIFTTTEvent() {
  std::unique_ptr<BearSSL::WiFiClientSecure> client(new BearSSL::WiFiClientSecure);
  client->setInsecure();

  HTTPClient https;

  Serial.print("[HTTPS] begin...\n");
  if (https.begin(*client, "https://maker.ifttt.com/trigger/gas_leakd/with/key/" + String(gasIFTTTKey))) {
    Serial.print("[HTTPS] GET...\n");
    int httpCode = https.GET();

    // httpCode will be negative on error
    if (httpCode > 0) {
      Serial.printf("[HTTPS] GET... code: %d\n", httpCode);
      if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
        String payload = https.getString();
        Serial.println(payload);
      }
    } else {
      Serial.printf("[HTTPS] GET... failed, error: %s\n", https.errorToString(httpCode).c_str());
    }

    https.end();
  } else {
    Serial.printf("[HTTPS] Unable to connect\n");
  }
}
```
## Machine Learning Algorithm:
```
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_squared_error
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
import matplotlib.pyplot as plt
from twilio.rest import Client

# Read data from CSV file
file_path = 'Tox.csv'
data = pd.read_csv(file_path)
# Your Twilio client
client = Client(account_sid, auth_token)

# Threshold for high toxicity (adjust as needed)
high_toxicity_threshold = 0.100
# Split the data into training and testing sets
train_data, test_data, train_toxicity, test_toxicity = train_test_split(
    data[['PPM', 'Humidity', 'Temperature']],
    data['Toxicity'],  # Change 'Leak' to 'Toxicity'
    test_size=0.2,
    random_state=42
)

# Create a simple machine learning pipeline for regression
model = Pipeline([
    ('scaler', StandardScaler()),  # Standardize features
    ('regressor', RandomForestRegressor(random_state=42))  # Use a RandomForestRegressor for regression
])

# Train the model
model.fit(train_data, train_toxicity)

# Make predictions on the test set
predictions = model.predict(test_data)

# Evaluate the model
mse = mean_squared_error(test_toxicity, predictions)
print(f'Mean Squared Error: {mse:.2f}')

# Plot the flow of three values
plt.figure(figsize=(10, 6))
plt.plot(predictions, label='Predictions')
plt.plot(test_toxicity.values, label='Actual Toxicity')
plt.xlabel('Data Points')
plt.ylabel('Toxicity Level')
plt.title('Predicted vs Actual Toxicity')
plt.legend()
plt.show()
plt.figure(figsize=(10, 6))
plt.plot(data['PPM'], label='PPM')
plt.plot(data['Humidity'], label='Humidity')
plt.plot(data['Temperature'], label='Temperature')
plt.xlabel('Data Points')
plt.ylabel('Values')
plt.title('Values from CSV File')
plt.legend()
plt.show()
# Now, you can use this trained model to make predictions on new data
# Replace the following values with your actual sensor readings
new_data = pd.DataFrame({
    'PPM': [500],
    'Humidity': [50],
    'Temperature': [25]
})

predicted_toxicity = model.predict(new_data)[0]

# Classify toxicity level based on specified ranges
if predicted_toxicity < 100:
    toxicity_level = "Low toxicity"
elif 100 <= predicted_toxicity < 500:
    toxicity_level = "Moderate toxicity"
else:
    toxicity_level = "High toxicity"

print(f'Predicted Toxicity: {predicted_toxicity:.2f} - {toxicity_level}')

if predicted_toxicity >= high_toxicity_threshold:
    # Send SMS notification
    message_body = f'Gas toxicity level is high: {predicted_toxicity:.2f}'
    message = client.messages.create(
        body=message_body,
        from_=twilio_phone_number,
        to=recipient_phone_number
    )
    print(f'SMS Sent: {message.sid}')
else:
    print('Gas toxicity level is not high. No SMS sent.')

```
## Basic Trigger HTML Code:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="refresh" content="0;https://maker.ifttt.com/trigger/gas_leakd/with/key/bm3rXYTDiawPYVrzmRvBmY">
    <title>Redirecting...</title>
</head>
<body>
    <p>If you are not redirected, <a href="URL_TO_REDIRECT">click here</a>.</p>
</body>
</html>
```
## DataSet Transmission Code:
```
void sen()
{
// HTTPClient https;
  if (!client.connect(host, httpsPort))
  {
    Serial.println("connection failed");
    return;
  }

  String url = "/macros/s/" + GAS_ID + "/exec?value1="+name1+"&value2="+age1;
  Serial.print("requesting URL: ");
  Serial.println(url);
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
         "Host: " + host + "\r\n" +
         "User-Agent: BuildFailureDetectorESP8266\r\n" +
         "Connection: close\r\n\r\n");

  Serial.println("request sent");
  while (client.connected()) {
    String line = client.readStringUntil('\n');
    if (line == "\r") {
      Serial.println("headers received");
      break;
    }
  }
  String line = client.readStringUntil('\n');
  if (line.startsWith("{\"state\":\"success\"")) 
  {
    Serial.println("esp8266/Arduino CI successfull!");
  } 
  else 
  {
    Serial.println("esp8266/Arduino CI has failed");
  }
  Serial.print("reply was : ");
  Serial.println(line);
  Serial.println("closing connection");
  Serial.println("==========");
  Serial.println();
  ```
## Output
<!--Embed the Output picture at respective places as shown below as shown below-->
## Blynk Monitering:
![image](https://github.com/user-attachments/assets/e8a8125b-f7c1-4e39-a03b-dac706dc4f05)

## Arduino Serial Moniter:
![image](https://github.com/user-attachments/assets/8deb674d-7e38-44ef-b294-98ef4c0c01ff)

## ML Algorithm Predictions:
![image](https://github.com/user-attachments/assets/16a021b1-dfbc-4ba0-a2b9-117cfccbae81)

## Graphical Representation:
![image](https://github.com/user-attachments/assets/3afaa1ac-dc99-4a91-a1d1-e1c0947f6b95)

![image](https://github.com/user-attachments/assets/7bbdca6a-21c7-4f76-ae10-b5e0bd03b9b4)

## ALERTS GENERATED:
![image](https://github.com/user-attachments/assets/a2e2de0c-db83-4461-8fb5-ec2be666698a)

## Results and Impact
<!--Give the results and impact as shown below-->
The results of implementing the IoT-based air pollution monitoring system demonstrate its effectiveness in providing real-time, accurate data on key pollutants such as CO and PM2.5, which are critical to environmental health. Users benefit from the ability to visualize air quality data through an intuitive dashboard, receive instant notifications when pollution levels exceed safe limits, and analyze historical trends to understand pollution patterns. The system’s predictive capabilities also allow users to anticipate pollution spikes, enabling proactive interventions that help reduce exposure to harmful pollutants. Overall, the project has a significant impact by enhancing awareness, promoting healthier living and working conditions, and supporting data-driven decisions for pollution management in urban and industrial areas.

## Articles published / References
1. X. Mao, X. Miao, Y. He, X.-Y. Li, and Y. Liu, “CitySee: Urban CO2 monitoring with sensors,” in INFOCOM, 2012 Proceedings IEEE,pp. 1611–1619. 

2. M. S. Munir, S. N. Khan, M. J. Khan, and M. U. Farooq, "Design and development of IoT enabled air quality monitoring system for smart cities," 2019 IEEE 23rd International Conference on Automation and Computing (ICAC).

3. A. R. Al-Ali, I. Zualkernan, and F. Aloul, "A mobile GPRS-sensors array for air pollution monitoring," IEEE Sensors Journal, vol. 10, no. 10, pp. 1666-1671, 2010.

4. B. Sharma and V. H. Mahalle, ”Air pollution monitoring system using IoT,”3rd International Conference on Trends in Electronics and Informatics (ICOEI), IEEE, 2019.





