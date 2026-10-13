# gRPC Hands On

## Overview

[Lesson 10: APIs](../2026-04-09-Part-7-API.md) introduced gRPC as an API style that optimizes the efficiency of data transmission.
In this appendix we build a small gRPC example: a Python service acting as the server and a Go service acting as the client.
The client sends a question, and the server streams the answer back one word at a time, the way a chat assistant streams tokens.
Because the two services are written in different languages, the example also shows how a shared schema lets them talk to each other.

The complete code is in the [`notes/appendix/grpc101`](https://github.com/pockerman/distributed-systems-notes/tree/main/notes/appendix/grpc101) directory of this book's repository.
To run it, you need [Python](https://www.python.org/) and [Go](https://go.dev/) installed on your machine.

## gRPC 101

Google open-sourced gRPC in 2015, and version 1.0 was released in 2016 [1]. By default, gRPC uses [protocol buffers](https://protobuf.dev/) to encode the data it sends over the wire.
Protocol buffers are a platform- and language-neutral way to encode structured data, with a strongly typed schema. Below is an example of a message definition:

```protobuf
message Person {
  string name = 1;
  string surname = 2;
  int32 age = 3;
}
```

Each field has a type, a name, and a field number. The field number, not the name, identifies the field in the binary encoding, so it must not change once the message is in use.

::: {.callout-note}
Protocol buffers are serialized into a compact binary format before they are sent across the network. This makes messages smaller and faster to parse than JSON, but harder to read when debugging.
:::

A gRPC service is also defined in a `.proto` file. It lists the remote procedures the service offers, together with their request and response messages:

```protobuf
service RegisterPerson {
  rpc Register(Person) returns (RegistrationResponse) {}
}

message Person {
  string name = 1;
  string surname = 2;
  int32 age = 3;
}

message RegistrationResponse {
  string status = 1;
}
```

From this file, the protocol buffer compiler `protoc` generates code for each language: classes for the messages, a _stub_ that clients call as if it were a local object, and a base class that servers implement.

`Register` above is a _unary_ call: one request, one response. gRPC also supports _server streaming_ (one request, a stream of responses), _client streaming_, and _bidirectional streaming_, all carried over a single HTTP/2 connection (see [Lesson 11: HTTP](../2026-04-09-Part-8-HTTP.md)). Our example uses server streaming.

## The example

### The service definition

The two services share the following definition, in `proto/message.proto`:

```protobuf
syntax = "proto3";

package grpc101;

option go_package = "grpc101/proto;proto";

service Service {
  rpc Ask(Request) returns (stream Response);
}

message Request {
  string query = 1;
}

message Response {
  string response = 1;
}
```

The `stream` keyword in the return type makes `Ask` a server-streaming call.

### Generating the code

First install the tools. From the `grpc101` directory, install the Python packages (`requirements.txt` requires protobuf 6.31.1 or newer, the version the committed Python code was generated with), then the Go plugins for `protoc` (you also need `protoc` itself, see [Protocol Buffer Compiler Installation](https://protobuf.dev/installation/)):

```bash
pip install -r requirements.txt
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

Then create the output directories and compile the definition for both languages:

```bash
mkdir -p grpc101/go grpc101/python
protoc --go_out=./grpc101/go --go-grpc_out=./grpc101/go ./proto/message.proto
python -m grpc_tools.protoc -I./proto --python_out=./grpc101/python --grpc_python_out=./grpc101/python ./proto/message.proto
```

The repository already contains the generated code, so you can skip this step unless you change `message.proto`.

### The Python server

The server implements the generated `ServiceServicer` base class. Because `Ask` is a server-streaming call, the method is a generator: every `yield` sends one `Response` message to the client. The `time.sleep` call simulates a model producing one token at a time.

```python
import time
import grpc
from concurrent import futures

import grpc101.python.message_pb2 as message_pb2
import grpc101.python.message_pb2_grpc as message_pb2_grpc


class Service(message_pb2_grpc.ServiceServicer):

    def Ask(self, request, context):
        print(f"Received Query: {request.query}")

        text = f"Answer for '{request.query}'"
        text += " I don't know what are you talking about"

        # simulate streaming tokens
        for word in text.split():
            yield message_pb2.Response(response=word + " ")
            time.sleep(0.1)


def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    message_pb2_grpc.add_ServiceServicer_to_server(Service(), server)

    server.add_insecure_port("[::]:50051")
    server.start()

    print("Python gRPC server running on :50051")

    server.wait_for_termination()


if __name__ == "__main__":
    serve()
```

The server listens on port 50051 without TLS (`add_insecure_port`), which is fine for a local example but not for production; see [Lesson 7: TLS 101](../2026-04-06-Part-5-TLS-101.md).

### The Go client

The client creates a channel to the server with `grpc.NewClient`, creates a stub with `NewServiceClient`, and calls `Ask`. `NewClient` does not connect straight away: the connection is opened on the first call, so an unreachable server shows up as an error from `Ask` rather than from `NewClient`. Like the server, the client does not use TLS here (`insecure.NewCredentials()`). Instead of a single response, it gets a stream, and reads messages from it with `Recv` until the server closes the stream (`io.EOF`).
The call runs under a context with a 10-second timeout, which gRPC sends to the server as a deadline; see the discussion of timeouts in [Lesson 32: Downstream Resiliency](../2026-02-04-Downstream-Resiliency.md).

```go
package main

import (
	"context"
	"fmt"
	"io"
	"log"
	"time"

	pb "grpc101/grpc101/go/grpc101/proto"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

func main() {
	conn, err := grpc.NewClient("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatal("could not create client:", err)
	}
	defer conn.Close()

	client := pb.NewServiceClient(conn)

	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	stream, err := client.Ask(ctx, &pb.Request{
		Query: "Explain DDD to me",
	})
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Streaming response:")

	for {
		res, err := stream.Recv()
		if err == io.EOF {
			break
		}
		if err != nil {
			log.Fatal(err)
		}

		fmt.Print(res.Response)
	}

	fmt.Println("\nDone.")
}
```

### Running the example

Open a terminal in the `grpc101` directory and start the server:

```bash
PYTHONPATH=./grpc101/python python server.py
```

The generated `message_pb2_grpc.py` imports `message_pb2` as a top-level module, which is why `grpc101/python` has to be on the `PYTHONPATH`.

Then open a second terminal in the same directory and run the client. The repository already contains `go.mod`, so `go mod tidy` is enough to download the dependencies:

```bash
go mod tidy
go run client.go
```

The client prints the answer word by word as the server streams it:

```text
Streaming response:
Answer for 'Explain DDD to me' I don't know what are you talking about
Done.
```

## Summary

::: {.callout-note title="Summary"}
gRPC is a remote procedure call framework that uses protocol buffers to define services and messages in a language-neutral `.proto` file and to encode them in a compact binary format, and HTTP/2 to carry the calls. The `protoc` compiler generates client stubs and server base classes for each language, so services written in different languages can call each other as if they were local objects. Besides simple request-response (unary) calls, gRPC supports client, server, and bidirectional streaming. In this appendix a Python server implements a server-streaming `Ask` call that yields its answer one word at a time, and a Go client calls it with a deadline and reads the stream until the server closes it.
:::

## References

1. [About gRPC](https://grpc.io/about/)
2. [gRPC documentation](https://grpc.io/docs/)
3. [Protocol Buffers documentation](https://protobuf.dev/)
4. [What is gRPC? (Remote Procedure Calls)](https://www.youtube.com/watch?v=hVrwuMnCtok), YouTube
5. [What is RPC? gRPC Introduction.](https://www.youtube.com/watch?v=gnchfOojMk4), YouTube
