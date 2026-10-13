# gRPC Hands On: sample code

A Python gRPC server that streams its answer, one word at a time, to a Go client.
The walk-through is in [`../2026-05-05-API-Basics-gRPC-101.md`](../2026-05-05-API-Basics-gRPC-101.md).
Run all the commands below from this directory.

## Install the tools

You need Python, Go and `protoc` installed.

```bash
pip install -r requirements.txt
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

## Generate the code

The generated code is already committed, so this step is only needed after changing `proto/message.proto`.

```bash
mkdir -p grpc101/go grpc101/python
protoc --go_out=./grpc101/go --go-grpc_out=./grpc101/go ./proto/message.proto
python -m grpc_tools.protoc -I./proto --python_out=./grpc101/python --grpc_python_out=./grpc101/python ./proto/message.proto
```

## Run the Python server

```bash
PYTHONPATH=./grpc101/python python server.py
```

## Run the Go client

In a second terminal:

```bash
go mod tidy
go run client.go
```
