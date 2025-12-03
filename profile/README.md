# Corc

I'm proud to present **Corc**, a multi-channel remote control stack for physical devices, developed under the **[`co-rc`](https://github.com/co-rc)** organization.

Corc lets different kinds of controllers orchestrate different kinds of devices over multiple transports, starting with:

- **BLE** – Android ↔ bridge  
- **IR** – bridge ↔ TV / IR devices  
- **USB HID** – bridge ↔ host

The core idea is a **transport gateway** that lets you replace existing remotes and build custom ones, while keeping the transport plumbing reusable.

- **Possible expansions:** *cooperative Remote Control*, *Common RC transport*  
- **Status:** early design / prototyping, APIs and layout may change  
- **License:** Apache-2.0 (planned)  
- **Scope:** protocol-focused, modular stack for controllers, bridges and devices

## Example topology

### 1. Android app → BLE-IR bridge → TV (simple use case)

```text
+-----------------------------------------------------------+
|                 Android TV Remote app                     |
|                                                           |
|  [ Power ]  [ Vol+ ]  [ Vol- ]  [ Mute ]  [   Input   ]   |
|                                                           |
|     (Android application acting as a TV remote control)   |
+-------------------------------+---------------------------+
                                |
                                |  Bluetooth LE (commands)
                                v
+-------------------------------+---------------------------+
|                        BLE-IR bridge                      |
|     (Pico W / ESP32 + MicroPython running Corc firmware)  |
|                                                           |
|  - receives commands from Android over BLE                |
|  - sends IR codes like a normal TV remote                 |
+-------------------------------+---------------------------+
                                |
                                |  IR (infrared)
                                v
                .---------------------------------.
                |     ___________                 |
                |    |           |                |
                |    |   TV      |                |
                |    |  screen   |                |
                |    |___________|                |
                |   (IR-controlled end device)    |
                '---------------------------------'
```
