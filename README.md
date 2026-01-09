#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "YOUR_WIFI";
const char* password = "YOUR_PASSWORD";
const char* mqtt_server = "broker.hivemq.com";

WiFiClient espClient;
PubSubClient client(espClient);

#define FLOW_SENSOR 34

float waterUsed = 0;

void setup() {
  Serial.begin(9600);
  pinMode(FLOW_SENSOR, INPUT);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  client.setServer(mqtt_server, 1883);
}

void loop() {
  if (!client.connected()) {
    client.connect("AquaTrackDevice");
  }

  int sensorValue = analogRead(FLOW_SENSOR);
  waterUsed += sensorValue * 0.001;  // simulated flow

  char data[50];
  sprintf(data, "%.2f", waterUsed);

  client.publish("aquatrack/water", data);
  Serial.println(data);

  delay(5000);
}





## Backend Code
from flask import Flask, jsonify
from mqtt_client import water_data

app = Flask(__name__)

DAILY_LIMIT = 100  # liters

@app.route("/status")
def status():
    alert = "NORMAL"
    if water_data > DAILY_LIMIT:
        alert = "LIMIT EXCEEDED"

    return jsonify({
        "water_used": water_data,
        "status": alert
    })

if __name__ == "__main__":
    app.run(debug=True)


import paho.mqtt.client as mqtt

water_data = 0

def on_message(client, userdata, msg):
    global water_data
    water_data = float(msg.payload.decode())
    print("Water Used:", water_data)

client = mqtt.Client()
client.connect("broker.hivemq.com", 1883)
client.subscribe("aquatrack/water")
client.on_message = on_message
client.loop_start()


import sqlite3

conn = sqlite3.connect("aquatrack.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS usage (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    water_used REAL
)
""")

def insert_data(value):
    cursor.execute("INSERT INTO usage (water_used) VALUES (?)", (value,))
    conn.commit()
flask
paho-mqtt
sqlite3

import pandas as pd
from sklearn.linear_model import LinearRegression

data = pd.read_csv("../data/sample_data.csv")

X = data[['day']]
y = data['water_used']

model = LinearRegression()
model.fit(X, y)

future_day = [[31]]
prediction = model.predict(future_day)

print("Predicted Water Usage:", prediction[0])
 ## Data Samples
 day,water_used
1,80
2,82
3,85
4,90
5,95
6,100
## DashBoard 
import matplotlib.pyplot as plt

water_usage = [80, 85, 90, 95, 110]

plt.plot(water_usage)
plt.xlabel("Days")
plt.ylabel("Water Usage (Liters)")
plt.title("AquaTrack Water Consumption")
plt.show()

 
 






