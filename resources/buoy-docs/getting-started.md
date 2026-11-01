---
title: "Keychain getting started"
summary: "Install keychain and use its core API with explicit application ownership."
category: "Guides"
order: 10
keywords: ["keychain", "installation", "example"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Keychain getting started

## Install and import

Copy the sealed `keychain-0.1.0.buipkg` archive into your project `packages/` directory, or install a source/registry dependency named `keychain` using BPM and sync it. Then use `Import keychain`. The archive includes this documentation as package resources.

Use a consuming source filename such as `main.bui`; a local module named exactly like the package shadows its import.

## Complete usage example

The example is compiler-checked as a complete Buoy module. Functions and Subs are intended to be called from the consuming app at the appropriate user action or lifecycle event.

```buoy
Import keychain

Function ReadSavedPassword(account As String) As Optional Of String
    If Not KeychainBackendSupported() Or Not KeychainAvailable() Then Return None
    Dim store As New KeychainStore("com.example.buoy-demo")
    Try
        Return store.GetPassword(account)
    Catch e As KeychainException
        If e.Kind = KeychainErrorKind.ServiceUnavailable Or e.Kind = KeychainErrorKind.LibraryUnavailable Then Return None
        Throw e
    End Try
End Function
```

The wrapper in this example deliberately returns None when no store is usable, as an application memory-only fallback. That choice belongs to the wrapper: the native GetPassword method itself uses None only for an absent item and throws for locked, denied, cancelled or incompatible entries, which this example re-raises. Never overwrite a saved credential after such a failure, and do not log returned credentials.

## Integrate with the application

Choose a stable app-specific reverse-DNS service name. Keychain methods are synchronous, can block indefinitely and can invoke OS authorization or unlock UI, so call them off the UI thread. There is no cancellation or deadline API; a worker thread does not make a call cancellable. Managed Buoy strings are not zeroized by this API.

## Handle failures and unsupported targets

Blank services and empty accounts, and identities a backend cannot represent (Windows: / or \\ in the service, over 128/512 UTF-16 units, NUL, passwords over 2560 bytes; Linux: NUL in a password), throw InvalidArgumentException before any native call. GetPassword returns None only for an absent identity; an existing empty password is Some(""). Denial, cancellation, locked or unavailable stores, missing libraries and incompatible (binary) entries throw KeychainException with Operation, Status, Kind, Backend, Domain and NativeCode. On Linux a read or a delete that removes nothing while any collection is locked throws Locked rather than reporting absence. A confirmed successful deletion returns True despite an unrelated lock; observed matching locked duplicates still raise Locked. Error messages exclude service, account, label and password values.

KeychainSupported keeps its legacy meaning: true on macOS only. KeychainBackendSupported is true on macOS, Linux and Windows: the macOS Keychain, the Linux Secret Service through libsecret (fixed schema org.buoy.Integrations.Password with buoy-application/buoy-key attributes), and Windows Credential Manager (generic target Buoy/<service>/<account>, case-insensitive, local-machine persistence by default). KeychainAvailable is a library-only probe that never connects, prompts or reads. No backend has a plaintext or file fallback.

Return to the [overview](index.md) or consult the [API reference](api.md).
