# docker-tor-hidden-service-v3

## What's New?

* This is the new v3 images. It creates the newer tor v3 (56 character) onions addresses. There is a bug/workaround though. In order to view your v3 addresses, you must run:

```docker exec [your tor container] v3onions```

I'm not a python wiz and it may take a while to make it as easy to use as the v2 onions, but I think this is a fair workaround for now.

* /version is a text file with the current of Tor version generated with each build 
* Weekly builds. The Goldy's original image hadn't been updated in some time. Using the latest version of Tor is always best practice.


Create a tor hidden service with a link

```sh
# run a container with a network application
$ docker run -d --name hello_world tutum/hello-world

# and just link it to this container
$ docker run -ti --link hello_world jsevans/tor-hidden-service-v3
```

The .onion URLs are displayed to stdout at startup.

To keep onion keys, just mount volume `/var/lib/tor/hidden_service/`

```sh
$ docker run -ti --link something --volume /path/to/keys:/var/lib/tor/hidden_service/ jsevans/tor-hidden-service-v3
```

Look at the `docker-compose.yml` file to see how to use it.

## Setup

### Set private key

Private key is settable by environment or by copying the file in `hostname/private_key` in docket volume (`hostname` is the link name).

It's easier to pass key in environment with `docker-compose`.

```yaml
    links:
      - hello
      - world
    environment:
        # Set private key
        HELLO_KEY: |
            -----BEGIN RSA PRIVATE KEY-----
            MIICXQIBAAKBgQDR8TdQF9fDlGhy1SMgfhMBi9TaFeD12/FK27TZE/tYGhxXvs1C
            NmFJy1hjVxspF5unmUsCk0yEsvEdcAdp17Vynz6W41VdinETU9yXHlUJ6NyI32AH
            dnFnHEcsllSEqD1hPAAvMUWwSMJaNmBEFtl8DUMS9tPX5fWGX4w5Xx8dZwIDAQAB
            AoGBAMb20jMHxaZHWg2qTRYYJa8LdHgS0BZxkWYefnBUbZn7dOz7mM+tddpX6raK
            8OSqyQu3Tc1tB9GjPLtnVr9KfVwhUVM7YXC/wOZo+u72bv9+4OMrEK/R8xy30XWj
            GePXEu95yArE4NucYphxBLWMMu2E4RodjyJpczsl0Lohcn4BAkEA+XPaEKnNA3AL
            1DXRpSpaa0ukGUY/zM7HNUFMW3UP00nxNCpWLSBmrQ56Suy7iSy91oa6HWkDD/4C
            k0HslnMW5wJBANdz4ehByMJZmJu/b5y8wnFSqep2jmJ1InMvd18BfVoBTQJwGMAr
            +qwSwNXXK2YYl9VJmCPCfgN0o7h1AEzvdYECQAM5UxUqDKNBvHVmqKn4zShb1ugY
            t1RfS8XNbT41WhoB96MT9P8qTwlniX8UZiwUrvNp1Ffy9n4raz8Z+APNwvsCQQC9
            AuaOsReEmMFu8VTjNh2G+TQjgvqKmaQtVNjuOgpUKYv7tYehH3P7/T+62dcy7CRX
            cwbLaFbQhUUUD2DCHdkBAkB6CbB+qhu67oE4nnBCXllI9EXktXgFyXv/cScNvM9Y
            FDzzNAAfVc5Nmbmx28Nw+0w6pnpe/3m0Tudbq3nHdHfQ
            -----END RSA PRIVATE KEY-----

```

Options are set using the following pattern: `LINKNAME_KEY`

### Setup port


__Caution__: Using `PORT_MAP` with multiple ports on single service will cause `tor` to fail.

Use link setting in environment with the following pattern: `LINKNAME_PORTS`.

Like docker, the first port is exposed port and the second one is service internal port.

```yaml
links:
  - hello
  - world
  - hey
environment:
    # Set mapping ports
    HELLO_PORTS: 80:80

    # Multiple ports can be coma separated
    WORLD_PORTS: 8000:80,8888:80,22:22

    # Socket mapping is supported
    HEY_PORTS: 80:unix:/var/run/socket.sock

```

__DEPRECATED:__
By default, ports are the same as linked containers, but a default port can be mapped using `PORT_MAP` environment variable.

#### Socket

To increase security, it's possible to setup your service through socket between containers and turn off network in your app container. See `docker-compose.v2.sock.yml` for an example.

__Warning__: Due to a bug in `tor` configuration parser, it's not possible to mix network link and socket link in the same `tor` configuration.

### Group services

Multiple services can be hosted behind the same onion address.

```yaml
links:
  - hello
  - world
  - hey
environment:
    # Set mapping ports
    HELLO_PORTS: 80:80

    # Multiple ports can be coma separated
    WORLD_PORTS: 8000:80,8888:80,22:22

    # Socket mapping is supported
    HEY_PORTS: 80:unix:/var/run/socket.sock

    # hello and world will share the same onion address
    # Service name can be any string as long there is not special char
    HELLO_SERVICE_NAME: foo
    WORLD_SERVICE_NAME: foo

```

__Warning__: Be carefull to not use the same exposed ports for grouped services.

### Compose v2 support

Links setting are required when using docker-compose v2. See `docker-compose.v2.yml` for example.

### Copose v3 support and secrets

Links setting are required when using docker-compose v3. See `docker-compose.v3.yml` for example.

#### Secrets

Secret key can be set through docker `secrets`, see `docker-compose.v3.yml` for example.

### Tools

A command line tool `v3onions` is available in the container to get the `.onion` url when the container is running.

```sh
# Get services
$ docker exec my_tor_container v3onions
/var/lib/tor/hidden_service/my_tor_container/hostname
p7gyaqryx6hru34lodxorn7cr6jglnpe3huwzqffo6mogwkfwn6d7iyd.onion
```

### Auto reload

Changing `/etc/tor/torrc` file trigger a `SIGHUP` signal to `tor` to reload configuration.

To disable this behavior, add `ENTRYPOINT_DISABLE_RELOAD` in environment.


### pyentrypoint

This container is using [`pyentrypoint`](https://github.com/cmehay/pyentrypoint) to generate its setup.

If you need to use the legacy version, please checkout the `legacy` branch or pull `goldy/tor-hidden-service:legacy`.
