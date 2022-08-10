Victorian smart home

Version 1.0

We're renovating an 1830s end-terraced home to bring it into the 21st century. This means in many cases that we're removing inappropriate tech which has been introduced more recently (like plasterboard / sheetrock!), and in other cases lightly augmenting our house to make sure we can lower our energy footprint, anticipte problems, and make living just a little bit more fun and comfortable. I've developed this repository in part to document my work as it goes along, but also on the possibilities that others in similar situations might benefit from some of the knowledge I've gained along the way. There are a number of issues pertaining to homes built before 1900 which can benefit from smart devices which really don't pertain to newer builds. I'll get into this a bit further below.

# Prerequisites:

These are the key pieces of infrastructure that needed to be in place before we could get moving:

## (1) Home server

This is a relatively old hardware build, using a low-voltage intel CPU. I generally run about 80% CPU and 90% of the RAM on this server, so these specifications work for me. Some day in the future, I'll switch to a newer low-voltage board and CPU with 32GB RAM and that should more or less sort me for the next decade. This server has been running more or less continuously without any hardware upgrades since 2015. 

### Server hardware specifications:

- Motherboard: GA-Z97N-WIFI
- CPU: Intel Core i5-4690K 3.5 GHz Quad-Core Processor
- Memory: 16GB DDR3 (2x8gb)
- Case: Cooler Master Elite 120 Advanced Mini ITX Tower Case
- PSU: Corsair CSM 450 W 80+ Gold Certified Semi-modular ATX Power Supply
- Storage: 1TB SSD (for OS) + 12TB (Western Digital drives @ 4tb x 3)
- USB RTL-SDL: Nooelec NESDR SMArt v4 SDR (RTL2832U & R820T2-Based Software Defined Radio)

(fyi: amazon affiliate links above)

### Software:

If you're just getting started, I'd highly recommend the guides by [Alex Kretzschmar](https://github.com/ironicbadger) who I've followed almost entirely. The only thing I've been ahead of Alex on was using traefik. You can find his first guide, which got me started here: [Perfect Media Server 2017](
https://www.linuxserver.io/blog/2017-06-24-the-perfect-media-server-2017), though I'd note that he doesn't switch to proxmox (which I've done last year) until the [guide from 2019](https://perfectmediaserver.com/).

- I'm running a proxmox hypervisor, with several virtual machines underneath, mostly running debian / docker. Most of my docker containers are running off a Debian install, which includes traefik. I run all our services behind a wildcard domain hosted on cloudflare. Traefik re-directs some traffic to other containers, and for the sake of giving some services unique IP addresses I'm running the following additional servers as separate VMs:

(2) Home Assistant OS - this is the brain of our whole smart home system. I can't provide extensive detail here, but if you want to run a system like I've got here, you could strip things down considerably to just run HAOS on a smaller RaspberryPi based computer, for around £100-200

## (2) Network infrastructure:

