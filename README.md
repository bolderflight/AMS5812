# ams5812
This library communicates with [AMS-5812](https://www.analog-micro.com/en/products/pressure-sensors/board-mount-pressure-sensors/ams5812/) pressure transducers.
   * [License](LICENSE.md)
   * [Changelog](CHANGELOG.md)
   * [Contributing guide](CONTRIBUTING.md)

# Description
The Analog Microelectronics AMS-5812 pressure transducers are fully signal conditioned, amplified, and temperature compensated over a temperature range of -25 to +85 C. These sensors generate data with high precision, high stability and low drift. Digital measurements are sampled with a 14 bit resolution. The AMS 5812 sensors are available in a wide variety of pressure ranges and in configurations suited for barometric, differential, and bidirectional differential measurement.

# Usage
This library communicates with the AMS-5812 sensors using an I2C interface. The default I2C address for the AMS-5812 is 0x78; however, a USB starter kit may be purchased to enable programming additional slave addresses. Pressure and temperature data can be provided at rates of up to 2 kHz.

## Installation
CMake is used to build this library, which is exported as a library target called *ams5812*. The header is added as:

```
#include "ams5812/ams5812.h"
```
Note that you'll need CMake version 3.13 or above; it is recommended to build and install CMake from source, directions are located in the [CMake GitLab repository](https://github.com/Kitware/CMake).

The library can be also be compiled stand-alone using the CMake idiom of creating a *build* directory and then, from within that directory issuing:

```
cmake .. -DMCU=MK66FX1M0
make
```

This will build the library and an example executable called *ams5812_example*. The example executable source files are located at *examples/ams5812_example.cc*. This code is built and tested on AARCH64 and AMD64 systems running Linux and AMD64 systems running the Windows Subsystem for Linux (WSL). The [arm-none-eabi](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads) toolchain must be installed in your Linux environment.

Notice that the *cmake* command includes a define specifying the microcontroller the code is being compiled for. This is required to correctly configure the code, CPU frequency, and compile/linker options. The available MCUs are:
   * MK20DX128
   * MK20DX256
   * MK64FX512
   * MK66FX1M0
   * MKL26Z64
   * IMXRT1062_T40
   * IMXRT1062_T41

These are known to work with the same packages used in Teensy products. Also switching the MK66FX1M0 or MK64FX512 from BGA to LQFP packages is known to work well. Swapping packages of other chips is probably fine, as long as it's only a package change.

The *ams5812_example* targets creates an executable for communicating with the sensor using I2C communication. The example target also has a *_hex* for creating the hex file and a *_upload* to upload the software to the microcontroller. 

## Namespace
This library is within the namespace *sensors*.

## Methods

**Ams5812(i2c_t3 &ast;bus, uint8_t addr, Transducer type)** Creates an Ams5812 object. A pointer to the I2C bus object is passed along with the I2C address of the sensor and the AMS-5812 transducer type. The enumerated transducer types are:

| Sensor Name       | Enumerated Type  | Pressure Type              | Pressure Range       |
| -----------       | ---------------  | ---------------            | ---------------      |
| AMS 5812-0000-D   | AMS5812_0000_D   | differential / relative    | 0...517 Pa           |
| AMS 5812-0001-D   | AMS5812_0001_D   | differential / relative    | 0...1034 Pa          |
| AMS 5812-0000-D-B | AMS5812_0000_D_B | bidirectional differential | -517...+517 Pa       |
| AMS 5812-0001-D-B | AMS5812_0001_D_B | bidirectional differential | -1034...+1034 Pa     |
| AMS 5812-0003-D   | AMS5812_0003_D   | differential / relative    | 0...2068 Pa          |
| AMS 5812-0008-D   | AMS5812_0008_D   | differential / relative    | 0...5516 Pa          |
| AMS 5812-0015-D   | AMS5812_0015_D   | differential / relative    | 0...10342 Pa         |
| AMS 5812-0003-D-B | AMS5812_0003_D_B | bidirectional differential | -2068...+2068 Pa     |
| AMS 5812-0008-D-B | AMS5812_0008_D_B | bidirectional differential | -5516...+5516 Pa     |
| AMS 5812-0015-D-B | AMS5812_0015_D_B | bidirectional differential | -10342...+10342 Pa   |
| AMS 5812-0030-D   | AMS5812_0030_D   | differential / relative    | 0...20684 Pa         |
| AMS 5812-0050-D   | AMS5812_0050_D   | differential / relative    | 0...34474 Pa         |
| AMS 5812-0150-D   | AMS5812_0150_D   | differential / relative    | 0...103421 Pa        |
| AMS 5812-0300-D   | AMS5812_0300_D   | differential / relative    | 0...206843 Pa        |
| AMS 5812-0600-D   | AMS5812_0600_D   | differential / relative    | 0...413685 Pa        |
| AMS 5812-1000-D   | AMS5812_1000_D   | differential / relative    | 0...689476 Pa        |
| AMS 5812-0030-D-B | AMS5812_0030_D_B | bidirectional differential | -20684...+20684 Pa   |
| AMS 5812-0050-D-B | AMS5812_0050_D_B | bidirectional differential | -34474...+34474 Pa   |
| AMS 5812-0150-D-B | AMS5812_0150_D_B | bidirectional differential | -103421...+103421 Pa |
| AMS 5812-0150-B   | AMS5812_0150_B   | barometric                 | 75842...120658 Pa    |
| AMS 5812-0150-A   | AMS5812_0150_A   | absolute                   | 0...103421 Pa        |
| AMS 5812-0300-A   | AMS5812_0300_A   | absolute                   | 0...206843 Pa        |

For example, the following code declares an AMS5812 object called *ams* with an AMS5812-0008-D sensor located on I2C bus 0 with an I2C address of 0x10:

```C++
sensors::Ams5812 ams(&Wire, 0x10, sensors::Ams5812::AMS5812_0008_D);
```

**bool Begin()** Initializes communication with the sensor and configures the sensor driver for the specified transducer. True is returned if communication is able to be established with the sensor and configuration completes successfully, otherwise, false is returned.

```C++
bool status = ams.Begin();
if (!status) {
  // ERROR
}
```

**bool Read()** Reads data from the AMS-5812 and stores the data in the Ams5812 object. Returns true if data is successfully read, otherwise, returns false.

```C++
/* Read the sensor data */
if (ams.Read()) {
}
```

**float pressure_pa()** Returns the pressure data from the Ams5812 object in units of Pa.

```C++
float pressure = ams.pressure_pa();
```

**float die_temperature_c** Returns the die temperature of the sensor from the Ams5812 object in units of degrees C.

```C++
float temperature = ams.die_temperature_c();
```
