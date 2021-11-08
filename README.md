# Ethereum Validator APIs

***Warning: Super fresh repo, expect rapid iteration and breaking changes***

Collection of RESTful APIs provided by Ethereum Validator Clients (VC)

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
