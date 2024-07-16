---
title: Streaming Data from Arduino to Website Using Python
author: arpiku 
date: 2024-03-12 15:45:00 +0530
categories: [Python, Arduino, Programming, Embedded Programming]
tags: [WebSockets ,Python,Arduino, Embedded Programming]
pin: false 
---


So say you have an Arduino, and it is actively observing some data, say the humidity, temperature, ppm and such and you
would like to somehow be able to stream this data to a website, how can you achieve that?

The following is a documentation of a simple server and Arduino code that I wrote a while back, it's a rasberry pi server running some
python code that continuously polls data from the serial port (data being sent by Arduino), you can also make it completely wireless and send the data
through a Bluetooth connection, the principle remains the same. The basic structure is as follows:

1. Have an arduino connected to sensors collecting and streaming data to some port or over Bluetooth.
2. A receiver and parser on the server side to take that data and process it or in this case simply display it.
3. Afterwards you can create a socketed server with some pretty basic JavaScript and flask based python server to directly serve the collected data in realtime on a website.

### Handling things on Arduino Side
Well let's look at the Arduino side first, the following code will print the data on the serial monitor available in the arduino IDE.
```C++
#include <Arduino.h>
#include <Wire.h>
#include <BMP180I2C.h> //For setting up the BMP180 Sensor for pressure and temprature readings
#define I2C_ADDRESS 0x77
#include "SoftwareSerial.h"
#include <DHT.h>

#define DHT_PIN 2  // Pin to which the DHT sensor is connected
#define DHT_TYPE DHT11  // Change this to DHT11 if you are using DHT11 sensor
#define MQ_PIN A3

SoftwareSerial bt(10,11); //RX,TX


//DH11 Sensor - For temp and humidity
float temperature_dh11;
float humidity_dh11;

//MQ-135 Sensor - PPM Sensor
int airQuality_mq135;


//create an BMP180 object using the I2C interface
BMP180I2C bmp180(I2C_ADDRESS);
DHT dht(DHT_PIN, DHT_TYPE);


void setup() {
  Serial.begin(9600);
  bt.begin(9600);

  while (!Serial);
  Wire.begin();
  dht.begin();
  if (!bmp180.begin())
  {
    Serial.println("begin() failed. check your BMP180 Interface and I2C Address.");
    while (1);
  }
  bmp180.resetToDefaults();
  bmp180.setSamplingMode(BMP180MI::MODE_UHR);

  
}

void loop() {
  //Code to check if you are able to communicate with Arduino, you can send serial data from from bluetooth using terminal!
  if(bt.available()) {
    Serial.write(bt.read());    
  }

  //DH11 values
  temperature_dh11 = dht.readTemperature();
  humidity_dh11 = dht.readHumidity();
  
  //MQ-135 reading
  airQuality_mq135 = getAirQuality();


  Serial.print("Temperature (DHT): ");
  Serial.print(isnan(temperature_dh11) ? "Error" : String(temperature_dh11));
  Serial.println(" °C");
  if(Serial.available()) {
    bt.write(temperature_dh11);
  }
  
  Serial.print("Humidity (DHT): ");
  Serial.print(isnan(humidity_dh11) ? "Error" : String(humidity_dh11));
  Serial.println(" %");

  Serial.print("Air Quality (MQ-135 - PPM of contaminents): ");
  Serial.println(airQuality_mq135);

  //start a temperature measurement, the following code is particular for the BMP180 sensor
  if (!bmp180.measureTemperature())
  {
    Serial.println("could not start temperature measurement, is a measurement already running?");
    return;
  }

  //wait for the measurement to finish. proceed as soon as hasValue() returned true. 
  do
  {
    delay(100);
  } while (!bmp180.hasValue());

  Serial.print("Temperature: "); 
  Serial.print(bmp180.getTemperature()); 
  Serial.println(" degC");

  //start a pressure measurement. pressure measurements depend on temperature measurement, you should only start a pressure 
  //measurement immediately after a temperature measurement. 
  if (!bmp180.measurePressure())
  {
    Serial.println("could not start perssure measurement, is a measurement already running?");
    return;
  }

  do
  {
    delay(100);
  } while (!bmp180.hasValue());

  Serial.print("Pressure: "); 
  Serial.print(bmp180.getPressure());
  Serial.println(" Pa");
}


int getAirQuality() {
  int rawValue = analogRead(MQ_PIN);
  float voltage = (rawValue / 1024.0) * 5.0;
  float ratio = voltage / 5.0;
  float ppm = (ratio - 0.16) / 0.04 * 1000;
  return ppm;
}

```
The above code can be modified to include the following block, since we already have the `bt` object we can send (write) data to it.

```cpp
 // Send formatted data string over Bluetooth
  bt.print("DHT Temp: ");
  bt.print(temperature_dh11);
  bt.print(" C, DHT Humidity: ");
  bt.print(humidity_dh11);
  bt.print(" %, Air Quality: ");
  bt.print(airQuality_mq135);
  bt.print(" ppm, BMP Temp: ");
  bt.print(bmpTemperature);
  bt.print(" C, BMP Pressure: ");
  bt.print(bmpPressure);
  bt.println(" Pa");
```

This finally gives us an interface to observe the data wirelessly, you can read this [artice](https://www.emcraft.com/imxrt1050-evk-board/using-bluetooth-serial-port-profile) to learn how to use rfcomm to achieve similar results.
Remember to match the baud rate as baud rate basically represents the speed at which the devices are expected to communicate, a mismatches value will
result in missed data.

<video width="600" controls>
  <source src="../assets/Arduino.MOV" type="video/quicktime">
  Your browser does not support the video tag.
</video>


### The website and server side of things
This is a simple python flask server, it uses the Serial library to collect the data, but you could've used some system commands to pipe the output of the rfcomm to this code (Exercise left to the reader?).

```python
import json
from flask import Flask, render_template
from flask_socketio import SocketIO, emit
from time import sleep

import threading
import serial
from datetime import datetime
import time

app = Flask(__name__)
app.config['SECRET_KEY'] = 'asdfasdf'
socketio = SocketIO(app)


ser = serial.Serial('/dev/ttyACM0', 9600)

@app.route('/')
def index():
    return render_template('index.html')

@socketio.on('connect', namespace='/')
def connected():
    print("Client Connected..")


@socketio.on('disconnect', namespace='/')
def disconnected():
    print("Client Disconnected")


def read_from_port(ser):
    while True:
        reading = ser.readline().decode('utf-8', errors='replace').rstrip()
        try:
            if reading:
                data = json.loads(reading) # Try to parse JSON
                socketio.emit('metrics', data, namespace='/')  # Use socketio.emit here
                print(data)
        except json.JSONDecodeError:
            print("Error decoding JSON from Arduino")
        time.sleep(1.5)

thread = threading.Thread(target=read_from_port, args=(ser,))
thread.start()

if __name__ == '__main__':
    socketio.run(app,host = '0.0.0.0', port=5000, debug=True)
 

```

A simple socket script on the Javascript side that connects to this little server to pull the data.

```javascript
let lastTemp = 0;
let lastPpm = 0;
let lastHumidity = 0;
let lastPressure = 0;

function countUp(start, end, elementId, duration = 2000) {
            let current = start;
            const step = (end - start) / (duration / 10);
            const counterElement = document.getElementById(elementId);
            const interval = setInterval(() => {
                current += step;
                if (current >= end) {
                    current = end;
                    clearInterval(interval);
                }
                counterElement.textContent = String(Math.round(current)).padStart(3, '0');
            }, 10);
        }


const socket = io.connect('http://' + location.hostname + ':' + location.port, {path: '/socket.io', namespace: '/'});

socket.on('metrics', (data) => {
  countUp(lastTemp, data.temp, 'temp', 1000);
  lastTemp = data.temp; // Update the last value to the new one
  
  countUp(lastPpm, data.ppm, 'ppm', 1000);
  lastPpm = data.ppm; // Update the last value to the new one
  
  countUp(lastHumidity, data.humidity, 'humidity', 1000);
  lastHumidity = data.humidity; // Update the last value to the new one
  
  countUp(lastPressure, data.pressure, 'pressure', 1000);
  lastPressure = data.pressure; // Update the last value to the new one
});

```
And a little pizzaz added by a some CSS.
```html
<!DOCTYPE html>
<html>

  <head>
    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='style.css') }}">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.0.1/socket.io.js"></script>
    <script  type="module" src="{{ url_for('static', filename='sockets.js') }}"></script>

  </head>
  <body>
    <div class="heading">
     Arduino Streamer 
    </div>

<div>
      <div class = "container">
<div class="rounded-box current" id="currentLabel">
   Air Quality Stats 
</div></div>

    <div class="container">
      <div class="odometer-row">
        <div id="tempOdo" class="odometer-container current">
          <div class="label">Temperature(Celsius)</div>
          <div id="temprature"><span> </span><span id="temp"> 0000</span></div>
        </div>


        <div id="ppmOdo" class="odometer-container current">
          <div class="label">Contaminents (PPM)</div>
          <div id="ppm"> 0000</div>
        </div>

        <div id="humidityOdo" class="odometer-container current">
          <div class="label">Humidity(%)</div>
          <div id="humidity"> 0000</div>
        </div>

        <div id="pressureOdo" class="odometer-container current">
          <div class="label">Pressure(Pa)</div>
          <div id="pressure"> 0000</div>
        </div>
      </div>
    </div>

  </body>
</html>
```

```css
@import url('https://fonts.googleapis.com/css2?family=Orbitron&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@500&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@500&display=swap');


.heading {
  text-align: center;
  font-family: 'Montserrat', sans-serif;
  font-size: 3em;
  margin-bottom: 40px;
}


.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 1;
}


.odometer-row {
  padding-top:180px;
  display: flex;
  flex-direction: row;
  justify-content: center;
  z-index: 1;
}

.odometer-container {
      font-size: 2em;
      width: 300px;
      height: 100px;
      text-align: center;
      color: #fff;
      margin-top: 5px;  
      margin-left: 10px;  
      margin-right: 10px;  
      margin-bottom: 5px;  
      padding: 15px;
      border-radius: 5px;
      font-family: 'Montserrat', sans-serif;
    }
    .odometer-container.current{
      background-color: #3341ff;
    }
    .label {
      font-size: 0.8em;
      margin-bottom: 5px;
    }
}
```
And that's it.
