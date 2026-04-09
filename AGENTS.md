# AGENTS.md

## Cursor Cloud specific instructions

### Project Overview

GNU Radio is a C++/Python signal processing toolkit (v3.11.x). It uses CMake as its build system and has no external services (databases, web servers, etc.) — it is a pure native application.

### Build

The build requires an out-of-tree build directory. CMake prevents in-tree builds.

```bash
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_CXX_COMPILER=g++ \
      -DCMAKE_C_COMPILER=gcc ..
cmake --build . --parallel $(nproc)
sudo cmake --install .
sudo ldconfig
```

**Gotcha:** The default compiler on Ubuntu 24.04 may be Clang (via `/usr/bin/c++`), which can fail to link due to missing `-lstdc++`. Always specify `-DCMAKE_CXX_COMPILER=g++ -DCMAKE_C_COMPILER=gcc` explicitly.

After `cmake --install`, run `sudo ldconfig` so the shared libraries are found at runtime.

### Environment Variable

Set `LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH` before running Python scripts that import `gnuradio`, or add `/usr/local/lib` to `/etc/ld.so.conf.d/` (already handled by `ldconfig` after install).

### Tests

```bash
cd build && ctest --output-on-failure -j2
```

All 265 tests should pass. Tests use Boost.Test (C++) and pytest (Python).

### Lint

- **C++ formatting:** `clang-format-14 --dry-run --Werror <file>` (see `.clang-format` in repo root)
- **Python formatting:** `pycodestyle --max-line-length=120 --ignore=E265,E266,E275,E402,E501,E704,E712,E713,E714,E711,E721,E722,E741,W504,W605 --exclude='*.yml.py' <file>`
- CI config for lint args is in `.github/workflows/make-test.yml` and `tox.ini`.

### Verify Installation

```bash
python3 -c "import gnuradio.blocks; print(gnuradio.blocks.complex_to_float())"
```

### Disabled (optional) components

These components are disabled because their hardware/GUI dependencies are not installed in the cloud VM. They are not required for core development:
- `gr-qtgui` (needs Qt5 + Qwt + PyQt5)
- `gnuradio-companion` (needs GTK3/PyGObject or Qt for the GUI)
- `gr-uhd` (needs USRP Hardware Driver)
- `gr-iio` (needs libiio)
- `gr-soapy` (needs SoapySDR)
- `doxygen` (needs Doxygen for API docs)
