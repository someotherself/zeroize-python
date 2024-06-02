# Zeroize

[![PyPI version](https://badge.fury.io/py/rencrypt.svg)](https://badge.fury.io/py/rencrypt)
[![PyPI](https://github.com/radumarias/zeroize-python/actions/workflows/CI.yml/badge.svg)](https://github.com/radumarias/zeroize-python/actions/workflows/PyPI.yml)
[![tests](https://github.com/radumarias/zeroize-python/actions/workflows/tests.yml/badge.svg)](https://github.com/radumarias/zeroize-python/actions/workflows/tests.yml)

Securely clear secrets from memory. Built on stable Rust primitives which guarantee memory is zeroed using an operation will not be 'optimized away' by the compiler.
Uses a portable pure Rust implementation that works everywhere.

It uses [zeroize](https://crates.io/crates/zeroize) crate under the hood.  
It can work with `bytearray` and numpy array.

> ⚠️ **Warning**  
> **Currently it doens't work in the case of [Copy-on-write fork](https://en.wikipedia.org/wiki/Copy-on-write)  
> Also it doesn't work if memory is moved or moved to swap file. You can use `crypes` with `libc.mlockall` to lock the memory from being swapped, see example below.***

# Example

```python
import zeroize
import numpy as np


# regular array
arr = bytearray(b'1234567890')

# numpy array
arr_np = np.array([0] * 10, dtype=np.uint8)
arr_np[:] = arr
assert arr_np.tobytes() == b'1234567890'

print("zeroize'ing...: ")
zeroize.zeroize1(arr)
zeroize.zeroize_np(arr_np)

print("checking if is zeroized...")
assert arr == bytearray(b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
assert all(arr_np == 0)

print("all good, bye!")
```

# Use `crypes` with `libc.mlockall()` to lock the memory from being swapped

```python
import ctypes

MCL_CURRENT = 1
MCL_FUTURE  = 2

libc = ctypes.CDLL('libc.so.6', use_errno=True)

def mlockall(flags=MCL_CURRENT|MCL_FUTURE):
    result = libc.mlockall(flags)
    if result != 0:
        raise Exception("cannot lock memmory, errno=%s" % ctypes.get_errno())

def munlockall():
    result = libc.munlockall()
    if result != 0:
        raise Exception("cannot lock memmory, errno=%s" % ctypes.get_errno())


if __name__ == '__main__':
    mlockall()
    print("memmory locked")

    # allocate your data here
    # ...
    # zeroize it

    munlockall()
    print("memmory unlocked")
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
