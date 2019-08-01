## Quickstart
On a fresh installation of Raspbian Stretch Lite on a Raspberry Pi, run the following commands to install everything you need to run the Nido thermostat:

```bash
cd ~
mkdir nido
cd nido
curl -fsSL get-nido.moveolabs.com | source /dev/stdin
```

For the extra cautious, you can verify the checksum of the install script, below, and the download domain [ownership](https://keybase.io/alexmensch).

```bash
shasum -a 256 get-nido.sh 
ab43579d6b546e789292b54f802d118af3e2a55e00ba86308e7729a6387bf773  get-nido.sh
```

Once Nido is up and running, you can add it as a new accessory in the Apple Home app using the default code `94812494` or by scanning the QR code in the Homebridge startup logs. Make sure your Apple device is on the same network as the Raspberry Pi.

Of course, you'll need some hardware to make the software useful, so keep reading...

## Say "NEE-doh", an IoT thermostat for ancient technology
### A bridge between the mechanical era and the Internet

I live in an apartment with a [gas wall furnace](https://www.williamscomfortprod.com/product/monterey-plus-home-furnaces/) that's operated by [184-year-old technology](https://www.jondetech.se/technology/thermopile-history/). One of the highlighted features of these furnaces is that they **don't require wall power**, which is great from the standpoint of keeping you warm during a power outage, but if you want to control your furnace with a smart thermostat, you're out of luck. (Though that hasn't stopped [some people](https://medium.com/@chrisvale/controlling-an-ancient-millivolt-heater-with-a-nest-b9493bbc59da) from [trying](https://scottshapiro.com/hacking-nest-uk-san-francisco-heater/).) Commercial smart thermostats generally require power drawn directly from the heating/cooling system to power themselves and also (usually) lack the necessary circuitry to control a heater of this type.

The ancient wall heater in question is very common in California (the mecca of cutting edge technology!), and although it frustrates assimilation by the modern IoT movement, it's very elegant from an engineering standpoint. Here's how it works:

1. You light the pilot light in the heater.
2. Right in the middle of the pilot flame is a [thermopile](https://en.wikipedia.org/wiki/Thermopile). This thermopile coverts the heat of the pilot flame into a small voltage of just 0.4V (or 400mV, which it what gives it the name "millivolt heater".)
3. As long as the pilot flame is lit, the voltage generated by the thermopile is sufficient to keep the pilot gas valve open. This positive feedback loop acts as a safety measure: if the pilot light goes out, the gas supply is switched off automatically.
4. This tiny voltage is also used to control a second, larger gas valve that turns the heat on and off. Two wires are routed out of the heater that connect to a mechanical thermostat. When the circuit is closed (ie. the two wires are connected to each other), the gas valve opens and the heat turns on. Without this voltage (if the pilot light is out, say), the gas valve cannot open.
5. The mechanical thermostat is operated by a combination of a coiled [bimetallic strip](https://en.wikipedia.org/wiki/Bimetallic_strip), a magnet, a [reed switch](https://en.wikipedia.org/wiki/Reed_switch), and a physical dial to set the target temperature.
6. When you set the target temperature, you're rotating the bimetallic strip such that the magnet attached to the end of it closes the reed switch when the target temperature is reached.
7. When the reed switch is closed, the circuit is closed, which turns on the gas valve, which turns on the heat.
8. As the air heats up, the bimetallic strip moves the magnet sufficiently such that the reed switch opens again and the heat turns off.

It's clever and it works (including some natural advantages like some built-in mechanical [hysteresis](https://en.wikipedia.org/wiki/Hysteresis)), so why ruin a great thing?

### Nido is the solution to the millivolt heater problems you didn't know you had
- **The mechanical thermostat, though clever, is *really* inaccurate.** Sometimes it can be off by as much as 5F (~3C). Especially at night, this is the difference between waking up sweaty or shivering in the morning.
- **Convenient control from anywhere.** With a few taps on my phone physical goods just show up at my doorstep the next day, so why can't I set the heat remotely so that my apartment is a comfortable temperature when I get home? This is what the Internet has been promising us for [the last 20 years](https://www.businessinsider.com/the-complete-history-of-internet-fridges-and-connected-refrigerators-2016-1).
- **Complete customization.** You can customize the heater's schedule in as much detail as you'd like. Only want it to turn on when someone's home? Done. Want to customize settings based on *who's* home? Done. Override it manually any time? Sure.
- **Integration with other workflows.** One of the benefits of having an open API is that you easily integrate Nido with any other service, like [IFTTT](https://ifttt.com), for totally customized workflows that suit you and any other home automation that you've configured.
- **Data-driven decisions are good decisions.** Having an accurate temperature sensor not only means that the thermostat action is precise, it also gives you feedback on what **"comfortable" means to *you***. For example, I now know that setting the thermostat to 19C overnight means that I sleep comfortably. 21C seems to do the trick when I'm up and about during the day.

So, what are you waiting for? Bring that gas heater and the comfort of your living space into the 21st century.

## Getting started
### Shopping list
1. **A [Raspberry Pi Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/), or any other Raspberry Pi.** It needs to be powered, connected to your home network, and you'll need access to the GPIO pins.
2. **Bosch BME280 sensor module [from Adafruit](https://www.adafruit.com/product/2652).** The current version of the software depends on communicating with this specific module over [I²C](https://en.wikipedia.org/wiki/I²C).
3. **Custom electronics.** You'll need to get your soldering iron out for this one. See details on this, below. (I'm working on a custom PCB so that this and the previous step can be skipped. Please see the [collaborators](#collaborators-wanted) section if you're interested in helping out.)
4. **A screwdriver.** You're going to need to detach the control wires from the mechanical thermostat that's currently on your wall.

**Total cost:** Approximately $30, not including tools

### Custom electronics
I've implemented a very simple circuit to control the heater. This circuit effectively replaces the magnet and reed switch and allows a GPIO control pin on the Raspberry Pi to control the valve that turns on the heating.

__Circuit diagram:__

![Circuit diagram](https://raw.githubusercontent.com/alexmensch/nido/master/doc/circuit.png)

__How it works:__
1. To turn on the heat, the control software drives the GPIO control pin high.
2. This high voltage state turns on an NPN transistor, which drives enough current to close the contacts on a small relay.
3. The relay contacts are connected to the heater control wires (formerly connected to the mechanical thermostat), which replaces the reed switch in the mechanical thermostat.
4. When the voltage falls low on the GPIO pin, the transistor is turned off, the relay contacts open, and the heat turns off again.

**Note:** It's important to use a "normally open" relay so that the whole system fails safe. In other words, the heat should be off by default if your Raspberry Pi loses power, not the other way around!

__Parts list:__

- Relay: 1 x SPST (NO), Hamlin Electronics P/N HE3621A0510 ([Jameco](https://www.jameco.com/z/HE3621A0510-Hamlin-Electronics-Electromechanical-SIL-Relay-SPST-NO-500mA-5-Volt-500-Ohm-Through-Hole_1860088.html))
- Transistor: 1 x BC337 TO-92 NPN ([Jameco](https://www.jameco.com/z/BC337-Major-Brands-Transistor-BC337-TO-92-NPN-800ma-45-Volt_254810.html))
- Resistor: 1 x 1kΩ

**Total cost:** Less than $2

### Hardware interface with the Raspberry Pi

__Relay circuit:__

There are just three wires that need to be connected to the Raspberry Pi GPIO pins:

GPIO Pin|Circuit interface                               |Notes
:-------|:------------------------------------------------|:-----------------------------
GPIO 26 |HIGH/LOW input to the base of the NPN transistor.|This is the only true GPIO pin used in the relay circuit. This pin number is pre-defined in the software.
5V      |5V rail for the relay driver.                    |There are two 5V pins in the GPIO header.
GND     |Ground rail for the relay driver.                |There are eight GND pins available in the GPIO header.

__BME280 Adafruit module:__

Only two data wires needs to be connected to the GPIO header for the I²C data bus other than power:

GPIO Pin |BME280 Pin |Notes
:--------|:----------|:--------------------
SCL      |SCK        |Timing (clock) signal
SDA      |SDI        |Data signal
5V       |VIN        |Alternatively, the corresponding 3Vo/3V3 pins can also be used. I shared the same 5V source for both circuits to avoid an extra wire.
GND      |GND        |There are eight GND pins to choose from on the GPIO header, and this can be shared with the relay circuit to minimize wires.

## Hardware design
### A $5 computer and some electronics
At its core, a thermostat is a very simple device. Much simpler hardware solutions than Nido are possible, but around the time that I started working on this project in January 2016, the [Raspberry Pi Zero](https://www.raspberrypi.org/products/raspberry-pi-zero/) had just been released, and I was really intrigued by the idea of building around a $5 Linux computing device. Running a full operating system and being able to write software in widely-used languages like Python and JavaScript meant that it would be much easier to integrate Nido with other systems and make extensions to the product in the future. Despite the importance of the hardware interface with the heater, the heart of this project relies on software, and the Raspberry Pi has made it easy to expand the product's functionality over time.

Aside from the choice of the Raspberry Pi as the underlying platform, there were just a few main hardware decisions:

- The **BME280 temperature / humidity / pressure sensor** chip. I'd been eyeing this chip for a different project that also required a pressure sensor, and this one is packaged in a compact and easy-to-use format by the folks at [Adafruit](https://www.adafruit.com/about). An alternative Bosch chip, the BMP280, which lacks the pressure sensor, is about half the cost and would be my choice for the next hardware revision of the product.
- The **I²C data bus** protocol. This was primarily a decision around limiting the number of wires compared to SPI. I²C only requires two wires for data transmission, and its low data rate was not an issue for this project.
- The small **control relay**. This was mostly a decision of expediency of choice as I was up against a deadline at the time, but I'd love to find something smaller for the next hardware revision. The requirements on the contact side are minimal: low voltage and virtually no current. There's probably a solid state relay that's a great choice here, but it's hard to beat the price on the existing hardware!

### Wait...no physical interface?
Counter to every other thermostat you've interacted with, Nido has no screen and no buttons. Initially, this made the hardware design faster to implement, but I ultimately came to the conclusion that a physical interface is unnecessary. The most pleasing human interaction with technology is when it works invisibly for us in the background, only requiring our attention during exceptional moments when our input is really needed.

Nido is typically set up to have the following behavior:

- It turns on automatically when the first person arrives home.
- It turns off automatically when the last person leaves home.
- You define a comfortable indoor temperature that you choose for your home during the day.
- You define a comfortable overnight temperature.

For the times when you feel like you do need to intervene in its normal operation:

- You can see its status and adjust its settings from your iOS device from anywhere in the world.
- It responds to voice commands via Siri, too.

For the advanced home automation enthusiast:

- It has an open API that lets you fine-tune its control beyond the automation built into HomeKit on iOS.

With all of those possibilities, why have yet another device in your home to distract and fight for your attention? Don't you carry around an advanced display and controller in your pocket already, anyway?

## Software design
### Python, NodeJS and MQTT

This is a high level overview as each of the repositories below go into more detail on their own operation. At the conclusion of the "get-nido" install script, the Raspberry Pi host will be running four services that work together to provide a full range of functionality. These services are:

1. **"nido-supervisor"**. This is the heart of Nido. This container runs all of the backend logic necessary for the thermostat to interact with the world, including the hardware interface, the thermostat control logic, a simple database, datalogging capabilities, and an RPC service. These capabilties are packaged together into the [nido](https://pypi.org/project/nido/) Python package.
2. **"nido-api"**. This container uses a Gunicorn WSGI server to host a REST API for Nido. This frontend interfaces with the backend via RPC. This service is also part of the [nido](https://pypi.org/project/nido/) Python package.
3. **"homebridge"**. I'm running Nick Farina's [Homebridge](https://github.com/nfarina/homebridge) project to provide a bridge interface to the HomeKit Accessory Protocol and advertise Nido on the user's local network. I wrote a NodeJS plugin, [homebridge-nido](https://www.npmjs.com/package/homebridge-nido), that connects Homebridge to the Nido native API.
4. **"mosquitto"**. This is an off-the-shelf version of the [Mosquitto](https://mosquitto.org) MQTT broker, By default, the Nido backend is configured to publish state and environmental sensor information to this broker every 60 seconds. Any other client on the network can subscribe to these updates from the broker. (More on this, [below](#data-logging)).

__Code repositories:__

[alexmensch/nido-python](https://github.com/alexmensch/nido-python)

- The build output of this repository is the [nido](https://pypi.org/project/nido/) Python package. It has a built in simulator for the expected input and output hardware so that development can take place on a non-Raspberry Pi machine.

[alexmensch/nido-docker](https://github.com/alexmensch/nido-docker)

- This respository is used for automated builds to publish the [alexmensch/nido](https://hub.docker.com/r/alexmensch/nido) Docker image. This ARM32v6-compatible image is based on the minimalist [Alpine Linux](https://alpinelinux.org) image. The timezone on the image is set to UTC, and the [nido](https://pypi.org/project/nido/) Python package is installed, including compliation of the GPIO Python interface library. No other steps are performed, making this a lightweight image.

[alexmensch/nido-homebridge](https://github.com/alexmensch/nido-homebridge)

- This NodeJS plugin is published as [homebridge-nido](https://www.npmjs.com/package/homebridge-nido) on npm. It is a very simple translation interface between Homebridge (itself an interface to the HomeKit Accessory Protocol) and the native Nido API.

[alexmensch/nido](https://github.com/alexmensch/nido)

- This repository. Aside from this documentation, it holds everything needed to run the finished product: default service configuration files, the Docker Compose service definitions and the installation script.

### Thermostat control loop
The control loop for the thermostat is very simple. In the absence of any changes being made to the thermostat settings, every 5 minutes the supervisor checks the current mode setting, the temperature set point, what the current state of the heater is, and the current temperature. If changes are made to the thermostat settings, then this process is triggered immediately. The following decisions are then made:

1. If any exceptions are raised in the software, shut off the heater by default.
2. If the mode is set to "Off", shut off the heater.
3. If the mode is set to "Heat", and the heater is currently heating, only shut off the heater if the current temperature is above the set temperature.
4. If the mode is set to "Heat", and the heater is currently off, only turn on the heater if the current temperature is less than 0.6C cooler than the set temperature.
4. Else, shut off the heater.

**Why 0.6C?**

This corresponds approximately to a difference of 1F, but the reason to include this margin is to introduce [hysteresis](https://en.wikipedia.org/wiki/Hysteresis) into the control loop. This means that we want the system to have some lag in its response, which is also the reason that we only check to see if we need to turn the heater on or off every 5 minutes. Without introducing this lag, the precision of the sensor would generate the undesireable effect of the heater turning off and on in quick succession as the room temperature moved just a fraction above and below the set temperature.

### Data logging
The BME280 chip is an amazing piece of technology, and it's capable of very high precision. It's an under-appreciated piece of technology in this project, but you can make better use of it by capturing the data it generates. By default, when you run Nido, a local MQTT broker ([Mosquitto](https://mosquitto.org)) is also started. You can subscribe to this broker and receive the data that's logged from Nido every 60 seconds.

MQTT Topic            | Description
:---------------------|:-----------
nido/set_temp         |The current set temperature of the thermostat.
nido/controller       |The current state of the heater (0 = Off, 1 = Heating).
nido/temp_c           |The current temperature in Celsius.
nido/pressure_mb      |The current pressure in millibars.
nido/relative_humidity|The current relative humidity.

Messages are formatted in InfluxDB [line protocol format](https://docs.influxdata.com/influxdb/v1.7/write_protocols/line_protocol_tutorial/). For example:
```
thermostat set_temp=19.4 1564466359638010112
thermostat controller=0 1564466359638010112
thermostat temp_c=22.4 1564466359638010112
thermostat pressure_mb=1010.1 1564466359638010112
thermostat relative_humidity=61.4 1564466359638010112
``` 

Point an MQTT subscriber to port 1883 on the Raspberry Pi host to receive these messages. You can also configure Nido to broadcast to any MQTT broker (hint: pick your favorite cloud service).

## Collaborators wanted!
It's been really fun getting the project this far, but there's only so much time in the day, and my skills are rusty in some areas and only extend so far in others! I'd love to get some help in the following domains. You can contact me at amars\[at\]alumni\[dot\]stanford\[dot\]edu.

### Mechanical and CAD
- I'd love to create a CAD design for a 3D-printable case. The current prototype uses a standard Raspberry Pi Zero case and some pretty rudimentary mounting. I think it would be pretty straightforward to create a case in nearly the exact same form factor as the original mechanical thermostat. This would make it a truly great direct in-place replacement for the old thermostat!

### PCB and electronics
- There's a great opportunity to create an all-in-one PCB that incorporates not only the relay driver circuit, but also the BME280/BMP280 sensor. With some surface mount components, the sensor and driver circuitry can be given a much smaller footprint, too.
- Developing the PCB in a form factor where it can be plugged directly into the GPIO header without any wires required would be even better. There's a good opportunity to interface with the case design, too.

### Software
- My day job is not as a professional software developer, so I'm sure there are at least a few things that could be improved. I have lots of ideas for extensions and additions to the project to make it even better.
- Data visualization is an interesting opportunity that I haven't yet explored in detail.
- Empirical measurement and feedback of overall energy efficiency would be a really interesting extension to this project.
