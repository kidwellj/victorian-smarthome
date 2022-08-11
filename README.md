[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/X8X0EBKKN)

# Victorian smart home

![photo of brick Victorian home](/images/the_fron.jpg)


We're renovating an 1830s end-terraced home to bring it into the 21st century. This means in many cases that we're removing inappropriate tech which has been introduced more recently (like plasterboard / sheetrock!), and in other cases lightly augmenting our house to make sure we can lower our energy footprint, anticipte problems, and make living just a little bit more fun and comfortable. I've developed this repository in part to document my work as it goes along, but also on the possibilities that others in similar situations might benefit from some of the knowledge I've gained along the way. There are a number of issues pertaining to homes built before 1900 which can benefit from smart devices which really don't pertain to newer builds. I'll get into this a bit further below.

# Prerequisites:

These are the key pieces of infrastructure that needed to be in place before we could get moving. Quick note - everything I use is either (a) free + open source software / hardware or (b) inexpensive and hackable.

## (1) Home server

This is a relatively old hardware build, using a low-voltage intel CPU. I generally run about 80% CPU and 90% of the RAM on this server, so these specifications work for me. Some day in the future, I'll switch to a newer low-voltage board and CPU with 32GB RAM and that should more or less sort me for the next decade. This server has been running more or less continuously without any hardware upgrades since 2015. 

### Server hardware specifications:

- Motherboard: GA-Z97N-WIFI
- CPU: Intel Core i5-4690K 3.5 GHz Quad-Core Processor
- Memory: 16GB DDR3 (2x8gb)
- Case: Cooler Master Elite 120 Advanced Mini ITX Tower Case
- PSU: Corsair CSM 450 W 80+ Gold Certified Semi-modular ATX Power Supply
- Storage: 2.5" [1TB SSD](https://amzn.to/3zPVpHP) sitting in a [3.5" adapter](https://amzn.to/3JNwWHt) and 12TB ([4tb Seagate Ironwolf Drives](https://amzn.to/3AcucAi) x 3)
- USB Zigbee: [Phoscon ConBee II](https://amzn.to/3bHqlli)
- USB RTL-SDL: [Nooelec NESDR SMArt v4 SDR](https://amzn.to/3bFIgZP) (RTL2832U & R820T2-Based Software Defined Radio)
- USB Tado TRV controller

(fyi: amazon affiliate links above)

### Software:

If you're just getting started, I'd highly recommend the guides by [Alex Kretzschmar](https://github.com/ironicbadger) who I've followed almost entirely. The only thing I've been ahead of Alex on was using traefik. You can find his first guide, which got me started here: [Perfect Media Server 2017](
https://www.linuxserver.io/blog/2017-06-24-the-perfect-media-server-2017), though I'd note that he doesn't switch to proxmox (which I've done last year) until the [guide from 2019](https://perfectmediaserver.com/).

- I'm running a proxmox hypervisor, with several virtual machines underneath, mostly running debian / docker. Most of my docker containers are running off a Debian install, which includes traefik. I run all our services behind a wildcard domain hosted on cloudflare. Traefik re-directs some traffic to other containers, and for the sake of giving some services unique IP addresses I'm running the following additional servers as separate VMs:

(2) Home Assistant OS - this is the brain of our whole smart home system. I can't provide extensive detail here, but if you want to run a system like I've got here, you could strip things down considerably to just run HAOS on a smaller RaspberryPi based computer, for around £100-200

(3) Unifi Controller - if you're already using Proxmox, [the helper scripts by @tteck are pretty great](https://github.com/tteck/Proxmox).

(4) Logitech Media Server - I'm running Logitech Media Server, and here am grateful to @StevenSeifried for the [helper script](https://github.com/StevenSeifried/proxmox-scripts).


## (2) Network infrastructure:

- The house is wired with Cat6 ethernet wire running in 20mm round PVC conduits
- [TekLager open-source router](https://teklager.se/en/products/routers/) running [PFSense](https://www.pfsense.org/) - I highly recommend TekLager and PFSense as enterprise class, hobbyist friendly firewalls
- Network Downstairs: HP J9625A PoE+ (2620-24) Managed Switch - you can generally find used PoE gigabit switches by HP or Cisco on ebay for around £100
- Network Upstairs: Ubiquiti Unifi US-8-150W Switch
- Wifi: UAP-nanoHD hotspots (4)
- The Zigbee USB sensor above creates a whole-home zigbee network, which is a low voltage wireless mesh network. This is terrific for sensors which can run off a small disc battery for more than a year in more cases!
- The RTL-SDR sensor is basically a radio antenna in the server. I run [RTL433](https://github.com/merbanan/rtl_433) on the Debian server which basically detects and parses any incoming radio signal (e.g. RF). This means that we can purchase a cheap RF remote, sensors etc and the server can pick up the signal and integrate it into Home Assistant. In many cases, old tech you can get from a thrift store is given new life!

So the house has Gigbit Ethernet, 2.4 / 5ghz Wifi, Zigbee mesh and RF networks all running (more on this below)

# Sensors

That's all the hardware and networking kit, now on to the fun, with our sensor network!

## Moisture & temperature

There are a number of issues with victorian homes, the main one you hear about being moisture. We had a damp inspector come to our house and tell us to remove half the plaster on the walls downstairs. I finished taking 4cm thick beautiful lime plaster (with horsehair!) from part of one wall and given that it was the middle of Autumn and pretty wet outside, was surprised to find that it was bone dry. Actually, I was more angry than surprised, given that I'd paid someone to inspect the home and they'd given me a quite specific recommendation. Turns out many RICS inspectors have no knowledge of pre-1940s homes or the ways that walls are intended to breathe moisture. A four hour conversation with [Pete Ward over at Hertage House restoration](https://www.heritage-house.org/contact-us.html) gave me the necessary background, and I've taken a more cautious approach, choosing to monitor moisture in the air and trends rather than using damp meters on walls. Thankfully I didn't get any further along and most walls remain intact. There were a few rooms that I was concerned about moisture in, given the ways that the kitchen cabinets and floor joists in the front room were completely rotted out! I'll spare you the long story, and say that in all cases the problems related to recent introductions of technology (an incorrectly installed damp proof course, cement floors in the back of the kitchen and broken rainwater guttering).

When you're looking to purchase temp/humidity sensors, there's really no reason to DIY unless you're terribly bored. They can be purchased for barely more than you'd pay for raw parts. I'm partial to the Aqara temperature sensors. You can get these from the [official UK store](https://amzn.to/3bH2BOr) for £20 each. Being a patient man, I bought them directly from China via aliexpress.com for around £7-9 each and they came a month later. We have one of these in each room, which reports relative humidity and temperature for the room. They're pretty accurate and integrate painlessly with Home Assistant.

TODO: add screen shot

In home assistant, I'm using two additional plugins: Thermal Comfort (available on the [HACS](https://github.com/dolezsa/thermal_comfort) and Mold detection, which take in the information about temperature and humidity and make it more interesting and pointed with respond to water issues. By using monitoring like this, we've been able to pinpoint rooms where there were moisture issues (kitchen!) and add inverventions (fans) and also rule-out rooms which, contrary to the damp inspector's report didn't actually have any issues (the rest of the ground floor).

## Water Leaks

Water leak sensors are in the same genre for me, and I have one of these under every sink and tap in the house. Same prices via [UK store](https://amzn.to/3C0QMgN) or China.

## Outside Weather

We keep track of outside conditions in a few ways. I'm running a cheap but accurate [Bresser weather station](https://amzn.to/3vSuf1u), which keeps track of all the important things. You can purchase expensive (like £400) weather stations that are wifi connected, but I can much more easily get data from this device using the RTL433 RF signal which the weather station uses to send information to the "wireless" display. Job done. Al this shows up in Home Assistant pretty seamlessly, and I can track historic weather conditions in Grafana (more on this later).

## TBD

I have three projects in the works, which will be a bit more DIY as there aren't good solutions already out there. I'll include details as they are developed, but here are the basic parameters for now:

### (1) under-floor earth moisture monitoring

Given the fact that most of our floors are suspended timber over bare earth, my current project in process is to deploy a series of wifi soil moisture sensors under the floors in key locations to test for unexpected water ingress which might damage floor joists. I had to remove all the joists under our front room which were above an cellar, but most joists in the house are still sound, and these are giant hand-hewn oak beams more than a century old. I'm hoping a bit of tech can ensure that I can proactively monitor for broken things that might be unnoticed at first like rainwater guttering but which could cause unexpected and unnecessary damage.

### (2) airflow, fans, etc.

We need to, long term, dry out the cellar a bit with some better airflow. I'd like to run a good cross-flow ventilation system, and the most sustainable options would involve air heat recovery and dehumidification. So you draw air into the room with a low-energy fan, dehumidify the incoming air (rather than the room) and extract heat before venting humid air out. You can see a commercial version of this which is sold by [VapourFlow](https://www.vapourflow.com/product-category/basement-ventilation/) with h/t to Pete Ward for recommending them. Their solutions are a bit too expensive for my taste, and they don't interface with DIY home monitoring, so a bit too proprietary for my taste. I'm planning to wire up fans and dehumidifiers in a similar kind of system, but using HA and Node Red as the brains rather than the fan so it can be replaced easily and inexpensively. Research pending, I'll share when I do some prototyping. 

I'm also planning on putting the fans in bathroom and kitchen on a sonoff relay so I can track when they're off/on and automatically shut them off when a window is opened (windows have sensors on them too). This saves a bit of energy.

### (3) water flow monitoring and automatic shut-off

One of the key causes of water damage is water leaks, whether a sink or a pipe in a wall. An easy way to detect even invisible leaks is to monitor water flow levels and flag up unexpected flow. So I'm planning on installing flow hall sensors at key points in the plumbing, which I'll connect to ESP32 microprocessors and connect to Home Assistant using ESPHome. There are a ton of good tutorials on using ESP32 devices, which can be had for as little as £4 each if you buy direct from China and they provide a low energy bluetooth + wifi connection to whatever sensor you connect. ESPHome has really painless templates prepared for a whole bunch of devices, nearly anything you could think of actually. Unexpected egress of water will be flagged for inspection. 22mm brass compatible flow sensors run around £8-10 each, so not very expensive. I'm planning to couple these with zigbee-enabled water valves which basically just attach to the top handle of a ball valve. They're around £20 each.

### (4) air quality sensors

I've got a few sensors I'll be putting together using ESP32 microprocessors and ESPHome software to measure VoC, CO2 and PM25 particulates. These sensors are super cheap and sensors are not hard to assemble! I'm hoping to install these in places that are adjacent to smoke detectors, which I'm monitoring using RF sniffing (again) on that RTL-SDR device.

## Heating

### Heat Pump

One of the first upgrades we made to the house was to replace the gas boiler with a heat pump. After a fair bit of research, we settled on a Nibe heat pump. Nibe makes fantastic heat pumps, but their digital connection is pretty disappointing. But the upside of being a hacker is that I can take their great pump and whip up my own monitoring solution that is superior using open source tech.

Nibe sells an expensive subscription to a monthly service which takes data off the heat pump running in your home, uploads it to their cloud and then licenses the content back to you as the homeowner. Not only is the service (and this is the case with most utilities where this is an upsell dreamed up in a board room and not a genuine product) pretty painful to use, but it's overpriced, and generally for anything running in my home, which I paid for, I don't want a subscription.

The details are a bit complex, but most large appliances, since the 1970s have been built to report performance information using a serial connection and a simple protocol called ModBus. You'd seriously be surprised at how many modbus devices you have sitting around your house. Home Assistant makes modbus pretty easy to pull, and I've build some fairly snappy statistics pages which show performance data for our heat pump and the heating liquid at every stage in the system. I've been able to detect and correct several performance issues that our installer didn't notice, and which would never have shown up in regular maintenance, so a good reason to run a system like this!

You can read more where I've written it all up here: [https://community.home-assistant.io/t/how-to-connect-to-nibe-heat-pump-without-the-cloud/381099]

### Radiators

There's nothing wrong with radiators and hot water heating systems. I know water-based underfloor heating is the most efficient option, but in houses like ours, it's really impractical to try and put in new concrete floors. It's possible to work with limecrete as a breathable option, but radiators work fine for us. Half the rooms in this house didn't have any radiators at all, so I needed to learn a bit of plumbing (with thanks to Tony Jones at ASJ heating!) and managed to source radiators on Facebook marketplace and ebay. The key upgrade here was to go with [Tado smart radiator TRVs](https://amzn.to/3BWDiCz). There's no need for a thermostat as every single radiator is an individual heating zone. So if we turn on the woodstove, the system will progressively turn down radiators as rooms get warmer and avoid wasting energy. The system also allows us to run everything on schedules, and we can turn everything off when we're away as needed. The valves go on sale a few times a year and we picked ours up for around £35. The only down side of Tado system is that they use a proprietary wireless protocol and it's not a mesh system, so users sometimes have issues with range with the outermost radiators struggling to catch the signal of the USB controller. I got around this by using a 15m Active (powered) USB cable and moving the controller USB stick to a central position in the house. But in the future, I'd prefer to use zigbee TRVs. Trouble is, there aren't really any yet. Shelly has just announced WiFi TRVs at a good price point, but TBH, they're pretty ugly. I shall be watching this space closely...

## Lights

We're just getting started with lights, but it's really pretty simple. A lot of people put a huge amount of money into smart bulbs, but from my perspective it's a bit too much tech for a simple situation. So we've gone with smart relays at each light switch. I use [shelly](https://amzn.to/3PgrJcp) devices for lighting. They're designed to be hackable, are inexpensive and rock solid. For most rooms, we're using Shelly1 relays. In a few we're running the Shelly Dimmer unit. I'll post a longer explainer in a bit as, to be fair, light switches are a bit of a science. Suffice it to say for now, it's nice to be able to switch off lights automatically, but given the whole house is on LED bulbs, it isn't really a major electricity outlay.

## Plants

We're relying mostly on the weather station outside to monitor growing conditions and the garden. Next spring we'll be adding more food growing, so I'll add in details here regarding the smart watering and soil monitoring system we deploy in due course. Indoors, I have a germination box with an inexpensive soil warming cable in sand which is powered by a Sonoff smart-relay. The Sonoff relay has a built-in temperature sensor and allows me to track the soil temperature. We have a separate temp/humidity sensor in the room which I also average in to control the soil warming cable. This allows us to germinate seeds a month or two early using relatively little energy.

## Energy Use Monitoring

We're gradually adding energy monitoring to keep devices throughout the house. Every major appliance is plugged into a cheap [Gosund smart switch](https://amzn.to/3zJmHPP) which cost around £7 each and have built-in energy monitoring. Home assistant does the rest and tracks our usage on each appliance. I was already able to realise that the dishwasher was accidentally running on a high energy setting, and we were able to adjust that and save many KW from being wasted. We use Octopus Energy, for whom I have absolutely nothing but high praise. Their supply is sourced from 100% renewable energy, especially now that we've removed the gas meter from our home (the induction stove was the last step after the heat pump got installed). They also work quite actively with the DIY hacker community, providing information about energy prices and allowing users to actively monitor their usage and pricing. Shameless plug: If you aren't already using Octopus, you can get £100 (I do also!) back when you sign up using [our referral link](https://share.octopus.energy/azure-crab-741). Even though Octopus has installed a smart meter in our home, I wanted to have real-time monitoring, so we installed a [Shelly EM device](https://amzn.to/3bI44UG). I can subtract usage from most devices which are being monitored using smart plugs, and this just leaves a few things left. 

As a project, I wanted to see if I could get a sense of the energy use on each individual component in our heating and hot water system, so I've just finished a new design which uses an ESP32 board, an series of inexpensive [PZEM-004 energy monitoring boards](https://amzn.to/3AbSQRB) - note, you can get these for around £3-4 via aliexpress.com. The PZEM boards communicate via another serial protocol (not modbus this time) called UART, and you can string together a number of devices on a single ESP32 board. This will allow me to monitor the flow pump, hot water immersion heater, heat pump controller, heat pump unit, etc. each individually as well as see when they're on and how hard they're working. I'll have this data integrated into the heat pump monitoring system. I'll add schematics etc. here as soon as this unit is done, hopefully later in 2022.

# HiFi

What home is complete without multi-room streaming music? This is worth a separate guide, which I've developing here: [https://github.com/kidwellj/multi-room-hifi]

# Car Charging

We have an EV, and are currently charging this with an Ohme device. Ohme

# Solar

