---
title: "Keychain API"
summary: "Store one generic password identity at a time on macOS, Linux and Windows with explicit, structured error reporting."
category: "Reference"
order: 20
keywords: ["keychain", "API", "lifecycle"]
symbols: ["KeychainSupported", "KeychainBackendSupported", "KeychainAvailable", "KeychainBackendCapabilities", "KeychainCapabilities", "KeychainPersistence", "KeychainBackend", "KeychainErrorKind", "KeychainStore.Persistence", "KeychainException.Kind", "KeychainException.Backend", "KeychainException.Domain", "KeychainException.NativeCode", "KeychainStore", "KeychainStore.Service", "KeychainStore.GetPassword", "KeychainStore.SetPassword", "KeychainStore.DeletePassword", "KeychainException", "KeychainException.Operation", "KeychainException.Status"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# Keychain API

Import the `keychain` package. Constructors below use the class name; invoke them with `New`. Listed members are public client APIs; native declarations and trampoline helpers are implementation details.

## Public surface

| Symbol | Contract |
| --- | --- |
| `KeychainSupported() As Boolean` | Legacy: true only for the macOS implementation. |
| `KeychainBackendSupported() As Boolean` | True when the compiled target has a native backend (macOS, Linux, Windows). |
| `KeychainAvailable() As Boolean` | Library-only probe; never connects to a service, prompts or reads. |
| `KeychainBackendCapabilities() As KeychainCapabilities` | Backend, persistence options, identity casing, delete semantics, NUL support and size limits. |
| `KeychainStore(service As String)` | Creates a persistent wrapper with a nonblank stable service identity. |
| `KeychainStore(service As String, persistence As KeychainPersistence)` | Explicit Persistent or SessionOnly storage (SessionOnly on Linux/Windows only). |
| `KeychainStore.Service As String` | Read-only service string. |
| `KeychainStore.Persistence As KeychainPersistence` | Read-only persistence choice. |
| `KeychainStore.GetPassword(account As String) As Optional Of String` | None means absent; Some preserves empty strings. |
| `KeychainStore.SetPassword(account As String, password As String, label As String = "")` | Creates or replaces exactly the service/account identity; macOS retries when another writer wins an initial create race. |
| `KeychainStore.DeletePassword(account As String) As Boolean` | True when deleted; false when already absent. Linux removes every unlocked duplicate. |
| `KeychainException` | Exception for native, unavailable and unsupported failures. |
| `KeychainException.Operation As String` | Operation that failed. |
| `KeychainException.Status As Integer` | Original OSStatus on macOS; the native code elsewhere. |
| `KeychainException.Kind As KeychainErrorKind` | Failed, Unsupported, LibraryUnavailable, ServiceUnavailable, Locked, Denied, Cancelled, InvalidArgument or IncompatibleEntry. |
| `KeychainException.Domain As String`, `NativeCode As Integer` | OSStatus, GLib GError domain, Win32 or buoy-keychain, with the unchanged native code. |

## Ownership and timing

Choose a stable app-specific reverse-DNS service name. Keychain methods are synchronous, can block indefinitely and can invoke OS authorization or unlock UI, so call them off the UI thread. There is no cancellation or deadline API; a worker thread does not make a call cancellable. Managed Buoy strings are not zeroized by this API. On macOS, stable app signing helps avoid repeated authorization after development rebuilds or relocation.

## Errors and result interpretation

Blank services and empty accounts, and identities a backend cannot represent (Windows: / or \\ in the service, over 128/512 UTF-16 units, NUL, passwords over 2560 bytes; Linux: NUL in a password), throw InvalidArgumentException before any native call. GetPassword returns None only for an absent identity; an existing empty password is Some(""). Denial, cancellation, locked or unavailable stores, missing libraries and incompatible (binary) entries throw KeychainException with Operation, Status, Kind, Backend, Domain and NativeCode. On Linux a read or a delete that removes nothing while any collection is locked throws Locked rather than reporting absence. A confirmed successful deletion returns True despite an unrelated lock; observed matching locked duplicates still raise Locked. Error messages exclude service, account, label and password values.

## Platform behavior

KeychainSupported keeps its legacy meaning: true on macOS only. KeychainBackendSupported is true on macOS, Linux and Windows: the macOS Keychain, the Linux Secret Service through libsecret (fixed schema org.buoy.Integrations.Password with buoy-application/buoy-key attributes), and Windows Credential Manager (generic target Buoy/<service>/<account>, case-insensitive, local-machine persistence by default). KeychainAvailable is a library-only probe that never connects, prompts or reads. No backend has a plaintext or file fallback.

See the [complete usage example](getting-started.md) and the [package overview](index.md).

[Full typed API: properties, parameters, defaults, and returns](typed-api.md)
