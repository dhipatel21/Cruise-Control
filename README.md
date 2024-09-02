## Adaptive Cruise Control with Haptic Wheel

## Introduction
Our project was creating a Simulink model that closely models a simple vehicle with an autonomous cruise control system. It was a Hardware in the Loop simulation using a Haptic wheel and Simulink-generated C (simulation built by staff in Unreal Engine). We used a bicycle model for vehicle dynamics and linearized the non-linear components of a true vehicle for sake of simplicity. Basically, the car is supposed to have three different modes, manual, position control, and velocity control, and is supposed to autonomously steer and autonomously switch between them while on the road and communicating with other students’ vehicles. All communication is done via CAN, with other cars’ positions and speeds all periodically updated on the CAN bus. The pick lead logic allows us to determine whether we are the lead car, as well as if our automatic cruise control should be in position or velocity mode, given that automatic cruise control is enabled. We began by setting our initial min<sub>dis</sub> value to 1,000,000, and looping through the other cars to see if anyone was in front. If the positive distance to another vehicle is less than our min<sub>dis</sub>, then we update the min<sub>dis</sub> and save the index of the car. Upon reaching the end of the loop, we either have the minimum distance and the car closest to and in front of us, or we still have a value of 1,000,000, indicating we are the lead car, in which case we use the velocity control.

Haptic Wheel to Control the Car in the Simulation

<img src="https://github.com/dhipatel21/Cruise-Control/blob/cc176974310d9b9b154a7a48cffe3c63b319f5a4/Haptic_Wheel.png" alt="drawing" width="400"/>

Car Simulation in Unreal Engine

<img src="https://github.com/dhipatel21/Cruise-Control/blob/cc176974310d9b9b154a7a48cffe3c63b319f5a4/acc.png" alt="drawing" width="400"/>

## Automatic Steering Controllers
The automatic steering implementation is a subsystem consisting of two cascading discrete PD controllers, an outer loop and inner loop controllers. Both the outer and inner loop implementations of discrete PD controllers followed the same simple PD design with varying inputs, and Kp and Kd values. The inputs for the outer and inner loops are n and delta, respectively. The input n is the summation of n, the output from the Vehicle Dynamics subsystem, and n<sub>desired</sub>, which was set to 0 to indicate we want to stay in the middle of the road (the double white line in between the wheels of the vehicle). We used an adder since our steering was reversed, which was addressed in our parameters.m file with a negative sign in front of R. The input delta is the subtraction of the Steering Angle input from the Inputs subsystem from the output of the outer loop. As we only used PD controllers, we only had to tune four variables, S<sub>Kd1</sub>, S<sub>Kp1</sub>, S<sub>Kd2</sub>, S<sub>Kp2</sub>, where S<sub>Kd1</sub> and S<sub>Kp1</sub> correspond to the gains for the outer loop controller, and S<sub>Kd2</sub> and S<sub>Kp2</sub> correspond to the gains for the inner loop controller. We tuned our gain values by solving for S<sub>Kd1</sub> and S<sub>Kp1</sub> with a starting point of w<sub>n</sub> = 10 and zeta = 0.7, and equating the coefficients, where u is our desired speed (25 m/s for the purpose of the demo). Solving for S<sub>Kd2</sub> and S<sub>Kp2</sub> followed a similar set up; however, to incorporate the self-aligning torque we used different coefficients with the same zeta = 0.7, but different w<sub>n</sub> = 30:

<img src="https://github.com/dhipatel21/Cruise-Control/blob/cc176974310d9b9b154a7a48cffe3c63b319f5a4/PD_Equations.png" alt="drawing" width="400"/>


Upon inspection of the vehicle, we saw that the haptic wheel jittered on turns and the vehicle did not completely stay in the middle of the road. Our first change was to decrease zeta to 0.5. We noticed that the car moved a bit smoother; however, it still had jitters and was not fully centered. From there we incrementally increased (based on the scale of the gain values) the S<sub>Kd1</sub>, S<sub>Kp1</sub>, S<sub>Kd2</sub>, and S<sub>Kp2</sub> values until the car consistently moved in the middle of the road with no jittering, since we were already relatively close to the desired behavior. For example, since S<sub>Kd1</sub> and S<sub>Kp1</sub> were initially really small, less than 0.03 and 0.3, we increased the gains by 0.01 and 0.369, respectively; whereas S<sub>Kd2</sub> and S<sub>Kp2</sub> were initially really large, around 95 and 575, we increased the gains by 1 and 1, respectively. Since we were quite close to the desired behavior of the automatic steering control, we determined that less tuning was required; however, this small increase in gains helped the consistency.

Final gains were S<sub>Kp1</sub> = 0.625, S<sub>Kd1</sub> = .0356, S<sub>Kp2</sub> = 574.45 , and S<sub>Kd2</sub> = 97.45.

The outer loop, representing the first of our two discrete PD controllers in our automatic steering control system.

<img src="https://github.com/dhipatel21/Cruise-Control/blob/cc176974310d9b9b154a7a48cffe3c63b319f5a4/Outer_PD.png" alt="drawing" width="400"/>



The inner loop, representing the second of our two discrete PD controllers in our cascading PD control system for the automatic steering control system.                   
<img src="https://github.com/dhipatel21/Cruise-Control/blob/cc176974310d9b9b154a7a48cffe3c63b319f5a4/Inner_PD.png" alt="drawing" width="400"/>
