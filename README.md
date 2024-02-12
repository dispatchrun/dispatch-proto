[![Build](https://github.com/stealthrocket/dispatch-sdk-protobuf/actions/workflows/buf.yml/badge.svg)](https://github.com/stealthrocket/dispatch-sdk-protobuf/actions/workflows/buf.yml)
[![MIT License](https://img.shields.io/badge/license-Apache%202-blue.svg)](LICENSE)
[![Docs](https://img.shields.io/badge/API-reference-lightblue.svg)](https://buf.build/stealthrocket/dispatch-sdk/docs/main:dispatch.sdk.v1)
<img align="right" src="https://github.com/stealthrocket/dispatch-sdk-protobuf/assets/865510/d1079e83-d283-4351-9660-8abab0f23d66" width="250"/>

# Dispatch SDK

This module contains the protobuf definitions for the Dispatch SDK.

[connectrpc]:   https://connectrpc.com/
[grpc]:         https://grpc.io/
[signup]:       https://docs.stealthrocket.cloud/dispatch/getting-started
[rpc-dispatch]: https://buf.build/stealthrocket/dispatch-sdk/docs/main:dispatch.sdk.v1#dispatch.sdk.v1.DispatchService.Dispatch
[rpc-function]: https://buf.build/stealthrocket/dispatch-sdk/docs/main:dispatch.sdk.v1#dispatch.sdk.v1.FunctionService.Run

## What is Dispatch?

Dispatch is a platform to develop reliable distributed systems. Dispatch
provides a simple programming model based on durable coroutines to manage the
scheduling of function calls across a fleet of service instances. Orchestration
of function calls is managed by Dispatch, providing **fair scheduling**,
transparent **retry of failed operations**, and **durability**.

To interact with the Dispatch scheduler, the client SDK uses this module to
generate [connectrpc][connectrpc] or [gRPC][grpc] clients and servers.

## Authentication

To authenticate with the Dispatch control plane, a client must present a valid
API key in the `Authorization` header of the HTTP requests, for example:

```sh
curl https://api.stealthrocket.cloud/dispatch.sdk.v1.DispatchService/Dispatch \
    -H "Authorization: Bearer $DISPATCH_API_KEY" \
    ...
```

To obtain an API key, follow the instructions to [sign up for Dispatch][signup] 🚀.

## Dispatching Calls to Functions

The dispatch service is the frontend to the platform. Clients can submit calls
to functions implemented in their application, using the
[`dispatch.sdk.v1.DispatchService/Dispatch`][rpc-dispatch] method.

The request contains the list of calls that will be performed asynchronously by
the scheduler.

## Running Function Calls

Function endpoints are implemented by exposing the
[`dispatch.sdk.v1.FunctionService/Run`][rpc-function]
method on a [connectrpc][connectrpc] or [gRPC][grpc] endpoint.

The scheduler will make the function calls that were submitted via the API.

### Verification of Function Calls

When calling functions, requests are signed using asymmetric key pairs. The private
key is stored on the Dispatch platform, and the server uses the private key to
verify the signature of requests it receives.

The signature is embedded in a `Signature` header, for example:

    Signature: <signature>

The request signature is generated using the ed25519 cryptographic function,
using the request body as input.

## Contributing

Contributions are always welcome! Would you spot a typo or anything that needs
to be improved, feel free to send a pull request.

Pull requests need to pass all CI checks before getting merged. The buf linters
run on every code push and will detect any breaking changes that are made to the
generated code or the wire format. In general, we never accept breaking changes,
but there are cases where we might make exceptions for fixes to the generated
code because consumers are expected to pin their dependency and are responsible
for updating their code when needed. Breaking changes to the wire format are
never acceptable.

Remember to be respectful and open minded!
