<h1 align="center">
  <img src="./docs/logo-horizontal.png" alt="Smocker" height="100" title="Smocker logo by mandyellow" />
</h1>

[![CI](https://github.com/smocker-dev/smocker/actions/workflows/main.yml/badge.svg)](https://github.com/smocker-dev/smocker/actions/workflows/main.yml)
[![Docker Repository](https://img.shields.io/badge/ghcr.io%2Fsmocker--dev%2Fsmocker-blue?logo=docker&label=docker)](https://github.com/smocker-dev/smocker/pkgs/container/smocker)
[![Github Release](https://img.shields.io/github/v/release/smocker-dev/smocker.svg?logo=github)](https://github.com/smocker-dev/smocker/releases/latest)
[![Go Report Card](https://goreportcard.com/badge/github.com/smocker-dev/smocker)](https://goreportcard.com/report/github.com/smocker-dev/smocker)
[![License](https://img.shields.io/github/license/smocker-dev/smocker?logo=open-source-initiative)](https://github.com/smocker-dev/smocker/blob/main/LICENSE)

**Smocker** (server mock) is a simple and efficient HTTP mock server.

The documentation is available on [smocker.dev](https://smocker.dev).

## Table of contents

- [Installation](#installation)
  - [With Docker](#with-docker)
  - [Manual Deployment](#manual-deployment)
  - [Healthcheck](#healthcheck)
  - [User Interface](#user-interface)
- [Usage](#usage)
  - [Hello, World!](#hello-world)
- [Development](#development)
  - [Backend](#backend)
  - [Frontend](#frontend)
  - [Docker](#docker)
  - [Caddy](#caddy)
  - [HTTPS](#https)
- [Authors](#authors)
- [Contributors](#contributors)

## Installation

### With Docker

```sh
docker run -d \
  --restart=always \
  -p 8080:8080 \
  -p 8081:8081 \
  --name smocker \
  ghcr.io/smocker-dev/smocker
```

### Manual Deployment

```sh
# This will be the deployment folder for the Smocker instance
mkdir -p /opt/smocker && cd /opt/smocker
wget -P /tmp https://github.com/smocker-dev/smocker/releases/latest/download/smocker.tar.gz
tar xf /tmp/smocker.tar.gz
nohup ./smocker -mock-server-listen-port=8080 -config-listen-port=8081 &
```

### Healthcheck

```sh
curl localhost:8081/version
```

### User Interface

Smocker exposes a configuration user interface. You can access it in your web browser on http://localhost:8081/.

![History](docs/screenshots/screenshot-history.png)

![Mocks](docs/screenshots/screenshot-mocks.png)

## Usage

Smocker exposes two ports:

- `8080` is the mock server port. It will expose the routes you register through the configuration port
- `8081` is the configuration port. It's the port you will use to register new mocks. This port also exposes a user interface.

### Hello, World!

To register a mock, you can use the YAML and the JSON formats. A basic mock might look like this:

```yaml
# helloworld.yml
# This mock register two routes: GET /hello/world and GET /foo/bar
- request:
    # Note: the method could be omitted because GET is the default
    method: GET
    path: /hello/world
  response:
    # Note: the status could be omitted because 200 is the default
    status: 200
    headers:
      Content-Type: application/json
    body: >
      {
        "hello": "Hello, World!"
      }

- request:
    method: GET
    path: /foo/bar
  response:
    status: 204
```

You can then register it to the configuration server with the following command:

```sh
curl -XPOST \
  --header "Content-Type: application/x-yaml" \
  --data-binary "@helloworld.yml" \
  localhost:8081/mocks
```

After your mock is registered, you can query the mock server on the specified route, so that it returns the expected response to you:

```sh
$ curl -i localhost:8080/hello/world
HTTP/1.1 200 OK
Content-Type: application/json
Date: Thu, 05 Sep 2019 15:49:32 GMT
Content-Length: 31

{
  "hello": "Hello, World!"
}
```

To cleanup the mock server without restarting it, you can execute the following command:

```sh
curl -XPOST localhost:8081/reset
```

For more advanced usage, please read the [project's documentation](https://smocker.dev).

## Development

### Backend

The backend is written in Go. You can use the following commands to manage the development lifecycle:

- `make start`: start the backend in development mode, with live reload
- `make build`, `make VERSION=xxx build`: compile the code and generate a binary
- `make lint`: run static analysis on the code
- `make format`: automatically format the backend code
- `make test`: execute unit tests
- `make test-integration`: execute integration tests

### Frontend

The frontend is written with TypeScript and React. You can use the following commands to manage the development lifecycle:
npm install

- `yarn install`: install the dependencies
- `yarn start`: start the frontend in development mode, with live reload
- `yarn build`: generate the transpiled and minified files and assets
- `yarn lint`: run static analysis on the code
- `yarn format`: automatically format the frontend code
- `yarn test`: execute unit tests
- `yarn test:watch`: execute unit tests, with live reload

### Docker

The application can be packaged as a standalone Docker image. You can use the following commands to manage the development lifecycle:

- `make build-docker`, `make VERSION=xxx build-docker`: build the application as a Docker image
- `make start-docker`, `make VERSION=xxx start-docker`: run a Smocker Docker image

### Caddy

If you need to test Smocker with a base path, you can use the Caddyfile provided in the repository ([Caddy v2](https://caddyserver.com/v2)):

- `make start-release`, `make VERSION=xxx start-release`: create a released version of Smocker and launch it with `/smocker/` as base path
- `make start-caddy`: start Caddy to make Smocker accessible at http://localhost:8082/smocker/

### HTTPS

If you need to test Smocker with HTTPS enabled, the easiest way is to generate a locally signed certificate with [mkcert](https://github.com/FiloSottile/mkcert):

```sh
# Install the local certificate authority
mkcert -install

# Create a certificate for localhost
mkcert -cert-file /tmp/cert.pem -key-file /tmp/key.pem localhost
```

Then, start Smocker with TLS enabled, using your generated certificate:

```sh
./smocker -mock-server-listen-port=44300 -config-listen-port=44301 -tls-enable -tls-cert-file=/tmp/cert.pem -tls-private-key-file=/tmp/key.pem
```

## Authors

- [Thibaut Rousseau](https://github.com/Thiht)
- [Gwendal Leclerc](https://github.com/gwleclerc)

## Contributors

- [Amanda Yoshiizumi (mandyellow)](https://github.com/mandyellow): thank you for your awesome logo!



## local build
copy build/client from https://github.com/smocker-dev/smocker/releases/latest/download/smocker.tar.gz
run `make VERSION=1.0.0 build` and then test with http://localhost:8081
run `make VERSION=1.0.0 build-docker`  to create docker image
run `docker push keauw/smocker:1.0.0`  

