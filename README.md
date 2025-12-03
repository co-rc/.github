# Corc

I'm proud to present **Corc**, a multi-channel remote control stack for physical devices.

Corc lets a single controller app orchestrate different kinds of devices over multiple transports – starting with BLE/GATT and USB HID.

> Organization: [`co-rc`](https://github.com/co-rc)
> Project name: **Corc** (`Control & Remote Core`)

---

## License

Corc is distributed under the **Apache License 2.0** (Apache-2.0).

---

## Status

Early design / prototyping phase.

* APIs and protocol layout are still evolving.
* Expect breaking changes at this stage.
* Repository layout and names may change while things are being bootstrapped.

If you are reading this and the code is not here yet, this README is describing the intended scope and architecture.

---

## Goals

* **One brain, many transports**
  A single control app (initially Android) talks to multiple backends over a unified protocol.
* **Pluggable transports**
  BLE → GATT, GATT → GATT fan-out, USB HID keyboard – and more in the future – all share the same command model.
* **Simple mental model**
  From the app’s perspective there is “a device” and “a command”, not “BLE vs HID vs GATT”.
* **Hackable and explicit**
  Clear separation between:

  * UI / control logic
  * protocol / model
  * transport-specific adapters

---

## High-level architecture

Corc is split into a few logical components.

### 1. Controller app (Android)

The main “brain” of the system:

* defines and issues **commands** (e.g. “button X pressed”, “send key Y”, “toggle Z”),
* discovers and manages **devices**,
* routes commands to the right **transport**.

Planned repository (example):

* `co-rc/android` or `co-rc/corc-android`

### 2. BLE pilot (BLE → GATT)

A “pilot” that talks over BLE to physical hardware exposed via GATT.

Responsibilities:

* handle BLE discovery / connection,
* expose a GATT-based API,
* translate Corc commands to GATT operations and back (notifications, indications, reads/writes).

Planned repository:

* `co-rc/ble-pilot` or `co-rc/corc-ble-pilot`

### 3. GATT gateway (GATT → GATT fan-out)

A gateway that takes commands from a single pilot and fans them out to multiple downstream GATT devices.

Responsibilities:

* accept a **GATT-based control channel** from the pilot,
* maintain connections to multiple GATT end devices,
* route and multiplex commands, map logical devices to physical addresses.

Planned repository:

* `co-rc/gatt-gateway` or `co-rc/corc-gatt-gateway`

### 4. USB HID keyboard (host-side bridge)

A bridge that makes it possible to drive a USB HID keyboard from the same controller app (no GATT involved).

Responsibilities:

* expose a Corc-compatible command interface for “keyboard events”,
* map Corc key/shortcut events to USB HID reports,
* run on a host that has access to USB devices.

Planned repository:

* `co-rc/hid-keyboard` or `co-rc/corc-hid-keyboard`

### 5. Shared protocol / model

A small, shared model used across all components:

* **Device** – logical identifier, type, capabilities
* **Command** – abstract “thing to do”, transport-agnostic
* **Transport** – adapter that knows how to encode/decode commands for a specific channel

Planned repository:

* `co-rc/protocol` or `co-rc/corc-protocol`

---

## Conceptual model

From the controller’s point of view, Corc is:

```text
Device       = something that can receive commands
Command      = description of what should happen
Transport    = implementation detail
```

Examples:

* BLE pilot:

  * Device: “Pilot v1”
  * Commands: “button A”, “button B”, “mode X”
  * Transport: BLE + GATT

* GATT gateway:

  * Device: “Bulb #1”, “Bulb #2”, “Strip #1”
  * Command: “set brightness 50%”, “turn off”
  * Transport: GATT

* HID keyboard:

  * Device: “Host keyboard”
  * Command: “send key A”, “send ENTER”, “send Ctrl+Alt+Del”
  * Transport: USB HID

The controller never has to know whether a specific command is executed via BLE, GATT, HID, or something else – each channel is just an implementation of the same command interface.

---

## Repository layout (planned)

Under the `co-rc` organization:

* `android` / `corc-android` – controller app
* `ble-pilot` / `corc-ble-pilot` – BLE → GATT pilot
* `gatt-gateway` / `corc-gatt-gateway` – GATT → GATT fan-out gateway
* `hid-keyboard` / `corc-hid-keyboard` – USB HID keyboard bridge
* `protocol` / `corc-protocol` – shared protocol, model, and contracts
* `examples` – example integrations, sample configs, debug tools

Exact names may differ; check the organization page for the final layout.

---

## Getting started

> This section will evolve as the first reference implementations are published.

### Prerequisites

* Android development environment (Android Studio, recent SDK) for the controller app.
* BLE-capable hardware for the pilot.
* GATT-capable end devices for the gateway.
* A host with USB HID support for the keyboard bridge.

### Typical setup

1. Clone the relevant repositories from the `co-rc` organization.
2. Build and install the Android app on a device.
3. Deploy the BLE pilot to the target hardware.
4. Run the GATT gateway and/or USB HID keyboard bridge on the appropriate host.
5. Pair devices in the Android app and send commands.

As the project matures, this section will contain concrete build commands, example configs, and step-by-step guides.

---

## Roadmap (draft)

* [ ] Define and stabilize the core Corc protocol / command model.
* [ ] Publish the Android controller app skeleton.
* [ ] First BLE pilot reference implementation.
* [ ] First GATT gateway reference implementation.
* [ ] USB HID keyboard bridge (basic keys).
* [ ] Unified discovery and device registry.
* [ ] Example setups and demo scenarios.

---

## Contributing

Right now the project is in an early design phase.
If you are interested in contributing:

* open an issue in the relevant repository under `co-rc`,
* or start a discussion about use-cases and transports you would like to see.
