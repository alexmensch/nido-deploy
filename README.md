## Quickstart
On a fresh installation of Raspbian Stretch Lite, run the following commands to install everything you need to run the Nido thermostat:

```bash
cd ~
mkdir nido
cd nido
curl -fsSL get-nido.moveolabs.com | source /dev/stdin
```

Once Nido is up and running, you can add it as a new accessory in the Apple Home app using the default code `94812494`, or by scanning the QR code in the Homebridge startup logs. Make sure your Apple device is on the same network as the Raspberry Pi.

Of course, you'll need some hardware to make the software useful, so keep reading...

## Nido, an IoT thermostat for ancient technology
### Bridging the mechanical era and the Internet

I live in an apartment with a [gas wall furnace](https://www.williamscomfortprod.com/product/monterey-plus-home-furnaces/) that's operated by [184-year-old technology](https://www.jondetech.se/technology/thermopile-history/). One of the highlighted features of these furnaces is that they **don't require wall power**, which is great from the standpoint of keeping you warm during a power outage, but if you want to control your furnace with a smart thermostat, you're out of luck. (Though that hasn't stopped [some people](https://medium.com/@chrisvale/controlling-an-ancient-millivolt-heater-with-a-nest-b9493bbc59da) from [https://scottshapiro.com/hacking-nest-uk-san-francisco-heater/](trying).) Commercial smart thermostats generally require power drawn directly from the heating/cooling system to power themselves and also (usually) lack the necessary circuitry.

The ancient wall heater in question is very common in California (the mecca of cutting edge technology!), and although it frustrates assimilation by the modern IoT movement, it's very elegant from an engineering standpoint. Here's how it works:

1. You light the pilot light in the heater.
2. Right in the middle of the pilot flame is a [thermopile](https://en.wikipedia.org/wiki/Thermopile). This thermopile coverts the heat of the pilot flame into a small voltage of just 0.4V (or 400mV, which it what gives it the name "millivolt heater".)
3. As long as the pilot flame is lit, the voltage generated by the thermopile is sufficient to keep the pilot gas valve open. This positive feedback loop acts as a safety measure: if the pilot light goes out, the gas supply is switched off automatically.
4. This tiny voltage is also used to control a second, larger gas valve that turns the heat on and off. Two wires are routed out of the heater that connect to a mechanical thermostat. When the circuit is closed (ie. the two wires are connected to each other), the gas valve opens and the heat turns on. Without this voltage (if the pilot light is out, say), the gas valve cannot open.
5. The mechanical thermostat is operated by a combination of a coiled [bimetallic strip](https://en.wikipedia.org/wiki/Bimetallic_strip), a magnet, a [reed switch](https://en.wikipedia.org/wiki/Reed_switch), and a physical dial to set the target temperature.
6. When you set the target temperature, you're rotating the bimetallic strip such that the magnet attached to the end of it closes the reed switch when the target temperature is reached.
7. When the reed switch is closed, the circuit is closed, which turns on the gas valve, which turns on the heat.

It's clever and it works (including some natural advantages like some built-in mechanical [hysteresis](https://en.wikipedia.org/wiki/Hysteresis)!), so why ruin a great thing?

### Nido is the solution to the millivolt heater problems you didn't know you had
- **The mechanical thermostat, though clever, is *really* inaccurate.** Sometimes it can be off by as much as 5F (~3C). Especially at night, this is the difference between waking up sweaty or shivering in the morning.
- **Convenient control from anywhere.** With a few taps on my phone physical goods just show up at my doorstep the next day, so why can't I set the heat remotely so that my apartment is a comfortable temperature when I get home? This is what the Internet has been promising us for [the last 20 years](https://www.businessinsider.com/the-complete-history-of-internet-fridges-and-connected-refrigerators-2016-1).
- **Complete customization.** You can customize the heater's schedule in as much detail as you'd like. Only want it to turn on when someone's home? Done. Want to customize settings based on *who's* home? Done. Override it manually any time? Sure.
- **Integration with other workflows.** One of the benefits of having an open API is that you easily integrate Nido with any other service, like [IFTTT](https://ifttt.com), for totally customized workflows that suit you and any other home automation that you've configured.
- **Data-driven decisions are good decisions.** Having an accurate temperature sensor not only means that the thermostat action is accurate, it also gives you feedback on what "comfortable" means to you. For example, I now know that setting the thermostat to 19C overnight means that I sleep comfortably. 21C seems to do the trick when I'm up and about during the day. (Oh, and Nido also collects temperature, humidity and pressure information every minute, giving you unprecedented visibility into the micro climate of your home if you're so inclined to capture that data. I learned that I can detect when someone comes home because the temperature in the apartment increases by a measureable amount.)

So, what are you waiting for? Bring that gas heater and the comfort of your living space into the 21st century.

## Hardware design
### A $5 computer and a some electronics


### Wait...no physical interface?


## Software design
### Python, NodeJS and MQTT


### Thermostat control loop


### Data logging


## Collaborators wanted!
### Mechanics and CAD


### PCB and electronics


### Software

