# Archs

Platform and processor architecture compatibility notes for FasterEdge components.

## Go Cross-Platform Capabilities

The Go toolchain uses two environment variables to describe the target platform:

- `GOOS`: the target operating system or execution environment.
- `GOARCH`: the target processor instruction set architecture.

For most projects that do not depend on the target platform's C toolchain, you can cross-compile programs for other platforms from a single development machine by setting `GOOS`, `GOARCH` and disabling CGO:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build ./...
CGO_ENABLED=0 GOOS=windows GOARCH=arm64 go build ./...
CGO_ENABLED=0 GOOS=freebsd GOARCH=riscv64 go build ./...
```

The official Go download page mainly provides precompiled toolchains for common platforms. For other official ports, see the **Other Ports** section on the [Go Downloads](https://go.dev/dl/) page:

> Use https://go.dev/dl/ under "Other Ports".

Not every combination in `go tool dist list` has a directly downloadable precompiled binary toolchain; when no precompiled package is provided, you can cross-compile with an existing Go toolchain, or build the toolchain following [Installing Go from source](https://go.dev/doc/install/source).

## Official Target List

The following list is from Go `1.25.5`:

```bash
go tool dist list
```

It contains **15 GOOS, 14 GOARCH, and 48 valid GOOS/GOARCH combinations**. Validity is per-combination; GOOS and GOARCH from the table below cannot be mixed arbitrarily.

### By GOOS

| GOOS | Official GOARCH combinations |
|---|---|
| `aix` | `ppc64` |
| `android` | `386`, `amd64`, `arm`, `arm64` |
| `darwin` | `amd64`, `arm64` |
| `dragonfly` | `amd64` |
| `freebsd` | `386`, `amd64`, `arm`, `arm64`, `riscv64` |
| `illumos` | `amd64` |
| `ios` | `amd64`, `arm64` |
| `js` | `wasm` |
| `linux` | `386`, `amd64`, `arm`, `arm64`, `loong64`, `mips`, `mips64`, `mips64le`, `mipsle`, `ppc64`, `ppc64le`, `riscv64`, `s390x` |
| `netbsd` | `386`, `amd64`, `arm`, `arm64` |
| `openbsd` | `386`, `amd64`, `arm`, `arm64`, `ppc64`, `riscv64` |
| `plan9` | `386`, `amd64`, `arm` |
| `solaris` | `amd64` |
| `wasip1` | `wasm` |
| `windows` | `386`, `amd64`, `arm64` |

### Full GOOS/GOARCH Combinations

```text
aix/ppc64
android/386
android/amd64
android/arm
android/arm64
darwin/amd64
darwin/arm64
dragonfly/amd64
freebsd/386
freebsd/amd64
freebsd/arm
freebsd/arm64
freebsd/riscv64
illumos/amd64
ios/amd64
ios/arm64
js/wasm
linux/386
linux/amd64
linux/arm
linux/arm64
linux/loong64
linux/mips
linux/mips64
linux/mips64le
linux/mipsle
linux/ppc64
linux/ppc64le
linux/riscv64
linux/s390x
netbsd/386
netbsd/amd64
netbsd/arm
netbsd/arm64
openbsd/386
openbsd/amd64
openbsd/arm
openbsd/arm64
openbsd/ppc64
openbsd/riscv64
plan9/386
plan9/amd64
plan9/arm
solaris/amd64
wasip1/wasm
windows/386
windows/amd64
windows/arm64
```

## GOARCH Reference

| GOARCH | Processor or execution environment |
|---|---|
| `386` | 32-bit x86 |
| `amd64` | 64-bit x86-64 |
| `arm` | 32-bit ARM |
| `arm64` | 64-bit ARM / AArch64 |
| `loong64` | 64-bit LoongArch |
| `mips` | 32-bit big-endian MIPS |
| `mipsle` | 32-bit little-endian MIPS |
| `mips64` | 64-bit big-endian MIPS |
| `mips64le` | 64-bit little-endian MIPS |
| `ppc64` | 64-bit big-endian PowerPC |
| `ppc64le` | 64-bit little-endian PowerPC |
| `riscv64` | 64-bit RISC-V |
| `s390x` | IBM z/Architecture |
| `wasm` | WebAssembly |

## Querying the Current Toolchain's Supported Targets

On the current host:

```bash
go env GOOS GOARCH
```

All targets supported by the current Go toolchain:

```bash
go tool dist list
```

List as JSON only:

```bash
go tool dist list -json
```

Check whether a specific combination exists:

```bash
go tool dist list | grep '^linux/arm64$'
```

The static list in this document represents Go `1.25.5` at the time of writing. Later Go versions may add, adjust or remove ports, so before a release build you should rely on the output of the toolchain you are actually using.

## Cross-Compilation Limitations

### CGO

With `CGO_ENABLED=0`, pure Go projects can usually be cross-compiled directly. When CGO is enabled, you must also prepare the target platform's C compiler, linker, system headers and target libraries:

```bash
CGO_ENABLED=1 \
GOOS=linux \
GOARCH=arm64 \
CC=aarch64-linux-gnu-gcc \
go build ./...
```

### Platform-Specific Code

A project may still fail to build or run on an official target for the following reasons:

- Using source files with GOOS/GOARCH build constraints;
- Depending on third-party libraries that support only some platforms;
- Calling operating-system-specific syscalls;
- Depending on dynamic libraries, drivers, GUI or hardware capabilities;
- The target environment does not implement the network, filesystem or process capabilities the program needs.

Therefore, the Go toolchain supporting a given GOOS/GOARCH does not automatically mean every Go project is compatible with that combination.

### WebAssembly

Go officially includes two WebAssembly targets:

- `js/wasm`: JavaScript host environment; usually requires the `wasm_exec.js` shipped with the Go distribution.
- `wasip1/wasm`: WASI Preview 1 host environment.

The two differ in system interfaces and host capabilities; you cannot consider the runtime environments equivalent merely by swapping GOOS.

## Official References

- [Go Downloads](https://go.dev/dl/) — official toolchain downloads and Other Ports.
- [Installing Go from source](https://go.dev/doc/install/source) — building Go from source and platform requirements.
- [`go` command documentation](https://pkg.go.dev/cmd/go) — Go build commands and environment variables.
- [`go tool dist`](https://pkg.go.dev/cmd/dist) — querying the platform combinations supported by the current toolchain.
- [Go Porting Policy](https://go.dev/wiki/PortingPolicy) — policies for new ports and port maintenance.
- [Minimum Requirements](https://go.dev/wiki/MinimumRequirements) — minimum runtime requirements per architecture.