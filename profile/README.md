# Corc

I'm proud to present **Corc**, a multi-channel remote control stack for physical devices.

Corc lets different kinds of remote controllers orchestrate different kinds of devices over multiple transports – starting with **BLE (Android ↔ bridge)**, **IR (bridge ↔ TV / IR devices)** and **USB HID (bridge ↔ host)**.

> Organization: [`co-rc`](https://github.com/co-rc)  
> Project name: **Corc**  
> Possible expansions: *cooperative Remote Control*, *Common RC transport*

The common idea is a **transport gateway** that lets you replace existing remote controllers and build unusual / custom remotes, while keeping the transport plumbing reusable.

---

## License

Corc is distributed under the **Apache License 2.0** (Apache-2.0).

---

## Doc

https://co-rc.github.io/

## Status

Early design / prototyping phase.

- APIs and protocol layout are still evolving.
- Expect breaking changes at this stage.
- Repository layout may change while things are being bootstrapped.

If you are reading this and the code is not here yet, this README is describing the intended scope and architecture.

---

## Example topologies (ASCII diagrams)

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
````

The BLE-IR bridge behaves like a normal IR remote towards the TV,
and like a BLE peripheral towards the Android app.

---

### 2. Android app → USB HID keyboard bridge

```text
+-----------------------------------------------------------+
|                 Android Remote app                        |
|                                                           |
|  [ PowerOff ]  [ Sleep ]  [ Vol+ ]  [ Vol- ]  [ Mute ]    |
|                                                           |
|  (Android application acting as a PC / TV remote control) |
+-------------------------------+---------------------------+
                                |
                                |  Bluetooth LE (commands)
                                v
+-------------------------------+---------------------------+
|                 USB HID keyboard bridge                   |
|      (microcontroller acting as a USB keyboard device)    |
|                                                           |
|  - receives commands from Android over BLE                |
|  - sends USB HID scan codes: PowerOff, Sleep, Vol+/-,     |
|    Mute                                                   |
+-------------------------------+---------------------------+
                                |
                                |  USB HID (scan codes)
                                v
                      +---------+----------+
                      |   PC / TV host     |
                      |   sees: keyboard   |
                      +--------------------+
```

Here Corc lets an Android-based remote controller drive a **USB keyboard-like device** that exposes HID scan codes such as **PowerOff, Sleep, Volume Up/Down, Mute**.
No additional software is required on the host – it just sees a standard USB keyboard.

---

## Goals

* **Reusable transport gateway**
  Reuse the same “transport plumbing” (Bluetooth, IR, USB HID, etc.) across different remote controllers and scenarios.

* **One remote controller per setup (but not globally one brain)**
  In some setups the brain is an **Android app**; in others it might be an **IR remote** or something else. Corc focuses on the **transport layer**, not on forcing a single global controller.

* **Pluggable transports**
  BLE-IR bridges, USB HID keyboard bridges – and future bridges – all share the same command model.

* **Simple mental model**
  From the controller’s perspective there is “a device” and “a command”, not “BLE vs HID vs IR”.

* **Hackable and explicit**
  Clear separation between:

  * remote controller logic (Android app, IR remote, etc.),
  * protocol / model,
  * transport-specific bridges / adapters.

---

## High-level architecture

Corc is split into a few logical components.

### 1. Controller app (Android remote controller)

One of the possible “brains” of a setup:

* defines and issues **commands** (e.g. “button X pressed”, “send key Y”, “toggle Z”),
* discovers and manages **devices**,
* routes commands to the right **transport** via Corc bridges.

Implementation details:

* Android app developed using **IntelliJ IDEA + Android plugin**,
  not necessarily Android Studio.

---

### 2. BLE-IR bridge (Android → IR devices)

A BLE-based remote controller bridge that talks to an Android app over BLE and to end devices over IR, like a normal infrared remote.

Responsibilities:

* act as a BLE peripheral receiving commands from the Android app,
* map Corc commands to IR codes,
* send IR signals compatible with the target device’s remote (TV, amplifier, etc.).

Implementation notes:

* based on a **microcontroller board (e.g. Raspberry Pi Pico W or ESP32)**,
* running **MicroPython**,
* firmware and reference hardware design (schematics / wiring, IR LED driver, etc.) will live inside Corc repositories.

---

### 3. USB HID keyboard bridge (remote-controlled keyboard device)

A **device**, similar in spirit to the BLE-IR bridge, controlled from an Android remote controller and exposing itself as a USB keyboard to a host (PC, TV, etc.).

Responsibilities:

* receive Corc commands from a remote controller (e.g. Android app),
* map those commands to USB HID **scan codes**, such as:

  * `PowerOff`
  * `Sleep`
  * `Volume Up`
  * `Volume Down`
  * `Mute`
* present itself as a standard USB HID keyboard to the host.

Implementation notes:

* based on a **microcontroller board acting as a USB device**,
* firmware is part of Corc (no host-side software required),
* the host (PC / TV) only sees “a keyboard” sending the above scan codes.

---

### 4. Shared protocol / model

A small, shared model used across all components:

* **Device** – logical identifier, type, capabilities.
* **Command** – abstract “thing to do”, transport-agnostic.
* **Transport / Bridge** – adapter that knows how to encode/decode commands for a specific channel
  (BLE→IR, BLE→USB HID, future transports).

The same protocol can be used by:

* Android remote controller,
* other future remote controllers or front-ends.

---

## Conceptual model

From the controller’s point of view, Corc is:

```text
Device       = something that can receive commands
Command      = description of what should happen
Transport    = implementation detail handled by a Corc bridge
```

Examples:

* BLE-IR bridge:

  * Device: “TV”
  * Commands: “volume up”, “volume down”, “power toggle”
  * Transport path: **BLE** (Android → bridge) + **IR** (bridge → TV)

* USB HID keyboard bridge:

  * Device: “Host keyboard”
  * Command: “PowerOff”, “Sleep”, “Volume Up/Down”, “Mute”
  * Transport path: **BLE** (Android → bridge) + **USB HID** (bridge → host)

The controller does not need to know which concrete transport is used – each channel is just an implementation of the same command interface.

---

## Repository layout (planned)

Under the `co-rc` organization, Corc will be split into functional parts such as:

* Android remote controller application.
* BLE-IR bridge:

  * MicroPython firmware for BLE→IR remote controller.
  * Reference hardware (e.g. Pico W / ESP32 wiring / board, IR LED circuit).
* USB HID keyboard bridge:

  * Microcontroller firmware for a remote-controlled USB keyboard device.
* Shared protocol / model:

  * common command and device abstractions, serialization, etc.
* Examples and tools:

  * demonstration setups, debug tools, small utilities.

Exact repository names may evolve; check the organization page for the current layout.

---

## Getting started (high-level)

This section will evolve as the first reference implementations are published.

A typical BLE-IR setup might look like:

1. Clone the relevant Corc repositories from `co-rc`.
2. Open the Android project in **IntelliJ IDEA with the Android plugin**.
3. Build and install the Android remote controller app on a device.
4. Build and flash the BLE-IR bridge firmware (MicroPython + Corc code) onto a supported microcontroller (e.g. Pico W / ESP32) using the hardware and instructions provided in the Corc repos.
5. Power up the bridge and pair it with the Android app over BLE.
6. Configure IR codes for the target TV and control it from the app.

For the USB HID keyboard bridge, the steps will be similar:

* use Corc-provided firmware and reference hardware designs,
* flash the keyboard-bridge firmware onto a supported microcontroller board,
* connect the board to the target host via USB,
* pair the bridge with the Android remote controller app.

As the project matures, this section will contain concrete build commands, example configs, and step-by-step guides.

---

## Roadmap (draft)

* [ ] Define and stabilize the core Corc protocol / command model.
* [ ] Publish the first Android remote controller skeleton.
* [ ] First BLE-IR bridge (MicroPython + Pico W / ESP32) reference implementation.
* [ ] USB HID keyboard bridge (PowerOff/Sleep/Vol/Mute scan codes).
* [ ] Unified discovery and device registry.
* [ ] Example setups and demo scenarios (Android BLE-IR TV remote, USB keyboard control).

---

## Contributing

At this stage Corc is still being sketched and bootstrapped.

There is no formal contribution process yet, but **feel free to contribute** once code is available:

* by opening issues with ideas or bug reports,
* by sending pull requests to improve the code or documentation.

Details about code style, review process, and governance will be added once the project stabilizes.
