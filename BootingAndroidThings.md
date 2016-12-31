# Booting Android Things
This session is based on Things preview version v1.
Android Things is brand name on launch of IoT framework while it was branded as Brillo on pre-launch in 2015.

## Architecture

![Android Things](https://developer.android.com/things/images/platform-architecture.png)

Things comes from Android stack.

If you have any prior experience to Android, you are not new to Things as well.
* If one has worked for developing Android application, he is ready to go with writing application for Things.
* If one is new for Android, one has to go through basic elements of Android, such as Activity, Menifest,
intent, broadcast receiver at minimum to get a feel of your first sample application.

### Benifits
* Easy integration and use of Google services which is key part here in embedded zone.
* Very stable and developer friendly development tools


## Developer Access

![Developer Access](https://developer.android.com/things/images/driver-stack.png)

## Things Support Library
Library provide device interface protocols.
* GPIO
* PWM
* I2C
* SPI
* UART

Things provide above protocols access at application layer.
API details for these interfaces are available at [Understanding APIs](#) section.

### What we can do
While writing application, in most cases, we will need additional driver, specific to the device. These drivers has be handled by the developer by adding or implementing. Some of the sample drivers are provided at [contrib-drivers](https://github.com/androidthings/contrib-drivers). For example, if we are uing barometric sensor BMX280, we will need to add compiler dependency in build.gradle for application.

## Development Platforms
### Hardware Platforms
List of development platforms, which are already ported with Things OS, are specified at [Developer Kits](https://developer.android.com/things/hardware/developer-kits.html) section of Things official website.

During Preview image v1, devices available for development are:
* Raspberry Pi 3
* Intel® Edison
* NXP Pico i.MX6UL

We will be using Raspberry Pi 3 to see Things booting during this session.

### Development Environment:
Things uses same build tools and development tools as Android. Get Android Studio, Android SDK and you are ready to write and test you application. Unlike Android, you need to have hardware platform to test Things application as of version v1.

## Booting Things on Raspberry Pi 3
Flashing OS image to Raspberry Pi 3 is same as usual. Use dd command to flash and you are done.
```
$ sudo dd bs=4M if=./iot_rpi3.img of=/dev/<mmc_device>
```
For the first time you need to connect Raspberry Pi 3 to router by ethernet cable. Check if DHCP is enabled in router.
```
$ adb connect <ip-address>
```
You can get IP address provided by router DHCP by either of following:
* Login to router and see connected clients IP address to router.
* You can connect Raspberry Pi 3 to TV/Monitor HDMI cable. And can view IP on screen.
* You can wait for about 2 mins and take snapshot using Android Studio and see the screen image.


Once you get adb access over ethernet, you can configure WiFi by sending intent to WifiSetupService with SSID and passphrase as intent extra. Once configured, configuration remains with WiFi service and no further configuration is needed from next boot onwards.
Configure WiFi with following command:
```
$ adb shell am startservice \
    -n com.google.wifisetup/.WifiSetupService \
    -a WifiSetupService.Connect \
    -e ssid <Network_SSID> \
    -e passphrase <Network_Passcode>
```
Now after WiFi configuration success you can get IP address over WiFi and disconnected ethernet and connect ADB with WiFi IP address. And your setup is completed.

### Process Analysis
Process after boot complete
[adb ps](adb_ps_log.txt) shows running process snapshot in Things preview version v1.

```
init -> logd
init -> debuggerd
init -> vold
init -> kauditd
init -> healthd
init -> lmkd
init -> servicemanager
init -> surfaceflinger
init -> adbd
init -> zygote
init -> audioserver
init -> cameraserver
init -> drmserver
init -> installd
init -> keystore
init -> media.codec
init -> mediadrmserver
init -> media.extractor
init -> mediaserver
init -> netd
init -> peripheralman
init -> gatekeeperd
init -> userinputdriverservice
init -> metrics_collector
init -> metricsd
init -> perfprofd
init -> update_engine
init -> mdnsd
init -> system_server
vold -> sdcard
init -> wpa_supplicant
zygote -> com.android.settings
zygote -> android.ext.services
zygote -> android.process.media
zygote -> com.android.iotlauncher
zygote -> com.google.android.gms.feedback
zygote -> com.google.android.gms.persistent
zygote -> com.android.managedprovisioning
zygote -> com.android.onetimeinitializer
zygote -> com.android.providers.calendar
zygote -> com.google.process.gapps
```

When you are checking the logs then you should also have a look at below lines and at your free time, dig into the concept. 
```

USER      PID   PPID  VSIZE  RSS   WCHAN            PC  NAME
root      1     0     7336   1552           0 00000000 S /init
root      2     0     0      0              0 00000000 S kthreadd
...
...
root      102   2     0      0              0 00000000 S binder
...
root      117   1     2932   1132           0 00000000 S /sbin/ueventd
...
root      152   1     965944 95168          0 00000000 S zygote
```

### Services Analysis

[service list](service_list_log.txt) shows list of services running in preview version v1 after boot complete.

## Things Application

### Home Activity

* Things needs an application to declear a home activity. And it is done by declearing intent filter, with category IOT_LAUNCHER and DEFAULT.
* To execute application from Android studio, we need additional seperate intent filter with category LAUNCHER.

```
<application
    android:label="@string/app_name">
    <activity android:name=".HomeActivity">
        <!-- Launch activity as default from Android Studio -->
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>

        <!-- Launch activity automatically on boot -->
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.IOT_LAUNCHER"/>
            <category android:name="android.intent.category.DEFAULT"/>
        </intent-filter>
    </activity>
</application>
```
### Connecting peripheral
Raspberry Pi 3 peripheral I/O interface pins are shown here:
![Raspberry Pi 3 I/O](https://developer.android.com/things/images/pinout-raspberrypi.png)

Locate pin BCM6, which is pin number 31, as we will be connecting LED to the same pin.

### Cloning sample application
We will be cloning sample LED blink sample provided by Google for developer.
```
$ git clone https://github.com/androidthings/sample-simplepio.git
```
For getting better connectivity diagram, refer README page of [sample-simplepio](https://github.com/androidthings/sample-simplepio) link. As we are running sample from Android studio, we will not need to mentioned gradle commands mentioned instead we will use Android Studio GUI for the same. Although you can try it if you want.

### Executing sample application

From Android studio, select cloned sample application,
File -> Open ... -> (sample-simplepio)
It may ask for gradle updates, go for it. And you are ready to test your first application.
Just execute it and you are done. You can see LED blinking.

### Memory Usage

## References

All of the references are taken from https://developer.android.com/things/hardware/index.html which is the only source as of now.

## About Me
* **Abhishek Dwivedi** - *independent technical consultant* - [OERDev](https://github.com/abhishekkumardwivedi)
* *contact to hire [abhishekk@oerdev.com](abhishekk@oerdev.com) or sponser for helping in your project R&D.*
