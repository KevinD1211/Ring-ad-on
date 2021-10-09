![ring-mqtt-logo](https://raw.githubusercontent.com/tsightler/ring-mqtt-ha-addon/master/logo.png)

![aarch64-shield](https://img.shields.io/badge/aarch64-yes-green.svg)
![amd64-shield](https://img.shields.io/badge/amd64-yes-green.svg)
![armhf-shield](https://img.shields.io/badge/armhf-yes-green.svg)
![armv7-shield](https://img.shields.io/badge/armv7-yes-green.svg)
# About
This Home Assistant add-on provides integration of Ring devices using the [ring-mqtt](https://github.com/tsightler/ring-mqtt) script.  Currently, most official Ring alarm devices, as well as many 3rd party Z-wave devices that can be connected to the Ring Alarm hub, are supported.  The addon also support Ring Smart Lighting, Cameras and Chimes.  Camera and Chime support is disabled by default since these are already supported via the native Home Assistant Ring component, but can be easily enabled if you prefer to use the support available in this addon.

The optional camera support includes the ability to stream both live and recorded events (streaming recorded events requires a Ring Protect Plan subscription) and includes the required metadata needed to automate downloading recorded videos.  The automatically discovered cameras use the Home Assistant MQTT camera component and these cameras only support snapshots so using the video streaming support will requireme manually adding generic IP cameras to the Home Assistant configuration.yaml file.  Please read the documentation for more details on configuring the streaming cameras.

***This add-on requires a working MQTT broker.  The Mosquitto MQTT add-on for Home Assistant is highly recommended.***
