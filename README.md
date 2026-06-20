<div align="center">

# Rill

A BusyBox-style multi-call binary written in pure x86_64 NASM assembly.

[![Version](https://img.shields.io/github/v/release/Real-Fruit-Snacks/Rill?color=89b4fa)](https://github.com/Real-Fruit-Snacks/Rill/releases)
[![License](https://img.shields.io/github/license/Real-Fruit-Snacks/Rill?color=cba6f7)](LICENSE)

</div>

## Overview

Rill is an aggressively minimal multi-call binary consisting of 41 POSIX utilities bundled into a single ~108 KB static executable. It acts as the assembly cousin to other multi-call tools in the family (like `jib` in Rust, `topsail` in Go, and `mainsail` in Python).

Built at the lowest practical level, every applet in Rill is hand-written in NASM assembly. It executes via direct `syscall`s, bypasses the dynamic linker, ships with zero `libc` dependencies, avoids global allocators, and strips away standard startup glue code. Every byte the kernel interacts with originates from readable `.asm` source files.

## Key Features

- **Extremely Lightweight**: Single static ELF binary (~108 KB) with only two `PT_LOAD` segments and `.bss` / stack frame buffers.
- **41 Utilities**: Includes essential POSIX tools spanning file operations, text manipulation, path processing, and system queries.
- **Zero Dependencies**: Pure System V AMD64 ABI executing raw Linux x86_64 syscalls. No `libc`, no dynamic execution.
- **Multi-call Dispatch**: Uses standard multi-call mechanics, resolving its target utility either via `argv[0]` symlinks or direct subcommands.
- **Rigorous Testing**: Enforced by a 329-case `bats` integration suite that enforces byte-for-byte parity against standard `/usr/bin/<applet>` equivalents.

## Getting Started / Installation

### Prerequisites

To compile Rill from source, you need a Linux x86_64 environment (or WSL) equipped with:
- `nasm`
- `ld` (GNU Linker)
- `make`
- `bats` (for testing)

### Installation

**From Precompiled Release:**

```bash
curl -LO https://github.com/Real-Fruit-Snacks/Rill/releases/latest/download/rill-linux-x64
chmod +x rill-linux-x64
mv rill-linux-x64 rill
./rill date
```

**Build from Source:**

```bash
git clone https://github.com/Real-Fruit-Snacks/Rill.git
cd Rill
make             # Compiles the binary to build/rill
make symlinks    # Generates build/<applet> symlinks for argv[0] dispatch
make test        # Executes the bats integration suite
make size        # Outputs the final binary size
```

## Usage

Rill utilizes a multi-call dispatch mechanism. You can invoke an applet by using it as a subcommand or by accessing it through a symbolic link.

### Subcommand Dispatch

```bash
./build/rill true && echo ok
./build/rill printenv
```

### Symlink Dispatch

When invoked via a symlink, Rill automatically detects its invocation name:

```bash
ln -s build/rill ls
./ls -la
echo "test data" | ./build/grep -i 'T.'
```

### Included Applets

- **File Ops**: `ls`, `cp`, `mv`, `rm`, `mkdir`, `rmdir`, `touch`, `ln`, `readlink`, `chmod`, `chown`, `stat`, `find`
- **Text**: `cat`, `wc`, `head`, `tail`, `cut`, `tr`, `sort`, `uniq`, `grep`, `tee`
- **Paths**: `pwd`, `basename`, `dirname`, `which`
- **System**: `whoami`, `id`, `uname`, `hostname`, `date`, `kill`, `ps`, `env`, `printenv`
- **Control**: `true`, `false`, `echo`, `yes`, `sleep`

## Architecture / File Structure

```text
Rill/
├── src/
│   ├── start.asm            # ELF entrypoint (_start), sys_exit handler, and applet dispatch table
│   ├── core/                # Shared assembly macros, string utils, and memory routines
│   └── applets/             # Individual .asm files for each of the 41 applets
├── tests/
│   └── integration/         # 329 bats testing cases comparing behavior against coreutils
├── linker.ld                # Custom GNU ld script defining the ELF layout
├── Makefile                 # Build orchestration
└── pyproject.toml           # (If applicable, configuration metadata)
```

Adding a new applet is highly systematic: place a new `src/applets/<name>.asm` exporting `applet_<name>_main`, wire the extern into `src/start.asm`, and append the corresponding `bats` integration file.

## Contributing

Review the contribution guidelines in [CONTRIBUTING.md](CONTRIBUTING.md) before submitting pull requests. All new applets must include full integration test coverage.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
