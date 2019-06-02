This page contains informaiton about the software and hardware components of our project:

## Software Components

Component | Description | Link
--- | --- | ---
Sensor Controller | Raspberry Pi which runs a custom python program to advertise connected sensor values by operating as a Bluetooth Low Energy peripheral | [Link](https://github.com/dangpg/BikeVR-Sensor-Controller)
Unity Framework BikeVR | C# Unity Framework which enables to read the sensor controller's values on a mobile platform (Android, IOS) | [Link](https://github.com/dangpg/BikeVR-Unity-Framework)
Google VR Application | Sample Google Cardboard Unity project which combines all components to form a sample VR gaming application | [Link](https://github.com/cschierle/stationaryBikeVR)

## Hardware Components

Component | Description
--- | --- 
Stationary Bike | Ordinary fitness machine which will be used to act as a input controller
IR Sensor - TCRT5000 | For the beginning we will only measure the bike’s revolution count and forward it as the proposed input value. In order to measure the current revolution count of the bike we will use an IR sensor which can be easily installed on any fitness machine. 
Raspberry Pi 3 Model B+ | By connecting the IR sensor to a Raspberry Pi, we can preprocess its signal and calculate the user’s performance. The sensor controller will then forward this data to the user’s smartphone by Bluetooth.
Virtual Reality Platform | We will develop our VR software by using Google Cardboard. Our main platform will be Android.
Wiimote | In order to further extend the user’s input possibilities, we are considering the usage of additional controllers (e.g. Wiimote which can be also paired over bluetooth).