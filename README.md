# Native credential store

There is no enumerate-all, read-all or delete-all API, no plaintext or
preferences-file fallback, no master-password collection, and importing the
package performs no native operation.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add keychain --git https://github.com/BuoyBay/keychain.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## Example

```basic
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

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer keychain
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
