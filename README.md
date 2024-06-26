<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github.com/dispatchrun/.github/blob/main/profile/dispatch_logo_dark.png?raw=true">
    <img alt="dispatch logo" src="https://github.com/dispatchrun/.github/blob/main/profile/dispatch_logo_light.png?raw=true" height="64">
  </picture>
</p>

# dispatch-proto

[![Build](https://github.com/dispatchrun/dispatch-proto/actions/workflows/buf.yml/badge.svg)](https://github.com/dispatchrun/dispatch-proto/actions/workflows/buf.yml)
[![Docs](https://img.shields.io/badge/API-reference-lightblue.svg)](https://buf.build/stealthrocket/dispatch-proto/docs/main:dispatch.sdk.v1)

This module contains the protobuf definitions to integrate with Dispatch.

[connectrpc]:      https://connectrpc.com/
[grpc]:            https://grpc.io/
[http-signatures]: https://datatracker.ietf.org/doc/draft-ietf-httpbis-message-signatures/19/
[signup]:          https://docs.dispatch.run/getting-started
[rpc-dispatch]:    https://buf.build/stealthrocket/dispatch-proto/docs/main:dispatch.sdk.v1#dispatch.sdk.v1.DispatchService.Dispatch
[rpc-function]:    https://buf.build/stealthrocket/dispatch-proto/docs/main:dispatch.sdk.v1#dispatch.sdk.v1.FunctionService.Run

- [What is Dispatch?](#what-is-dispatch)
- [Authentication](#authentication)
- [Dispatching Calls to Functions](#dispatching-calls-to-functions)
- [Running Function Calls](#running-function-calls)
  - [Verification of Function Calls](#verification-of-function-calls)
- [Contributing](#contributing)

## What is Dispatch?

Dispatch is a cloud service for developing scalable and reliable applications,
including:

- **Event-Driven Architectures**
- **Background Jobs**
- **Transactional Workflows**
- **Multi-Tenant Data Pipelines**

Dispatch differs from alternative solutions by allowing developers top write
simple Python code: it has a **minimal API footprint**, which usually only
requires using a function decorator (no complex framework to learn), failure
recovery is built-in by default for transient errors like rate limits or
timeouts, with a **zero-configuration** model.

To interact with the Dispatch scheduler, the client SDK uses this module to
generate [connectrpc][connectrpc] or [gRPC][grpc] clients and servers.

## Authentication

To authenticate with the Dispatch control plane, a client must present a valid
API key in the `Authorization` header of the HTTP requests, for example:

```sh
curl https://api.dispatch.run/dispatch.sdk.v1.DispatchService/Dispatch \
    -H "Authorization: Bearer $DISPATCH_API_KEY" \
    ...
```

To obtain an API key, follow the instructions to [sign up for Dispatch][signup] 🚀.

## Dispatching Calls to Functions

Clients can submit calls to functions implemented in their application, using
the [`dispatch.sdk.v1.DispatchService/Dispatch`][rpc-dispatch] method.

The request contains the list of calls that will be performed asynchronously by
the scheduler.

## Running Function Calls

Function endpoints are implemented by exposing the
[`dispatch.sdk.v1.FunctionService/Run`][rpc-function]
method on a [connectrpc][connectrpc] or [gRPC][grpc] endpoint.

The scheduler will make the function calls that were submitted via the API.

### Verification of Function Calls

When calling functions, requests are signed using asymmetric key pairs. The private
key is stored by Dispatch, and the server uses the private key to verify the
signature of requests it receives.

The signature mechanism uses the [HTTP Signatures IETF Draft][http-signatures],
with the following signing fields:

    @method, @path, @authority, content-type, content-digest

The request signature is generated using the ed25519 cryptographic function,
the `keyid` in the signature is set to `default`.

### HTTP Status Codes

The protocol uses HTTP status codes to communicate errors that occurred before
any function could be run (e.g., due to invalid requests).

| Status | Reason                                             |
| :----- | :------------------------------------------------- |
| 200    | The requested function ran successfuly             |
| 400    | Malformed requests (e.g., missing function name)   |
| 401    | Missing or malformed signature in the HTTP request |
| 403    | An invalid signature was found in the HTTP request |
| 404    | The requested function did not exist               |

When responses containing those status codes are returned, the content type is
**application/json** and the body is a JSON object with this structure:
```json
{
  "code": "unauthenticated",
  "message": "missing request signature"
}
```

This format follows the [connectrpc][connectrpc] protocol, the **code** is set
according to the [http-to-error-code](https://connectrpc.com/docs/protocol/#http-to-error-code)
specification, and the **message** contains a description of the error intended
to provide operators insight into the reason why the request failed.

Proxies on the request path can also return other HTTP status codes such as
429 to apply rate limits or 504 if a timeout occurred.

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
