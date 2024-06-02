# Zeroize

[![PyPI version](https://badge.fury.io/py/rencrypt.svg)](https://badge.fury.io/py/rencrypt)
[![PyPI](https://github.com/radumarias/zeroize-python/actions/workflows/PyPI.yml/badge.svg)](https://github.com/radumarias/zeroize-python/actions/workflows/PyPI.yml)
[![tests](https://github.com/radumarias/zeroize-python/actions/workflows/tests.yml/badge.svg)](https://github.com/radumarias/zeroize-python/actions/workflows/tests.yml)

Clear secrets from memory. Built on stable Rust primitives which guarantee memory is zeroed using an operation will not be 'optimized away' by the compiler.
Uses a portable pure Rust implementation that works everywhere.

It uses [zeroize](https://crates.io/crates/zeroize) crate under the hood.  
It can work with `bytearray` and `numpy array`.

> ⚠️ **Warning**  
> **Currently it doens't work in the case of [Copy-on-write fork](https://en.wikipedia.org/wiki/Copy-on-write)  
> Also by itself it doesn't work if memory is moved or moved to swap file. You can use `crypes` with `libc.mlockall()` to lock the memory, see example below.***

# Example

```python
import zeroize
import numpy as np
import ctypes


# Lock memory using ctypes
def lock_memory():
    libc = ctypes.CDLL("libc.so.6")
    # Lock all current and future pages from being swapped out
    libc.mlockall(ctypes.c_int(0x02 | 0x04))  # MCL_CURRENT | MCL_FUTURE


def unlock_memory():
    libc = ctypes.CDLL("libc.so.6")
    # Unlock all locked pages
    libc.munlockall()


print("locking memory")
lock_memory()

print("allocate memory")

# regular array
arr = bytearray(b"1234567890")

# numpy array
arr_np = np.array([0] * 10, dtype=np.uint8)
arr_np[:] = arr
assert arr_np.tobytes() == b"1234567890"

print("zeroize'ing...: ")
zeroize.zeroize1(arr)
zeroize.zeroize_np(arr_np)

print("checking if is zeroized")
assert arr == bytearray(b"\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00")
assert all(arr_np == 0)

print("unlocking memory")
unlock_memory()

print("all good, bye!")
```

# Building from source

## Browser

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/radumarias/zeroize-python)

[![Open in Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new/?repo=radumarias%2Fzeroize-python&ref=main)

## Geting sources from GitHub
Skip this if you're starting it in browser.

```bash
git clone https://github.com/radumarias/zeroize-python && cd zeroize-python
```

## Compile and run

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
To configure your current shell, you need to source
the corresponding env file under $HOME/.cargo.
This is usually done by running one of the following (note the leading DOT):
```bash
. "$HOME/.cargo/env"
```
```
python -m venv .env
source .env/bin/activate
pip install maturin
pip install numpy
maturin develop
python main.py
```
