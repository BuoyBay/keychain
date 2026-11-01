---
title: "Keychain"
summary: "Store one generic password identity at a time on macOS, Linux and Windows with explicit, structured error reporting."
category: "Overview"
order: 0
keywords: ["keychain"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# Keychain

KeychainStore wraps generic-password items identified by a service and account. It provides read, create/update, and delete operations without enumerate-all or delete-all APIs, and has no dependency on Calls or CouchDB.

## Platform support

KeychainSupported keeps its legacy meaning: true on macOS only. KeychainBackendSupported is true on macOS, Linux and Windows: the macOS Keychain, the Linux Secret Service through libsecret (fixed schema org.buoy.Integrations.Password with buoy-application/buoy-key attributes), and Windows Credential Manager (generic target Buoy/<service>/<account>, case-insensitive, local-machine persistence by default). KeychainAvailable is a library-only probe that never connects, prompts or reads. No backend has a plaintext or file fallback.

## Start reading

- [Getting started](getting-started.md) walks through installation and a complete example.
- [API reference](api.md) describes public symbols, state, errors, and ownership.

## Scope

Choose a stable app-specific reverse-DNS service name. Keychain methods are synchronous, can block indefinitely and can invoke OS authorization or unlock UI, so call them off the UI thread. There is no cancellation or deadline API; a worker thread does not make a call cancellable. Managed Buoy strings are not zeroized by this API.
