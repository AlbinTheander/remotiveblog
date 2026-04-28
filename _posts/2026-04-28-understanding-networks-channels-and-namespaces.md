---
layout: default
title: "Understanding networks, channels and namespaces in RemotiveTopology"
date: 2026-04-28
categories: [RemotiveTopology, Tutorial]
---

One of the more mysterious concepts when starting to work with [RemotiveTopology](https://www.remotivelabs.com/products/remotivetopology) is namespaces. In this article, we try to
shed some light over how networks, channels and namespaces relate to each other.

## Channels: Where ECUs Communicate

A **channel** defines where a group of ECUs communicate with each other. Think of it as a communication bus where multiple ECUs exchange messages. This concept aligns with the "physical channel" in AUTOSAR ARXML terminology.

It's important to understand that a channel is a logical part of the architecture. It doesn't always correspond to a physical network. For example, several ethernet VLAN channels could use the same physical ethernet network.

## Namespaces: How ECUs view channels

A **namespace** represents how an individual ECU interprets the signals on a channel. It's the ECU's "view" of the communication on that channel.

Think about it this way: multiple ECUs can be connected to the same CAN channel, but each ECU might have its own understanding of what the signals mean. The namespace is defined by the signal database (DBC, ARXML, etc.) that the ECU uses to decode messages.

One ECU can even have multiple namespaces on the same channel. For point-to-point ethernet, each connection to another ECU is its own namespace. The same frame ID could mean different things depending on from which ECU the frame originates.

The word "**namespace**" is used since each namespace guarantees the uniqueness of the signal names and ids.

## Networks: Virtual or physical media to transport data

A **network** is a way of actually transporting data. While channels are logical building blocks to design an architecture, networks are physical entities that are used when the topology is up and running.

RemotiveTopology uses Linux network interfaces to interact with networks. The traffic that is flowing on these interfaces is real. You can use Wireshark or tcpdump to inspect the traffic as you would when working with real hardware.

A RemotiveTopology instance is what connects the logical channel concept to real world network interfaces on your computer.

Lets look at an example.

## Example: DriverCan, a CAN channel

Very often, all or most of the channel information is defined in arxml files. When using other signal database formats, you can add more information in platform files of RemotiveTopology.

Here is a the definition of a channel that is defined in dbc file.

```yaml
schema: remotive-topology-platform:0.15
channels:
  DriverCan0:
    type: can
    database: ./databases/driver_can.dbc
    can_physical_channel_name: DriverCan0
```

We need to add the `can_physical_channel_name` property, that would have been specified in an arxml file, since the dbc format doesn't include the name of the channel.

You can add optional details if needed:

```yaml
channels:
  DriverCan0:
    type: can
    database: ./databases/driver_can.dbc
    can_physical_channel_name: DriverCan0
    baudrate: 500000
    baudrate_fd: 2000000
    brs: true  # Bit rate switching for CAN FD
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

(In reality, there is more information in a platform. To see all properties, you can use the RemotiveCLI tool and run `remotive topology show platform <file.platform.yaml>`.


{: .note}
> When you write the platform files, you don't need to specify information that is automatically extracted. The above example is equivalent to
> ```yaml
> schema: remotive-topology-platform:0.15
> channels:
>   DriverCan0:
>     type: can
>     can_physical_channel_name: DriverCan0
>     database: ../databases/driver_can.dbc
> ```
> or, if you have an arxml that contains all the information, it could be suffice with
> ```yaml
> schema: remotive-topology-platform:0.15
> include:
>   ./databases/driver_can.arxml
> ```

Finally, we need an instance when executing this instance. The instance describes how `DriverCan0`is connected to a network interface on your computer. In our case, we have a physical CAN interface that is connected to the `lin5` socketCAN network interface. We will use the `remotivebus` driver to communicate with `lin5`.

```yaml
schema: remotive-topology-instance:0.15
platform:
  include: ./driver_can.platform.yaml

channels:
  DriverCan0:
    type: can
    driver:
      type: remotivebus
      config:
        type: can
        device: myvlin
        host_device: lin5
```

### Example: Using namespace to interact with a channel

Namespaces are used by code to interact with a channel as an ECU. This can be from a behavioral model for that ECU or for a test case where you want to read data from a channel as an ECU.

Namespaces get names from the ECU and the channel. In the example above, there would be two namespaces: `BCM-DriverCan0` and `SCCM-DriverCan0`.

These can be used to read or write data:

```python
# Subscribe to signals in the BCM namespace  
sub = await broker_client.subscribe(
    (
        "BCM-DriverCan0",  # namespace
        [
            "TurnLightControl.LeftTurnLightRequest",
            "TurnLightControl.RightTurnLightRequest"
        ]
    )
)

# Publish data to the SCCM namespace
await broker_client.publish(
    "SCCM-DriverCan0",
    WriteSignal("TurnLightControl.LeftTurnLightRequest", 1)
)
```

## Conclusion

When defining your vehicle platform in RemotiveTopology:

- **Channels** define where ECUs communicate (the logical communication buses)
- **Namespaces** define how each ECU interprets signals on those channels (based on their signal databases)
- **Networks** are real world entities that can transport data. When realizing logical channels, they get connected to networks.


### Key Takeaways

1. Channels are logical groupings, not physical networks
2. ECUs connect to channels to communicate
3. Each ECU's namespace defines its view of a channel's signals
4. Platform files define channels and ecus in an architecture
5. Physical networks are used to implement channels when realizing the platform
6. An instance file describes how channels are implemented as networks.

By mastering channels and namespaces, you can describe complex vehicle architectures as infrastructure-as-code and mastering instances allows you to realize the channels with real life networks.

---

*Want to try RemotiveTopology? Get a [free 30-day trial](https://docs.remotivelabs.com/docs/remotive-topology/install) and start defining your vehicle platform!*

*For more details, check out the [official platform documentation](https://docs.remotivelabs.com/docs/remotive-topology/usage/platform).*
