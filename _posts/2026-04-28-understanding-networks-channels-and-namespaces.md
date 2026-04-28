---
layout: default
title: "Understanding Channels and Namespaces in RemotiveTopology"
date: 2026-04-28
categories: [RemotiveTopology, Tutorial]
---

When describing your vehicle architecture in [RemotiveTopology](https://www.remotivelabs.com/products/remotivetopology), two concepts are fundamental: **channels** and **namespaces**. Understanding how these work together is essential for accurately modeling your vehicle platform as code.

This article focuses on defining your topology architecture - the logical structure of ECUs and how they communicate. We won't cover physical network instantiation here; that comes later when you deploy your topology.

## Channels: Where ECUs Communicate

A **channel** defines where a group of ECUs communicate with each other. Think of it as a communication bus where multiple ECUs exchange messages. This concept aligns with the "physical channel" in AUTOSAR ARXML terminology.

In RemotiveTopology's platform definition, you describe channels by their type and the ECUs connected to them. The actual physical network details (like which CAN interface to use) are specified later during instantiation.

### Defining a CAN Channel

The simplest channel definition just specifies the type:

```yaml
schema: remotive-topology-platform:0.15
channels:
  DriverCan0:
    type: can
```

That's it! RemotiveTopology now knows there's a CAN channel called `DriverCan0` where ECUs can communicate.

You can add optional details if needed for configuration or clarity:

```yaml
channels:
  DriverCan0:
    type: can
    baudrate: 500000
    baudrate_fd: 2000000
    brs: true  # Bit rate switching for CAN FD
```

### Adding ECUs from DBC Files

DBC files describe CAN signals and ECUs, but they don't include the channel name. You specify which channel should use the DBC using the `database` field:

```yaml
schema: remotive-topology-platform:0.15
channels:
  DriverCan0:
    type: can
    database: ../databases/driver_can.dbc
    can_physical_channel_name: DriverCan0
```

RemotiveTopology automatically extracts the ECUs from the DBC and connects them to this channel. For example, if `driver_can.dbc` contains ECUs named BCM and SCCM, the resulting platform looks like:

```yaml
schema: remotive-topology-platform:0.15
channels:
  DriverCan0:
    type: can
    can_physical_channel_name: DriverCan0
    database: ../databases/driver_can.dbc
ecus:
  BCM:
    channels:
      DriverCan0:
        database: ../databases/driver_can.dbc
  SCCM:
    channels:
      DriverCan0:
        database: ../databases/driver_can.dbc
```

### Other Channel Types

RemotiveTopology supports various automotive protocols:

**LIN channels** require an LDF (LIN Description File):

```yaml
channels:
  RearLightLin:
    type: lin
    database: ../databases/rear_light.ldf
    baudrate: 19200
```

**Ethernet channels** need subnet information:

```yaml
channels:
  VehicleEthernet:
    type: ethernet
    subnet: 10.1.0.0/24
    vlan: 2  # optional
```

## Connecting ECUs to Channels

To connect an ECU to a channel, you simply specify that the channel exists for that ECU:

```yaml
ecus:
  BCM:
    channels:
      DriverCan0: {}
```

For Ethernet channels, you must also specify the ECU's IP address:

```yaml
ecus:
  BCM:
    channels:
      VehicleEthernet:
        config:
          type: ethernet
          host: 10.1.0.50
```

## Namespaces: How ECUs View Channels

Here's where it gets interesting. A **namespace** represents how an individual ECU interprets the signals on a channel. It's the ECU's "view" of the communication on that channel.

Think about it this way: multiple ECUs can be connected to the same CAN channel, but each ECU might have its own understanding of what the signals mean. The namespace is defined by the signal database (DBC, ARXML, etc.) that the ECU uses to decode messages.

### Why "Namespace"?

The term comes from programming, where namespaces prevent naming conflicts. In RemotiveTopology, a namespace ensures that when you refer to a signal like `TurnStalk.TurnSignal`, there's no ambiguity about which ECU's interpretation you mean.

A namespace is typically formatted as `<ECU>-<Channel>`, like `SCCM-DriverCan0` or `BCM-BodyCan0`.

### Namespaces in Platform Definition

When you specify a database for an ECU's channel connection, you're defining that ECU's namespace:

```yaml
ecus:
  BCM:
    channels:
      DriverCan0:
        database: ../databases/driver_can.dbc
  SCCM:
    channels:
      DriverCan0:
        database: ../databases/driver_can.dbc
```

Each ECU gets its own namespace based on the database it uses to interpret signals on `DriverCan0`. The resulting namespaces would be `BCM-DriverCan0` and `SCCM-DriverCan0`.

### Using Namespaces in Code

When you interact with signals using the RemotiveTopology framework, you reference them through their namespace:

```python
# Update signals in the SCCM namespace
await client.restbus.update_signals(
    (
        "SCCM-DriverCan0",  # namespace
        [
            RestbusSignalConfig.set(
                name="TurnStalk.TurnSignal", 
                value="LEFT"
            ),
        ],
    )
)

# Subscribe to signals in the BCM namespace  
sub = await broker_client.subscribe(
    (
        "BCM-BodyCan0",  # namespace
        [
            "TurnLightControl.LeftTurnLightRequest",
            "TurnLightControl.RightTurnLightRequest"
        ]
    )
)
```

### Overriding Namespace Databases

Sometimes you want to give an ECU only a partial view of a channel. You can override the database for a specific ECU's channel connection:

```yaml
channels:
  DriverCan0:
    type: can
    database: ../databases/driver_can.dbc  # Full channel view

ecus:
  TestECU:
    channels:
      DriverCan0:
        database: ../databases/limited_view.dbc  # Partial view
```

This is useful when testing ECUs that should only see a subset of the signals on a channel.

## Building Modular Platforms

One of RemotiveTopology's strengths is modularity. You can combine multiple platform files to build complex topologies from smaller subsystems:

```yaml
schema: remotive-topology-platform:0.15
includes:
  - ./lighting.platform.yaml
  - ./climate.platform.yaml
  - ./driver.platform.yaml
```

This is especially powerful when different teams work on different subsystems. Each team maintains their own platform definition, and you combine them to create the complete vehicle topology.

### Using ARXML

If you have ARXML files (ECU extracts or system descriptions), simply include them:

```yaml
schema: remotive-topology-platform:0.15
includes:
  - ../databases/body_control.arxml
  - ../databases/steering_control.arxml
```

ARXML contains channels, ECUs, and their connections - RemotiveTopology extracts all the necessary information automatically.

## Putting It All Together

Let's visualize how channels and namespaces work together:

```
┌─────────────────────────────────────────────────┐
│ Channel: DriverCan0                             │
│ Type: CAN                                       │
│ Database: driver_can.dbc                        │
└─────────────────────────────────────────────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
┌──────────────────┐  ┌──────────────────┐
│ Namespace        │  │ Namespace        │
│ SCCM-DriverCan0  │  │ BCM-DriverCan0   │
│                  │  │                  │
│ Signals:         │  │ Signals:         │
│ - TurnStalk.*    │  │ - VehicleSpeed   │
│ - WiperControl   │  │ - BrakeStatus    │
└──────────────────┘  └──────────────────┘
```

Each ECU connected to `DriverCan0` has its own namespace, allowing it to have its own interpretation of the signals on that channel.

## Why These Concepts Matter

Understanding channels and namespaces is crucial for several reasons:

### 1. Accurate Architecture Definition

When describing your vehicle platform, you define the logical structure - which ECUs communicate on which channels. This is independent of how you'll physically instantiate it later.

### 2. Flexibility in Testing

The same platform definition works whether you're:
- Running entirely virtually on your laptop
- Connecting to physical CAN interfaces
- Mixing virtual and hardware ECUs
- Running in CI/CD pipelines

The physical network mapping happens during instantiation, not in the platform definition.

### 3. ECU Isolation and Testing

By using namespaces, you can:
- Mock specific ECUs and control their signals
- Subscribe to signals from different ECUs separately
- Give test ECUs limited views of channels
- Test ECU interactions without ambiguity

### 4. Team Collaboration

Modular platform files let different teams work independently:
- The steering team maintains `steering.platform.yaml`
- The lighting team maintains `lighting.platform.yaml`  
- Integration testing combines all subsystems
- Changes to one subsystem don't break others

## Complete Example: Turn Signal System

Here's a complete platform definition for a turn signal system spanning two channels:

```yaml
schema: remotive-topology-platform:0.15

# Define the channels
channels:
  DriverCan0:
    type: can
    database: ../databases/driver_can.dbc
    can_physical_channel_name: DriverCan0
  
  BodyCan0:
    type: can
    database: ../databases/body_can.dbc
    can_physical_channel_name: BodyCan0

# Define the ECUs
ecus:
  # Steering Column Control Module - detects turn stalk position
  SCCM:
    channels:
      DriverCan0:
        database: ../databases/driver_can.dbc
  
  # Body Control Module - acts as gateway between channels
  BCM:
    channels:
      DriverCan0:
        database: ../databases/driver_can.dbc
      BodyCan0:
        database: ../databases/body_can.dbc
  
  # Front Light Control Module - controls turn lights
  FLCM:
    channels:
      BodyCan0:
        database: ../databases/body_can.dbc
  
  # Rear Light Control Module - controls turn lights  
  RLCM:
    channels:
      BodyCan0:
        database: ../databases/body_can.dbc
```

### What This Defines

**Channels:**
- `DriverCan0`: Communication between steering and body control
- `BodyCan0`: Communication between body control and lighting

**ECUs:**
- `SCCM`: Connected to DriverCan0, namespace `SCCM-DriverCan0`
- `BCM`: Connected to both channels, namespaces `BCM-DriverCan0` and `BCM-BodyCan0`
- `FLCM`: Connected to BodyCan0, namespace `FLCM-BodyCan0`
- `RLCM`: Connected to BodyCan0, namespace `RLCM-BodyCan0`

### Testing with This Platform

With this platform defined, you can write tests like:

```python
# Set turn stalk to LEFT in SCCM namespace
await client.restbus.update_signals(
    (
        "SCCM-DriverCan0",
        [
            RestbusSignalConfig.set(
                name="TurnStalk.TurnSignal", 
                value="LEFT"
            ),
        ],
    )
)

# Verify BCM received and forwarded the signal
# Subscribe to FLCM namespace to check light control
sub = await broker_client.subscribe(
    (
        "FLCM-BodyCan0",
        [
            "TurnLightControl.LeftTurnLightRequest",
            "TurnLightControl.RightTurnLightRequest"
        ]
    )
)

# Wait for expected values
await await_at_most(seconds=2).until(
    partial(take_values, sub),
    equal_to({
        "TurnLightControl.LeftTurnLightRequest": 1,
        "TurnLightControl.RightTurnLightRequest": 0
    }),
)
```

This test verifies the complete signal flow: SCCM → BCM (gateway) → FLCM, using namespaces to interact with each ECU's view of the channels.

## Viewing Your Platform

Use the RemotiveTopology CLI to inspect your platform definition:

```bash
$ remotive topology show platform lighting.platform.yaml
```

This displays all channels and ECUs, helping you verify your topology structure before instantiation.

## What's Next: Instantiation

The platform definition we've discussed describes the **architecture** of your vehicle topology - which ECUs exist and how they communicate logically. 

The next step is **instantiation** - actually running this topology. This is where you specify:
- Whether ECUs are mocks, behavioral models, or real hardware
- How to map channels to physical network interfaces
- Configuration for running in Docker containers
- Connection to CAN/LIN/Ethernet adapters

Instantiation is covered in a separate configuration (the instance file), keeping the architecture definition clean and reusable across different test environments.

## Conclusion

When defining your vehicle platform in RemotiveTopology:

- **Channels** define where ECUs communicate (the logical communication buses)
- **Namespaces** define how each ECU interprets signals on those channels (based on their signal databases)

This separation creates a flexible, modular architecture that works whether you're testing virtually on a laptop, in a CI/CD pipeline, or connected to real hardware. The platform definition stays the same - only the instantiation changes.

### Key Takeaways

1. Channels are logical groupings, not physical networks
2. ECUs connect to channels to communicate
3. Each ECU's namespace defines its view of a channel's signals
4. Platform files can be combined for modular development
5. Physical network details come later, during instantiation

By mastering channels and namespaces, you can describe complex vehicle architectures as infrastructure-as-code, enabling early integration testing and seamless transitions from virtual to physical testing environments.

---

*Want to try RemotiveTopology? Get a [free 30-day trial](https://docs.remotivelabs.com/docs/remotive-topology/install) and start defining your vehicle platform!*

*For more details, check out the [official platform documentation](https://docs.remotivelabs.com/docs/remotive-topology/usage/platform).*
