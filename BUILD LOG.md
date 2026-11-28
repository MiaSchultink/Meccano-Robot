Started by trying to revive the existing Meccano robot. It was covered in dust and looked like any robot that hadn't been touched in 10 years would look.

I figured the battery was dead and looked up the manual to figure out how to charge it. I quickly learned that it has a custom battery pack and charger which I didn't have anymore. Bonus, the product and its charger are non longer manufactured. I started looking for replacement chargers sold locally online. I was also considering buying a whole new battery pack as I wasn't sure the original one still works.

Since that was a bust, I started looking at how I could move the servos on it with an Arduino. I decideed I would try to move a single servo first as that wouldn't require that much power. I noticed some people made open source libraries for controlling these types of motors from an Arduino. The one I decided to start with is documented here. https://github.com/alexfrederiksen/MeccanoidForArduino/tree/master

But before I could get to using it, I had to figure out how to even connect a Meccanoid servo to an Arduino Uno. For that, I stumbled upon this helpful video https://www.youtube.com/watch?v=8y_xMW0Kr5c. The video was creaed by MECCAControl which apparently makes a Windows software you can use to easily program Meccanoid components, but I have a mac. This video also gave me a glimps into how these servos actually work. Aparently you can construct chains of multiple servos and reference each component of your chain in the code.

I started putting the simple hack-job electronics setup together. I basically stole the breadboard and MB102 from the servo arm project I made and got an Arduino R3 I and lying around. I unpluged the left elbow from the rest of the motors in the chain and used the jumper wires I had bought for the servo arm to get a male ending for the breadboard.

<img width="768" height="1024" alt="EFA4A1D8-7A63-4029-84BD-F92A2819829F_1_105_c" src="https://github.com/user-attachments/assets/1039d99e-146b-43d2-9f0e-2d0a2f990ba7" />

I had a good idea of how the power, ground, and signal wires should be connected from the servo-arm project. I'm not really an electronics expert so I consulted gemini before testing my setup and it suggested using a resistor, so I modified my setup to include it. I created a little Tinkercad sketch to keep things organized. It has a simple servo connected because that's all they had.

<img width="840" height="549" alt="Screenshot 2026-11-20 at 23 03 51" src="https://github.com/user-attachments/assets/2f37fe45-3be4-41c4-ae42-fb3b737c5701" />

<img width="768" height="1024" alt="509C9D99-A73B-4192-8C4D-B48E4EF4200B_1_105_c" src="https://github.com/user-attachments/assets/f4df30eb-5140-47f0-ba3c-125eb9ef571f" />

<img width="768" height="1024" alt="1017F8BD-3E1F-4B87-8629-8C4B22434A03_1_105_c" src="https://github.com/user-attachments/assets/1c9a3763-d0ad-466d-abe0-cd2634a238db" />

I moved the whole setup to my desk as that's the only place I have working plugs for the wall outlet I still relied on.

I used an open source library someone adapted from the oringinal MeccaBrain library or something similar: MeccanoidForArduino. I read a bit of its documentation on github to udnerstand how to use it. I pieced together some basic parts of its test code. The video from before came in handy. I knew why the code was referencing servo 0 in the chain. https://github.com/alexfrederiksen/MeccanoidForArduino/blob/master/meccanoid_test/meccanoid_test.ino

```
#include <Meccanoid.h>

/* Note that their needs to be a 22k ohm pull-up resistor on chain input,
 * See http://www.meccano.com/meccanoid-opensource Smart Module Protocol 
 * (bottom of PDF) for more details. */

/* PWM pin for chain */
int pin = 8;

/* define chain and PWM pin for it */
Chain chain(pin);

/* define modules to use and their positions */
MeccanoServo servo1 = chain.getServo(0);  // first on chain  (0)

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
}

void loop() {
  // update chain
  chain.update();

  if (servo1.justConnected()) {
    // set color to red (r: 1, g: 0, b: 0) and pos to 45 degrees
    servo1.setColor(1, 0, 0)
      .setPosition(90);
    delay(500);
  }
  if (servo1.isConnected()) {
    Serial.println(servo1.getPosition());
  }
}
```

I will admit that when I first tested it I was sure I'd fry an electrical component. I braced for impact, but surprisingly enough the LED of the motor turned solid green and it made a little buzzing sound! Honesly more than anything I was thrilled to see that the servo is still alive after 10 or so years. That meant we had something to work with. It didn't move though...

<img width="1080" height="1920" alt="ABAC661B-B3A4-4CB8-A3C6-935A6E56DB24_1_102_o" src="https://github.com/user-attachments/assets/7a8ee01a-66cb-47d6-9428-aebe7fe32797" />

I hit a bit of a wall here. I tried to print some values to the SerialMonitor and discovered that the Arduino wasn't picking up the motor. The position was -20 and the isConnected value remained 0. I also tried to mess with the IDs a little. I figured maybe it was 1, or maybe each motor on the robot had a unique ID. No luck.

At this point, I consulted the internet. Specicially, gemini. I described the problem. We went back and forth for about half an hour. Every clarification I made helped its advice be more specific. I gave it my wiring setup on the breadboard-Arduino side. I confirmed that its wired correctly.

<img width="1700" height="1058" alt="image" src="https://github.com/user-attachments/assets/84d21ef9-65a4-4dcd-89e1-758d04cc6cfc" />

<img width="1630" height="998" alt="image" src="https://github.com/user-attachments/assets/e50896ac-6f8d-4bdc-a6e6-cbfd06225bc8" />

Turns out the culperate was the setup's connection to the servo. Apparently I connected it to the OUTPUT wires not the INPUT wires on the servo. I didn't know it had 2 different streams.

<img width="1526" height="1028" alt="image" src="https://github.com/user-attachments/assets/938ddca0-4357-4651-95b3-69aae08cc727" />

I plugged the breaboard jumper wires to the input cable and the LED turned blue (like I programmed it to in test code I wrote)! I added the motor movement back in and it moved a litle! Very exciting.

<img width="4284" height="5712" alt="874BB715-B941-4033-96F5-80B52405BC9E" src="https://github.com/user-attachments/assets/d6f47215-fc39-4f5e-a5ac-3ddef532f3cb" />

https://github.com/user-attachments/assets/fafa95e1-296c-4535-a9a4-e90b0a00662a

Now that I knew one motor worked, I started planning how I would set up the electronics to control all the motors at once. This was harder than I thought. Initially, I thought of hijacking the Meccabrain as it conveniently connects the motors and the battery internally. However, I quickly learned that connecting external electronics I would need for my custom controls would be difficult and require custom soldering in this case, so I decided agains it. I was curious to see how the brain implemented this connection though.

<img width="768" height="1024" alt="D208BE2A-5D3D-4526-9333-5D83CB47E187_1_105_c" src="https://github.com/user-attachments/assets/92959225-63aa-41ab-b57b-01a679259c09" />

<img width="768" height="1024" alt="0B5F13F8-39B3-4083-9CB8-7B81B614F7CB_1_105_c" src="https://github.com/user-attachments/assets/d3b79cab-b56d-4056-91d8-9168cccce6b0" />

While I knew I would eventually have the robot move around independently when I incorperated the Raspberry Pi and camera, I wanted to have the option to control the robot manually. So, this time, unlike the previous arm project, I would couldn't escape buying an ESP32 unless I planned to walk around after it with a laptop. The R4's bluetooth capabiliteis don't let it pair with a PS5 controller.

I began planning the electronics layout around this change. Some online research revealed that I was missing several components. Additionally, it seemed like I didn't actually need the Arduino as the ESP32 had everything I needed. So, I dumped the Arduino. One complication this invites is conveting between 3.3V used by the ESP32 and 5V.

**Rough diagram**

<img width="1558" height="2016" alt="7991F9D7-8C99-4B2E-BE52-E5974A8D3633_1_102_o" src="https://github.com/user-attachments/assets/392ac2de-b045-4d95-9e3e-372f19d30284" />

**Shopping list**

<img width="2606" height="940" alt="image" src="https://github.com/user-attachments/assets/1852ecdb-f172-45b7-953f-3d2425379229" />

I ordered from about 4 different local suppliers. Some of the components arrived. I had accumulated enough electronics and tools to justify freeing one of my desk drawers for them :)

<img width="768" height="1024" alt="1EF1A557-C54D-4FD5-8160-3223DC0EE159_1_105_c" src="https://github.com/user-attachments/assets/9dca8999-7efa-401b-b4e5-d7ba62e3b058" />

While I waited for the rest to arrive, I tested some more of the motors to make sure they still worked. I couldn't test the feet as I didn't have proper equipment to communicate with the DC motors yet.

https://github.com/user-attachments/assets/68c55994-7971-4090-b4d5-8631b27985c9











