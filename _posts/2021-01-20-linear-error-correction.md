---
date: 2020-01-20 00:28:00 +0000
---

# Error correcting a miscalibrated sensor

As you may know from my last post, I used to have a
[Winsen MH-Z19B](https://uk.banggood.com/MH-Z19B-Upgrade-Version-0-5000PPM-Infrared-CO2-Sensor-For-CO2-Indoor-Air-Quality-Monitor-UART-or-PWM-p-1094463.html)
CO2 sensor. In fact, I have 4 of them now, since after my little incident I was
left with 0. Hopefully I'll keep these ones alive.

![MH-Z19B](https://imgaz2.staticbg.com/thumb/large/oaupload/banggood/images/50/7F/7baf213e-7e68-42a2-9dea-fd4df79e8e29.JPG.webp)

As a refresher, this is a CO2 sensor that uses NDIR (non-dispersive infrared) to
detect the concentration of CO2 in the atmosphere. Every 6 seconds, it performs
a measurement, and in order to do so it needs a relatively accurate temperature
measurement. This is because the measurement method for CO2 used is generating
some heat, causing [far infrared](https://en.wikipedia.org/wiki/Far_infrared)
radiation to be emitted through black-body radiation, which is then bounced
around a chamber and measured by an IR sensor, which is much like one in a
thermal imaging camera or a motion sensor.

## First attempt: Univariate linear functions

The particular sensor I got (and broke) was off in its temperature measurement
by about 6 Kelvin. This meant that all its measurements were off by a certain
amount (according to my Temtop P1000 air quality measurement station). So
naturally, my first idea was to assume (hope?) that the error is a linear
factor, maybe with an additional offset, and if I could just compute the
function `f(x) = mx + b`, I could easily correct the measurement it gave me. A
line can be described by 2 points, so I would just need 2 measurements and have
my function. Unfortunately, my measurements are pretty inaccurate, because CO2
concentration constantly changes, especially when I'm in the room, turning O2
and food into CO2. So the next step was to turn to least-squares optimisation.

1. Collect a bunch of data points: (CO2 according to my sensor, CO2 according to
   Temtop reference sensor)
2. Optimise a linear function to have the least amount of error given all the
   data points.

The Temtop also has the same sensor in it, so I figured it would be easy to
match them up.

![Temtop P1000](https://cdn.shopify.com/s/files/1/0428/8338/3449/products/elitech-p1000-air-quality-monitor-1_1200x.jpg)

Of course, storing a bunch of data points as well as the code for computing
least squares and also having to run that code on startup sounded like a really
bad idea to me and my 8KB of program memory. Fortunately, there's `constexpr`,
the promise that C++ can be executed at compile time.

*Un*fortunately, however, since I was using C++11, and a rather old version of
AVR-GCC (5.x), I could only really use single return statement `constexpr`. This
resulted in a rather convoluted implementation of least-squares for one variable
([see for yourself](https://github.com/homectl/homectl/blob/f540097359afba1a886e5051e3b3d33ca61dd561/include/linreg.h)).

## The problem of temperature

The solution worked perfectly. Or so I thought.

Since I had been doing all my measurements in my room at a pleasant 294-295
Kelvin, my corrective function worked perfectly inside that temperature range,
but became wildly wrong at lower temperatures like the less pleasant 288 Kelvin
that my room would have after opening the window for a bit. Clearly, the error
was not linear in one variable - it was at least two! I had to consider
temperature.

The sad news here is that in order to do multivariate linear optimisation, you
basically need matrix operations like multiplication, determinant, and inverse.
Writing that in C++11 `constexpr` would prove to be a _massive_ pain. Not
impossible, mind you, and I have done the kind of crazy `template`
metaprogramming required for this in the distant past, but I'd really prefer to
keep that past distant. So I had no choice but to try and upgrade my compiler.
It took me a while, but it turned out to be
[possible](https://github.com/homectl/homectl/blob/ee5cbda2fcb9ce243b9149e17aaba11d40ecbb78/platformio.ini#L15)
to upgrade the AVR-GCC to a more reasonable version like 7.3.0, which nicely
supports C++14 `constexpr` where you can basically run any
global-side-effect-free code at compile time (woohoo!).

## Multivariate least squares

The final solution, which ended up actually working pretty decently, is based
around a small `constexpr`
[matrix library](https://github.com/homectl/homectl/blob/be6fcc8878d2278c1622ffb512db1a6ea0072782/include/homectl/Matrix.h)
I wrote, and an algorithm from
[Wikipedia](https://en.wikipedia.org/wiki/Ordinary_least_squares#Matrix/vector_formulation).
It is now
[used](https://github.com/homectl/homectl/blob/514c9035a80152d546ed851b582719d57ed29111/src/CO2.cpp#L8)
to correctly return CO2 values that agree with my reference sensor.

Is this solution correct? I don't know. The temperature sensor inside is
probably a thermistor, which has a logarithmic sensitivity curve.

![Thermistor Curve](https://i.stack.imgur.com/YUqJF.jpg)

It's possible that the function seems linear only in the narrow range I measured
things in (much like how Earth looks flat (no, I'm not going to hyperlink to
flat earthers here) when looking at a narrow slice of its surface), but is
actually logarithmic in a wider range. If that's the case, the problem is pretty
easy to solve still, even with the ordinary least-squares method. I'd just make
the logarithm of a value one of the variables and compute its weight. In fact, I
could do that right now to see whether the function is in fact logarithmic, but
for the temperature range that's relevant to me, the function is linear enough
and now I'll move on to the next thing :).
