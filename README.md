# Azure software radio Out of Tree Module

The gr-azure Out of Tree (OOT) Module allows users to easily leverage Azure cloud resources from within a GNU Radio flowgraph. You can use this OOT module with your existing GNU Radio development environment, or within a VM in the cloud.  Example use-cases involve storing and retrieving RF recordings from Blob (file) storage, communicating with [DIFI](https://dificonsortium.org/about/) devices from within GNU Radio, or creating complex cloud applications using Azure Event Hubs as a bridge between your flowgraph and [Azure services](https://azure.microsoft.com/en-us/services/).  We are excited to see what can be created by combining GNU Radio with the power and scalability of the cloud!

<center><img src="images/example_flowgraph.png"></center>

## Table of Contents
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installing Azure software radio OOT](#installing-azure-software-radio-oot)
  - [Running the Unit Tests](#running-the-unit-tests)
  - [Resolutions to Common Problems During Installation and Tests](#resolutions-to-common-Problems-During-Installation-and-Tests)
- [Examples](#examples)
- [Blocks Documentation](#azure-software-radio-out-of-tree-module-blocks)
  - [Key Vault Block](#key-vault-block)
  - [Blob Blocks](#blob-blocks)
  - [Event Hub Blocks](#event-hub-blocks)
  - [DIFI Blocks using the IEEE-ISTO Std 4900-2021: Digital IF Interoperability Standard](#difi-blocks-using-the-ieee-isto-std-4900-2021-digital-if-interoperability-standard)
  - [REST API Block](#rest-api-block)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Support](#support)
- [Contributing](#contributing)
- [License](#license)
- [Trademarks](#trademarks)

## Getting Started

The following installation instructions will get you up and running with the Azure OOT Module on your local machine.

### Prerequisites
This project depends on the GNU Radio 3.9.x or 3.10.x runtime and development dependencies. See the [GNU Radio installation instructions](https://wiki.gnuradio.org/index.php/InstallingGR#From_Binaries) for steps on
installing GNU Radio from binaries (note that GNU Radio packaged with Ubuntu 20 is only 3.8). Some package managers do not automatically install all of the development dependencies,
so you may need to separately install and configure some of them. The Azure software radio OOT module requires the following:

- GNU Radio 3.9.x or 3.10.x
- Python 3.8 or greater
- python3-pip
- cmake
- liborc-dev
- doxygen
- pytest
- pybind11

See the installation steps below for how to install these dependencies.

**NOTE:** If using the Azure CLI, you will need version 2.17.1 or newer. This module is not compatible with
the Azure CLI availabile in the default apt repository on Ubuntu 20. If this older version of the Azure CLI is present
on your system, the installation of this OOT module may fail or the module may crash at runtime. Please install the
Azure CLI according to the recommendations found in [AZ CLI Installation in Linux](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt).

### Installing Azure software radio OOT

The following steps show how to install this OOT module on a Debian-based OS with GNU Radio already installed.  They have been tested to work under Ubuntu 20.  If you see error messages after running any of the following steps, stop and check our [FAQ](./docs/FAQ.md) for how to
resolve the problem.

```bash
sudo apt-get install python3-pip cmake liborc-dev doxygen
sudo pip install pytest pybind11

git clone https://github.com/microsoft/gr-azure.git
cd gr-azure

sudo pip install -r python/requirements.txt

mkdir build
cd build
cmake ..
make -j4
sudo make install
sudo ldconfig
```

(If you run into a non-existent path error after `cmake ..`, try recreating your build directory and use `cmake -DCMAKE_FIND_ROOT_PATH=/usr ..` instead)

At this point the OOT module should have been installed, and you should see additional blocks within GNU Radio Companion.

### Running the Unit Tests
If you would like to run the QA tests, there are two methods:
 1. From within the build directory, run:
    ```
    make test
    ```

    You can review detailed test output (including any failures) in Testing/Temporary/LastTest.log.

 2. From within the python directory, run:
    ```
    python -m pytest qa_*
    ```

    Pytest will show detailed test results directly in the output of this command.

### Resolutions to Common Problems During Installation and Tests
For a list common problems and resolutions, please check our [FAQ](./docs/FAQ.md) to see if your issue has been addressed.

## Examples
The [examples](./examples) folder has a collection of flowgraphs and supporting files that illustrate common ways of
using the blocks provided in this module. See the [README in the examples folder](./examples/README.md) to get started.

## Azure software radio Out of Tree Module Blocks

### Key Vault Block
The Key Vault block allows users to pull down keys and secrets from an [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview) in GNU Radio.
It is expected that the user will setup and store secrets in an Azure Key Vault prior to pulling down keys using this block. To create a Key Vault, see [Create Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/secrets/quick-create-cli).

For a brief tutorial on using this block, see the [Key Vault Example](./examples/README.md#key-vault-example).


### Blob Blocks
The two Blob blocks (source and sink) provide an interface to read and write samples to [Azure Blob storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) in GNU Radio.
It is expected that the user will setup a storage account and a container prior to accessing Blob storage with the Blob source and sink blocks. To create a storage account, see [Create Storage Account](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal).

 * __Blob Source Block__\
	The Blob source block reads samples from Azure Blob storage. This block currently supports block blobs and the following outputs: Complex float32, Complex int16, Complex int8, float, int, short and byte (Page blobs and append blobs are not supported at this time).

 * __Blob Sink Block__\
	The Blob sink block writes samples to Azure Blob storage. This block currently supports block blobs and the following inputs:  Complex float32, Complex int16, Complex int8, float, int, short and byte (Page blobs and append blobs are not supported at this time).

There are several ways to authenticate to the Azure blob backend, these blocks support authentication using a connection string, a URL with an embedded SAS token, or use credentials supported by the DefaultAzureCredential class.

For a brief tutorial on using these blocks, see the [Blob Examples](./examples/README.md#Blob-Source-and-Sink-Examples).

### Event Hub Blocks
The Event Hub blocks (source and sink) provide an interface to send and receive events to [Azure Event Hubs](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-about) using the message passing interface in GNU Radio.
It is expected that the user will create an Event Hubs namespace, Event Hub entity and consumer group prior to using the Event Hub source and sink blocks. To create an Event Hub, see [Create an Event Hub](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-create).

 * __EventHub Source Block__\
	The EventHub source block receives a JSON formatted event message from Azure Event Hub and converts it to GNU Radio PMT format.

 * __EventHub Sink Block__\
	The EventHub sink block converts a PMT message to JSON and sends it to Azure Event Hub.

These blocks support multiple ways to authenticate to the Azure Event Hub backend, such as using a connection string, a SAS token, or use credentials supported by the DefaultAzureCredential class.

For a brief tutorial on using these blocks, see the [Event Hub Examples](./examples/README.md#Event-Hub-Examples).

### DIFI Blocks Using the IEEE-ISTO Std 4900-2021: Digital IF Interoperability Standard
This is a set of GNU Radio blocks based on IEEE-ISTO Std 4900-2021: Digital IF Interoperability Standard version 1.0.

There are two DIFI blocks (source and sink) as part of this OOT module. The Bit Depths currently supported are 8 and 16 with upcoming support for the full range of bit depths specified in the DIFI standard.

 * __DIFI Source Block__\
	The DIFI source block receives UDP DIFI packets from a given IP address and port. It then forwards them to GNU Radio as a complex64 (gr_complex) or signed complex 8 (std::complex<char>).
	This block emits the following tags in the following situations:
	  - pck_n tag: Emitted when a missed packet occurs, will update the upstream blocks with the current packet number to expect and the current time stamps
	  - context tag: Emitted when a new DIFI context packet is received with the context packet dynamic information
	  - static_change: Emitted when the static parts of the DIFI context packet changes

    The DIFI Advanced tab contains more advanced settings for the DIFI block and should be used by users who know the devices and network in use.

   Context Packet Mismatch Behavior:
      - Default: Throws exceptions if context packet is incorrect or non-compliant
      - Ignore Mismatches - Forward data, no warnings: Entirely ignore the context packet, only forwards data
      - Throw Warnings - Forward: Displays Warnings about context packet mismatch or non-compliant context packets, but still forward DIFI data
      - Throw Warnings - No Forward: Displays Warnings about context packet mismatch or non-compliant context packets, but won't forward data until a correct context packet is received or one that matches the given settings

 * __DIFI Sink Block__\
	The DIFI sink block forwards packets to a given IP address and port number and packets the data with the given bit depth. This block operates in two modes, standalone and paired:

	- Pair Mode: The block expects to be paired with a DIFI source block that sends context packets, timing information, and packet count information. The sink block forwards context packets received via tags. For data packets, it adds the correct timestamps and packet number. The data format is packeted as complex64 (gr_complex) or complex signed 8 (std::complex<char>) samples.

	- Standalone Mode: In standalone mode, it is expected the user will supply the context packet information via GRC or the constructor of the class. For now, the context packet payload data are static once specified by the user. Similar to paired mode, the data format to pack is complex64 (gr_complex) or complex signed 8 (std::complex<char>) samples.

	Scaling Mode: To help mitigate quantization error, the DIFI Sink has an optional helper feature to apply a gain & offset to the input signal. The first mode "Manual" allows a user to manually set gain & offset. In Min-Max mode the user supplies the max and min expected I & Q values and the block solves for a gain & offset based on these and the specified bit depth.

	Note: this block converts from float 32 I and Q down to the specified bit depth for I and Q, which can cause significant quantization error for small signals.

For a brief tutorial on using these blocks, see the [DIFI Examples](./examples/README.md#difi-examples).


### REST API Block
The REST API block allows users to get status and configure a running top block in GNU Radio. It starts a server in the configured port and restricts which settings and variables in a flowgraph are readable, writable or callable.

To get status from a flowgraph, a user can hit the status endpoint as follow

``curl -X GET http://<IP>:<port>/status``

To configure or write the exposed variables, the following command

``curl -X PUT http://<IP>:<port>/config -H 'Content-Type: application/json' -d '{"<variable>":<value>}``

To execute a callback or function within a top block, use the following route

``curl -X PUT http://<IP>:<port>/call -H 'Content-Type: application/json' -d '{"<function name>":<parameter>}``

For a brief tutorial on using this block, see the [REST API Example](./examples/README.md#rest-api-example).


## Frequently Asked Questions
For a list of common questions, including problems and resolutions, please check our [FAQ](./docs/FAQ.md)

## Support

This project uses [GitHub Issues](https://github.com/microsoft/gr-azure/issues) to track bugs and feature requests. Please refer to our [Support Guide](SUPPORT.md#how-to-file-issues-and-get-help) for more details.

Before filing a new issue, please check our [FAQ](./docs/FAQ.md) for potential solutions to common problems.

Starting with [GNU Radio](https://github.com/gnuradio/gnuradio) maint-3.9, this project will support the same set of [maintenance branches](https://github.com/gnuradio/gnuradio/branches) tracked by GNU Radio.

## Contributing

[Contributing Guide](./CONTRIBUTING.md)

## License

This project is licensed under the GNU General Public License v3.0 or later - see the [LICENSE](LICENSE.txt) file for details

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft trademarks or logos is subject to and must follow Microsoft's Trademark & Brand Guidelines. Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship. Any use of third-party trademarks or logos are subject to those third-party's policies.