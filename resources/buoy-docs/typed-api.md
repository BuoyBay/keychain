---
title: "keychain typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["KeychainBackend", "KeychainCapabilities", "KeychainCredential", "KeychainErrorKind", "KeychainException", "KeychainGError", "KeychainPersistence", "KeychainStore"]
dependencies: []
symbols: ["KeychainBackend", "KeychainCapabilities", "KeychainCredential", "KeychainErrorKind", "KeychainException", "KeychainGError", "KeychainPersistence", "KeychainStore"]
related: ["api.md", "getting-started.md", "index.md"]
---

# keychain typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## KeychainBackend

`Public Enum KeychainBackend As Integer`

Native credential store behind KeychainStore on the compiled target.

### Values

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `Unsupported` | `KeychainBackend` | `0` | — |
| `MacKeychain` | `KeychainBackend` | `1` | — |
| `SecretService` | `KeychainBackend` | `2` | — |
| `WindowsCredentialManager` | `KeychainBackend` | `3` | — |

## KeychainErrorKind

`Public Enum KeychainErrorKind As Integer`

Safe classification of a failed operation. Absence is never an error: it is None/False.

### Values

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `Failed` | `KeychainErrorKind` | `0` | — |
| `Unsupported` | `KeychainErrorKind` | `1` | — |
| `LibraryUnavailable` | `KeychainErrorKind` | `2` | — |
| `ServiceUnavailable` | `KeychainErrorKind` | `3` | — |
| `Locked` | `KeychainErrorKind` | `4` | — |
| `Denied` | `KeychainErrorKind` | `5` | — |
| `Cancelled` | `KeychainErrorKind` | `6` | — |
| `InvalidArgument` | `KeychainErrorKind` | `7` | — |
| `IncompatibleEntry` | `KeychainErrorKind` | `8` | — |

## KeychainPersistence

`Public Enum KeychainPersistence As Integer`

Persistent is the default and matches macOS Keychain items. SessionOnly is an explicit opt-in.

### Values

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `Persistent` | `KeychainPersistence` | `0` | — |
| `SessionOnly` | `KeychainPersistence` | `1` | — |

## KeychainException

`Public Class KeychainException Inherits Exception`

Operation and Status are unchanged (Status is the macOS OSStatus there). Kind, Backend, Domain and NativeCode add a structured, secret-free classification on every platform.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Status` | `Integer` | No | No | — | — |
| `Operation` | `String` | No | No | — | — |
| `Kind` | `KeychainErrorKind` | No | No | `KeychainErrorKind.Failed` | — |
| `Backend` | `KeychainBackend` | No | No | `KeychainBackend.Unsupported` | — |
| `Domain` | `String` | No | No | `""` | "OSStatus", a GLib GError domain such as "secret-error", "Win32" or KeychainDomain(). |
| `NativeCode` | `Integer` | No | No | `0` | — |

### Constructor

`Public Sub Constructor(operation As String, status As Integer)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `operation` | `String` | `ByVal` | Required |
| `status` | `Integer` | `ByVal` | Required |

Returns: no value.

### Constructor (overload 2)

`Public Sub Constructor(operation As String, kind As KeychainErrorKind, backend As KeychainBackend, domain As String, nativeCode As Integer)`

Messages contain only the operation, backend and native domain/code; never service, account, label, native error text or password content.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `operation` | `String` | `ByVal` | Required |
| `kind` | `KeychainErrorKind` | `ByVal` | Required |
| `backend` | `KeychainBackend` | `ByVal` | Required |
| `domain` | `String` | `ByVal` | Required |
| `nativeCode` | `Integer` | `ByVal` | Required |

Returns: no value.

### KeychainSupported

`Public Function KeychainSupported() As Boolean`

Legacy meaning, kept for existing callers: True only for the macOS Keychain.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### KeychainBackendSupported

`Public Function KeychainBackendSupported() As Boolean`

True when KeychainStore has a native backend for the compiled target (macOS, Linux, Windows). Implemented is not the same as available on this machine: see KeychainAvailable.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### KeychainAvailable

`Public Function KeychainAvailable() As Boolean`

Library-only probe. Never connects to D-Bus, starts a keyring, prompts, or reads an item, so True does not prove a running, unlocked service.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### KeychainBackendCapabilities

`Public Function KeychainBackendCapabilities() As KeychainCapabilities`

No parameters.

#### Returns

| Type |
| --- |
| `KeychainCapabilities` |

## KeychainCapabilities

`Public Class KeychainCapabilities`

Feature-specific, nonprompting description of the compiled backend.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Backend` | `KeychainBackend` | No | No | `KeychainBackend.Unsupported` | — |
| `Supported` | `Boolean` | No | No | `False` | Implemented for this target; not a promise that the service works on this machine. |
| `LibraryAvailable` | `Boolean` | No | No | `False` | Same library-only probe as KeychainAvailable(). |
| `PersistentStorage` | `Boolean` | No | No | `False` | — |
| `SessionOnlyStorage` | `Boolean` | No | No | `False` | — |
| `CaseInsensitiveIdentity` | `Boolean` | No | No | `False` | Windows target names match case-insensitively; other backends compare exactly. |
| `DeleteRemovesDuplicates` | `Boolean` | No | No | `False` | Linux: DeletePassword removes every unlocked duplicate of the identity. |
| `EmbeddedNulPasswords` | `Boolean` | No | No | `False` | Length-based password storage (macOS). Linux libsecret text passwords are C strings. |
| `MaxPasswordBytes` | `Integer` | No | No | `0` | Largest UTF-8 password in bytes; 0 means no fixed bound is enforced by this package. |
| `MaxServiceUnits` | `Integer` | No | No | `0` | UTF-16 code-unit limits for service/account; 0 means no package-enforced limit. |
| `MaxAccountUnits` | `Integer` | No | No | `0` | — |

## KeychainGError

`Sequential Structure KeychainGError`

### Fields

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `Domain` | `UInt32` | — | — |
| `Code` | `Int32` | — | — |
| `Message` | `Ptr` | — | — |

## KeychainCredential

`Sequential Structure KeychainCredential`

### Fields

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `CredentialFlags` | `UInt32` | — | — |
| `CredentialType` | `UInt32` | — | — |
| `TargetName` | `Ptr` | — | — |
| `Comment` | `Ptr` | — | — |
| `WrittenLow` | `UInt32` | — | — |
| `WrittenHigh` | `UInt32` | — | — |
| `BlobSize` | `UInt32` | — | — |
| `Blob` | `Ptr` | — | — |
| `Persistence` | `UInt32` | — | — |
| `AttributeCount` | `UInt32` | — | — |
| `Attributes` | `Ptr` | — | — |
| `TargetAlias` | `Ptr` | — | — |
| `UserName` | `Ptr` | — | — |

## KeychainStore

`Public Class KeychainStore`

Scope each app/integration to its own stable reverse-DNS service name.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Service` | `String` | Yes | No | — | — |
| `Persistence` | `KeychainPersistence` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(service As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `service` | `String` | `ByVal` | Required |

Returns: no value.

### Constructor (overload 2)

`Public Sub Constructor(service As String, persistence As KeychainPersistence)`

Explicit persistence choice. SessionOnly is rejected on macOS when an operation runs.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `service` | `String` | `ByVal` | Required |
| `persistence` | `KeychainPersistence` | `ByVal` | Required |

Returns: no value.

### GetPassword

`Public Function GetPassword(account As String) As Optional Of String`

None means missing; denial, cancellation, locked stores and other native errors throw.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `account` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Optional Of String` |

### SetPassword

`Public Sub SetPassword(account As String, password As String, label As String = "")`

Create or replace exactly one service/account password; duplicate races retry update.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `account` | `String` | `ByVal` | Required |
| `password` | `String` | `ByVal` | Required |
| `label` | `String` | `ByVal` | `""` |

Returns: no value.

### DeletePassword

`Public Function DeletePassword(account As String) As Boolean`

Delete only this exact service/account. False means the item was already absent.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `account` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |
