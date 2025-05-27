# Ethereum keymanager APIs

![CI](https://github.com/ethereum/keymanager-APIs/workflows/CI/badge.svg)
[![GitPOAP Badge](https://public-api.gitpoap.io/v1/repo/ethereum/keymanager-APIs/badge)](https://www.gitpoap.io/gh/ethereum/keymanager-APIs)

Collection of RESTful APIs provided by Ethereum consensus keymanagers

API browser: [https://ethereum.github.io/keymanager-APIs/](https://ethereum.github.io/keymanager-APIs/)

## Outline

This document outlines an application programming interface (API) which is exposed by a validator client implementation
which aims to facilitate validator management tools without sacrificing security.

The API is a REST interface, accessed via HTTP. The API should not, unless protected by additional security layers,
be exposed to the public Internet as the API includes multiple endpoints which can control the state of a deployed validator (e.g. Exits, etc).
Currently, the only supported return data type is JSON.

The validator client (VC) maintains private information-related validators such as keys and an anti-slashing database.
A VC is also in charge of signing messages related to validation duties (e.g. Attestations, and Beacon Blocks) as well as state changes (e.g. Exits).

The goal of this specification is to promote interoperability between various validator client implementations to aid in the creation of common validator management tools
such as GUIs, alerts, etc.

## Client Support

Learn about the Consensus Validator Clients that implement these APIs on the [Ethereum.org](https://ethereum.org/)

### v1 APIS

| Validator Client/Remote Managers | local keymanager | remote keymanager | fee recipient | gas limit  | graffiti |
| -------------------------------- | ---------------- | ----------------- | ------------- | ---------- | -------- |
| Prysm                            | production       | production        | production    | production | -        |
| Teku                             | production       | production        | production    | production | -        |
| Lighthouse                       | v2.1.2           | v2.3.0            | v2.4.0        | v3.0.0     | -        |
| Nimbus                           | production       | production        | 22.7.0        | -          | -        |
| Lodestar                         | v0.35.0          | v0.40.0           | v1.2.0        | v1.2.0     | v1.12.0  |
| Vero                             | N/A              | v1.1.0            | v1.1.0        | v1.1.0     | v1.1.0   |
| Web3signer                       | production       | N/A               | N/A           | N/A        | N/A      |

## Use Cases

High-level information on how each API endpoint is used are listed in the [Use-cases Doc](/use-cases/README.md)

## Flows

Further detail on expected behavior and error states of the APIs are listed in the [Flows Doc](/flows/README.md)

## Running API Browser Locally

To render the spec in a browser you will need an HTTP server to serve the `index.html` file.
Rendering occurs client-side in JavaScript, so no changes are required to the HTML file between
edits.

##### Python

```sh
python -m http.server 8080
```

The API spec will render on [http://localhost:8080](http://localhost:8080).

##### NodeJs

```sh
npm install simplehttpserver -g

# OR

yarn global add simplehttpserver

simplehttpserver
```

The API spec will render on [http://localhost:8000](http://localhost:8000).

### Usage

Local changes will be observable if "dev" is selected in the "Select a definition" drop-down in the web UI.

Users may need to tick the "Disable Cache" box in their browser's developer tools to see changes after modifying the source.

## Contributing

Api spec is checked for lint errors before merge.

To run lint locally, install linter with

```sh
npm install -g @redocly/cli

# OR

yarn global add @redocly/cli
```

and run lint with

```sh
redocly lint keymanager-oapi.yaml
```

## Releasing

This repository supports both stable and pre-releases. The version of the next release has to be
determined based on the changes in `master` branch since the last stable release. It is recommended
to create a pre-release before releasing a new stable version.

### Stable releases

Steps to create a new stable release:

- Create and push a tag with the version of the release, e.g. `v1.1.0`
- CD will create the github release, upload bundled spec files, and open a release PR
- Review, approve and merge the release PR to `master` branch

### Pre-releases

Steps to create a new pre-release:

- Create and push a tag with the version of the release, e.g. `v1.1.0-alpha.0`
- CD will create the github release and upload bundled spec files

Pre-releases will not be listed in `index.html` and are intended to allow early testing against the spec.
