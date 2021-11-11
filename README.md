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

To render the spec in a browser you will need an HTTP server to serve the `index.html` file.
Rendering occurs client-side in JavaScript, so no changes are required to the HTML file between
edits.

##### Python

```
python2 -m SimpleHTTPServer 8080
```

The API spec will render on [http://localhost:8080](http://localhost:8080).

##### NodeJs

```
npm install simplehttpserver -g

# OR

yarn global add simplehttpserver

simplehttpserver
```

The API spec will render on [http://localhost:8000](http://localhost:8000).

## Contributing

The API spec is linted for issues before PRs are merged.

To run the linter locally, install `spectral`:

```
npm install -g @stoplight/spectral

# OR

yarn global add @stoplight/spectral
```

and run with

```
spectral lint keymanager-oapi.yaml
```

## Releasing

1. Create and push a tag

   - Make sure info.version in keymanager-oapi.yaml file is updated before tagging.
   - CD will create github release and upload bundled spec file

2. Add release entrypoint in index.html

In SwaggerUIBundle configuration (inside index.html file), add another entry in "urls" field (SwaggerUI will load first item as default).
Entries should be in the following format (replace `<tag>` with the real tag name from step 1):

```javascript
         {url: "https://github.com/ethereum/keymanager-APIs/releases/download/<tag>/keymanager-oapi.yaml", name: "<tag>"},
```
