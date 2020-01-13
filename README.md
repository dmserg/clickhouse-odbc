# ODBC Driver for ClickHouse [![Build Status](https://travis-ci.org/clickhouse/clickhouse-odbc.svg?branch=master)](https://travis-ci.org/clickhouse/clickhouse-odbc) <!-- omit in toc -->

## Introduction <!-- omit in toc -->

This is the official ODBC driver implementation for accessing ClickHouse as a data source.

For more information on ClickHouse go to [ClickHouse home page](https://clickhouse.yandex).

For more information on what ODBC is go to [ODBC Overview](https://docs.microsoft.com/en-us/sql/odbc/reference/odbc-overview).

The canonical repo for this driver is located at [https://github.com/ClickHouse/clickhouse-odbc](https://github.com/ClickHouse/clickhouse-odbc).

See [LICENSE](LICENSE) file for licensing information.

## Table of content <!-- omit in toc -->

- [Installation](#installation)
- [Configuration](#configuration)
  - [Troubleshooting: driver manager tracing and driver logging](#troubleshooting-driver-manager-tracing-and-driver-logging)
- [Building from sources](#building-from-sources)
- [Appendices](#appendices)
  - [Run-time dependencies: Windows](#run-time-dependencies-windows)
  - [Run-time dependencies: macOS](#run-time-dependencies-macos)
  - [Run-time dependencies: Red Hat/CentOS](#run-time-dependencies-red-hatcentos)
  - [Run-time dependencies: Debian/Ubuntu](#run-time-dependencies-debianubuntu)
  - [Configuration: MDAC/WDAC (Microsoft/Windows Data Access Components)](#configuration-mdacwdac-microsoftwindows-data-access-components)
  - [Configuration: UnixODBC](#configuration-unixodbc)
  - [Configuration: iODBC](#configuration-iodbc)
  - [Enabling driver manager tracing: MDAC/WDAC (Microsoft/Windows Data Access Components)](#enabling-driver-manager-tracing-mdacwdac-microsoftwindows-data-access-components)
  - [Enabling driver manager tracing: UnixODBC](#enabling-driver-manager-tracing-unixodbc)
  - [Enabling driver manager tracing: iODBC](#enabling-driver-manager-tracing-iodbc)
  - [Building from sources: Windows](#building-from-sources-windows)
  - [Building from sources: macOS](#building-from-sources-macos)
  - [Building from sources: Red Hat/CentOS](#building-from-sources-red-hatcentos)
  - [Building from sources: Debian/Ubuntu](#building-from-sources-debianubuntu)

## Installation

Pre-built binary packages of the release versions of the driver available for the most common platforms at:

- [Releases](https://github.com/ClickHouse/clickhouse-odbc/releases)

Note, that since ODBC drivers are not used directly by a user, but rather accessed through applications, which in their turn access the driver through ODBC driver manager, user have to install the driver for the **same architecture** (32- or 64-bit) as the application that is going to access the driver. Moreover, both the driver and the application must be compiled for (and actually use during run-time) the **same ODBC driver manager implementation** (we call them "ODBC providers" here). There are three supported ODBC providers:

- ODBC driver manager associated with **MDAC** (Microsoft Data Access Components, sometimes referenced as WDAC, Windows Data Access Components) - the standard ODBC provider of Windows
- **UnixODBC** - the most common ODBC provider in Unix-like systems. Theoretically, could be used in Cygwin or MSYS/MinGW environments in Windows too.
- **iODBC** - less common ODBC provider, mainly used in Unix-like systems, however, it is the standard ODBC provider in macOS. Theoretically, could be used in Cygwin or MSYS/MinGW environments in Windows too.

If you don't see a package that matches your platforms, or the version of your system is significantly different than those of the available packages, or maybe you want to try a bleeding edge version of the code that hasn't been released yet, you can always build the driver manually from sources:

- [Building from sources](#building-from-sources)

Note, that it is always a good idea to install the driver from the corresponding **native** package (<!-- `.deb`, `.rpm`, -->`.msi`, etc., which you can also easily create if you are building from sources), than use the binaries that were manually copied to some folder.

Native packages will have all the dependency information so when you install the driver using a native package, all required run-time packages will be installed automatically. If you use manual packaging, i.e., just extract driver binaries to some folder, you also have to make sure that all the run-time dependencies are satisfied in your system manually:

- [Run-time dependencies: Windows](#run-time-dependencies-windows)
- [Run-time dependencies: macOS](#run-time-dependencies-macos)
- [Building from sources: Red Hat/CentOS](#building-from-sources-red-hatcentos)
- [Run-time dependencies: Debian/Ubuntu](#run-time-dependencies-debianubuntu)

## Configuration

The first step usually consists of registering the driver so that the corresponding ODBC provider is able to locate it.

The next step is defining one or more DSNs, associated with the newly registered driver, and setting driver-specific parameters in the body of those DSN definitions.

All this involves modifying a dedicated registry keys in case of MDAC, or editing `odbcinst.ini` (for driver registration) and `odbc.ini` (for DSN definition) files for UnixODBC or iODBC, directly or indirectly.

This will be done automatically using some default values if you are installing the driver using native installers.

Otherwise, if you are configuring manually, or need to modify the default configuration created by the installer, please see the exact locations of files (or registry keys) that need to be modified in the corresponding section below:

- [Configuration: MDAC/WDAC (Microsoft/Windows Data Access Components)](#configuration-mdacwdac-microsoftwindows-data-access-components)
- [Configuration: UnixODBC](#configuration-unixodbc)
- [Configuration: iODBC](#configuration-iodbc)

The list of DSN parameters recognized by the driver is as follows:

|      Parameter      |                                                      Default value                                                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :-----------------: | :----------------------------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|        `Url`        |                                                          empty                                                           | URL that points to a running ClickHouse instance, may include username, password, port, database, etc.                                                                                                                                                                                                                                                                                                                       |
|       `Proto`       | deduced from `Url`, or from `Port` and `SSLMode`: `https` if `443` or `8443` or `SSLMode` is not empty, `http` otherwise | Protocol, one of: `http`, `https`                                                                                                                                                                                                                                                                                                                                                                                            |
| `Server` or `Host`  |                                                    deduced from `Url`                                                    | IP or hostname of a server with a running ClickHouse instance on it                                                                                                                                                                                                                                                                                                                                                          |
|       `Port`        |                         deduced from `Url`, or from `Proto`: `8443` if `https`, `8123` otherwise                         | Port on which the ClickHouse instance is listening                                                                                                                                                                                                                                                                                                                                                                           |
|       `Path`        |                                                         `/query`                                                         | Path portion of the URL                                                                                                                                                                                                                                                                                                                                                                                                      |
| `UID` or `Username` |                                                        `default`                                                         | User name                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `PWD` or `Password` |                                                          empty                                                           | Password                                                                                                                                                                                                                                                                                                                                                                                                                     |
|     `Database`      |                                                        `default`                                                         | Database name to connect to                                                                                                                                                                                                                                                                                                                                                                                                  |
|      `Timeout`      |                                                           `30`                                                           | Connection timeout                                                                                                                                                                                                                                                                                                                                                                                                           |
|      `SSLMode`      |                                                          empty                                                           | Certificate verification method (used by TLS/SSL connections, ignored in Windows), one of: `allow`, `prefer`, `require`, use `allow` to enable [`SSL_VERIFY_PEER`](https://www.openssl.org/docs/manmaster/man3/SSL_CTX_set_verify.html) TLS/SSL certificate verification mode, [`SSL_VERIFY_PEER \| SSL_VERIFY_FAIL_IF_NO_PEER_CERT`](https://www.openssl.org/docs/manmaster/man3/SSL_CTX_set_verify.html) is used otherwise |
|  `PrivateKeyFile`   |                                                          empty                                                           | Path to private key file (used by TLS/SSL connections), can be empty if no private key file is used                                                                                                                                                                                                                                                                                                                          |
|  `CertificateFile`  |                                                          empty                                                           | Path to certificate file (used by TLS/SSL connections, ignored in Windows), if the private key and the certificate are stored in the same file, this can be empty if `PrivateKeyFile` is specified                                                                                                                                                                                                                           |
|    `CALocation`     |                                                          empty                                                           | Path to the file or directory containing the CA/root certificates (used by TLS/SSL connections, ignored in Windows)                                                                                                                                                                                                                                                                                                          |
|     `DriverLog`     |                                  `on` if `CMAKE_BUILD_TYPE` is `Debug`, `off` otherwise                                  | Enable or disable the extended driver logging                                                                                                                                                                                                                                                                                                                                                                                |
|   `DriverLogFile`   |               `\temp\clickhouse-odbc-driver.log`  on Windows, `/tmp/clickhouse-odbc-driver.log` otherwise                | Path to the extended driver log file (used when `DriverLog` is `on`)                                                                                                                                                                                                                                                                                                                                                         |

### Troubleshooting: driver manager tracing and driver logging

To debug issues with the driver, first things that need to be done are:

- enabling driver manager tracing:
  - [Enabling driver manager tracing: MDAC/WDAC (Microsoft/Windows Data Access Components)](#enabling-driver-manager-tracing-mdacwdac-microsoftwindows-data-access-components)
  - [Enabling driver manager tracing: UnixODBC](#enabling-driver-manager-tracing-unixodbc)
  - [Enabling driver manager tracing: iODBC](#enabling-driver-manager-tracing-iodbc)
- enabling driver logging, see `DriverLog` and `DriverLogFile` DSN parameters above
- making sure that the application is allowed to create and write these driver log and driver manager trace files

## Building from sources

The general requirements for building the driver from sources are as follows:

- CMake 3.12 and later
- C++17 and C11 capable compiler toolchain:
  - Clang 4 and later
  - GCC 7 and later
  - Xcode 10 and later
  - Microsoft Visual Studio 2017 and later

Additional requirements exist for each platform, which also depend on whether packaging and/or testing is performed.

See the exact steps for each platform in the corresponding section below:

- [Building from sources: Windows](#building-from-sources-windows)
- [Building from sources: macOS](#building-from-sources-macos)
- [Building from sources: Red Hat/CentOS](#building-from-sources-red-hatcentos)
- [Building from sources: Debian/Ubuntu](#building-from-sources-debianubuntu)

The list of configuration options recognized during the CMake generation step is as follows:

|                 Option                 |                      Default value                       | Description                                                        |
| :------------------------------------: | :------------------------------------------------------: | :----------------------------------------------------------------- |
|           `CMAKE_BUILD_TYPE`           |                     `RelWithDebInfo`                     | Build type, one of: `Debug`, `Release`, `RelWithDebInfo`           |
|          `CH_ODBC_ENABLE_SSL`          |                           `ON`                           | Enable TLS/SSL (required for utilizing `https://` interface, etc.) |
|        `CH_ODBC_ENABLE_INSTALL`        |                           `ON`                           | Enable install targets (required for packaging)                    |
|        `CH_ODBC_ENABLE_TESTING`        |            inherits value of `BUILD_TESTING`             | Enable test targets                                                |
| `CH_ODBC_PREFER_BUNDLED_THIRD_PARTIES` |                           `ON`                           | Prefer bundled over system variants of third party libraries       |
|     `CH_ODBC_PREFER_BUNDLED_POCO`      | inherits value of `CH_ODBC_PREFER_BUNDLED_THIRD_PARTIES` | Prefer bundled over system variants of Poco library                |
|      `CH_ODBC_PREFER_BUNDLED_SSL`      |     inherits value of `CH_ODBC_PREFER_BUNDLED_POCO`      | Prefer bundled over system variants of TLS/SSL library             |
|  `CH_ODBC_PREFER_BUNDLED_GOOGLETEST`   | inherits value of `CH_ODBC_PREFER_BUNDLED_THIRD_PARTIES` | Prefer bundled over system variants of Google Test library         |
|    `CH_ODBC_PREFER_BUNDLED_NANODBC`    | inherits value of `CH_ODBC_PREFER_BUNDLED_THIRD_PARTIES` | Prefer bundled over system variants of nanodbc library             |
|     `CH_ODBC_RUNTIME_LINK_STATIC`      |                          `OFF`                           | Link with compiler and language runtime statically                 |
|   `CH_ODBC_THIRD_PARTY_LINK_STATIC`    |                           `ON`                           | Link with third party libraries statically                         |
|       `CH_ODBC_DEFAULT_DSN_ANSI`       |                 `ClickHouse DSN (ANSI)`                  | Default ANSI DSN name                                              |
|     `CH_ODBC_DEFAULT_DSN_UNICODE`      |                `ClickHouse DSN (Unicode)`                | Default Unicode DSN name                                           |
|               `TEST_DSN`               |       inherits value of `CH_ODBC_DEFAULT_DSN_ANSI`       | ANSI DSN name to use in tests                                      |
|              `TEST_DSN_W`              |     inherits value of `CH_ODBC_DEFAULT_DSN_UNICODE`      | Unicode DSN name to use in tests                                   |

Configuration options above can be specified in the first `cmake` command (generation step) in a form of `-Dopt=val`.

## Appendices

### Run-time dependencies: Windows

All modern Windows systems come with preinstalled MDAC driver manager.

Another run-time dependecies are `C++ Redistributable for Visual Studio 2017` or same for `2019`, etc., depending on the package being installed, however the required DLL's are redistributed with the `.msi` installer, and you can choose to install them from there, if you don't already have them installed in your system.

### Run-time dependencies: macOS

#### iODBC <!-- omit in toc -->

Homebrew: execute the following in the terminal (assuming you have [Homebrew](https://brew.sh/) installed):

```sh
brew install poco openssl libiodbc
```

#### UnixODBC <!-- omit in toc -->

Homebrew: execute the following in the terminal (assuming you have [Homebrew](https://brew.sh/) installed):

```sh
brew install poco openssl unixodbc
```

### Run-time dependencies: Red Hat/CentOS

#### UnixODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo yum install openssl unixODBC
```

#### iODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo yum install openssl libiodbc
```

### Run-time dependencies: Debian/Ubuntu

#### UnixODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo apt install openssl unixodbc
```

#### iODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo apt install openssl libiodbc2
```

### Configuration: MDAC/WDAC (Microsoft/Windows Data Access Components)

TODO

### Configuration: UnixODBC

TODO

### Configuration: iODBC

TODO

### Enabling driver manager tracing: MDAC/WDAC (Microsoft/Windows Data Access Components)

Comprehensive explanations (possibly, with some irrelevant vendor-specific details though) on how to enable ODBC driver manager tracing could be found at the following links:

- [5.8.1 Enabling ODBC Tracing on Windows](https://dev.mysql.com/doc/connector-odbc/en/connector-odbc-configuration-trace-windows.html)
- [ODBC Troubleshooting: How to Enable Driver-manager Tracing](https://www.simba.com/blog/odbc-troubleshooting-tracing/)
- [Enabling Tracing](https://docs.microsoft.com/en-us/sql/odbc/reference/develop-app/enabling-tracing)

### Enabling driver manager tracing: UnixODBC

Comprehensive explanations (possibly, with some irrelevant vendor-specific details though) on how to enable ODBC driver manager tracing could be found at the following links:

- [ODBC Troubleshooting: How to Enable Driver-manager Tracing](https://www.simba.com/blog/odbc-troubleshooting-tracing/)
- [How do I turn unixODBC tracing on or off?](https://www.easysoft.com/support/kb/kb00945.html)

### Enabling driver manager tracing: iODBC

Comprehensive explanations (possibly, with some irrelevant vendor-specific details though) on how to enable ODBC driver manager tracing could be found at the following links:

- [ODBC Troubleshooting: How to Enable Driver-manager Tracing](https://www.simba.com/blog/odbc-troubleshooting-tracing/)
- [Tracing Application Behavior](http://www.iodbc.org/dataspace/doc/iodbc/wiki/iodbcWiki/FAQ#Tracing%20Application%20Behavior)

### Building from sources: Windows

#### Build-time dependencies <!-- omit in toc -->

CMake bundled with the recent versions of Visual Studio can be used.

An SDK required for building the ODBC driver is included in Windows SDK, which in its turn is also bundled with Visual Studio.

You will need to install WiX toolset to be able to generate `.msi` packages. You can download and install it from [WiX toolset home page](https://wixtoolset.org/).

#### Build steps <!-- omit in toc -->

All of the following commands have to be issued in Visual Studio Command Prompt:

- use `x86 Native Tools Command Prompt for VS 2019` or equivalent for 32-bit builds
- use `x64 Native Tools Command Prompt for VS 2019` or equivalent for 64-bit builds

Clone the repo with submodules:

```sh
git clone --recursive git@github.com:ClickHouse/clickhouse-odbc.git
```

Enter the cloned source tree, create a temporary build folder, and generate the solution and project files in it:

```sh
cd clickhouse-odbc
mkdir build
cd build

# Configuration options for the project can be specified in the next command in a form of '-Dopt=val'

# Use the following command for 32-bit build only.
cmake -A Win32 -DCMAKE_BUILD_TYPE=RelWithDebInfo ..

# Use the following command for 64-bit build only.
cmake -A x64 -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
```

Build the generated solution in-place:

```sh
cmake --build . -C RelWithDebInfo
cmake --build . -C RelWithDebInfo --target package
```

...and, optionally, run tests (note, that for non-unit tests, preconfigured driver and DSN entries must exist, that point to the binaries generated in this build folder):

```sh
cmake --build . -C RelWithDebInfo --target test
```

...or open the IDE and build `all`, `package`, and `test` targets manually from there:

```sh
cmake --open .
```

### Building from sources: macOS

#### Build-time dependencies <!-- omit in toc -->

You will need Xcode 10 or later and Command Line Tools to be installed, as well as [Homebrew](https://brew.sh/).

#### Build-time dependencies: iODBC <!-- omit in toc -->

Homebrew: execute the following in the terminal (assuming you have [Homebrew](https://brew.sh/) installed):

```sh
brew install git cmake make poco openssl libiodbc
```

#### Build-time dependencies: UnixODBC <!-- omit in toc -->

Homebrew: execute the following in the terminal (assuming you have [Homebrew](https://brew.sh/) installed):

```sh
brew install git cmake make poco openssl unixodbc
```

#### Build steps <!-- omit in toc -->

Clone the repo with submodules:

```sh
git clone --recursive git@github.com:ClickHouse/clickhouse-odbc.git
```

Enter the cloned source tree, create a temporary build folder, and generate a Makefile for the project in it:

```sh
cd clickhouse-odbc
mkdir build
cd build

# Configuration options for the project can be specified in the next command in a form of '-Dopt=val'.

# You may also add '-G Xcode' to the next command, in order to use Xcode as a build system or IDE, and generate the solution and project files instead of Makefile.
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
```

Build the generated solution in-place:

```sh
cmake --build . -C RelWithDebInfo
cmake --build . -C RelWithDebInfo --target package
```

...and, optionally, run tests (note, that for non-unit tests, preconfigured driver and DSN entries must exist, that point to the binaries generated in this build folder):

```sh
cmake --build . -C RelWithDebInfo --target test
```

...or, if you configured the project with '-G Xcode' initially, open the IDE and build `all`, `package`, and `test` targets manually from there:

```sh
cmake --open .
```

### Building from sources: Red Hat/CentOS

#### Build-time dependencies: UnixODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo yum groupinstall "Development Tools"
sudo yum install centos-release-scl
sudo yum install devtoolset-8
sudo yum install git cmake openssl-devel unixODBC-devel
```

#### Build-time dependencies: iODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo yum groupinstall "Development Tools"
sudo yum install centos-release-scl
sudo yum install devtoolset-8
sudo yum install git cmake openssl-devel libiodbc-devel
```

#### Build steps <!-- omit in toc -->

All of the following commands have to be issued right after this one command issued in the same terminal session:

```sh
scl enable devtoolset-8 -- bash
```

Clone the repo with submodules:

```sh
git clone --recursive git@github.com:ClickHouse/clickhouse-odbc.git
```

Enter the cloned source tree, create a temporary build folder, and generate a Makefile for the project in it:

```sh
cd clickhouse-odbc
mkdir build
cd build

# Configuration options for the project can be specified in the next command in a form of '-Dopt=val'

cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
```

Build the generated solution in-place:

```sh
cmake --build . -C RelWithDebInfo
cmake --build . -C RelWithDebInfo --target package
```

...and, optionally, run tests (note, that for non-unit tests, preconfigured driver and DSN entries must exist, that point to the binaries generated in this build folder):

```sh
cmake --build . -C RelWithDebInfo --target test
```

### Building from sources: Debian/Ubuntu

#### Build-time dependencies: UnixODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo apt install build-essential git cmake libpoco-dev libssl-dev unixodbc-dev
```

#### Build-time dependencies: iODBC <!-- omit in toc -->

Execute the following in the terminal:

```sh
sudo apt install build-essential git cmake libpoco-dev libssl-dev libiodbc2-dev
```

#### Build steps <!-- omit in toc -->

Assuming, that the system `cc` and `c++` are pointing to the compilers that satisfy the minimum requirements from [Building from sources](#building-from-sources).

Clone the repo with submodules:

```sh
git clone --recursive git@github.com:ClickHouse/clickhouse-odbc.git
```

Enter the cloned source tree, create a temporary build folder, and generate a Makefile for the project in it:

```sh
cd clickhouse-odbc
mkdir build
cd build

# Configuration options for the project can be specified in the next command in a form of '-Dopt=val'

cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
```

Build the generated solution in-place:

```sh
cmake --build . -C RelWithDebInfo
cmake --build . -C RelWithDebInfo --target package
```

...and, optionally, run tests (note, that for non-unit tests, preconfigured driver and DSN entries must exist, that point to the binaries generated in this build folder):

```sh
cmake --build . -C RelWithDebInfo --target test
```
