# Polyglot - a universal grpc client

Polyglot is a grpc client which can talk to any grpc server. In order to make a call, the following are required:
* A compiled Polyglot binary, 
* the .proto files for the service,
* and a request proto instance in text format.

In particular, it is not necessary to generate grpc classes for the service or to compile the protos into the Polyglot binary.

## Status

Polyglot now has most of the features for min-viable ability to talk to any server. As such, it is usable but a bit rough around the edges, more usability improvements are on their way.

## Usage

### Basics

For now, it is required to build polyglot before usage (currently only works on Linux):

`$ bazel build src/main/java/polyglot`

After calling this, you should have a fresh binary at:

`./bazel-bin/src/main/java/polyglot/polyglot`

Then, the binary itself can be invoked as follows:

```
$ echo <json-request> | polyglot \
    --endpoint=<host>:<port> \
    --full_method=<some.package.Service/doSomething> \
    --proto_discovery_root=<path>
```

For an example, see `run-client-example.sh`.

### Configuration

Some of the features of Polyglot (such as Oauth, see below) require some configuration. Moreover, that sort of configuration tends to remain constant across multiple Polyglot runs. In order to improve usability, Polyglot supports loading a configuration proto from a Json file at runtime. This configuration file can contain multiple `Configuration` objects (schema defined [here](https://github.com/dinowernli/polyglot/blob/master/src/main/proto/config.proto#L14)). An example configuration could look like this:

```
{
  "configurations": [
    {
      "name": "production",
      "call_config": {
        "use_tls": "true",
        "oauth_config": {
          "refresh_token_credentials": {
            "token_endpoint_url": "https://auth.example.com/token",
            "client": {
                "id": "example_id",
                "secret": "example_secret"
            },
            "refresh_token_path": "/path/to/refresh/token"
          }
        }
      },
      "proto_config": {
        "proto_discovery_root": "/home/dave/protos",
        "include_paths": [
          "/home/dave/lib"
        ]
      }
    },
    {
      "name": "staging",
      "call_config": {
        "oauth_config": {
          "refresh_token_credentials": {
            "token_endpoint_url": "https://auth-staging.example.com/token"
          }
        }
      },
      "proto_config": {
        "proto_discovery_root": "/home/dave/staging/protos",
        "include_paths": [
          "/home/dave/staging/lib"
        ]
      }
    }
  ]
}
```

By default, Polyglot tries to find a config file at `$HOME/.polyglot/config.pb.json`, but this can be overridden with the `--config_set_path` flag. By default, Polyglot uses the first configuration in the set, but this can be overridden with the `--config_name` flag.

The general philosophy is for the configuration to drive Polyglot's behavior and for command line flags to allow selectively overriding parts of the configuration. For a full list of what can be configured, please see [`config.proto`](https://github.com/dinowernli/polyglot/blob/master/src/main/proto/config.proto#L14).

### Using TLS

Polyglot uses statically linked boringssl libraries under the hood and doesn't require the host machine to have any specific libraries. Whether or not the client uses TLS to talk to the server can be controlled using the `--use_tls` flag or the corresponding configuration entry.

### Authenticating requests using OAuth

Polyglot has built-in support for authentication of requests using OAuth tokens in two ways:
* Loading an access token from disk and attaching it to the request.
* Loading a refresh token from disk, exchanging it for an access token, and attaching the access token to the request.

In order to use this feature, Polyglot needs an `OauthConfiguration` inside its `Configuration`. For details on how to populate the `OauthConfiguration`, please see the documentation of the fields in [`config.proto`](https://github.com/dinowernli/polyglot/blob/master/src/main/proto/config.proto#L14).

## Build requirements

We do not yet have a binary distribution, so right now it is required to build the binary from source in order to use it. The build requirements are:

* Java 8
* Bazel

## Run the example

First, start the server: 

`$ ./run-server.sh`

Then, in a different terminal, run the client example:

`$ ./run-client-example.sh`

## Build and run tests

`$ bazel test ...`

## Features

* OAuth integration for authenticated requests
* Ability to parse .proto files at runtime
* Support for receiving streams of responses

## Upcoming features

* Executing the full OAuth dance (rather than relying on a refresh token on disk)
* Binary distribution
* Building on Mac and Windows
