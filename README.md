# gRPC Elixir

[![Hex.pm](https://img.shields.io/hexpm/v/grpc.svg)](https://hex.pm/packages/grpc)
[![Travis Status](https://travis-ci.org/elixir-grpc/grpc.svg?branch=master)](https://travis-ci.org/elixir-grpc/grpc)
[![GitHub actions Status](https://github.com/elixir-grpc/grpc/workflows/CI/badge.svg)](https://github.com/elixir-grpc/grpc/actions)

An Elixir implementation of [gRPC](http://www.grpc.io/).

**WARNING: Be careful to use it in production! Test and benchmark in advance.**

**NOTICE: Erlang/OTP needs >= 22 **

**NOTICE: gun**

The Gun library doesn't have a full 2.0 release yet, so we depend on `:grcp_gun 2.0.1` for now.
This is the same as `:gun 2.0.0-rc.2`, but Hex doesn't let us depend on RC versions for releases.

## Installation

The package can be installed as:

  ```elixir
  def deps do
    [
      {:grpc, "~> 0.5.0"},
      # We don't force protobuf as a dependency for more
      # flexibility on which protobuf library is used,
      # but you probably want to use it as well
      {:protobuf, "~> 0.10"}
    ]
  end
  ```

## Usage

1. Generate Elixir code from proto file as [protobuf-elixir](https://github.com/tony612/protobuf-elixir#usage) shows(especially the `gRPC Support` section).
2. Implement the server side code like below and remember to return the expected message types.
```elixir
defmodule Helloworld.Greeter.Server do
  use GRPC.Server, service: Helloworld.Greeter.Service

  @spec say_hello(Helloworld.HelloRequest.t, GRPC.Server.Stream.t) :: Helloworld.HelloReply.t
  def say_hello(request, _stream) do
    Helloworld.HelloReply.new(message: "Hello #{request.name}")
  end
end
```

3. Start the server

You can start the gRPC server as a supervised process. First, add `GRPC.Server.Supervisor` to your supervision tree.

```elixir
# Define your endpoint
defmodule Helloworld.Endpoint do
  use GRPC.Endpoint

  intercept GRPC.Logger.Server
  run Helloworld.Greeter.Server
end

# In the start function of your Application
defmodule HelloworldApp do
  use Application
  def start(_type, _args) do
    children = [
      # ...
      {GRPC.Server.Supervisor, endpoint: Helloworld.Endpoint, port: 50051}
    ]

    opts = [strategy: :one_for_one, name: YourApp]
    Supervisor.start_link(children, opts)
  end
end
```

4. Call rpc:
```elixir
iex> {:ok, channel} = GRPC.Stub.connect("localhost:50051")
iex> request = Helloworld.HelloRequest.new(name: "grpc-elixir")
iex> {:ok, reply} = channel |> Helloworld.Greeter.Stub.say_hello(request)

# With interceptors
iex> {:ok, channel} = GRPC.Stub.connect("localhost:50051", interceptors: [GRPC.Logger.Client])
...
```

Check [examples](examples) and [interop](interop)(Interoperability Test) for some examples.

## TODO

- [x] Unary RPC
- [x] Server streaming RPC
- [x] Client streaming RPC
- [x] Bidirectional streaming RPC
- [x] Helloworld and RouteGuide examples
- [x] Doc and more tests
- [x] Authentication with TLS
- [x] Improve code generation from protos ([protobuf-elixir](https://github.com/tony612/protobuf-elixir) [#8](https://github.com/elixir-grpc/grpc/issues/8))
- [x] Timeout for unary calls
- [x] Errors handling
- [x] Benchmarking
- [x] Logging
- [x] Interceptors(See `GRPC.Endpoint`)
- [x] [Connection Backoff](https://github.com/grpc/grpc/blob/master/doc/connection-backoff.md)
- [x] Data compression
- [x] Support other encoding(other than protobuf)

## Benchmark

1. [Simple benchmark](examples/helloworld/README.md#Benchmark) by using [ghz](https://ghz.sh/)

2. [Benchmark](benchmark) followed by official spec

## Sponsors

This project is being sponsored by [Tubi](https://tubitv.com/). Thank you!

<img src="https://user-images.githubusercontent.com/1253659/37473536-4db44048-28a9-11e8-90d5-f8a2f5a8d53c.jpg" height="80">

## Contributing

You contributions are welcome!

Please open issues if you have questions, problems and ideas. You can create pull
requests directly if you want to fix little bugs, add small features and so on.
But you'd better use issues first if you want to add a big feature or change a
lot of code.
