# esp32spi_mqtt

This mqtt client is an adaptation of Paul Sokolovsky's umqtt library for micropython available at https://github.com/micropython/micropython-lib to make it compatible with Adafruit's CircuitPython adafruit_esp32spi library https://github.com/adafruit/Adafruit_CircuitPython_ESP32SPI.

## Notes:
This has only been modified and tested to work with a mosquitto MQTT server with no username or password, and only with a non TLS connection.  Bug reports and pull requests are welcome!

This works well with the pyportal running CircuitPython 4.0.0-rc.2 UF2 firmware with a version of the CircuitPython adafruit_esp32spi library that includes the following socket timeout fix: https://github.com/adafruit/Adafruit_CircuitPython_ESP32SPI/commit/fce3410075f08d33d98b59ac81f876041ea72108

## Installation
Copy the esp32spi_mqtt.py file to your somewhere on your circuitpython's path so it can be found with an import statement.

## Usage
```
#sample usage on the pyportal

import board
import busio
from digitalio import DigitalInOut

from adafruit_esp32spi import adafruit_esp32spi
import adafruit_esp32spi.adafruit_esp32spi_socket as socket

from circuit_python_mqtt import MQTTClientSimple
from secrets import secrets

esp32_cs = DigitalInOut(board.ESP_CS)
esp32_ready = DigitalInOut(board.ESP_BUSY)
esp32_reset = DigitalInOut(board.ESP_RESET)

spi = busio.SPI(board.SCK, board.MOSI, board.MISO)
esp = adafruit_esp32spi.ESP_SPIcontrol(spi, esp32_cs, esp32_ready, esp32_reset)

socket.set_interface(esp)

client = MQTTClientSimple('testclient', '192.168.50.10', port=1883)
def recv_message(topic, msg):
    print('=============')
    print('mqtt callback')
    print('-------------')
    print(topic)
    print(msg)
    print('=============')

client.set_callback(recv_message)

while not esp.is_connected:
    try:
        esp.connect_AP(secrets['ssid'], secrets['password'])
    except RuntimeError as e:
        print("could not connect to AP, retrying: ",e)
        continue

client.connect()
client.subscribe("test")
client.publish("test", "Hello World!")
client.wait_msg()
client.disconnect()
```