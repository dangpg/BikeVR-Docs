The whole project consist of three main components:

Component | Description
--- | --- 
Sensor Controller | Raspberry Pi which runs a custom python program to advertise connected sensor values by operating as a Bluetooth Low Energy peripheral
Unity Framework BikeVR | C# Unity Framework which enables to read the sensor controller's values on a mobile platform (Android, IOS)
Google VR Application | Sample Google Cardboard Unity project which combines all components to form a VR gaming application