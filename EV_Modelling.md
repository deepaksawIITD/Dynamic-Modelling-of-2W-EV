## Input parameters of the vehicle
$
Kerb \: Weight = 108 \: kg \\
Passenger \: Mass = 80 \: kg \\
Vehicle \; accelerates \: 0-40 \: \frac{km}{hr} \: in \: 3.3 sec \\
Vehicle \: Velocity, v_{accel} = 40 \times  \frac{5}{18}  m/s\\
Vechicle \: Acceleation \: Time,t_{accel} = 3.3 s \\
Gradeability = 0 ^\circ
$


```python
import scipy as sp
import numpy as np
import math
m_kerb = 108;       # Kerb weight (Weight of vehicle in kg)
m_passenger = 80;   # Mass of the passenger (in kg)
v_accel = 40*5/18;  # Vehicle accelerates 0-40 km/hr in 3.3 sec (in m/s)
t_accel = 3.3;      # Vehicle acceleration time (in s) to reach 40 km/hr
gradeability = 0;   # Vehicle gradeability in degrees
```

## Tyre Size (90/90-12 tubeless for Ather 450x)


```python
wheel_dia = 12*2.54*10;         # Wheel diameter in mm
nominal_section_width = 90;     # Nominal section width of the tyre in mm
tyre_aspect_ratio = 0.9;        # Height of the tyre sidewall as percentage of the nominal section width
d_tyre = wheel_dia + 2*nominal_section_width*tyre_aspect_ratio; # diameter of the tyre (in mm)
r_tyre = d_tyre*0.5/1000;       # Radius of the tyre (in m)
print(r_tyre)
```

    0.2334
    

# Tractive Effort

## Calculation of Rolling Resistance Force

$ Coefficient \: of \: Rolling \: Resistance, \mu_{rr} = 0.005 \\
g=9.8 m/s^2 \\
Total\:Weight = Kerb\:Weight + Passenger\:Weight \\
Rolling\:Resistance = \mu_{rr} \times Total\:Weight \times g $


```python
mu_rr = 0.005;                          # mu_rr = coefficient of rolling resistance
g = 9.8;                                # acceleration of gravity (in m/s^2)
m = m_kerb + m_passenger;               # Total mass of vehicle and passanger (in kg)
Frr = mu_rr*m*g                         # Rolling resistance force (in N)
print(Frr)
```

    9.212000000000002
    

## Calculation of Aerodynamic Drag
$ Air \: Density, \rho = 1.25 \: kg/m^3 \\
Frontal \: Area, A = 0.6 \: m^2 \\
Drag \: Coefficient \: Constant, C_d = 0.7 \\
Aerodynamic \: Drag, F_{ad} = \rho \times A \times C_d \times V_{accel}^2 $


```python
ro = 1.25;          # density of the air (in kg/m^3)
A = 0.6;            # frontal area (in m^2)
Cd = 0.7;           # drag coefficient constant 
Fad = 0.5*ro*A*Cd*math.pow(v_accel,2)  # Aerodynamic drag (in N)
print(Fad)
```

    32.4074074074074
    

## Calculation of Hill Climbing Force
$ Upward \: Slope \: Angle, \psi = Gradeability \times \frac{\pi}{180} \: rad \\
Hill \: Climbing, F_{hc} = m \times g \times sin(\psi)  $


```python
psi = gradeability*math.pi/180;       # Upward slope angle (in rad)
Fhc = m*g*math.sin(psi);                    # Hill Climbing (in N)
print(Fhc)
```

    0.0
    

## Calculation of Acceleration Force

$ Linear \: Acceleration, a_{linear} = \frac {Vehicle \: Velocity} {Vehicle \: Acceleration \: Time} \\ 
Gear \: Ratio, G = 10 \\
Moment \: of \: Inertia, I = 0.025 \: kg.m^2 \\
Gear \: System \: Efficiency, \eta_g = 1 \\ \\
Force \: due \: to \: Linear \: Acceleration, F_{la} = m \times a_{linear} \\
Axle \: Torque = Tractive \: Effort \: Delivered \: by \: the \: Powertrain \times Radius \: of \: the \: tyre \\
Peak \: Torque = Axle \: Torque / Gear \: Ratio \\ \\
Tractive \: Effort \: Delivered \: by \: the \: Powertrain, F_{te} = G \times \frac{T_{peak}}{r_{tyre}} \\
Axle \: Angular \: Speed = \frac {Vehicle \: Velocity}{r_{tyre}} \: rad/s \\
Motor \: Angular \: Speed, \omega = G \times  \frac {Vehicle \: Velocity}{r_{tyre}} \: rad/s \\
N = \omega \times \frac {60}{2 \times \pi} rpm \\
Motor \: Angular \: Acceleration, \frac {d\omega}{dt} = G \times \frac {a_{linear}} {r_{tyre}} \: rad/s^2 \\
T_{peak} = I \times \frac{d \omega}{dt} \\
Force \: at \: the \: wheels \: needed \: to \: provide \: the \: angular \: acceleration, F_{\omega a} = I \times G^2 \times \frac {a_{linear}} {\eta_g \times r_{tyre}^2} 
$


```python
a_linear = v_accel/t_accel;        # linear acceleration (in m/s^2) (0-40km/hr in 3.3sec)
G = 10                             # gear ratio of the system connecting
                                   # the motor to the axle
I = 0.025;                         # moment of inertia of the rotor of the motor (kg.m^2)
eta_g = 1;                         # gear system efficiency
         
Fla = m*a_linear;                  # Force due to linear acceleration

# Fte*r_tyre                       # Axle torque 
# T_peak = Fte*r_tyre/G            # Motor torque
# Fte = G*T_peak/r_tyre            # tractive effort delivered by the powertrain
# v_accel/r_tyre                   # axle angular speed (in rad/s)

w = G*v_accel/r_tyre;              # motor angular speed (in rad/s)
N = w*60/(2*math.pi)                    # motor max speed (in rpm)
#w1 = G*a_linear/r_tyre;           # motor angular acceleration (rad/s^2)
#T_peak = I*w1;                    # Torque required for the angular acceleration "w1"
Fwa = I*G*G*a_linear/(eta_g*math.pow(r_tyre,2))     # Force at the wheels needed to provide the angular acceleration    
print(Fwa)
```

    154.51898828591058
    

## Calculation of Total Tractive Effort             
$
Total \: Tractive \: Effort = Rolling \: Resistance \: Force + Aerodynamic \: Drag + Hill \: Climbing \:Force + \\ Force \: due \: to \: Linear \: Acceleration + Acceleration \: Force \\ \\
F_{te} = F_{rr} + F_{ad} + F_{hc} + F_{la} + F_{\omega a}
$


```python
Fte = Frr + Fad + Fhc + Fla + Fwa;
print(Fte)
```

    829.135028689951
    

# Modelling the Vehicle Acceleration 
## Acceleration Performance Parameters
$
T_{peak} = F_{te} \times \frac {r_{tyre}}{G} \\
Peak \: Power \: at \: 25 \: Percent \: of \: Maximum \: Speed \\%
P_{peak} = T_{peak} \times \omega \times 0.25
$


```python
T_peak = Fte*r_tyre/G           # Peak Motor torque (Nm)
P = T_peak*w*0.25;
print(T_peak)
print(P)
```

    19.352011569623457
    2303.1528574720865
    
