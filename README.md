
# Main-Engine-Control-System
By: Christopher Andersen

This project is the culmination of my efforts to re-create the original Tacho System developed by MAN B&W for ME main engines with my own input.

The main engine on the MV California is an electronically controlled two-stroke, straight six, direct reversible, crosshead type diesel engine with constant pressure turbocharging and air cooling.

In this instance, the absence of a camshaft is what interests me about this configuration. The role of the camshaft within the main engine is to mechanically determine the main engine cylinder and exhaust port timing. An ME engine completely replaces this mechanical system with a combination of FIVA (Fuel Injection Valve Actuation) valves, and the Tacho System. 

The hydraulic oil pumped through the FIVA valves by the hydraulic power supply uses a six micrometer autofilter with a backflush initiated every 30-120 minutes. The hydraulic oil is same oil as the main engine lube oil supply transported by three axial piston pumps with swashplates. The average pressure in the hydraulic system while under load is 210-300 Bar. There is a 4th hydraulic pump in this engine setup, a fixed displacement pump designed only to run at 85% capacity for emergency purposes only. The pumps are designed to share load when one pump reaches above 90% load. If a pump reaches this load amount, a second pump will activate, splitting the load until both pumps reach 90%, repeating the splitting cycle until the system reaches the fixed displacement pump.

The Tacho System is a measurement system for engine speed and crankshaft position for the control of events in the main engine. This system is an electrohydraulic replacement for the camshaft that utilizes eight sensors across two systems that measure degrees on a 360 tooth gear. To find the position of the gear, the system utilizes four sensors: Marker Master A (MMA), Marker Slave A (MSA), Quadratur 1A (Q1A), Quadratur 2A (Q2A), Marker Master B (MMB), Marker Slave B (MSB), Quadratur 1B (Q1B), and Quadratur 2B (Q2B).

Sensor truth table:
```
Pos   0-44 45-89 90-134 135-179 180-224 225-269 270-314 315-359
---------------------------------------------------------------
MMA    1     1     1       1       0       0       0       0
MMB    0     1     1       1       1       0       0       0
MSA    0     0     1       1       1       1       0       0
MSB    0     0     0       1       1       1       1       0
```
For my replication of this system, I take advantage of the 8 divisions of the flywheel to create a finite state machine with eight states (3 bits). For this construction I use JK flip flops in my state memory portion instead of D flip flops due to the simplification of the output logic. I also believe that if there were to be more states required for an engine with more cylinders, JK flip flops would be necessary due to the reduction in circuit complexity.

Side note: I believe it is possible to make this system modular for engines with a variable amount of cylinders. How this could be done is by writing a combined boolean function that will calculate all future logic statements as more cylinders are added. The reason I believe this is possible is because there is a relationship between the cylinder count and the amount of states needed on the flywheel to accommodate these cylinders. If the amount of cylinders increases, so must the resolution of the sensor on the flywheel, which requires more states. I believe this relationship is exponential but the issue holding me back is I have no idea how to write a cumulative boolean statement for this eight state system and form a function out of it.

When designing this system, there are two schools of thought: either build a sensor that would directly connect the flywheel to the hydraulic operating valves as MAN B&W had done, or electronically emulate the flywheel and with a digital replica that simulates the existence of a flywheel (the flywheel is still required to exist physically for momentum purposes) and the camshaft at the same time. For this demonstration, I selected option two.

The three inputs to my system are a forward/reverse input from the engine order telegraph and the engine speed defined as clock speed. The outputs are the state (or position) that the flywheel is in, and the direction of spin of the propeller. As an example, S0 corresponds to position 0-44 degrees on the flywheel, S1 corresponds to position 45-89 degrees on the flywheel, etc. Below is the truth table for this system along with the state diagram:

```
           Current state      | Input |                 Next state              |              Output
Pos.     Q2_cur Q1_cur Q0_cur |  F R  | Q2_nxt J2 K2 Q1_nxt J1 K1 Q0_nxt J0 K0  |  CW CCW S0 S1 S2 S3 S4 S5 S6 S7
------------------------------|-------|-----------------------------------------|--------------------------------
0-44        0      0     0    |  0 0  |    0    0  x    0    0  x    0    0  x  |   0  0   1  0  0  0  0  0  0  0	
            0      0     0    |  0 1  |    1    1  x    1    1  x    1    1  x  |   0  1   0  0  0  0  0  0  0  1
            0      0     0    |  1 0  |    0    0  x    0    0  x    1    1  x  |   1  0   0  1  0  0  0  0  0  0
            0      0     0    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
------------------------------|-------|-----------------------------------------|--------------------------------
45-89       0      0     1    |  0 0  |    0    0  x    0    0  x    1    x  0  |   0  0   0  1  0  0  0  0  0  0
            0      0     1    |  0 1  |    0    0  x    0    0  x    0    x  1  |   0  1   1  0  0  0  0  0  0  0
            0      0     1    |  1 0  |    0    0  x    1    1  x    0    x  1  |   1  0   0  0  1  0  0  0  0  0
            0      0     1    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
------------------------------|-------|-----------------------------------------|--------------------------------
90-134      0      1     0    |  0 0  |    0    0  x    1    x  0    0    0  x  |   0  0   0  0  1  0  0  0  0  0
            0      1     0    |  0 1  |    0    0  x    0    x  1    1    1  x  |   0  1   0  1  0  0  0  0  0  0
            0      1     0    |  1 0  |    0    0  x    1    x  0    1    1  x  |   1  0   0  0  0  1  0  0  0  0
            0      1     0    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
------------------------------|-------|-----------------------------------------|--------------------------------
135-179     0      1     1    |  0 0  |    0    0  x    1    x  0    1    x  0  |   0  0   0  0  0  1  0  0  0  0
            0      1     1    |  0 1  |    0    0  x    1    x  0    0    x  1  |   0  1   0  0  1  0  0  0  0  0
            0      1     1    |  1 0  |    1    1  x    0    x  1    0    x  1  |   1  0   0  0  0  0  1  0  0  0
            0      1     1    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
------------------------------|-------|-----------------------------------------|--------------------------------
180-224     1      0     0    |  0 0  |    1    x  0    0    0  x    0    0  x  |   0  0   0  0  0  0  1  0  0  0
            1      0     0    |  0 1  |    0    x  1    1    1  x    1    1  x  |   0  1   0  0  0  1  0  0  0  0
            1      0     0    |  1 0  |    1    x  0    0    0  x    1    1  x  |   1  0   0  0  0  0  0  1  0  0
            1      0     0    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
------------------------------|-------|-----------------------------------------|--------------------------------
225-269     1      0     1    |  0 0  |    1    x  0    0    0  x    1    x  0  |   0  0   0  0  0  0  0  1  0  0
            1      0     1    |  0 1  |    1    x  0    0    0  x    0    x  1  |   0  1   0  0  0  0  1  0  0  0
            1      0     1    |  1 0  |    1    x  0    1    1  x    0    x  1  |   1  0   0  0  0  0  0  0  1  0
            1      0     1    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
------------------------------|-------|-----------------------------------------|--------------------------------
270-314     1      1     0    |  0 0  |    1    x  0    1    x  0    0    0  x  |   0  0   0  0  0  0  0  0  1  0
            1      1     0    |  0 1  |    1    x  0    0    x  1    1    1  x  |   0  1   0  0  0  0  0  1  0  0
            1      1     0    |  1 0  |    1    x  0    1    x  0    1    1  x  |   1  0   0  0  0  0  0  0  0  1
            1      1     0    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
------------------------------|-------|-----------------------------------------|--------------------------------
315-359     1      1     1    |  0 0  |    1    x  0    1    x  0    1    x  0  |   0  0   0  0  0  0  0  0  0  1
            1      1     1    |  0 1  |    1    x  0    1    x  0    0    x  1  |   0  1   0  0  0  0  0  0  1  0
            1      1     1    |  1 0  |    0    x  1    0    x  1    0    x  1  |   1  0   1  0  0  0  0  0  0  0
            1      1     1    |  1 1  |    x    x  x    x    x  x    x    x  x  |   x  x   x  x  x  x  x  x  x  x
```
State diagram:

![State Diagram](/images/State-diagram.jpg "State Diagram")

Assumtions: 

There will not be a forward and reverse input given in the same moment because the engine order telegraph (EOT) can only give one input at a time.

The system will initialize with the flywheel in state 000 (S0) or between 0-44 degrees for startup purposes only. After the system has been initialized, the digital flywheel will always record and remember its position.

During engine startup, the RPM input for the clock will run off a script with a set RPM increase for consistency. After engine startup the clock will be influenced by the actual RPM of the propulsion shaft in real time.


----------------------------------------------------------------------------------------------------------------


Karnaugh maps derived from the truth table in sum of products form:

Vertical axis in order of bits: F, R

Horizontal axis in order of bits: Q2_cur, Q1_cur, Q0_cur

J2: 
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  0  |  0  |  x  |  x  |  x  |  x  |
 01 |  1  |  0  |  0  |  0  |  x  |  x  |  x  |  x  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  0  |  1  |  0  |  x  |  x  |  x  |  x  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
J2 = (!Q1_cur)(!Q0_cur)R + (Q1_cur)(Q0_cur)F

K2:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  x  |  x  |  x  |  x  |  0  |  0  |  0  |  0  |
 01 |  x  |  x  |  x  |  x  |  0  |  0  |  0  |  1  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  x  |  x  |  x  |  x  |  0  |  1  |  0  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
K2 = (!Q1_cur)(!Q0_cur)R + (Q1_cur)(Q0_cur)F

J1:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  x  |  x  |  x  |  x  |  0  |  0  |
 01 |  1  |  0  |  x  |  x  |  x  |  x  |  0  |  1  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  1  |  x  |  x  |  x  |  x  |  1  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
J1 = (!Q0_cur)R + (Q0_cur)F

K1:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  x  |  x  |  0  |  0  |  0  |  0  |  x  |  x  |
 01 |  x  |  x  |  0  |  1  |  1  |  0  |  x  |  x  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  x  |  x  |  1  |  0  |  0  |  1  |  x  |  x  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
K1 = (!Q0_cur)R + (Q0_cur)F

J0:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  x  |  x  |  0  |  0  |  x  |  x  |  0  |
 01 |  1  |  x  |  x  |  1  |  1  |  x  |  x  |  1  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  1  |  x  |  x  |  1  |  1  |  x  |  x  |  1  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
J0 = F + R

K0:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  x  |  0  |  0  |  x  |  x  |  0  |  0  |  x  |
 01 |  x  |  1  |  1  |  x  |  x  |  1  |  1  |  x  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  x  |  1  |  1  |  x  |  x  |  1  |  1  |  x  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
K0 = F + R

S0:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  1  |  0  |  0  |  0  |  0  |  0  |  0  |  0  |
 01 |  0  |  1  |  0  |  0  |  0  |  0  |  0  |  0  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  0  |  0  |  0  |  0  |  1  |  0  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S0 = (!Q2_cur)(!Q1_cur)(!Q0_cur)(!F)!R + (!Q2_cur)(!Q1_cur)(Q0_cur)R + (Q2_cur)(Q1_cur)(Q0_cur)F

S1:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  1  |  0  |  0  |  0  |  0  |  0  |  0  |
 01 |  0  |  0  |  0  |  1  |  0  |  0  |  0  |  0  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  1  |  0  |  0  |  0  |  0  |  0  |  0  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S1 = (!Q2_cur)(!Q1_cur)(Q0_cur)(!F)!R + (!Q2_cur)(Q1_cur)(!Q0_cur)R + (!Q2_cur)(!Q1_cur)(!Q0_cur)F

S2:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  0  |  1  |  0  |  0  |  0  |  0  |
 01 |  0  |  0  |  1  |  0  |  0  |  0  |  0  |  0  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  1  |  0  |  0  |  0  |  0  |  0  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S2 = (!Q2_cur)(Q1_cur)(!Q0_cur)(!F)!R + (!Q2_cur)(Q1_cur)(Q0_cur)R + (!Q2_cur)(!Q1_cur)(Q0_cur)F

     
S3:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  1  |  0  |  0  |  0  |  0  |  0  |
 01 |  0  |  0  |  0  |  0  |  0  |  0  |  0  |  1  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  0  |  0  |  1  |  0  |  0  |  0  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S3 = (!Q2_cur)(Q1_cur)(Q0_cur)(!F)!R + (Q2_cur)(!Q1_cur)(!Q0_cur)R + (!Q2_cur)(Q1_cur)(!Q0_cur)F
     
S4:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  0  |  0  |  0  |  0  |  0  |  1  |
 01 |  0  |  0  |  0  |  0  |  0  |  0  |  1  |  0  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  0  |  1  |  0  |  0  |  0  |  0  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S4 = (Q2_cur)(!Q1_cur)(!Q0_cur)(!F)!R + (Q2_cur)(!Q1_cur)(Q0_cur)R + (!Q2_cur)(Q1_cur)(Q0_cur)F
     
S5:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  0  |  0  |  0  |  0  |  1  |  0  |
 01 |  0  |  0  |  0  |  0  |  1  |  0  |  0  |  0  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  0  |  0  |  0  |  0  |  0  |  0  |  1  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S5 = (Q2_cur)(!Q1_cur)(Q0_cur)(!F)!R + (Q2_cur)(Q1_cur)(!Q0_cur)R + (Q2_cur)(!Q1_cur)(!Q0_cur)F     

S6:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  0  |  0  |  1  |  0  |  0  |  0  |
 01 |  0  |  0  |  0  |  0  |  0  |  1  |  0  |  0  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  0  |  0  |  0  |  0  |  0  |  1  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S6 = (Q2_cur)(Q1_cur)(!Q0_cur)(!F)!R + (Q2_cur)(Q1_cur)(Q0_cur)R + (Q2_cur)(!Q1_cur)(Q0_cur)F

S7:
```
  \   000   001   011   010   110   111   101   100
   \ _______________________________________________
 00 |  0  |  0  |  0  |  0  |  0  |  1  |  0  |  0  |
 01 |  1  |  0  |  0  |  0  |  0  |  0  |  0  |  0  |
 11 |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |
 10 |  0  |  0  |  0  |  0  |  1  |  0  |  0  |  0  |
     ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅ ̅
```
S7 = (Q2_cur)(Q1_cur)(Q0_cur)(!F)!R + (!Q2_cur)(!Q1_cur)(!Q0_cur)R + (Q2_cur)(Q1_cur)(!Q0_cur)F

Circuit diagram modeled here: https://www.multisim.com/content/Kr2qwFthUyTpyucijUgyRN/tacho-system/open/

Circuit diagram:

![Circuit Diagram](/images/Circuit-diagram.png "Circuit Diagram")

The LEDs represent the output states and serve as a visual representation of what happens on the flywheel. These LEDs are hardwired to a FIVA valve for a cylinder, hypothetically, because the injection and exhaust port timing per state is not known. As an example, the firing order of this engine is 1-5-3-4-2-6. Relating the firing order to the state of the cylinder in the diesel cycle (which is unknown, this is hypothetical), we can say while the flywheel is within 0-44 degrees (S0), cylinder 1 is injecting fuel into the cylinder, cylinder 6 has just combusted its fuel and is on the start of its power stroke, cylinder 2 is halfway through its power stroke with the exhaust valves starting to open, cylinder 4 is at bottom dead center with its scavenge air valves open, cylinder 3 is halfway through its compression stroke, closing its exhaust valves, and cylinder 5 is finishing its compression stroke, awaiting fuel injection.

