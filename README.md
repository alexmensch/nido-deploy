## Quickstart
On a fresh installation of Raspbian Stretch Lite, run the following commands to install everything you need to run the Nido thermostat:

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

## Nido, an IoT thermostat for ancient technology
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

For the really advanced among us:

- It has an open API that lets you fine-tune its control beyond the automation built into HomeKit on iOS.

With all of those possibilities, why have yet another device in your home to distract and fight for your attention? Don't you carry about an advanced display and controller in your pocket already, anyway?

## Software design
### Python, NodeJS and MQTT



__Code repositories:__

[alexmensch/nido-python](https://github.com/alexmensch/nido-python)


[alexmensch/nido-docker](https://github.com/alexmensch/nido-docker)


[alexmensch/nido-homebridge](https://github.com/alexmensch/nido-homebridge)


[alexmensch/nido](https://github.com/alexmensch/nido)


### Thermostat control loop


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
### Mechanical and CAD


### PCB and electronics


### Software

