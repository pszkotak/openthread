# OpenThread on nRF52811 Example

This directory contains example platform drivers for Nordic Semiconductor nRF52811 SoC

This example platform is under development.

## Toolchain

Download and install [GNU toolchain for ARM Cortex-M][gnu-toolchain].

[gnu-toolchain]: https://launchpad.net/gcc-arm-embedded

In a Bash terminal, follow these instructions to install the GNU toolchain and
other dependencies.

```bash
$ cd <path-to-openthread>
$ ./script/bootstrap
```

## Building the examples

```bash
$ cd <path-to-openthread>
$ ./bootstrap
$ make -f examples/Makefile-nrf52811
```

After a successful build, the `elf` files can be found in
`<path-to-openthread>/output/nrf52811/bin`.  You can convert them to `hex`
files using `arm-none-eabi-objcopy`:
```bash
$ arm-none-eabi-objcopy -O ihex ot-cli-ftd ot-cli-ftd.hex
```

## Native SPI Slave support

You can build the libraries with support for native SPI Slave.
To do so, build the libraries with the following parameter:
```
$ make -f examples/Makefile-nrf52811 NCP_SPI=1
```

With this option enabled, SPI communication between the NCP example and wpantund is possible
(provided that the wpantund host supports SPI Master). To achieve that, an appropriate SPI device
should be chosen in wpantund configuration file, `/etc/wpantund.conf`. You can find an example below.
```
Config:NCP:SocketPath "system:/usr/bin/ot-ncp /usr/bin/spi-hdlc-adapter -- '--stdio -i /sys/class/gpio/gpio25 /dev/spidev0.1'"
```

[spi-hdlc-adapter][spi-hdlc-adapter]
is a tool that can be used to perform communication between NCP and wpantund over SPI.
In the above example it is assumed that `spi-hdlc-adapter` is installed in `/usr/bin`.

The default SPI Slave pin configuration for nRF52811 is defined in `examples/platforms/nrf52811/platform-config.h`.

[spi-hdlc-adapter]: https://github.com/openthread/openthread/tree/master/tools/spi-hdlc-adapter

## Flashing the binaries

Flash the compiled binaries onto nRF52811 using `nrfjprog` which is
part of the [nRF5x Command Line Tools][nRF5x-Command-Line-Tools].

[nRF5x-Command-Line-Tools]: https://www.nordicsemi.com/eng/Products/nRF52840#Downloads

```bash
$ nrfjprog -f nrf52 --chiperase --program output/nrf52811/bin/ot-cli-mtd.hex --reset
```

## Running the example

1. Prepare two boards with the flashed `CLI Example` (as shown above).
2. The CLI example uses UART connection. To view raw UART output, start a terminal
   emulator like PuTTY and connect to the used COM port with the following UART settings:
    - Baud rate: 115200
    - 8 data bits
    - 1 stop bit
    - No parity
    - HW flow control: RTS/CTS

   On Linux system a port name should be called e.g. `/dev/ttyACM0` or `/dev/ttyACM1`.
3. Open a terminal connection on the first board and start a new Thread network.

 ```bash
 > panid 0xabcd
 Done
 > ifconfig up
 Done
 > thread start
 Done
 ```

4. After a couple of seconds the node will become a Leader of the network.

 ```bash
 > state
 Leader
 ```

5. Open a terminal connection on the second board and attach a node to the network.

 ```bash
 > panid 0xabcd
 Done
 > ifconfig up
 Done
 > thread start
 Done
 ```

6. After a couple of seconds the second node will attach and become a Child.

 ```bash
 > state
 Child
 ```

7. List all IPv6 addresses of the first board.

 ```bash
 > ipaddr
 fdde:ad00:beef:0:0:ff:fe00:fc00
 fdde:ad00:beef:0:0:ff:fe00:9c00
 fdde:ad00:beef:0:4bcb:73a5:7c28:318e
 fe80:0:0:0:5c91:c61:b67c:271c
 ```

8. Choose one of them and send an ICMPv6 ping from the second board.

 ```bash
 > ping fdde:ad00:beef:0:0:ff:fe00:fc00
 16 bytes from fdde:ad00:beef:0:0:ff:fe00:fc00: icmp_seq=1 hlim=64 time=8ms
 ```

For a list of all available commands, visit [OpenThread CLI Reference README.md][CLI].

[CLI]: ./../../../src/cli/README.md

## Logging module

By default, the OpenThread's logging module provides functions to output logging
information over SEGGER's Real Time Transfer (RTT).

RTT output can be viewed in the J-Link RTT Viewer, which is available from SEGGER.
The viewer is also included in the nRF Tools. To read or write messages over RTT,
connect an nRF5 development board via USB and run the J-Link RTT Viewer.

Select the correct target device (nRF52) and the target interface "SWD".

The intended log level can be set using `OPENTHREAD_CONFIG_LOG_LEVEL` define.

## Disabling the Mass Storage Device

Due to a known issue in Segger’s J-Link firmware, depending on your version, you might experience data corruption or drops if you use the serial port. You can avoid this issue by disabling the Mass Storage Device:

 - On Linux or macOS (OS X), open JLinkExe from the terminal.
 - On Microsoft Windows, open the J-Link Commander application.

Run the following command: `MSDDisable`

## Diagnostic module

nRF52811 port extends [OpenThread Diagnostics Module][DIAG].

You can read about all the features [here][nRFDIAG].

[DIAG]: ./../../../src/diag/README.md
[nRFDIAG]: DIAG.md

## Radio driver documentation

The radio driver comes with documentation that describes the operation of state
machines in this module. To open the `*.uml` sequence diagrams, use [PlantUML][PlantUML-url].

[PlantUML-url]: http://plantuml.com/
