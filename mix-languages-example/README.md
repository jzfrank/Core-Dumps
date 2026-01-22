This little demo project is inspired by https://www.youtube.com/watch?v=XJC5WB2Bwrc and I asked Gemini for help. Below are Gemini's instructions.

# Instruction
Yes, this is absolutely doable on an **Apple Silicon M2 chip**. In fact, because macOS is Unix-based, the process is very similar to Linux, but there are a few "Apple-specific" flags you need to know to avoid architecture errors.

On M2, your code will run natively on the **ARM64** architecture. Here is the adjusted walkthrough specifically for your Mac.

---

## 1. Environment Check

Before starting, ensure your terminal is running natively (not through Rosetta) so that Python and the C++ compiler both target `arm64`.

1. Open **Terminal** and type: `uname -m`
* It should return **`arm64`**.


2. Ensure you have the Xcode Command Line Tools:
```bash
xcode-select --install

```



---

## 2. The Code (Same as before)

Create a file named `example.cpp`:

```cpp
#include <pybind11/pybind11.h>

int add(int i, int j) {
    return i + j;
}

PYBIND11_MODULE(example, m) {
    m.def("add", &add, "A function that adds two numbers");
}

```

---

## 3. Compiling on M2 (The "Secret Sauce")

The main difference on Mac is that you must tell the linker to look for Python symbols at runtime rather than at compile-time. If you don't, you'll get an "Undefined symbols for architecture arm64" error.

Run this command in your terminal:

```bash
c++ -O3 -Wall -shared -std=c++11 -undefined dynamic_lookup $(python3 -m pybind11 --includes) example.cpp -o example$(python3-config --extension-suffix)

```

### Breakdown of the Mac-specific flags:

* **`c++`**: On Mac, this points to Apple Clang (the standard compiler).
* **`-undefined dynamic_lookup`**: **Crucial for macOS.** This tells the compiler, "Don't worry if you can't find the Python internal functions right now; the Python interpreter will provide them when this library is loaded."
* **`$(python3-config --extension-suffix)`**: On M2, this will generate a filename like `example.cpython-311-darwin.so`.

---

## 4. The "Pro" Way: Using `setup.py`

If you don't want to manage long terminal commands, use a `setup.py` script. This is the recommended way for M1/M2 users because it automatically handles architecture detection.

**`setup.py`**

```python
from setuptools import setup
from pybind11.setup_helpers import Pybind11Extension, build_ext

ext_modules = [
    Pybind11Extension("example", ["example.cpp"]),
]

setup(
    name="example",
    ext_modules=ext_modules,
    cmdclass={"build_ext": build_ext},
)

```

**To build it:**

```bash
python3 setup.py build_ext --inplace

```

This creates the binary right in your folder. You can then run your Python script normally:

```python
import example
print(example.add(5, 5))

```

---

### Common Troubleshooting for M2

* **Architecture Mismatch:** If you get an error about `x86_64` vs `arm64`, it usually means your Python was installed via an Intel-based installer. It is best to use Python installed via [Homebrew](https://brew.sh) or a native ARM64 installer from python.org.
* **Header not found:** Ensure you ran `pip install pybind11` first.

**Would you like me to show you how to pass a more complex data type, like a Python List or a NumPy array, into your C++ function on the M2?**

[C++ Development on Mac Tutorial](https://www.youtube.com/watch?v=v2_Pth8MrKA)
This video provides a deep dive into setting up your environment for C++ development on Apple Silicon, ensuring your compiler and tools are correctly configured for native performance.
