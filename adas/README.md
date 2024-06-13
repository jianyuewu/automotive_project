# Advanced Driver Assistance System (ADAS)
Autonomous Driving need ADAS and safety systems. Contains:
Adaptive Cruise Control(ACC): maintian distance with previous car.
Vehicle Exit Alert: Open the door, will indicate if there is people beside the car.
Lane Departure Warning/Prevention: Warn the driver if leave the lane.
Also Rear Cross Traffic Alert(RCTA): Driver monitoring, Automatic parking and others.

# ADAS Working Flow
Sensors(Radar, Lidar, GPS) -> Processing Unit(Data fusion & decision making) -> Activation(Braking, steering).

# Autonomous Driving
ADAS -> Autonomous Driving: visual info + audio info + interfering the vehicle motion.
Normal drive -> Possbile collision: Active safety(ADAS), Passive safety, tertiary safety(Airbag).
SAE levels: 1 -> 5.

# Detection
## Algorithms
Detection algorithms should aim for higher recall at the cost of lower precision when detecting relevant objects if necessary, as safety is most important.
3D object detectors are trained to recognize and track both moving and static objects such as people, vehicles, and traffic signs.
Usually select CNN based algorithms, and combine different sensors' data from ADAS.

## Camera
Tesla is mainly using camera for autonomous driving.
Mainly detect: Any moving/ static objects. i.e. Traffic sign, road signal, lane.
Passive light sensors to produce digital images, need GPU to process images, so power usage is high.
Low cost, high availability.

## RADAR
RAdio Detection And Ranging.
Mainly detect: distance, angle, velocity.
Frequency(mm Wave): 24GHz, 76-77GHz, 77-81GHz.

## LiDAR
Light Detection And Ranging. Get reflection of the object. Generate point clouds.
Mainly detect: All the 360 degrees' surroundings, i.e. people, cars, with 3D environment.
Has higher resolution than Radar.

## Ultrasonic sensor
Use sound waves to detect, cheapest sensor.
Mainly detect: Car parking, distance to wall.

## GPS
Mainly use GNSS(GPS, BeiDou, Galileo, GLONASS, QZSS)
IMU: Inertial Measurement Unit.
Mainly detect: Car position.

# ADAS testing
Requirement -> development -> functional test
CARLO: Driver, sensors, traffic, realistic vehicle dynamics.
VTD(Virtual Test Drive).
ADTF(Automotive Data & Time framework).
ROS(Robot Operating System).

SIL(Software In Loop): App + Software in loop simulation. Scene + Sensor + SW (Embeded C/C++) + Output.
HIL(Hardware In loop): Use real HW and DYNA4 Studio, vTESTstudio. Environment + HW (Sensors + Camera + ADAS ECU).
DIL(Driver In Loop): Claytex, i.e. Braking test.

# References
udemy.com: Advanced driver assistance systems.
https://www.classcentral.com/course/youtube-advanced-driver-assistance-system-every-adas-levels-in-car-explained-144728
https://www.classcentral.com/course/youtube-adasa-a-conversational-in-vehicle-digital-assistant-for-advanced-driver-assistance-features-166932
