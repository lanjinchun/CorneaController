# Repository Guidance

## Cursor Cloud specific instructions

This repository targets a Windows station setup for real hardware. The documented
production-style environment is in `README.md` and `docs/ENV.md`: Qt 5.15.2 with
MSVC, Python 3.12, proprietary Cornea wheel packages, FTDI/libusb-win32 drivers,
HDF5 calibration files, and attached Cornea hardware.

For Cursor Cloud validation without hardware, use the Qt Debug simulation path.
The qmake project defines `DISABLE_SIM` by default, so a Linux Debug/SIM build
needs a `make` override that removes that define and links the local Python:

- Create build dir first: `mkdir -p build-qmake-debug`
- Generate: `qmake CorneaController.pro CONFIG+=debug CONFIG-=release "INCLUDEPATH+=/usr/include/python3.12" "INCLUDEPATH+=$(python3 -c 'import numpy; print(numpy.get_include())')" "LIBS+=-L/usr/lib/x86_64-linux-gnu -lpython3.12" -o build-qmake-debug/Makefile`
- Build from repo root: `make -C build-qmake-debug -j2 DEFINES="-DQT_WIDGETS_LIB -DQT_GUI_LIB -DQT_NETWORK_LIB -DQT_CONCURRENT_LIB -DQT_CORE_LIB" LIBS="-L/usr/lib/x86_64-linux-gnu -lpython3.12 /usr/lib/x86_64-linux-gnu/libQt5Widgets.so /usr/lib/x86_64-linux-gnu/libQt5Gui.so /usr/lib/x86_64-linux-gnu/libQt5Network.so /usr/lib/x86_64-linux-gnu/libQt5Concurrent.so /usr/lib/x86_64-linux-gnu/libQt5Core.so -lGL -lpthread"`

To run the app in Cloud SIM mode, place a temporary `cornea_config.json` beside
the built binary with `tcp.enabled=true`, a non-empty `python.venv_path`, and
`python.use_subprocess=false`. Start `build-qmake-debug/CorneaController` on the
desktop display; it should expose the TCP server on port 5566 and simulate
devices `SIM0000` through `SIM0011`.

Useful smoke checks after the app is running:

- Python worker protocol without proprietary wheels: send `ping` then `shutdown`
  to `python/panel_worker.py` over stdin/stdout.
- TCP SIM hello world: call `listDevices`, then `powerOn`, `setBrightness`,
  `setFlip`, `getTemperature`, `getStatus`, and `powerOff` for `SIM0000`.

Real end-to-end hardware tests still require the Windows station dependencies
and physical Cornea/FTDI devices described in the repository docs.
