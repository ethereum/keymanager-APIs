# Ethereum keymanager APIs

![CI](https://github.com/ethereum/keymanager-APIs/workflows/CI/badge.svg)

**_Warning: Super fresh repo, expect rapid iteration and breaking changes_**

Collection of RESTful APIs provided by Ethereum consensus keymanagers

API browser: [https://ethereum.github.io/keymanager-APIs/](https://ethereum.github.io/keymanager-APIs/)

## Outline

This document outlines an application programming interface (API) which is exposed by a validator client implementation
which aims to facilitate validator management tools without sacrificing security.

The API is a REST interface, accessed via HTTP. The API should not, unless protected by additional security layers,
be exposed to the public Internet as the API includes multiple endpoints which can control the state of a deployed validator (e.g. Exits, etc).
Currently, the only supported return data type is JSON.

The validator client (VC) maintains private information related validators such as keys and an anti-slashing database.
A VC is also in charge of signing messages relatied to validation duties (e.g. Attestations, and Beacon Blocks) as well as state changes (e.g. Exis).

The goal of this specification is to promote interoperability between various validator client implementations to aid in the creation of common validator management tools
such as GUIs, alerts, etc.

## Render

To render spec in browser you will need any http server to load `index.html` file
in root of the repo.

##### Python

```
python -m SimpleHTTPServer 8080
```

And api spec will render on [http://localhost:8080](http://localhost:8080).

##### NodeJs

```
npm install simplehttpserver -g

# OR

yarn global add simplehttpserver

simplehttpserver
```

And api spec will render on [http://localhost:8000](http://localhost:8000).

## Contributing

Api spec is checked for lint errors before merge.

To run lint locally, install linter with

```
npm install -g @stoplight/spectral

# OR

yarn global add @stoplight/spectral
```

and run lint with

```
spectral lint beacon-node-oapi.yaml
```

## Implementations

- [TypeScript Wrapper](https://www.npmjs.com/package/@chainsafe/eth2.0-api-wrapper)

https://www.npmjs.com/package/@chainsafe/eth2.0-api-wrapper

## Releasing

1. Create and push tag

   - Make sure info.version in beacon-node-oapi.yaml file is updated before tagging.
   - CD will create github release and upload bundled spec file

2. Add release entrypoint in index.html

In SwaggerUIBundle configuration (inside index.html file), add another entry in "urls" field (SwaggerUI will load first item as default).
Entry should be in following format(replace `<tag>` with real tag name from step 1.):

```javascript
         {url: "https://github.com/ethereum/keymanager-APIs/releases/download/<tag>/beacon-node-oapi.yaml", name: "<tag>"},
```
