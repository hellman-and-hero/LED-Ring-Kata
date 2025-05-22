## The initial situation

The legacy application [https://github.com/hellman-and-hero/musa-client](https://github.com/hellman-and-hero/musa-client) is to be expanded. Changes of the two sliders on the tab "Stock P / A" should also be visualized on LED rings. For this purpose the method [https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockPanel.java#L65](https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockPanel.java#L65) must be adapted. The StockPanel class is instantiated by the framework. Therefore, the signature of the constructor must not be changed, nor the content of the interface which is passed in the constructor.

The current value should be displayed with an LED ring for each slider on the panel. This hardware was purchased from a third party manufacturer.

- The hardware has 32 LEDs in the form of two rings, 16 LEDs each. These 32 LEDs are logically arranged one behind the other, i.e. the LEDs 1-16 are in the first ring, the LEDs 17-32 in the second ring <sup>[1](#myfootnote1)</sup>
- The color to be displayed must be sent to the hardware for each individual LED, i.e. the first three LEDs should be "switched" three messages are to be sent <sup>[2](#myfootnote2)</sup> <sup>[3](#myfootnote3)</sup>
- The values of the slider range from 0-100. Depending on the value, a corresponding number of LEDs should light up. For example, with a ring with 16 LEDs and a value greater than 0, the first LED should light up, with a value greater than 25 the first five, and with a value greater than 50 the first nine. If the value is 0, all LEDs should be off.
- In addition to the hardware with 16 LEDs per ring, there are also variants with 8 or 12 LEDs per ring. It must therefore be possible to configure the number of LEDs so that all three variants can be used.
- The name of the MQTT host, its port and the number of LEDs for ring 1 and ring 2 are in the [StockContext](https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockContext.java). which the [StockPanel in the constructor](https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockPanel.java#L39) receives.
- Since you do not have the real hardware available, you can use [the software simulation of the hardware](https://github.com/hellman-and-hero/tdd-demy-hardware-sim). When processing the kata, we equate the simulation with the real hardware (which we are normally not allowed to do, as the simulation can / could behave differently than the real hardware).

<a name="myfootnote1">1</a>: A picture of this can be found on page 11 of the slide set from [TDD demystified](https://www.xpdays.de/2018/downloads/174-tdd-demystified/tdd_demystified.pdf)

<a name="myfootnote2">2</a>: Technically, an MQTT message has to be sent to the MQTT broker for each LED, to which the hardware is also connected. The message must include the topic "some/led/\<led-number\>/rgb" and as payload have the color as a hex value ("#RRGGBB").

<a name="myfootnote3">3</a>: If you do not want to work out how to send MQTT messages yourself (is not the aim of this kata), you can use the ["simplified" branch](https://github.com/hellman-and-hero/musa-client/tree/simplified) and there the 
[MqttSender](https://github.com/hellman-and-hero/musa-client/blob/simplified/src/main/java/rgbledring/MqttSender.java). If you don't have an MQTT broker, you can either simply install mosquitto, [mosquitto via docker](https://hub.docker.com/_/eclipse-mosquitto) (_docker run -it -p 1883: 1883 -p 9001: 9001 eclipse-mosquitto:1.6_) or a [public MQTT broker](https://github.com/mqtt/mqtt.org/wiki/public_brokers).

### Sprint Backlog Sprint #1

1. The value of the first slider should be visualized on the first ring. In Sprint # 1, colors are not yet important, it is sufficient if LEDs can be switched on (#FFFFFF) and off (#000000)!
2. Direction of the LEDs
 The direction can be configured for each ring (clockwise or counter-clockwise)
3. Second ring
 Analogous to the first ring, the value of the second slider should now be visualized on the second ring. Here too, initially without colors

**Have you implemented the requirements? Great! Then on to the next page!**

### Sprint Backlog Sprint #2

1. The LEDs of the rings should light up in color (first third green, middle third yellow, last third red). Should work for rings with 8 (3/2/3), 12 (4/4/4) and 16 (5/6/5) LEDs. Rings of other sizes do not have to be taken into account.
2. Overload
 The ring should be filled from 0-90 as before from 0-100, i.e. 90 means "all LEDs on" (formerly 100), 45 means "the first half of the LEDs is on (formerly 50). If more than 90, **all** LEDs **light up red**. The threshold value (90) should be variable (e.g. 80, 95 or any other value)

**Have you implemented the requirement? Great! Then on to the next page!**

### Sprint Backlog Sprint #3

1. Peak hold
 When the level drops, the LED that signaled the maximum value should continue to glow for one second

2. So that the hardware can also be mounted on the ceiling, the start/end of the rings should be configurable. Also a lateral mounting on the wall should be possible. So in the first case (ceiling) for rings with 8 LEDs the order of the LEDs to be switched would be 5,6,7,8,1,2,3,4 and in the second case (wall right 7,8,1,2,3,4,5,6 or wall left 3,4,5,6,7,8,1,2). This setting applies to the entire hardware and thus to all rings equally. 

**Have you implemented the requirement? Great! Then on to the next page!**

### Sprint Backlog Sprint #4

1. It should be possible to switch the individual features on / off or configure each ring. The following can be configured for each ring:

- Number of LEDs
- Clockwise or counterclockwise direction
- Overload on / off (including individual threshold value)
- Peak hold on / off
- Configure position of first led for the rings

**Have you implemented the requirement? Congratulations! The product owner has no further requirements. On the next page you will find a few more comments**

## Remarks
- Did you stumble across the index of the LEDs (description speaks of LEDs 1-n, but the hardware assumes zero-based numbering)? This is only noticeable through trial and error or integration tests with the real hardware (or our simulated hardware).
- What effects would the requirement that a new hardware version be supported on your productive and test code have, which should not be controlled via MQTT but via the serial port (the existing support for MQTT is no longer necessary after a transition period and can be removed )?
- Did you use a public MQTT broker in your tests? One that exists in your environment? What are the consequences for your tests? What would be alternatives?
