## v4.8.4
**Fixed Bugs**  
- Event streams failed to update status to off/inactive after event finished playing, causing various replay issues
- Minor change to default filter for value templates entities to hopefully quiet warnings for openHAB users attempting to leverage this project via the Home Assistant MQTT binding

**Minor Enhancements**  
- Chimes snooze switch now includes additional JSON attribute "minutes_remaining" showing approximate number of minutes before snooze expires.

## v4.8.3
**New Features**  
- The event stream select entity now includes eventId and recordingUrl attributes with values updated based on the selected event to facilitate automatic downloading of recorded videos. See the [camera documentation](CAMERAS.md) for more information and an example automation using Home Assistant downloader service.
- Dome sirens (and perhaps other Z-wave sirens) are now supported

**Fixed Bugs**  
- Refactor and simplify snapshot functions, especially for battery cameras.  This should hopefully fix the issue of no motion snapshots for users with battery powered cameras.

**Breaking Changes**  
- Siren devices are now represented as a switch instead of a binary_sensor.

**Other Changes**  
- Device debug output now includes device name for all entries, including all received commands

## v4.8.2
**New Features**
- Support streaming video of historical motion/ding/on-demand events via new event stream RTSP path.  See the [camera documentation](https://github.com/tsightler/ring-mqtt/blob/main/docs/CAMERAS.md) for more details. (Note that this feature requires a Ring Protect plan that allows saving videos)
- Info sensor support for Ring Bridge (thanks to [alexanv1](https://github.com/alexanv1) for this PR!)

**Fixed Bugs**
- Bump ring-client-api to 9.21.2 to address issue with error on toggling camera lights on/off
- Fix alarm state attribute not resetting to all-clear after an alarm trigger event

## v4.8.1  
**Fixed Bugs**  
- Fix tamper entity state not updating on tamper events
- Fix typo in still image URL (thanks to [aneisch](https://github.com/aneisch) for the PR!)

**Minor Enhancements**  
- Add additional debug message for on-demand stream trigger
- Small enhancement to still image and stream URL generation to hopefully make a slightly better "guess"
- Monitor legacy hassio/status topic for birth/last will messages to inform of HA restarts for users with setups still using older defaults

**Other Changes**  
- Update to rtsp-simple-server 0.17.3, hopefully slightly reduces stream startup time
- Minor tweak to docker image to allow running under Docker as non-root user.  Requires adding S6_READ_ONLY_ROOT=1 to docker run environment.
- Documentation tweaks to clarify camera setup steps, especially for still image URL.

## v4.8.0
**New Features**  
Live Video Streaming is here!  

Since this plugin introduced support for cameras over 2 years ago, the single most requested feature, which I usually answered will "never be supported", is live streaming.  I didn't believe this feature would ever fit within this project simply because this script used MQTT for integration, which simply isn't a platform that can support streaming, other than the limited capabilities used for the snapshot feature.

However, because of continued demand for live streaming, I was reasearching and prototyping possible methods for integrating live streams when I saw a post from [gilliginsisland](https://github.com/jeroenterheerdt/ring-hassio/issues/51) on the ring-hassio Github issues page.  The final result uses rtsp-simple-server with an on-demand script that triggers the livestream via MQTT, and it was the concept in that post that provided the imputus to finally do the work in a way that felt like a proper fit within this project.

I have some additional features planned for the coming weeks, mainly the ability to play the last X recorded events, but I wanted to get something out there now for people to play with and see how it works in a wider range of environments than my test setup.  

Features included in this release:
- Easy(-ish) integration with Home Assistant, although note that it is not automatic.  Live streaming cameras must be manually added to Home Assistant configuration.yaml.  Please read [the camera docs](https://github.com/tsightler/ring-mqtt/blob/main/docs/CAMERAS.md) for more details.
- Support for on-demand live streams.  Streams are started automatically when viewed in Home Assistant and ended 5-10 seconds after the last viewer disconnects
- Support for external medial player by exposing the RTSP port on the addon any tool that supports RTSP streaming can consume the stream.
- Support for defining a username and password for authenticating the stream.
- Manual stream start via the "stream" switch entity or via MQTT command.  Allows for cool things like triggering a recording using automation.

**Minor Enhancments**  
- Increased maximum allowed time between snapshots from 3600 seconds (1 hour) to 604800 (7 days)
- Repopulate entities and states much sooner after Home Assistant restart is detected
- New algorithm for pulling motion snapshots from battery cameras.  Uses less CPU and should generate a more reliable image with less artifacts, but will be a little slower.

**Fixed Bugs**  
- Fix interval snapshots when using only "interval" setting vs "all"

**Other Changes**  
- Docker image now uses S6 init system for supervising node process
- Massive startup script cleanup and standardization

## v4.7.3
***** IMPORTANT NOTE *****  

If upgrading from version 4.6.x or earlier, please read the 4.7.0 change notes as well!

**Minor Enhancements**
- Documentation updates now note that Chimes only work with primary Ring account, not shared accounts
- Tweak logging color scheme to improve event readability
- Fix uncaught exception error handler to properly log error message

**Other Changes**
- Switch Docker base image to Node LTS-Alpine 3.14 (previously Alpine 3.12)

## v4.7.2 (this version was never released to the addon)
**Fixed Bugs**
- Add check for optional entities in publish and command processing to avoid crashes (most commonly an issue for Smart Lighting support where different devices have varying capabilities)
- Fix broken exit handler

## v4.7.1
**Fixed Bugs**
- Smart Lighting support caused crashes in 4.7.0
- Proper use of systemId with Ring authentication (addon only for now, hopefully eliminates spamming Authorized Client Devices in Account Control Center)

## v4.7.0
***** IMPORTANT NOTE *****

Due to changes in the way ring-mqtt generates configuration topics it is HIGHLY recommended to restart the Home Assistant instance as soon as possible after the upgrade of this addon.  Without this Home Assistant will log warnings/errors about non-unqiue entity IDs.  While ring-mqtt does generate unique IDs for entities, version 4.7.0 has standarized the generation of configuraiton topics which results in slightly different topics for some devices.  Because of this, the Home Assistant discovery process thinks it is seeing new devices with the same IDs as existing entities.  Restarting Home Assistant will allow for a fresh discovery cycle, and, since the entity IDs did not change from previous versions, only the configuration topics, there should be no changes required to existing devices.  For more details on the underlying engine changes you can read the "Other Changes" section below.

**New Device Support**
 - Ring Chimes
     - Chime Volume
     - Enable/Disable Snooze Mode (and display Snooze State) 
     - Set Snooze Minutes (must be set prior to enabling snooze mode)
     - Play Ding/Motion Sound
     - Wireless Signal Strength
 - Temperature Sensors
 - Thermostats (Currently only tested with Honeywell T6 Pro Z-wave Thermostat, would be interested in success/fail reports for others)

 **New Features**
  - Alarm devices now have individual entities for battery and tamper state (shoutout to @rechardhopton for the concept)
    Additional battery status data (charging state, auxillary batteries, etc) is available in the battery attributes
    All device attributes are still available in the Info sensor attributes to keep from breaking any existing monitoring
  - Battery cameras now show battery status as a separate entity with attributes for detailed battery status
  - Battery status will now report in battery column in Home Assistant Devices UI
  - Wifi strength now has it's own entity for alarm Base Station, Cameras and Chimes connected via wireless networks
    Wireless network name is available in wireless entity attributes

**Minor Enhancements**
  - Improved default icons for various entities
  - Debug output now includes devices names along with topics and state for easier identification of activity
  - On first startup a unique system ID is generated and stored in the state file.
  - Authorized Client entries for this addon now identify as "ring-mqtt-addon" or "ring-mqtt" (based on addon or docker/standalone mode) in the Ring Control Center
  
**Breaking Changes**
  - Due to the introduction of seperate entities for battery, tamper, and wireless status, the primary info sensor state for most devices has been changed to commStatus for most alarm devices (still alarmState for the Alarm Control Panel, and acStatus for Base Station, Range Extender, and Keypad).  For Cameras and Chimes the Info sensor state is now the last health update time (i.e. the last time health data was updated by Ring servers, usually every 4-8 hours, but sometimes longer).  Any automations or scripts that monitored the primary info sensor state, rather than a sepcific info sensor attribute, will need to be updated to use the new entity sensors.

**Fixed Bugs**
  - "Addressed 'dict object' has no attribute" warnings due to changes in Home Assistant >=2021.4
  
**Other Changes**

Underneath the covers there are quite a number of changes to the engine with the primary goal to simplify and standardize device support and, in turn, make it easier to maintain and add new device support.  The prior model, if it can be called that, was a disaster of my own making with different devices using inconsistent methods for generating unique entity IDs and configuration topics.  This is primarily because I never really thought much about the device model when ring-mqtt was first created as there was only alarm, motion, and contact sensors.  Other devices have been bolted on haphazardly along the way without much thought or consistency so that needed to change and no better time than now.
  
With the new model, device entities are defined in a consistent way and entity ID's, names, and MQTT topics are generated promgratically and consitently across all devices.  Key features of the new model:
- Entities are defined using a simple JSON format, sometimes requiring as little as one line to define a simple entity
- Home Assistant discovery messages are now built using a single, common function vs being hand coded for each device.  I've tried to maintain bug for bug compatibility with legacy versions, but I could have missed something so let me know if you see odd things.
- All MQTT topics are built automatically by the discovery function and saved to the entity object
- All device types (alarm, camera, chimes, smart lighting), now use a common base device and consistent functions
- Command processing is now unified for all devices
- All "special case" processing during device publishing/republishing is removed
- Entity topic and state properties use a more consistent naming across all devices

A primary goal of the new engine is to be 100% compatible with prior ring-mqtt to avoid breaking users during upgrades, however, this proved to be quite difficult.  I think I've managed to make the update nearly transparent, and I've tested the upgrade process on ~90% of supported devices, however, I don't own any locks, fans, or smart lighting devices, and, while I do attempt to fake them for testing purposes, I can't be 100% sure I didn't miss something.  Please feel free to report any devices or entities that either don't work or are duplicated after the upgrade.

## v4.6.3
 - Changes to snapshot interval now immediately cancel current interval and start new interval with the updated duration
 - Fix for Home Assistant with snapshot interval values > 100 seconds (valid values are now 10-3600 seconds)
 - Improved default icons for Home Assistant entities for snapshot interval, beam duration and volume level
 - Additional discovered devices debugging during startup including device name and id

## v4.6.2
 - Version bump to pull in changes required to fix 404 errors on startup due to Ring API changes
 
## v4.6.1
 - Add code to (hopefully) automatically remove legacy light based volume controls from Home Assistant

## v4.6.0
 - Adapt fan component to new fan schema introduced in Home Assistant 2021.3.  This schema is based on percentage vs using three preset speeds.  Presets for "low, medium, high" will still work but it's now possible to use the new percent based speed topics which map directly to Ring app and Home Assistant.  This is especially useful for fans which support more than 3 speeds.
 - Add support to define the default "on" duraton for Ring Smart Lighting via config option beam_duration (or BEAMDURATION envionment variable), please refer to documation (DOCS.md) for more details
 - Add ability to override "on" duration for individual Ring Smart Lights via MQTT topic, also uses number integraiton to present in Home Assistant for easy access via Lovalace UI or automations
 - Add support for "arming" state during exit delay
 - Add support for configuring a disarm code for Home Assistant (See disarm_code option in README)
 - Add support for reporting basic status and attributes of Ring External Siren
 - Docker images now enable debug logging by default (was already true of addon)
 - Removed "enable_volume" config option since the new number based integration will no longer be accidentally triggered by light based automations
 
 **Breaking Changes**
 - Fan automations in Home Assistant should continue to work with backwards compatibility, however, it's probably a good idea to update any automations to use the new methods (see Fan section of [Breaking Changes](https://www.home-assistant.io/blog/2021/03/03/release-20213/#breaking-changes) in Home Assistant 2021.3 release notes) and to review the new percent based speed settings as well.
 - Volume controls now use Home Assistant number component instead of the previously used light component.  Any automations for volume changes will need to be updated to use the new component.

## v4.5.7
 - Switch to custom ring-client-api with fix for hang during network/service outages
 - Add MotionDetectionEnabled attribute to camera motion entity attributes
 - Add support for snapshot interval setting via Home Assitant number entity

## v4.5.6 (***This build was experiemental and never intended for release***)
 - Experimental release with custom ring-client-api

## v4.5.5
 - Improve stream reliability with new Ring media servers by bumping to ring-client-api 9.18.0

## v4.5.4
 - New, lightweight, hopefully improved, snapshot from live stream implementation for battery cameras
 - Send "online" status prior to sending state data updates
 - Bump dependencies

## v4.5.3
 - Implement reconnect improvements for cameras after lost connections
 - Bump ring-client-api version

## v4.5.2
- Second attempt to fix truncation of video length (tries to read property if available, otherwise keeps stream alive for 60 seconds)

## v4.5.1
- When attempting to grab snapshot from livestream for battery cameras, set stream duration equal to video recording length setting to (hopefully) avoid truncating video recording

## v4.5.0
 - Snapshot on motion reliability improvements for line powered cameras
 - Snapshot on motion attempts to grab image from livestream for battery powered cameras.  This is slower, less reliable and sometimes produces lower quality images vs snapshots, but as battery cameras don't allow snapshots wile streaming, it's the only option for getting a snapshot of a motion event
 - Person detect attribute for camera motion events (needs testing, only works if person detection is enable on accout and show up as person detect events in history)
 - Date/Time uses standard format accross all attributes
 - Retrofit zones are now included as bypass eligible sensors when bypass arming is enabled
 - Docker image updated to Node v14 to hopefully address reconnect issues
 - Fix various crash bugs in camera support## v4.4.0
 - Add Arming Bypass Mode switch
 - Misc code cleanups and improvements
 - Drop support for i386 architecture for addon

## v4.4.0
 - Add Arming Bypass Mode switch
 - Misc code cleanups and improvements

## v4.3.2
 - Change enable_snapshots config option to snapshot_mode (fix for addon upgrade issue)

## v4.3.1
 - Add support for scheduled snapshot updates (see docs for details)
 
## v4.3.0
- Add support for sending snapshot images on camera motion events (must be explicitly enabled)
 
## v4.2.2
- Bump dependencies to latest versions
- Rebuild Docker image with latest base to pull in latest node v12 version (12.20.1)

## v4.2.1
- Add tilt sensor support (Garage Doors)
- Fix range extender device name
- Minor doc updates

## v4.2.0
- Add support for Zwave Range Extender info sensor data (thanks to @alexanv1)
- Make volume control support configurable
- Monitor legacy hass/status stopic for Home Assistant restarts
- Documentation updates and cleanups

## v4.1.1
- Fix enabling panic buttons for Docker
- Fix branch feature when running Docker as unprivileged user

## v4.1.0
- New Feature: Branches
  Simple testing of latest git repo versions from master or dev branch without updating addon.
- Use common image for addon and standalone Docker users
- Reduced Docker image size by ~65MB
- Fix issue with Docker image not being cleaned during upgrades/uninstall (unfortunately won't help for existing images, suggest existing users run "docker system prune" to remove old images)
- Web tweaks to work reasonably with both dark/light themes
- Minor fix for CO sensor to report correct manufacturer

## v4.0.4
- Minor fixes for smart lighting support
- Fix (hopefully) non-fatal resubscribe errors
- Fix a few typos in various devices
- Bump MQTT dependency to 4.2.1

## v4.0.3
- Fix for Smart Lighting motion sensor discovery bug introduced in 4.0

## v4.0.2
- Additional volume control fixes for base station and keypad
- Volume controls show up as lights due to limitations of MQTT component available but you change icons and labels in frontend.

## v4.0.1
- Fixes for various MQTT discovery issues
- Reintroduce manual options for MQTT broker settings (override automatic discovery)
- Minor fixes for Base Station volume and info sensor and Keypad volume

## v4.0.0
- Support for Home Assistant Device Registry
- Each device has an "info" sensor updated at least every 5 minutes. State is JSON data with values unique to each device type (battery level, tamper status, AC state, wireless strength, firmware info, serial number, etc)
- Support for monitoring alarm state
  - "Pending" state is equivalent to "entry delay"
  - "Triggered" state is any alarm
  - Detailed alarm state is available via info sensor
- Support for Fire and Polic Panic Buttons
  - *** Should be Used with Caution *** -- Can trigger alarm events with response
  - Can also be used as a high level monitor for burglar or fire alarms (panic state will trigger based on alarm type)
- Support for Base Station
  - Info sensor for monitoring battery/ac status
  - Ability to set volume (requires use of master account as other accounts don't have permission)
- Support for Keypad
  - Info sensor for monitoring batter/ac status, charging state, etc.
  - Ability to set volume
- Basic support for 3rd party Z-wave contact and motion sensors
  - Assumes device is concact sensor unless device name contains the word "motion"
- Significantly enhanced Web UI for token generation (mostly for Home Assistant Addon)
- Improved debug output with more organized location/device discovery output
- Simplified and standardized location/device handling code (still more work to do but becoming far more maintainable)

##Changes for v3.3.0 and earlier were not tracked in this file
