# How Does gRPC Work?

This note is adapted from ByteByteGo's _How does gRPC work?_ guide (2024) [1]. See [Part 7 | APIs](../2026-04-09-Part-7-API.md) for an introduction to gRPC.

![The data flow of a gRPC call. Image from ByteByteGo [1].](0210-grpc.png)

RPC (Remote Procedure Call) is called "remote" because it enables communication between services deployed on different servers, as in a microservice architecture. From the caller's point of view, it looks like a local function call.

The diagram above illustrates the overall data flow of a **gRPC** call:

- Step 1: the client makes a REST call to the order service. The request body is usually JSON.
- Steps 2–4: the order service (the gRPC client) receives the REST call, transforms it, and calls the payment service through its **client stub**. The stub encodes the request message into the binary protocol buffers format and hands it to the transport layer.
- Step 5: gRPC sends the packets over the network via HTTP/2. Thanks to binary encoding and HTTP/2 features such as multiplexing and header compression, gRPC is typically much faster than JSON over HTTP/1.1 (ByteByteGo quotes about 5x, but the actual gain depends on the payload and workload).
- Steps 6–8: the payment service (the gRPC server) receives the packets from the network, decodes them, and invokes the server application.
- Steps 9–11: the result is returned from the server application, encoded, and sent to the transport layer.
- Steps 12–14: the order service receives the packets, decodes them, and returns the result to the client application.

## References

1. ByteByteGo, _How does gRPC work?_, 2024
