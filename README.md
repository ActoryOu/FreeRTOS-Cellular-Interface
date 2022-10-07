# FreeRTOS Cellular Interface

## On this page:
* [Introduction](#Introduction)
* [Getting Started](#Getting-Started)
    * [Download the source code](#Download-the-source-code)
    * [Folder Structure](#Folder-Structure)
* [Integrate FreeRTOS Cellular Interface with MCU platforms](#Integrate-FreeRTOS-Cellular-Interface-with-MCU-platforms)
* [Adding support for new cellular modems](#Adding-support-for-new-cellular-modems)
* [Integrate FreeRTOS Cellular Interface with Cellular Modules](#integrate-freertos-cellular-interface-with-cellular-modules)
* [Building Unit Tests](#Building-Unit-Tests)
* [Generating documentation](#Generating-documentation)
* [Contributing](#Contributing)

## Introduction

The Cellular Interface library implement a simple unified [Application Programing Interfaces (APIs)](https://www.freertos.org/Documentation/api-ref/cellular/index.html) that hide the complexity of AT commands. The cellular modems to be interchangeable with the popular options built upon TCP stack and exposes a socket-like interface to C programmers.

Even though applications can choose to use the FreeRTOS Cellular Interface API directly, the API is not designed for such a purpose. In a typical FreeRTOS system, applications use high level libraries, such as the [coreMQTT](https://github.com/FreeRTOS/coreMQTT) library and the [coreHTTP](https://github.com/FreeRTOS/coreHTTP) library, to communicate with other end points. Those high level libraries use an abstract interface, the [Transport Interface](https://github.com/FreeRTOS/coreMQTT/blob/main/source/interface/transport_interface.h), to send and receive data. A Transport Interface can be implemented on top of the FreeRTOS Cellular Interface.

Most cellular modems implement more or less the AT commands defined by the [3GPP TS v27.007](https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=1515) standard. This project provides an implementation of such standard AT commands in a [reusable common component](https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface/tree/main/source/include/common). The three Cellular libraries in this project all take advantage of that common code. The library for each modem only implements the vendor-specific AT commands, then exposes the complete Cellular API.

The common component that implements the 3GPP TS v27.007 standard has been written in compliance of the following code quality criteria:

* GNU Complexity scores are not over 8.
* MISRA coding standard. Any deviations from the MISRA C:2012 guidelines are documented in source code comments marked by "`coverity`".

## Getting Started

### Download the source code

To clone using HTTPS:

```
git clone https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface.git
```

Using SSH:

```
git clone git@github.com/FreeRTOS/FreeRTOS-Cellular-Interface.git
```

### Folder structure

At the root of this repository are these folders:

* source : reusable common code that implements the standard AT commands defined by 3GPP TS v27.007.
* docs : documentations.
* test : unit test and cbmc.
* tools : tools for Coverity static analysis and CMock.

## Integrate FreeRTOS Cellular Interface with MCU platforms

The FreeRTOS Cellular Interface runs on MCUs.  It uses an abstracted interface - the [Comm Interface](https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface/tree/main/source/interface/cellular_comm_interface.h), to communicate with cellular modems. A Comm Interface must be implemented as well on the MCU platform.  The most common implementations of the Comm Interface are over UART hardware, but it can be implemented over other physical interfaces such as SPI as well. The documentation of the Comm Interface is found within the [Cellular API References](https://www.freertos.org/Documentation/api-ref/cellular/cellular_porting.html#cellular_porting_comm_if). These are example implementations of the Comm Interface:

* [FreeRTOS windows simulator comm interface](https://github.com/FreeRTOS/FreeRTOS/blob/main/FreeRTOS-Plus/Demo/FreeRTOS_Cellular_Interface_Windows_Simulator/Common/comm_if_windows.c)
* [FreeRTOS Common IO UART comm interface](https://github.com/aws/amazon-freertos/blob/feature/cellular/vendors/st/boards/stm32l475_discovery/ports/comm_if/comm_if_uart.c)
* [STM32 L475 discovery board comm interface](https://github.com/aws/amazon-freertos/blob/feature/cellular/vendors/st/boards/stm32l475_discovery/ports/comm_if/comm_if_st.c)
* [Sierra Sensor Hub board comm interface](https://github.com/aws/amazon-freertos/blob/feature/cellular/vendors/sierra/boards/sensorhub/ports/comm_if/comm_if_sierra.c)

The FreeRTOS Cellular Interface uses kernel APIs for task synchronization and memory management.

## Adding support for new cellular modems

FreeRTOS Cellular Interface now supports AT commands, TCP offloaded Cellular abstraction Layer. In order to add support for a new cellular modem, the developer can use the [common component](https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface/tree/main/source/include/common) that has already implemented the 3GPP standard AT commands.

In order to port the [common component](https://www.freertos.org/Documentation/api-ref/cellular/cellular_porting_module_guide.html):

1. Implement the cellular modem porting interface defined in [cellular_common_portable.h](https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface/tree/main/source/include/common/cellular_common_portable.h) ([Document](https://www.freertos.org/Documentation/api-ref/cellular/cellular__common__portable_8h.html)).
2. Implement the subset of Cellular Library APIs that use vendor-specific (non-3GPP) AT commands. The APIs to be implemented are the ones not marked with an "o" in [this table](https://www.freertos.org/Documentation/api-ref/cellular/cellular_common__a_p_is.html).
3. Implement Cellular Library callback functions that handle vendor-specific (non-3GPP) Unsolicited Result Code (URC). The URC handlers to be implemented are the ones not marked with an "o" in [this table](https://www.freertos.org/Documentation/api-ref/cellular/cellular_common__u_r_c_handlers.html).

The [Cellular common APIs document](https://www.freertos.org/Documentation/api-ref/cellular/cellular_porting_module_guide.html) provides detail information required in each steps.
It is recommended that you start by cloning the implementation of one of the existing modems, then make modifications where your modem's vendor-specific (non-3GPP) AT commands are different.

 Current Example Implementations:

* [Quectel BG96](https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface-Reference-Quectel-BG96)
* [Sierra Wireless HL7802](https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface-Reference-Sierra-Wireless-HL7802)
* [U-Blox Sara-R4](https://github.com/FreeRTOS/FreeRTOS-Cellular-Interface-Reference-ublox-SARA-R4)

## Integrate FreeRTOS Cellular Interface with Cellular Modules

FreeRTOS Cellular Interface provides common interface for modems, but it needs cellular modem to be implemented for different modems. You can follow [Adding support for new cellular modems](#adding-support-for-new-cellular-modems) to add a new cellular modem, or take example implementations (BG96 / HL7802 / Sara-R4) as cellular modem into your project. Follow below steps to integrate FreeRTOS Cellular Interface into your project:

1. Clone this repository into your project.
2. Clone modem repository or follow [Adding support for new cellular modems](#adding-support-for-new-cellular-modems) to implement your own cellular modem on your project.
3. Build and execute!

We also provide [Demos for FreeRTOS-Cellular-Interface on Windows simulator](https://github.com/FreeRTOS/FreeRTOS/tree/main/FreeRTOS-Plus/Demo/FreeRTOS_Cellular_Interface_Windows_Simulator) as references for these three example Implementations.

## Building Unit Tests

### Checkout CMock Submodule
By default, the submodules in this repository are configured with `update=none` in [.gitmodules](.gitmodules) to avoid increasing clone time and disk space usage of other repositories (like [amazon-freertos](https://github.com/aws/amazon-freertos) that submodules this repository).


To build unit tests, the submodule dependency of CMock is required. Use the following command to clone the submodule:
```
git submodule update --checkout --init --recursive test/unit-test/CMock
```

### Platform Prerequisites

- For building the unit tests, **CMake 3.13.0** or later and a **C90 compiler**.
- For running unit tests, **Ruby 2.0.0** or later is additionally required for the CMock test framework (that we use).
- For running the coverage target, **gcov** and **lcov** are additionally required.

### Steps to build unit tests

1. Go to the root directory of this repository. (Make sure that the **CMock** submodule is cloned as described [above](#checkout-cmock-submodule).)

1. Run the *cmake* command: `cmake -S test -B build`

1. Run this command to build the library and unit tests: `make -C build all`

1. The generated test executables will be present in `build/bin/tests` folder.

1. Run `cd build && ctest` to execute all tests and view the test run summary.

## CBMC

To learn more about CBMC and proofs specifically, review the training material [here](https://model-checking.github.io/cbmc-training).

The `test/cbmc/proofs` directory contains CBMC proofs.

In order to run these proofs you will need to install CBMC and other tools by following the instructions [here](https://model-checking.github.io/cbmc-training/installation.html).

## Reference examples

Please refer to the demos of the Cellular Interface library [here](https://github.com/FreeRTOS/FreeRTOS/tree/main/FreeRTOS-Plus/Demo/FreeRTOS_Cellular_Interface_Windows_Simulator) using FreeRTOS on the Windows simulator platform. These can be used as reference examples for the library API.

## Generating documentation

The Doxygen references were created using Doxygen version 1.9.2. To generate the
Doxygen pages, please run the following command from the root of this repository:

```shell
doxygen docs/doxygen/config.doxyfile
```

## Contributing

See [CONTRIBUTING.md](./.github/CONTRIBUTING.md) for information on contributing.
