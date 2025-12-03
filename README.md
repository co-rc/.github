# Corc

I'm proud to present **Corc**, a multi-channel remote control stack for physical devices.

Corc lets different kinds of remote controllers orchestrate different kinds of devices over multiple transports – starting with BLE/GATT and USB HID.

> Organization: [`co-rc`](https://github.com/co-rc)  
> Project name: **Corc**  
> Possible expansions: *cooperative Remote Control*, *Common RC transport*

The common idea is a **transport gateway** that lets you replace existing remote controllers and build unusual / custom remotes, while keeping the transport plumbing reusable.

> If you are not into Bluetooth: in this project you can think of **GATT** as  
> “the protocol a Bluetooth remote controller uses to talk to a TV”.

---

## License

Corc is distributed under the **Apache License 2.0** (Apache-2.0).

---

## Status

Early design / prototyping phase.

- APIs and protocol layout are still evolving.
- Expect breaking changes at this stage.
- Repository layout may change while things are being bootstrapped.

If you are reading this and the code is not here yet, this README is describing the intended scope and architecture.

---

## Example topologies (ASCII diagrams)

### 1. Android app → BLE bridge → TV (simple use case)

```text
+-----------------------------------------------------------+
|                 Android TV Remote app                     |
|  [ Power ]  [ Vol+ ]  [ Vol- ]  [ Mute ]  [   Input   ]   |
|                                                           |
|         (Android application acting as remote)            |
+-------------------------------+---------------------------+
                                |
                                |  Bluetooth LE (GATT)
                                v
+-------------------------------+---------------------------+
|                         BLE bridge                        |
|                  (PicoW / ESP32 + MicroPython             |
|                     running Corc firmware)                |
+-------------------------------+---------------------------+
                                |
                                |  GATT control of TV
                                v
                .---------------------------------.
                |     ___________                 |
                |    |           |                |
                |    |   TV      |                |
                |    |  screen   |                |
                |    |___________|                |
                |                                 |
                '---------------------------------'
````

BLE bridge is planned as a **microcontroller (e.g. Raspberry Pi Pico W or ESP32) running MicroPython** with firmware from this project.

---

### 2. IR remote → GATT bridge → multiple end devices

In this setup the **“brain”** is not an Android app but an existing IR remote.
Corc only provides the **GATT bridge / transport gateway**:

```text
+-------------------+       IR        +---------------------------+
|  IR remote        |  ------------>  |  GATT bridge              |
|  (remote controller)                |  (transport gateway)      |
+-------------------+                 +-------------+------------+
                                                    |
                                                    | GATT
                                                    v
                                       +------------+------------+
                                       |   Bulb #1               |
                                       +-------------------------+
                                       +-------------------------+
                                       |   Bulb #2               |
                                       +-------------------------+
```

The GATT bridge can translate between IR commands and GATT operations for multiple end devices.

---

### 3. Android app → USB HID keyboard bridge

```text
+-------------------+   Corc commands   +---------------------------+
| Android app       |  ---------------> |  USB HID keyboard bridge  |
| (remote controller|                   |  (USB device)             |
+-------------------+                   +-------------+------------+
                                                    |
                                                    | USB HID (scan codes)
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
  Reuse the same “transport plumbing” (Bluetooth, GATT, USB HID, etc.) across different remote controllers and scenarios.

* **One remote controller per setup (but not globally one brain)**
  In some setups the brain is an **Android app**; in others it might be an **IR remote** or something else. Corc focuses on the **transport layer**, not on forcing a single global controller.

* **Pluggable transports**
  BLE → GATT bridges, GATT → GATT bridges, USB HID keyboard bridges – and more in the future – all share the same command model.

* **Simple mental model**
  From the controller’s perspective there is “a device” and “a command”, not “BLE vs HID vs GATT”.

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

### 2. BLE bridge (BLE → GATT remote controller / transport)

A BLE-based remote controller bridge that talks to physical hardware exposed via GATT.

Responsibilities:

* handle BLE discovery / connection,
* expose a GATT-based API (Generic Attribute Profile),
* translate Corc commands to GATT operations and back (notifications, indications, reads/writes).

Implementation notes:

* based on a **microcontroller board (e.g. Raspberry Pi Pico W or ESP32)**,
* running **MicroPython**,
* firmware and reference hardware design (schematics / wiring) will live inside Corc repositories.

---

### 3. GATT bridge (GATT → GATT fan-out)

A GATT transport gateway that can be used with different “brains”
(e.g. an Android remote controller or an existing IR remote with an IR→GATT front-end).

Responsibilities:

* accept a **control channel** (exposed over GATT),
* maintain connections to multiple GATT end devices,
* route and multiplex commands, map logical devices to physical addresses.

Implementation notes:

* software and any reference hardware are part of Corc,
* meant to be reusable for multiple scenarios (IR remote, Android, etc.).

---

### 4. USB HID keyboard bridge (remote-controlled keyboard device)

A **device**, similar in spirit to the BLE bridge, controlled from an Android remote controller and exposing itself as a USB keyboard to a host (PC, TV, etc.).

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

### 5. Shared protocol / model

A small, shared model used across all components:

* **Device** – logical identifier, type, capabilities.
* **Command** – abstract “thing to do”, transport-agnostic.
* **Transport / Bridge** – adapter that knows how to encode/decode commands for a specific channel
  (BLE/GATT, GATT→GATT, USB HID, …).

The same protocol can be used by:

* Android remote controller,
* IR-based setups (through appropriate front-ends),
* other future remote controllers.

---

## Conceptual model

From the controller’s point of view, Corc is:

```text
Device       = something that can receive commands
Command      = description of what should happen
Transport    = implementation detail handled by a Corc bridge
```

Examples:

* BLE bridge (remote controller):

  * Device: “TV”, “LED strip”
  * Commands: “volume up”, “power toggle”, “brightness 50%”
  * Transport: BLE + GATT

* GATT bridge:

  * Device: “Bulb #1”, “Bulb #2”
  * Command: “set brightness 50%”, “turn off”
  * Transport: GATT

* USB HID keyboard bridge:

  * Device: “Host keyboard”
  * Command: “PowerOff”, “Sleep”, “Volume Up/Down”, “Mute”
  * Transport: USB HID

The controller does not need to know whether a specific command is executed via BLE, GATT, HID, or something else – each channel is just an implementation of the same command interface.

---

## Repository layout (planned)

Under the `co-rc` organization, Corc will be split into functional parts such as:

* Android remote controller application.
* BLE bridge:

  * MicroPython firmware for BLE / GATT remote controller.
  * Reference hardware (e.g. Pico W / ESP32 wiring / board).
* GATT bridge:

  * GATT→GATT fan-out transport gateway.
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

A typical BLE-based setup might look like:

1. Clone the relevant Corc repositories from `co-rc`.
2. Open the Android project in **IntelliJ IDEA with the Android plugin**.
3. Build and install the Android remote controller app on a device.
4. Build and flash the BLE bridge firmware (MicroPython + Corc code) onto a supported microcontroller (e.g. Pico W / ESP32) using the hardware and instructions provided in the Corc repos.
5. Power up the BLE bridge and pair it with the Android app.
6. Discover and control end devices from the app.

For GATT bridges and USB HID keyboard bridges, similar steps will apply:

* use Corc-provided firmware and reference hardware designs,
* flash the appropriate bridge firmware onto a supported microcontroller board,
* connect the board to the target host (e.g. USB for the keyboard bridge),
* pair the bridge with the chosen remote controller.

As the project matures, this section will contain concrete build commands, example configs, and step-by-step guides.

---

## Roadmap (draft)

* [ ] Define and stabilize the core Corc protocol / command model.
* [ ] Publish the first Android remote controller skeleton.
* [ ] First BLE bridge (MicroPython + PicoW/ESP32) reference implementation.
* [ ] First GATT bridge (GATT→GATT fan-out) reference implementation.
* [ ] USB HID keyboard bridge (PowerOff/Sleep/Vol/Mute scan codes).
* [ ] Unified discovery and device registry.
* [ ] Example setups and demo scenarios (Android BLE remote, IR remote with GATT bridge, USB keyboard control).

---

## Contributing

At this stage Corc is still being sketched and bootstrapped.

There is no formal contribution process yet, but **feel free to contribute** once code is available:

* by opening issues with ideas or bug reports,
* by sending pull requests to improve the code or documentation.

Details about code style, review process, and governance will be added once the project stabilizes.
