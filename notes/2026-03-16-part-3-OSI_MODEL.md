# Part 3 | Inter-Process Communication

## Overview

Distributed systems are made of processes that communicate over a network. Inter-process communication (IPC) is therefore at the heart of a distributed system,
and we need to understand the communication stack when designing one. In this note
we give a high-level overview of what happens behind the scenes when a process communicates with another process.

## Inter-process communication

When a process sends data to another process, the two must follow a
protocol in order to communicate effectively.
Communication protocols are arranged in a stack, where each layer uses the abstractions provided by the layer below it.
Lower layers are closer to the hardware.

One such model is the OSI, or Open Systems Interconnection, model [1].
In this model, the communication stack is split into seven abstraction layers:

1. Physical
2. Data link
3. Network
4. Transport
5. Session
6. Presentation
7. Application

According to [1]:

> The model describes communications from the physical implementation of transmitting bits across a transmission medium to the highest-level representation of data of a distributed application. Each layer has well-defined functions and semantics and serves a class of functionality to the layer above it and is served by the layer below it. Established, well-known communication protocols are decomposed in software development into the model's hierarchy of function calls.

The internet, however, is built on the simpler TCP/IP (internet protocol suite) model, which has four layers: application, transport, internet, and link [2].
Let's follow the journey an HTTP request takes to reach another process through these layers.

- **Application layer.** The request begins here. This is the layer where most software engineers work, and it defines protocols such as HTTP and DNS. It roughly corresponds to the session, presentation, and application layers of the OSI model.
- **Transport layer.** This layer transmits data between two processes, using port numbers to address them. The most important protocol at this layer is the [Transmission Control Protocol (TCP)](https://en.wikipedia.org/wiki/Transmission_Control_Protocol); UDP is another.
- **Internet layer.** This layer routes packets from one machine to another across networks. The [Internet Protocol (IP)](https://en.wikipedia.org/wiki/Internet_Protocol) is the core protocol at this layer [2]. It corresponds to the OSI network layer.
- **Link layer.** This layer operates on local network links, e.g. Ethernet or Wi-Fi, and is where the hardware lives. Switches at this layer forward
frames based on [MAC addresses](https://en.wikipedia.org/wiki/MAC_address) [2]. It corresponds to the OSI data link and physical layers.

Although a lot of engineering effort has gone into making these abstractions robust and secure, abstractions do leak. Troubleshooting network issues requires
that we understand some of the details of the layers below the one we are working on.

## Summary

::: {.callout-note title="Summary"}
Inter-process communication (IPC) is a core component of distributed systems, enabling processes on different machines to exchange data over a network using protocols organized in layers. The OSI model describes this layered approach with seven layers, from the physical transmission of bits up to the application-level representation of data. The internet uses the simpler four-layer TCP/IP model. When a request such as an HTTP request is sent, it travels through the application layer (protocols like HTTP and DNS), the transport layer (e.g., TCP, using ports for process-to-process communication), the internet layer (IP, routing packets between machines), and the link layer (technologies like Ethernet or Wi-Fi, using MAC addresses). While these abstractions simplify development, they leak in practice, so engineers need to understand the lower layers when diagnosing issues in distributed systems.
:::

The next part discusses [TCP](2026-03-17-part-4-TCP_101.md).

## References

1. [OSI model](https://en.wikipedia.org/wiki/OSI_model)
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. [Internet protocol suite](https://en.wikipedia.org/wiki/Internet_protocol_suite)
