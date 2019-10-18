# OpenChirp Code

```python
import sys
import busio
import digitalio
import board
import adafruit_mcp3xxx.mcp3008 as MCP
from adafruit_mcp3xxx.analog_in import AnalogIn
import time
import RPi.GPIO as GPIO
import threading
import pigpio
import dht11
import paho.mqtt.client as mqtt


# MQTT client
class Device(mqtt.Client):
    def __init__(self, username, password):
        super(Device, self).__init__()
        self.host = "mqtt.openchirp.io"
        self.port = 8883
        self.keepalive = 300
        self.username = username
        self.password = password

        # Set access credential
        self.username_pw_set(username, password)  # set username and pass
        self.tls_set('cacert.pem')

        # Create a dictionary to save all transducer states
        self.device_state = dict()

        # Connect to the Broker, i.e. OpenChirp
        self.connect(self.host, self.port, self.keepalive)
        self.loop_start()

    def on_connect(self, client, userdata, flags, rc):
        if rc == 0:
            print("Connection Successful")
        else:
            print("Connection Unsucessful, rc code = {}".format(rc))
        # Subscribing in on_connect() means that if we lose the connection and reconnect, the subscriptions will be renewed.
        # Subscribe to all transducers
        self.subscribe("openchirp/device/" + self.username + "/#")

        # The callback for when a PUBLISH message is received from the server.

    def on_message(self, client, userdata, msg):
        print(msg.topic + " " + str(msg.payload.decode()))

        # Change Actuator State based on Commands Issued from OpenChirp
        transducer = msg.topic.split("/")[-1]
        self.device_state[transducer] = msg.payload.decode()

    def on_publish(self, client, userdata, result):
        print("Data published")


# Modify here based on your own device
username = '5da4cf50466cc60c381e0d13'  # Use Device ID as Username
password = 'e8su1IUzbTPQ73ehJqdOgRrCkNOsTHmX'  # Use Token as Password

# Instantiate a client for your device
smart_gh = Device(username, password)



spi = busio.SPI(clock=board.SCK, MISO=board.MISO, MOSI=board.MOSI)
cs = digitalio.DigitalInOut(board.D5)

# Create an MCP3008 object
mcp = MCP.MCP3008(spi, cs)
# Create an analog input channel on the MCP3008 pin 0
channel = AnalogIn(mcp, MCP.P0)

threshold = 1 # Threshold for turn on/off LED
#set pwm
pi = pigpio.pi()
pi.set_PWM_frequency(21, 100)
pi.set_PWM_range(21, 100)
pi.set_PWM_dutycycle(21,100)

pi.set_PWM_frequency(20, 100)
pi.set_PWM_range(20, 100)
pi.set_PWM_dutycycle(20,100)

#set parameter
target = 0.6
light_ini = 0
light_read = 2
light_difF = 0
light_error = 0

P = 5
I = 1 
D = 5
pwm = 100

#set temp
temp_sensor = 12
instance = dht11.DHT11(pin = temp_sensor)

#set LED
red = 23
blue = 18

GPIO.setmode(GPIO.BCM)
GPIO.setup(red, GPIO.OUT)
GPIO.setup(blue, GPIO.OUT)
sensor = "light_intensity"
actuator = "led1"
led_pwm = "duty_cycle"
sensor_tem = "temperature"

# Initialize
smart_gh.device_state[sensor] = 0
smart_gh.device_state[actuator] = 0
smart_gh.device_state[led_pwm] = 0
smart_gh.device_state[sensor_tem] = 0

def fun_timer():
    global timer
    global light_ini, light_read, light_diff, light_error, target, pwm, P, I, D
    pre_error = light_error
    light_read = channel.voltage
    print(light_read)
    light_error = target - light_read
    light_ini += light_error
    light_diff = light_error - pre_error

    control = P * light_error + I * light_ini + D * light_diff
    pwm += control
    pwm = min(100, pwm)
    pwm = max(0, pwm)

    if pwm == 100 or pwm == 0:
        light_ini = 0
    
    #pwm = 0
    pi.set_PWM_dutycycle(21, pwm)
    pi.set_PWM_dutycycle(20, pwm)

    print(round(light_error,2), round(light_ini,2), round(light_diff,2), control, 100 - pwm)

    #temp part
    result = instance.read()
    #print(result.temperature)
    if result.temperature == 0:
        #GPIO.output(blue, GPIO.HIGH)
        #GPIO.output(red, GPIO.HIGH)
        pass
    else:
        if result.temperature >= 28:
            GPIO.output(blue, GPIO.HIGH)
            GPIO.output(red, GPIO.LOW)
        else:
            GPIO.output(blue, GPIO.LOW)
            GPIO.output(red, GPIO.HIGH)
    open_chirp()
    timer = threading.Timer(0.01, fun_timer)
    timer.start()


def open_chirp():
        # Read from sensor and Publish onto OpenChirp
        sensor_reading = channel.voltage
        smart_gh.publish("openchirp/device/" + username + "/" + sensor, payload=sensor_reading, qos=0, retain=True)
        pwm_value = pwm
        smart_gh.publish("openchirp/device/" + username + "/" + led_pwm, payload=pwm_value, qos=0, retain=True)
        tem_reading = instance.read().temperature
        smart_gh.publish("openchirp/device/" + username + "/" + sensor_tem, payload=tem_reading, qos=0, retain=True)
        # Update device state
        smart_gh.device_state[sensor] = sensor_reading
        smart_gh.device_state[led_pwm] = pwm_value
        smart_gh.device_state[sensor_tem] = tem_reading


def main():
    fun_timer()
    while True:
        pass

if __name__ == '__main__':
    try:
        main()
    except KeyboardInterrupt:
        GPIO.cleanup()
        sys.exit(0)

```
