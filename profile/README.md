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
