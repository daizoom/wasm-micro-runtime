"littlevgl" sample introduction
==============
This sample demonstrates that a graphic user interface application in WebAssembly by compiling the LittlevGL, an open-source embedded 2d graphic library into the WASM bytecode.

In this sample, the whole LittlevGL source code is built into the WebAssembly code with the user application. The platform interfaces defined by LittlevGL is implemented in the runtime and exported to the application through the declarations from source "ext_lib_export.c" as below:

        EXPORT_WASM_API(display_init),
        EXPORT_WASM_API(display_input_read),
        EXPORT_WASM_API(display_flush),
        EXPORT_WASM_API(display_fill),
        EXPORT_WASM_API(display_vdb_write),
        EXPORT_WASM_API(display_map),
        EXPORT_WASM_API(time_get_ms),

The runtime component supports building target for Linux and Zephyr/STM Nucleo board. The beauty of this sample is the WebAssembly application can have identical display and behavior when running from both runtime environments. That implies we can do majority of application validation from desktop environment as long as two runtime distributions support the same set of application interface.


Below pictures show the WASM application is running on an STM board with an LCD touch panel. 

![WAMR UI SAMPLE](../../doc/pics/vgl_demo2.png "WAMR UI DEMO STM32")

![WAMR UI SAMPLE](../../doc/pics/vgl_demo_linux.png "WAMR UI DEMO LINUX")


The number on top will plus one each second, and the number on the bottom will plus one when clicked. When users click the blue button, the WASM application increases the counter, and the latest counter value is displayed on the top banner of the touch panel. 

The sample also provides the native Linux version of application without the runtime under folder "vgl-native-ui-app". It can help to check differences between the implementations in native and WebAssembly.

Test on Linux
================================

Install required SDK and libraries
--------------
- 32 bit SDL(simple directmedia layer) (Note: only necessary when `WAMR_BUILD_TARGET` is set to `X86_32` when building WAMR runtime)
Use apt-get:
    `sudo apt-get install libsdl2-dev:i386`
Or download source from www.libsdl.org:
```
./configure C_FLAGS=-m32 CXX_FLAGS=-m32 LD_FLAGS=-m32
make
sudo make install
```
- 64 bit SDL(simple directmedia layer) (Note: only necessary when `WAMR_BUILD_TARGET` is set to `X86_64` when building WAMR runtime)
Use apt-get:
    `sudo apt-get install libsdl2-dev`
Or download source from www.libsdl.org:
```
./configure
make
sudo make install
```


Build and Run
--------------

- Build</br>
`./build.sh`</br>
    All binaries are in "out", which contains "host_tool", "vgl_native_ui_app", "ui_app.wasm" "ui_app_no_wasi.wasm "and "vgl_wasm_runtime".
- Run native Linux application</br>
`./vgl_native_ui_app`</br>

- Run WASM VM Linux applicaton & install WASM APP</br>
 First start vgl_wasm_runtime in server mode.</br>
`./vgl_wasm_runtime -s`</br>
 Then install wasm APP use host tool.</br>
`./host_tool -i ui_app -f ui_app.wasm`</br>

Test on Zephyr
================================
We can use a STM32 NUCLEO_F767ZI  board with ILI9341 display and XPT2046 touch screen to run the test. Then use host_tool to remotely install wasm app into STM32.
- Build WASM VM into Zephyr system</br>
 a. clone zephyr source code</br>
Refer to Zephyr getting started.</br>
https://docs.zephyrproject.org/latest/getting_started/index.html</br>
`west init zephyrproject`</br>
`cd zephyrproject`</br>
`west update`</br>
 b. copy samples</br>
    `cd zephyr/samples/`</br>
    `cp -a <wamr_root>samples/littlevgl/vgl-wasm-runtime vgl-wasm-runtime`</br>
    `cd vgl-wasm-runtime/zephyr_build`</br>
 c. create a link to wamr root dir</br>
   ` ln -s <wamr_root> wamr`</br>
 d. build source code</br>
    Since ui_app incorporated LittlevGL source code, so it needs more RAM on the device to install the application.
    It is recommended that RAM SIZE not less than 380KB.
    In our test use nucleo_f767zi, which is supported by Zephyr.

    `mkdir build && cd build`</br>
    `source ../../../../zephyr-env.sh`</br>
    `cmake -GNinja -DBOARD=nucleo_f767zi ..`</br>
   ` ninja flash`</br>

- Hardware Connections

```
+-------------------+-+------------------+
|NUCLEO-F767ZI       | ILI9341  Display  |
+-------------------+-+------------------+
| CN7.10             |         CLK       |
+-------------------+-+------------------+
| CN7.12             |         MISO      |
+-------------------+-+------------------+
| CN7.14             |         MOSI      |
+-------------------+-+------------------+
| CN11.1             | CS1 for ILI9341   |
+-------------------+-+------------------+
| CN11.2             |         D/C       |
+-------------------+-+------------------+
| CN11.3             |         RESET     |
+-------------------+-+------------------+
| CN9.25             |    PEN interrupt  |
+-------------------+-+------------------+
| CN9.27             |  CS2 for XPT2046  |
+-------------------+-+------------------+
| CN10.14            |    PC UART RX     |
+-------------------+-+------------------+
| CN11.16            |    PC UART RX     |
+-------------------+-+------------------+
```


- Install WASM application to Zephyr using host_tool</br>
First, connect PC and STM32 with UART. Then install to use host_tool.</br>
`./host_tool -D /dev/ttyUSBXXX -i ui_app -f ui_app_no_wasi.wasm`
**Note**: WASI is unavailable on zephyr currently, so you have to use the ui_app_no_wasi.wasm which doesn't depend on WASI.

- Install AOT version WASM application
`wamrc --target=thumbv7 --target-abi=eabi --cpu=cortex-m7 -o ui_app_no_wasi.aot ui_app_no_wasi.wasm`
`./host_tool -D /dev/ttyUSBXXX -i ui_app -f ui_app_no_wasi.aot`
