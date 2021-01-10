---
date: 2020-01-10 20:50:23 +0000
---

# Teardown of some air quality sensors

Recently, I broke two of my sensors (an expensive mistake involving USB-PD): the
MH-Z19B CO2 sensor and the PMS5003T combined particulate matter sensor. Of
course, I took the opportunity to dismantle them and see what's inside.

## Winsen MH-Z19B

This is a CO2 sensor that uses infrared light absorption to detect the
concentration of CO2 in the atmosphere. It is fairly precise and is the sensor
built into the Temtop P1000 air quality meter. It consumes around 130mA at peak,
which is once every 6 seconds when it performs a measurement.

It has a reflective chamber in the top part of the sensor in which a far infra
red (i.e. heat) emitter shines. This emitter needs around 3 minutes to heat up
to a stable temperature. We can see this chamber when removing the top-most
layer of the sensor. I had to do this with my Dremel 3000 and a cutting disc,
because this sensor is glued shut and the glue didn't melt at 300C from my hot
air station. Some other parts of the sensor did start to melt, so I stopped at
that temperature.

![Sensor](/images/MH-Z19B/PXL_20210110_190417458.jpg)

The following photo shows all the pieces of the sensor: the control board with
sensor and emitter; the separating reflective layer between board and chamber;
the top of the chamber; the walls of the sensor.

![All pieces](/images/MH-Z19B/PXL_20210110_191822552.jpg)

The MCU has the letters "F051K86", "GQ2AI 1L9U", "CHN 948B". This indicates that
it is an
[STM32F051K8](https://www.st.com/en/microcontrollers-microprocessors/stm32f051k8.html)
48 MHz ARM Cortex-M0 MCU. The sensor needs to measure its temperature, which may
be influenced by the surrounding atmosphere, so one of these components is a
thermistor. My guess would be it's R4, which says "1 QRD" on it, since it's the
resistor closest to the diode.

![PCB front](/images/MH-Z19B/PXL_20210110_191844527.jpg)

On the back, we see the Hd, Txd, Rdx, and V0 pins on the one side, and PWM, V-
and V+ pins on the other side. The actual pins are gone because I tried to open
up the box with a pair of pliars at first, which failed. Later I found out why
that failed: the box is 1mm thick plastic and it took a decent amount of force
on the Dremel to open it up.

![PCB back](/images/MH-Z19B/PXL_20210110_191900311.jpg)

The MH-Z19B sensor can not easily be disassembled and reassembled. If you break
it, it's broken forever.

## Plantower PMS5003T

This is a PM0.3, PM0.5, PM2.5, and PM10 sensor. It works with laser scattering:
a laser is aimed over a sensor, and small particles bend the light towards the
sensor, which can then measure how many of those particles are in the atmosphere
and how large they are based on the diameter of the ring shapes that appear on
the sensor.

The "T" at the end stands for Temperature, as this sensor also includes a
temperature and humidity sensor. These are not very precise, though, and I would
recommend against getting a PMS5003T, and instead getting a plain PMS5003 or
PMS7003. For temperature and humidity, a separate DHT22 would serve you much
better.

Unlike the MH-Z19B, this sensor can be disassembled and reassembled, so it can
theoretically be repaired. In my case, both the fan and the laser were broken,
so instead of trying to fix it, I bought a new one.

The sensor is shielded by an aluminium case which reduces influence on the
measurement from electromagnetic radiation outside the sensor.

![Shielding](/images/PMS5003T/PXL_20210110_202136700.jpg)

The laser itself is kept in a small plastic box (second item on the photo) to
keep dust from getting onto the laser itself. The goal is to measure dust
particles that flow through the case but not retain any of them, or even larger
ones. A fan (4th item) blows air from outside the sensor through the case.

![All pieces](/images/PMS5003T/PXL_20210110_192028839.jpg)

This is a close-up of the laser and sensor. The Molex socket provides power to
the fan.

![Laser and sensor](/images/PMS5003T/PXL_20210110_192131522.jpg)

The MCU has the letters "PLANTOWER", "PT-DSC0916", "1943", "635343", and it's
unclear to me what component this is. It has 12 pins on each of 4 sides, so 48
pins in total. It could be, for example, an
[STM32F101C6T6A](https://uk.rs-online.com/web/p/microcontrollers/7147676) ARM
Cortex M3, relabeled, or an in-house product.

![PCB front](/images/PMS5003T/PXL_20210110_192049591.jpg)

I've shaved off the top of the MCU and found something that looks like two
separate pieces of silicon inside. If anyone has an idea what this is, do let me
know. The MCU on this device was broken, so I couldn't debug it to see what it
might be.

![MCU shaved](/images/PMS5003T/PXL_20210110_224610343.jpg)
