# Presentation Structure 

## Problem 

Look at this product <Google.Nest.Protect.Image> 
It's a smart smoke detector by Google that runs on battery. 

Its basic functionality is 

- Connect To Wifi
- Detect Smoke
- Sends A Notification to your phone 

It's valued at 120$ at Google Store, Fare Deal Right? No Problem
The problem really shows up when the battery dies. 
you can't actually change the battery nor charge it. 

So this battery better dies after u. 

Let's put google's engineers' shoes and think about it as an engineering problem

- We want to make this device highly energy efficient
- so let's see what is taking up more energy 
- The real power drainer of this device actually is the Radio Device 

So we came up with IEEE 802.15.4 standard. It's actually very low on power usage but we found out that this tradeoffs two things `DateRate is only up to 250 kbps` which is a problem but not here second `Range is roughly 10 meter` which is a big issue here since you have a big house 

People tried to came up with other solutions but they all stood usless against this fundamental tradeoff 

"""
The more transmission power you use, the longer your range. But more power drains your battery faster. You cannot have both high energy efficiency AND high data rates `with a single radio`.
"""

So okay let there be no single radio -> meme (Married Radios)

## WUR Radio Solution (High Level Idea)

---

Our device already have a radio its c/cs are 

- High Speed Data Transmittion
- High Power Consumption
- High Range 

Let's call it (PCR) Primary Connectivity Radio

---

Since it's taking up so much energy we need it not to be awake all the time and go to sleep to save energy but who will wake it up?

--- 

Let's give it a Companion its c/cs are 

- Extremely Low Power (less than 1 mW)
- Always awake
- Only a receiver 
- Wakes up PCR when someone calls PCR 

Let's call it (CCR) Companion Connectivity Radio 

--- 

Let's describe how they work together

Scene Description 

```
1 AP (Access Point) -> Wifi Router
1 STA (Station) -> Your Device with 2 radios
```

Script 

1. The Access Point has data to send to your device but as we know your PCR is asleep
2. The AP sends a special wake-up frame to the WUR
3. The WUR, which is always listening, detects this frame
4. The WUR physically signals the PCR: "wake up, data is coming"
5. The PCR powers on, sends a response to the AP saying "I'm ready"
6. The AP sends the actual data to the PCR
7. After receiving, the PCR goes back to sleep

## Standard 

The WUR radio is actually standardized at IEEE 802.11ba 

- First draft was released in 2017
- 2018-2020 Iterations on draft
- 2021: official release 

Now let's get full technical and get to the details

### Wake-Up Procedure 

We already tackled this 

### Duty Cycle 

Let's put the engineer's shoes again and think how can we make this standard more power efficient.

The solution lies in letting the poor `WUR radio` get some sleep instead of being awake all the time. This trick is called `duty cycling`

For the WUR radio `a day` is called `duty cycle` and it's divided into `awake period` and `asleep period` just like anyone of us

===========================================
                                          
<----Duty Cycle-----> <----Duty Cycle-----> ....  
<-awake-><-sleeping-> <-awake-><-sleeping-> ....  
                                          
===========================================


But of course the AP need to know when you are up to know when to send you WakeUp Frames 
so some info need to be more communicated between ur device and the AP 

Note `no one wakes the WUR up these wakeup frames are for the PCR`

To proberly setup Duty Cycling these 4 parameters need to be communicated between the STA and AP

1. Minimum Wake-Up Time: The minimum duration the WUR must stay awake during a duty cycle
2. Duty Cycle Period: How long is one work day
3. Length of the "Awake" Period: How long does the WUR stay awake during a work day 
4. Starting Time of The "Awake" Period 

`The Negotiation Process` is as follows 

1. The AP first advertise its `Minimum Wake-Up Time` requirement
2. The STA replies with its desired `Duty Cycle Period` and `Length of the "Awake" Period`
3. The AP replies with the `Starting Time of The "Awake" Period` 
4. Negotiation Complete

Quick Quiz:

```

- Which radio is responsible for the duty cycle negotiation process?

1. Companion Connectivity Radio (WUR)
2. Primary Connectivity Radio (PCR)

```

### WUR Beacon 

Let's forget about WUR for a second 

in normal 802.11, AP used to broadcast periodically `beacon frames` 
`beacon frames` contain a 64-bit timestamp. These frames are useful for two reasons.

- Help synchronize all devices to a common clock.
- It act as heartbeat, it tells the device `you are still in range and connected to the AP`

In WUR, Your radio is no longer awake so it loses its synchronization and awareness of whether its in range or not 

Solution: Introduce a new type of frame called `WUR Beacon` 

- The AP periodically transmits WUR beacon frames
- This does the "heartbeat" function.
- It also carries synchronization timestamps but not 64-bit becuase it will take 1.024 ms to transmit and the WUR needs to sleep 

### Re-Discovery 

Let's again forget about WUR for a second 

in normal 802.11, If your devide stopped recieving `beacon frames`, It assumes it has become out of range and It needs to find a new AP. This process is called Rediscovery 

**Normal 802.11 Re-Discovery:**

1. Scan ALL available channels
2. Find one or more APs
3. Authenticate 
4. Assocate with a BSS

this proces is extremely power hungry because the device has to listeon on every channel for `beacon frames` from potential APs

- Let's put engineer's shoes and try to make this process power efficient 

Instead of all this channels we will dedicate a channel only for discovery 
All APs will send a newly introduced frame called `WUR Discovery Frame` 

A `WUR Discovery Frame` contain 

- AP's address
- AP's operating channel 

This way your device will only monitor this one channel and when it find a suitable AP it will send authentication and association frames 


Quick Quiz:

```

- Which is responsible for which?

1. WUR is responsbile for monitoring and authentication
2. PCR is responsbile for monitoring and authentication
3. WUR is responsbile for monitoring and PCR is reponsbile for authentication
4. PCR is responsbile for monitoring and WUR is reponsbile for authentication

```





