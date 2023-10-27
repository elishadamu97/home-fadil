---
layout: post-8-bit_bb_cpu
title: Arithmetic and Logic Unit
mathjax: true
similar: 8-bit-computers
date_child: "May 27, 2023"
category: children
parent: 8-bit_breadboard_CPU
permalink: /projects/8-bit_breadboard_CPU/8-bit_bb_cpu/ 
permalink: /projects/8-bit_breadboard_CPU/ALU/
summary: 8-bit Breadboard CPU. 

---

<meta name="viewport" content="width=device-width, initial-scale=2.0">

# Clock

# Basics Primer


<div class="grey-background">

What is a CPU’s clock?
<br>
<br>
Think of the "clock" in computers and most electronic systems as the heartbeat of the system. It is a simple but vital signal (a voltage) that alternates between on and off (high and low). This signal oscillation keeps everything in sync, making sure all sequential components inside the system play nicely together, and every time it "ticks," it tells parts of the system when to act.
<br>
<br>
In many ways, the speed of the clock dictates the pace at which the computer operates. Like a conductor leading an orchestra: the faster they wave their baton, the quicker the music plays. Similarly, a faster clock rate in a computer means that it can carry out actions or operations at a swifter pace. However, as discussed at the end of this post, there's a balance to strike. The physical properties and layout of a system's components can set a cap on how fast its clock can be pushed. Just as music played too quickly can become chaotic, a computer's clock speed needs to be optimized to ensure efficient and stable performance.

</div>



## Multivibrators


# ggg

```txt
   __|__            __|__              __|__              __|__              
  |     |_C3 =__   |     |_C3 =__     |     |_C3 =__     |     |_C3 =__   
  |B   A|          |B   A|            |B   A|            |B   A|          
   |‾‾‾|            |‾‾‾|              |‾‾‾|              |‾‾‾|
   0   1            1   1          
```


Multivibrators are digital oscillator circuits with two stable states. The transitions between the  states can be controlled or automatic, defining the multivibrator’s behavior and application. The three forms of a multivibrator circuits are astable, monostable, and bistable.

### Astable Multivibrator

The astable multivibrator continuously oscillates between two states, making it ideal for generating clock pulses in microprocessor circuits and other synchronized systems.

### Monostable Multivibrator

The monostable multivibrator has one stable state. When triggered, it shifts to a quasi-stable(temporary) state for a predetermined period before returning to its stable state.

### Bistable Multivibrator

The bistable multivibrator has two stable states. It changes from one state to another when triggered. A latch is a good example of a bistable multivibrator, as it maintains its output state either high or low until it receives a trigger(Set or Reset) signal to change state.

## The 555 Timer

Just like the SAP-1’s, the clock module of my CPU employs a 555 timer. The 555 timer is a versatile chip that can generate accurate pulses of various width, making it a suitable choice for this application. Compared to crystal oscillators, which are often used in many electronic devices for their precise frequency generation capabilities, the 555 timer offers greater flexibility. The circuitry of the 555, as we'll delve into shortly, allows us to easily manually adjust our clock speed.

On the other hand, crystal oscillators, as the name suggests, utilize the mechanical resonance of a vibrating crystal to produce an electrical frequency. A single crystal oscillator is typically designed to generate a specific fundamental frequency determined by the physical properties of the crystal, such as its size, shape, and the manner in which it is cut. It is possible for a crystal to support harmonics, which are integer multiples of its fundamental frequency. However, in the context of this breadboard CPU, the adaptability and control offered by the 555 timer are more convenient, especially for tasks like debugging and real-time modifications.

Below is a high-level(Modularized) schematic of the Texas Instruments LM_555 timer, simplified to show its working principles and pinout. The transistor-level schematic is linked at the bottom of the page.

![credit: (“LM555 Timer datasheet (Rev. D)”, n.d.) Texas Instruments]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/lm555.png)
credit: (“LM555 Timer datasheet (Rev. D)”, n.d.) Texas Instruments

As you can see there are three resistors in the center of the circuit, forming a voltage divider. And the 555 timer is called this way **because on the original model the value of R was 5 kΩ thus 555 because there are 3 Rs.**

# My Clocks

I’ve built two clocks for this project: a portable clock for debugging and a permanent clock for ongoing use.

## Debugging Clock

The debugging clock is similar to Ben Eater’s design. It has an astable and a monostable configuration, which can be selected for different testing needs using a switch. 

The difference between my clock and the SAP-1’s is the use of NAND logic for the astable-monostable output switching instead of AND logic.

Ben Eater’s simplified diagram of the 555 timer(which I minimally annotate throughout this post), shown below is, in my opinion, very helpful for understanding the intuition behind the functioning of the timer.

## Astable Multivibrator with 555

![Astable configuration of 555 timer - Ben Eater Clock Module video]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/555_astable_2.png)

In this straightforward representation, the functioning of the astable multivibrator using a 555 timer is clearly illustrated.

### 1. Initial Conditions

Upon powering up, let’s assume nodes ‘a’ and ‘b’ have 0V initially. No matter how fast the capacitor at node ‘b’ charges; the voltage divider’s node on C2 will reach 1.67V, before node ‘a’ reaches 1.67V. In other words, C2 will be on until the voltage at not ‘**b’** surpasses 1.67V.

While C2 is on, the ‘set’ input of the latch stays on, keeping its "not" output off; consequently, the BJT connected to the discharge pin (pin 7/node ‘a’) of the timer is also off, disconnecting the path to ground and allowing the charging of the external capacitor through the resistors.

![2.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/2.png)

### 2. Capacitor Charging

The external capacitor starts charging through the resistors. Current flows, and the voltage across the capacitor rises exponentially, following the RC charging curve.

As the capacitor charges, the voltage at node 'b' eventually exceeds 1.67V. C2's output goes low because the voltage at the inverting input is now higher than the non-inverting input. However, the state of the latch remains unaffected, since no input voltage at the set pin does not trigger a change in the output state.

![3.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/3.png)

### 2.1 Latch Reset

The capacitor continues to charge until the voltage at node 'b' surpasses 3.3V. At this point, the "reset" input of the latch is activated due to the output of C1 turning high (as the non-inverting input connected to node 'b' is now higher than the inverting input). The "not" output of the latch turns high, resulting in the activation of the BJT.

### 2.2 Capacitor Discharging

With the BJT on, the capacitor begins to discharge through the discharge pin of the timer, leading to a decrease in voltage at node 'b'. As it drops below 1.67V, C2's output goes high again, setting the latch and turning off the BJT. 

The cycle repeats, giving rise to an oscillating output at pin 3 of the timer. This self-sustaining oscillation produces a square wave output, characterized by the charging and discharging times of the capacitor, which are influenced by the values of the connected resistors and capacitor. The frequency of oscillation and duty cycle can then be adjusted by varying these component values.

Typically, to have flexibility on the clock rate, a variable resistor is added in addition to a resistor between nodes ‘a’ and ‘b’. The variable resistor alters the time it takes for the capacitor to charge and discharge, thereby altering the clock rate. 

The 1K resistor is series with the variable resistor is there so that in the event that the variable resistor is set to 0 ohm, there would still be some resistance between pin 6 and pin 7, otherwise the discharge time will theoretically be zero(the capacitor would immediately discharge every time the voltage at node ‘b’ hits 3.3V), keeping the timer’s output constantly on.

In this astable configuration of the 555, a variable resistor, paired with a fixed resistor, is typically inserted between node 'a' and 'b' to provide control over the clock rate. It influences the time constant of the RC circuit, thereby adjusting the frequency of oscillation.

The inclusion of a 1K resistor in series with the variable resistor ensures that a minimal level of resistance is maintained even if the variable resistor is dialed down to 0 ohms. Without this safeguard, the absence of resistance would result in an instantaneous discharge of the capacitor every time the voltage at node ‘b’ reaches 3.3V, causing the timer's output to remain perpetually active.

![3.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/3%201.png)

*Let the topmost resistor be Ra and the variable resistor plus the series minimal resistor next to it be Rb.*

The charging time for the capacitor exceeds the discharging time because the capacitor charges through both Ra and Rb, but only discharges through Rb. Ra doesn’t participate in the discharge path. As a result, the duty cycle exceeds 50%, with the exact excess percentage depending on the value of  Ra(The higher the value of Ra the higher above 50% the further above 50% the duty cycle will be, and vice versa.)

Achieving a precise 50% duty cycle isn’t feasible in this setup without modifications. A zero-ohm Ra would technically yield a 50% duty cycle, but it's impractical as it would create a direct short from VCC to ground via discharge pin 7.

However, adding a bypass diode between pin 7 and pin 6 can allow duty cycles of 50% and below. In the context of my project, the slightly imbalanced duty cycle resulting from the absence of such a modification is acceptable for my needs.

![555_astable_5.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/555_astable_5.png)

The 555 datasheet suggests adding a 10nf capacitor between pin 5(The control voltage pin) and ground, to reduce noise. As can be seen from the schematic, pin 5 is directly connected to the top node of the voltage divider, thereby setting the reference voltage for the comparator. Attaching a bypass capacitor to this pin helps keep the voltage level steady, making the timer more reliable.

More detail about bypass capacitors can be found in my reference post.

## Monostable Multivibrator with 555

The astable output on the debbuging clock is used as the ongoing clock would: To keep the CPU running continuously. However, when debugging, being able to trigger the clock pulses manually is key. 

A push button, by itself, can be used to the tie the reference voltage to the clock node/input of the CPU. The problem here is that when pressing the button we can unexpectedly make it close and open its switch more times than intended. Think about wristwatches with clicky buttons, where a single button press can sometimes result in multiple increments in the displayed time due to mechanical bounces of the button's internal contacts.

To mitigate this, we can use the 555 timer in a monostable configuration. In this setup, a trigger causes the output to change state for a specified period(Staying at that state no matter what happens to the trigger during the time period) before returning to its original state. This helps ensure each button press results in a single, clean clock cycle, despite any bouncing.

### Working Principle

When the push button is not being pressed, C2 stays off because of the reference voltage at pin 2 (through the 1K pull up resistor), and the latch stays on its reset sate with the “not” output high. This turns the transistor on and discharges the capacitor.

Upon pressing the button, pin 2 essentially gets connected to ground, activating C2. With C2 on, the output goes high, and the transistor turns off, initiating the capacitor's charging process. 

The cycle resets when the capacitor’s voltage exceeds 3.3V, turning on C1 and resetting the latch. 

This turns the transistor on and restarts the cycle.

![5.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/5.png)

This mechanism ensures that each button press results in a single, clean transition of the clock cycle, effectively mitigating the issue of button bounce.

![555_monostable_2.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/555_monostable_2.png)

## Bistable 555 Debouncer and Switching Logic

### Bistable Debouncer

Just as we could have used a push button by itself to trigger clock pulses, we could use a simple single-throw-double-throw switch, to switch between the two multivibrators outputs, as follows:

![simple_clock_switch.jpg]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/simple_clock_switch.jpg)

But just like push buttons, some switches can also bounce. My switch, just like the one in the original 8-bit CPU kit, is a Break-Before-Make switch(A concept I heard of for the first time while assembling the kit). 

The image below shows what happens within a single-pole-double-throw Break-Before-Make switch.

![credit:(What is a break-before-make circuit?) [toshiba.semicon-storage.com](http://toshiba.semicon-storage.com/)]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/break_before_make_switch.png)

credit:(What is a break-before-make circuit?) [toshiba.semicon-storage.com](http://toshiba.semicon-storage.com/)

It is clear that the pole within the switch can bounce with the last throw it made contact with due to slight vibrations during the switching process.

As mentioned early in this post, an SR latch is a bistable multivibrator by definition, and **its output** can be used with the switch through a switch-select logic to select(more so than switch) between the astable and monostable clock. 

![Bistable Debouncing Mechanism. Base SR Latch image from: Animated SR latch, By Napalm Llama, Wikimedia Commons]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/sr_latch_debouncer.png)

Bistable Debouncing Mechanism. Base SR Latch image from: Animated SR latch, By Napalm Llama, Wikimedia Commons

Ben Eater used a bistable configuration of the 555 timer to implement this debouncer just to expose learners to another use of the chip. The only component of the 555 timer used here is its SR latch, and in fact, an SR latch IC could have been used. 

An advantage that I initially personally found in using the 555 instead of an SR latch IC was that it’d have saved me room on the clock breadboard as I initially intended to use my debugging clock as my main clock, and I wanted to pack as many chips as possible on the same breadboard.

![555_bistable.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/555_bistable.png)

With this configuration, even if the switch bounces between a make(closed) and a floating state(break), it won’t change the output of the latch, because anytime the switch goes into one of the two make states, the latch will hold the output for that specific state, and won’t change unless and only unless the switch goes into the opposite make state.

In other words, unless the opposite input(Trigger or Reset) is grounded, which does not happen until the switch is fully actuated, the output of the latch is not going to change even if the current input is being toggled! 

### Switching Logic

Now that all the parts of the clock are done; we need a way to join them together. The goal is to have the following:

- A halt input that as its name implies, halts or stops the clock. This is useful to automatically stop the clock after executing a program.
- A select input, tied to the bistable output, to switch between the astable and monostable pulses.

The logic below is the one built that the original kit follows.

![3_ICs_clock_logic.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/3_ICs_clock_logic.png)

For the sake of saving room on the breadboard, I implemented the logic with NAND gates as it required only two ICs, instead of four with the implementation above.

![2_ICs_Clock_logic.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/2_ICs_Clock_logic.png)

![debugging_clock.png]({{ site.url }}{{ site.baseurl }}/assets/img/posts/8-bit_bb_cpu/clock/debugging_clock.png)

## Main Clock

******************************************Control lines involved:******************************************

- HLT (Halt |←)
- CLK (Clock | —> )
- ~CLs (Clockspeed Change)
- 

My main clock only uses an Astable multivibrator running at xxMH. The reason for this exact frequency will be discussed below.

Two 4-bit counters to divide the clockspeed in 8

1- 1x **LMC555CN CMOS Single 555 Timer Low Power DIP-8** ([Jameco](https://www.jameco.com/webapp/wcs/stores/servlet/ProductDisplay?langId=-1&storeId=10001&catalogId=10001&productId=126797), [Datasheet](https://www.jameco.com/Jameco/Products/ProdDS/126797Philips.pdf))

2- 1x 74HCT251 High-Speed CMOS Logic 8-Input Multiplexer ([Digikey](https://www.digikey.com/en/products/detail/texas-instruments/CD74HCT251E/38458), [Datasheet](https://www.ti.com/general/docs/suppproductinfo.tsp?distId=10&gotoUrl=https%3A%2F%2Fwww.ti.com%2Flit%2Fgpn%2Fcd74hc251))

3- 2x 74HCT161, Synchronous 4-Bit Binary Counter, ([Digikey](https://rocelec.widen.net/view/pdf/yyfxbvvpnb/cd54hc161.pdf?t.download=true&u=5oefqw), [Datasheet](https://www.jameco.com/Jameco/Products/ProdDS/46818.pdf)) .

4- 1x 74HCT173, 4-Bit D-type Registers with tri-state Outputs, ([Digikey](https://www.digikey.com/en/products/detail/texas-instruments/CD74HCT173E/38365), [Datasheet](https://www.ti.com/general/docs/suppproductinfo.tsp?distId=10&gotoUrl=https%3A%2F%2Fwww.ti.com%2Flit%2Fgpn%2Fcd74hc173)).

5- 1x 74HCT74 Dual D-Type Positive-Edge-Triggered Flip-Flops With Clear and Preset([Digikey](https://www.digikey.com/en/products/detail/texas-instruments/CD74HCT74E/38762), [Datasheet](https://www.ti.com/general/docs/suppproductinfo.tsp?distId=10&gotoUrl=https%3A%2F%2Fwww.ti.com%2Flit%2Fgpn%2Fcd74hc74)) **(Shared with:)**

6- 1x 74HCT02 Quadruple 2-Input Positive-NOR Gates ([Digikey](https://www.digikey.com/en/products/detail/texas-instruments/CD74HCT02E/38234), [Datasheet](https://www.ti.com/lit/ds/symlink/cd74hc02.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1687139484461&ref_url=https%253A%252F%252Fwww.ti.com%252Fgeneral%252Fdocs%252Fsuppproductinfo.tsp%253FdistId%253D10%2526gotoUrl%253Dhttps%253A%252F%252Fwww.ti.com%252Flit%252Fgpn%252Fcd74hc02)) **(Shared with:)**


<i class="fas fa-calendar-alt"></i> <span style="font-size: 15px; font-weight: bolder;">Updated:  </span><time>August 24, 2023</time>