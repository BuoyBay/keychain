# Keychain for Buoy · 0.1.0

Reusable generic-password storage, independent of Calls and CouchDB. One API,
three native backends: the macOS Keychain, the freedesktop Secret Service on
Linux (through libsecret), and Windows Credential Manager. Install
`packages/keychain-0.1.0.buipkg` in another Buoy project's `packages/`
directory and use `Import keychain`. Build it with `python3 tools/build_keychain.py`.

| API | Behavior |
| --- | --- |
| `KeychainSupported() As Boolean` | **Legacy meaning, unchanged:** true on macOS only. Existing callers (Calls) keep their macOS-only UI. |
| `KeychainBackendSupported() As Boolean` | True when `KeychainStore` has a native backend for the compiled target (macOS, Linux, Windows). Implemented, not proven usable on this machine. |
| `KeychainAvailable() As Boolean` | Library-only probe (libsecret/GLib/GObject/GIO on Linux, `advapi32` on Windows; always true on macOS). Never connects to D-Bus, starts a keyring, prompts or reads an item. |
| `KeychainBackendCapabilities() As KeychainCapabilities` | Nonprompting, feature-specific description of the compiled backend (see below). |
| `New KeychainStore(service As String)` | Use a stable, app-specific reverse-DNS service name. Persistent storage. |
| `New KeychainStore(service, persistence As KeychainPersistence)` | Explicit `Persistent` or `SessionOnly` (Linux/Windows; rejected on macOS). |
| `store.Persistence As KeychainPersistence` | Read-only persistence choice. |
| `store.GetPassword(account As String) As Optional Of String` | `None` only for an absent item; `Some("")` is an existing empty password. |
| `store.SetPassword(account, password, label = "")` | Creates or replaces exactly one identity. |
| `store.DeletePassword(account) As Boolean` | True if deleted, false if already absent. |

There is no enumerate-all, read-all or delete-all API, no plaintext or
preferences-file fallback, no master-password collection, and importing the
package performs no native operation.

## Errors

`KeychainException` keeps `Operation` and `Status` (on macOS the original
OSStatus; elsewhere the native code) and adds a structured, secret-free
classification:

| Member | Meaning |
| --- | --- |
| `Kind As KeychainErrorKind` | `Failed`, `Unsupported`, `LibraryUnavailable`, `ServiceUnavailable`, `Locked`, `Denied`, `Cancelled`, `InvalidArgument`, `IncompatibleEntry`. Absence is never an error. |
| `Backend As KeychainBackend` | `MacKeychain`, `SecretService`, `WindowsCredentialManager` (or `Unsupported`). |
| `Domain As String` | `OSStatus`, the native GLib GError domain (for example `secret-error`, `g-dbus-error-quark`, `g-io-error-quark`), `Win32`, or `buoy-keychain` for outcomes this package classifies itself. |
| `NativeCode As Integer` | The OSStatus, GError code or Win32 error, unchanged. GError codes are domain-specific, not errno. |

`buoy-keychain` codes: 1 unsupported option (session-only on macOS), 2 native
library or export unavailable, 3 incompatible entry, 4 no service connection,
5 attribute allocation, 6 locked (absence cannot be established), 7 store
refused, 8 an unlocked duplicate survived delete, 9 empty native result.
Exception messages contain only the operation, kind, domain and code: never the
service, account, label, native error text or password. The legacy
`New KeychainException(operation, status)` constructor and its message format
are unchanged; on macOS it also classifies common OSStatus values (`-128`
cancelled, `-25293`/`-34018`/`-25243` denied, `-25308` locked, `-25291`
unavailable). Blank service/account and identities a backend cannot represent
throw `InvalidArgumentException` before any native call.

## macOS

Reads, writes and deletes use Apple's `SecItemCopyMatching`, `SecItemUpdate`,
`SecItemAdd` and `SecItemDelete`, unchanged from earlier releases. Queries always
include generic-password class, service and account. Passwords are stored by
length as UTF-8 data (`EmbeddedNulPasswords` is true), subject to the runtime's
own string limits noted below. Access uses the normal macOS Keychain with its
OS-managed access controls; the library does not enable iCloud sync or weaken an
item's ACL. `SetPassword` retries an update if another writer wins the initial
create race. Keychain may ask the user to grant access; stable code signing is
recommended. This library does not use Buoy secure preferences' silent-read
behavior.

## Linux (Secret Service)

Identity uses the same fixed schema as `containers/integrations/linux/linuxsecrets.bui`:
`xdg:schema = org.buoy.Integrations.Password`, `buoy-application = service`,
`buoy-key = account`. Items written by `linuxsecrets.Store` and by
`KeychainStore` are the same items. Matching is exact and case-sensitive.
Persistent stores write to the `default` collection alias; `SessionOnly`
writes to the nonpersistent `session` collection. The label is the given label,
or the service when empty; labels and attributes are metadata, not secrets.

- Passwords are libsecret C strings: a password containing NUL is rejected with
  `InvalidArgumentException` (`EmbeddedNulPasswords` is false). This limit is
  not applied to the macOS API.
- `GetPassword` returns `None` only when nothing matches and every collection
  is unlocked. GNOME Keyring hides the items of a locked collection from
  searches, so a read or a delete that removes nothing while **any** collection is
  locked throws `Locked`. A matching locked item, or one libsecret will not
  return as UTF-8 text (binary content type, invalid UTF-8), throws `Locked` or
  `IncompatibleEntry` and is left untouched.
- `DeletePassword` keeps libsecret clear semantics: it removes every unlocked
  matching item (`DeleteRemovesDuplicates`) and returns whether any were
  removed. A surviving locked duplicate is not deletion: it throws `Locked`,
  even if unlocked duplicates were already removed.
- A missing library or export throws `LibraryUnavailable`. No session bus or no
  Secret Service on it throws `ServiceUnavailable` with the native GError
  domain/code. The bus check connects to the session bus only.
- Calls may activate the user's Secret Service, prompt to unlock, or block.
  The package never supplies a keyring password.

Requires 64-bit glibc Linux with libsecret 0.19.0 or later (`libsecret-1.so.0`), `libglib-2.0.so.0`,
`libgobject-2.0.so.0` and `libgio-2.0.so.0` (all through `Optional Declare`, no
link-time dependency), a session D-Bus and a Secret Service such as GNOME Keyring.

## Windows (Credential Manager)

Identity uses the existing `WinCredentials.bui` namespace: generic credential
target `Buoy/<service>/<account>`, `UserName = account`, and the password as a
UTF-8 blob. Target matching is case-insensitive (`CaseInsensitiveIdentity`):
`Alpha` and `alpha` are the same credential, and service/account casing is not
normalized by the package. Identities Windows cannot represent are rejected
explicitly, never truncated or remapped: a service containing `/` or `\`, a
service over 128 UTF-16 code units, an account over 512 UTF-16 code units, a
service or account containing NUL, or a password over 2560 UTF-8 bytes throws
`InvalidArgumentException`.

`KeychainStore` writes `CRED_PERSIST_LOCAL_MACHINE` by default, matching
persistent Keychain items; `KeychainPersistence.SessionOnly` writes
`CRED_PERSIST_SESSION`. `WinCredentials.StoreGeneric` keeps its own
session-only default. Unknown persistence enum values are rejected before native I/O. `SetPassword` replaces the whole target, including the
username and persistence of an entry written by another API. `GetPassword`
never rewrites anything: a blob containing invalid UTF-8 or NUL bytes throws
`IncompatibleEntry` and remains readable through `WinCredentials.ReadGeneric`. Windows generic credentials carry no text/binary encoding tag: a blob that is valid portable UTF-8 is interpreted as text, including one written through the byte-oriented legacy API.
A logon without a credential set (for example a key-authenticated SSH session)
fails with Win32 1312 → `ServiceUnavailable`, never absence.

## Capabilities

`KeychainCapabilities` fields: `Backend`, `Supported`, `LibraryAvailable`,
`PersistentStorage`, `SessionOnlyStorage`, `CaseInsensitiveIdentity`,
`DeleteRemovesDuplicates`, `EmbeddedNulPasswords`, `MaxPasswordBytes`,
`MaxServiceUnits` and `MaxAccountUnits` (0 = no package-enforced bound).

## Blocking, cancellation and memory

All operations are synchronous and can block indefinitely (an unlock prompt, an
unresponsive D-Bus service) or show OS UI; call them off the UI thread. No
cancellation or deadline API is offered: running a call on a worker does not
make it cancellable, and an abandoned worker could still mutate storage. Native
cancellation (for example libsecret `GCancellable` async operations) is a
separate, unimplemented step. Writes and deletes are never retried silently.
An application's memory-only mode remains its own explicit decision; do not
overwrite saved credentials after a transient read failure.

Credentials necessarily exist in process memory while in use; Buoy strings are
managed copies and this API does not promise zeroization. Buoy 0.72.0 strings
cannot carry U+0000 (`upstream-issues/BUI-003-string-nul-loss`): a NUL lost
before the call cannot be detected by any backend, and NUL bytes in stored data
are reported as `IncompatibleEntry` rather than truncated.

`link/*/libbuoy_security.tbd` contains symbol declarations only. Its install name
points to Apple's system Security framework, which is loaded at runtime; no
third-party security binary is bundled.

## Verification


Maintainers run the private validation suite before release. See the repository README for public build instructions.

