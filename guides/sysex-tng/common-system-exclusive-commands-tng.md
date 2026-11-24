# Common System Exclusive Commands TNG

**Revision 5**

## Overview

This document details the MIDI system exclusive messages used in some iConnectivity products. A list of products that support these messages is provided in the Product IDs section near the end of this document. An overview of the message format and a general guide for using these messages is provided in this introductory chapter.

### Message Classes, Data Classes, Data Blocks and Commands

Messages classes define specific operations that can be performed on a device. For example, getting values from a device, changing values in a device, or changing a device’s operating mode. Data classes are used to define groups of related values. For example, hardware configuration values, MIDI configuration values, or the current audio mixer values. Combining a message class with a data class creates a specific message type. For example, getting hardware configuration values, changing MIDI configuration values, or getting the current audio mixer values.

Data blocks are used as building material to construct a message. Different types of data blocks are used to denote a functional block type. For example, one kind of data block contains a list of parameters to retrieve from a device, another kind of data block contains the parameter values.

All messages begin with a common header and end with a common footer. Between the header and the footer is the message content comprising the message type (message class and data class) and a number of data blocks. For example, to retrieve a list of hardware values for a specific hardware port the following message would be used:

```text
Header
Message Type  = retrieve hardware values
Data Block #1 = hardware port ID
Data Block #2 = a list of hardware parameter IDs
Footer
```

After receiving the above message, the device would respond with the following message to return the hardware values:

```text
Header
Message Type  = return hardware values
Data Block #1 = hardware port ID
Data Block #2 = a list of hardware parameter IDs and their associated values
Footer
```

Commands are a type of message class that instructs a device to perform a function. Commands do not use data classes but they use data blocks. Examples include rebooting a device, saving presets in RAM to non-volatile storage, or starting a bulk data transfer operation.

### Parameters

The bulk of this document comprises data class parameter definitions. All data that can be read and written is accessed via a parameter ID unique to each data class. Many parameters are atomized down to a single value but in some cases bitmaps and structures are used to group several values together when it is convenient to do so. In some cases, structures use a sub-ID field to denote each member of the structure.

**Parameters have the following attributes:**

#### Read-Only or Read-Write

Read-Only parameters are those that cannot be modified by a user. Some read-only parameters have a constant value meaning that they never change (denoted as RC parameters). For example, the number of Ethernet ports on a device, or the MAC address associated with each Ethernet port. The remaining read-only parameters are dynamic meaning that the value can change as the device is operating (denoted as RD parameters). For example the dynamic IP address assigned to an Ethernet port, or the session name that initiated an RTP-MIDI session with the device. A host application need only read RC parameters once during a communication session but RD parameters should be refreshed as necessary (depending on the needs of the host application).

Read-Write parameters are those that can be modified by a user. Some read-write parameters require that the device be restarted before the new value is used (denoted as WB parameters). For example, the number of audio channels to use on a USB device port, or the audio sample rate and audio synchronization method used. However, most read-write parameters support real-time modification and these can be modified at any time and take effect instantly (denoted as WN parameters). For example, MIDI filtering, audio routing, or audio mixer settings. A host application should refresh all read-write parameters regularly during a communication session (depending on the needs of the host application).

#### Global or Preset

Some products support storing multiple configurations, or presets, “in the box” and allow changing between these presets at runtime without having to reboot the device. For these products, the parameters that can be specified on a per-preset level are denoted as preset (P) parameters. Parameters that apply to all presets (or for products that don’t support presets) are denoted as global (G) parameters. For example, most parameters related to MIDI port configuration (e.g. filtering, routing) are preset parameters whereas the configuration of an Ethernet port (e.g. IP address) is a global parameter.

#### Scene Applicable

Some products support multiple scenes within a preset. Typically, scenes are used to manage audio and MIDI routing for failover systems and employ special rules to deal with scene management and selection. Parameters that can be specified on a per-scene level are denoted as scene (S) parameters. Parameters that cannot be specified on a per-scene level are denoted as not-scene (T) parameters.

#### Parameter Attribute Notation

The following notation is used to summarize parameter attributes:

| Code | Meaning |
|---|---|
| RD | read-only dynamic parameter |
| RC | read-only constant parameter |
| WN | writeable normal parameter |
| WB | writeable reboot parameter |
| P | preset parameter |
| G | global parameter |
| S | scene parameter |
| T | not a scene parameter |

_Examples:_

- RCGT = global read-only constant parameter (no scene support)
- WBPT = preset writeable parameter that requires a reboot to take effect (no scene support)
- RDPT = preset read-only dynamic parameter (no scene support)
- WNPS = preset writeable normal scene parameter

### Memory Model

#### Non-Volatile Storage (FLASH)

The memory model for global, preset, and scene parameters is shown below for non-volatile storage (for a device that supports two scenes per preset and 4 presets). All WB parameters are in the global section so that changing presets does not require rebooting a device. At runtime, the globals and one preset are copied to the work area in RAM where they can be manipulated by system exclusive commands and then saved to non-volatile storage.

| Name | Description |
|---|---|
| Global<br>Parameters | RCGT/RDGT/WNGT parameters.<br>WBGT parameters. |
| Preset #1 Parameters | RCPT/RDPT/WNPT parameters for preset #1 including:<br>1. RCPS/RDPS/WNPS parameters for scene #1<br>2. RCPS/RDPS/WNPS parameters for scene #2 |
| Preset #2 Parameters | RCPT/RDPT/WNPT parameters for preset #2 including:<br>1. RCPS/RDPS/WNPS parameters for scene #1<br>2. RCPS/RDPS/WNPS parameters for scene #2 |
| Preset #3 Parameters | RCPT/RDPT/WNPT parameters for preset #3 including:<br>1. RCPS/RDPS/WNPS parameters for scene #1<br>2. RCPS/RDPS/WNPS parameters for scene #2 |
| Preset #4 Parameters | RCPT/RDPT/WNPT parameters for preset #4 including:<br>1. RCPS/RDPS/WNPS parameters for scene #1<br>2. RCPS/RDPS/WNPS parameters for scene #2 |

#### Volatile Storage (RAM)

The memory model for global, preset, and scene parameters is shown below for volatile storage (for a device that supports two scenes per presets, N presets, and one shadow area). Devices support a work area that always contains the currently active global and preset parameters (I.e. all the parameters that the device is currently using). Device may also contain additional shadow areas that can be used by software to manipulate global and preset parameters without modifying those in the device’s work area. Global and preset parameters can be moved between volatile (work and shadow areas) and non-volatile storage.

| Area | Name | Description |
|---|---|---|
| Work | Active<br>Global<br>Parameters | RCGT/RDGT/WNGT parameters.<br>WBGT parameters. |
| Work | Active<br>Preset<br>Parameters | RCPT/RDPT/WNPT parameters for active preset including:<br>1. RCPS/RDPS/WNPS parameters for scene #1<br>2. RCPS/RDPS/WNPS parameters for scene #2 |
| Shadow | Global<br>Parameters | RCGT/RDGT/WNGT parameters.<br>WBGT parameters. |
| Shadow | Preset<br>Parameters | RCPT/RDPT/WNPT parameters for any preset including:<br>1. RCPS/RDPS/WNPS parameters for scene #1<br>2. RCPS/RDPS/WNPS parameters for scene #2 |

### Session Management

A host communicates with a device using the MIDI system exclusive commands described in this document. Communication sessions are stateless and a device can support multiple concurrent sessions. The host needs to manage the session to gather information from the device. A host should not send multiple messages to a device and then wait for multiple responses; the host should send one message then wait for a response from the device before sending the next message (i.e. use handshaking). Guidelines for session management are given below.

#### Device Discovery

A host application typically only has access to a list of available MIDI ports on a computer, not to a list of connected MIDI devices. A host should use device discovery to find the best MIDI port for communication and to set communication parameters. A suggested process is described below.

1. Host creates a random 28-bit value for the session ID (used in every message header).

2. Host sends HstSesnVal messages to every MIDI output port. Each message has a unique transaction ID. Host also sets the HstInSizeMax parameter for each message. SNUM portion of Device ID should be 0. PID can be 0 or a specific product ID.

3. Device returns a DevSesnVal message for each HstSesnVal message it receives along with Device ID and connection parameters. A device only returns a DevSesnVal message if its Device ID matches the Device ID in the HstSesnVal message (wildcards are supported).

4. Host receives DevSesnVal messages on MIDI input ports and uses the session ID and transaction ID to find the matching HstSesnVal message to pair up MIDI input ports with MIDI output ports.

5. Host will likely find numerous MIDI input/output ports that are connected to the same device (use Device ID in DevSesnVal message to identify the device). Host uses DevOpMode and DevMIDIPortInfo to determine if the device is in the correct operating mode for the application and to determine which set of MIDI input/output ports to use. Host selects one set of MIDI input/output ports to use for the remainder of the session.

6. Host saves the value for DevInSizeMax (maximum length of messages sent to this device) and verifies that DevOutSizeMax is less than or equal to HstInSizeMax. If DevOutSizeMax is larger than HstInSizeMax, host sends another HstSesnVal message to the device and the device returns another DevSesnVal message. Repeat this step as necessary until DevOutSizeMax is less than or equal to HstInSizeMax.

7. Host and device are now ready for host to gather information from the device.

#### Get Device Info

Once device discovery has completed, the host gathers information from the device beginning with the DeviceInfo data class. A suggested process is described below.

1. Host sends a GetParmDef:DeviceInfo message to the device to get a list of supported DeviceInfo parameter IDs.

2. Device returns a RetParmDef:DeviceInfo message to the host containing a list of supported DeviceInfo parameter IDs.

3. Host sends a GetParmVal:DeviceInfo message to the device to retrieve parameter values that are of interest to the host. Host and device should use handshaking (to send several messages repeat steps 3-5 as needed).

4. Device returns a RetParmVal:DeviceInfo message to the host containing parameter values.

5. Host uses the DeviceInfo parameter values to determine the next set of DeviceInfo parameters to retrieve. Repeat steps 3-5 as needed until all DeviceInfo is retrieved.

The host can then repeat the above procedure using the DeviceFeature data class if there are any device features of interest to the host.

#### Get Hardware Info

Once DeviceInfo has been gathered, the host can gather hardware information. A suggested process is described below.

1. Host sends a GetParmDef:HardwareInfo message to the device to get a list of supported HardwareInfo parameter IDs.

2. Device returns a RetParmDef:HardwareInfo message to the host containing a list of supported HardwareInfo parameter IDs.

3. Host sends a GetParmVal:HardwareInfo message to the device to retrieve parameter values that are of interest to the host. Host and device should use handshaking (to send several messages repeat steps 3-5 as needed).

4. Device returns a RetParmVal:HardwareInfo message to the host containing parameter values.

5. Host uses the HardwareInfo parameter values to determine the next set of HardwareInfo parameters to retrieve. Repeat steps 3-5 as needed until all HardwareInfo is retrieved.

Note that steps 3-5 need to be repeated for each hardware port (i.e. each USB device port, each control port, each Ethernet port, etc.)

#### Get MIDI Info

The same process described for getting hardware info can be used for MIDI info once hardware info has been retrieved. Begin with the MIDIInfo data class to get common MIDI parameters then use the MIDIPortInfo data class to get port specific MIDI parameters and finally the MIDIFeature data class.

#### Change Parameter Values

Once all relevant parameters have been retrieved from the device, the host can then change parameter values using the process described below.

1. Host sends a SetParmVal message to the device containing new values for one or more parameters.

2. Device returns an Ack message to the host indicating if the parameter values were accepted or not.

#### Get Commands

Supported commands can be gathered from a device at any time using the process described below.

1. Host sends GetCmdDef message (no data class) to the device.

2. Device returns a RetCmdDef message (no data class) to the host containing a list of supported command IDs and command arguments.

#### Execute Commands

To execute a command, a host uses the process described below.

1. Host sends a SetCmdVal message (no data class) to the device.

2. Device returns an Ack message to the host indicating if the command was successful or not.

#### Register For Notifications

Parameter values that are not read-only constant (RC) can change at any time. A host can continuously poll a device to check for changes but for some host applications it may be more efficient for the host to ask the device to automatically inform the host whenever parameter values change. A suggested process is described below.

1. Host sends a SetCmdVal:Notification message to the device to register for parameter notifications.

2. Device returns an Ack message to the host indicating if the registration was successful or not.

3. Device sends NotParmVal messages to the host whenever parameter values change that the host did not instigate (i.e. from another session, from user interaction, from some automatic process, etc.) Host does not respond to the NotParmVal messages.

4. Host must send a system exclusive message (any message) to the device before a timeout interval otherwise the device will assume that the host is not listening and will stop sending NotParmVal messages.

5. Host sends a SetCmdVal:Notification message to the device to unregister for parameter notifications when the host no longer wants to receive NotParmVal messages.

6. Device returns an Ack message to the host indicating if the unregistration was successful or not.

See the section on Notification commands for more detailed information.

## Message Format

| Field | Description |
|---|---|
| _Header:_ |  |
| 0xF0 | 1 byte, start of system exclusive |
| 0x00, 0x01, 0x73 | 3 bytes, iConnectivity’s manufacturer ID code |
| 0x7D | 1 byte, system exclusive class code |
| _Body:_ |  |
| Device ID | 7 bytes (2 bytes for product ID, 5 bytes for serial number) |
| Session ID | 4 bytes (28 bit value) |
| Transaction ID | 4 bytes (28 bit value) |
| Message Length | 2 bytes, number of bytes in message content (14 bit value) |
| Message Content | length varies |
| Checksum | 1 byte |
| _Footer:_ |  |
| 0xF7 | 1 byte, end of system exclusive |

### Data Format

All fields are big-endian (most significant byte occurs first, least significant byte occurs last). All fields which accept values greater than 0x7F must have the value split across multiple bytes in order to be compatible with MIDI system exclusive format. The least significant 7 bits of the value are contained in the 7 least significant bits of the last byte. The next 7 bits are contained in the 7 least significant bits of the last-1 byte, etc. Here are some examples:

| Value | First Byte | Second Byte |
|---|---|---|
| 0x0003 | 0x00 | 0x03 |
| 0x007F | 0x00 | 0x7F |
| 0x0080 | 0x01 | 0x00 |
| 0x0081 | 0x01 | 0x01 |
| 0x1234 | 0x24 | 0x34 |
| 0x2CA5 | 0x59 | 0x25 |

See the section _Frequently Used Data Conversions_ for more information on commonly used data encodings used throughout this document.

### Device Identifier

The first 2 bytes of Device ID are product ID (PID) a 14-bit value split across 2 bytes (14x2 format). The remaining 5 bytes are serial number (SNUM) a 32-bit value split across 5 bytes (32x5 format). PID and SNUM are formatted so that each byte is 0x7F or less as explained in the Data Format section. Here are some examples:

| PID | SNUM | Device Identifier |
|---|---|---|
| 0x0001 | 0x00000001 | 0x00 0x01 0x00 0x00 0x00 0x00 0x01 |
| 0x0001 | 0x00000080 | 0x00 0x01 0x00 0x00 0x00 0x01 0x00 |
| 0x00A0 | 0x00000080 | 0x01 0x02 0x00 0x00 0x00 0x01 0x00 |
| 0x0ABC | 0x12345678 | 0x15 0x3C 0x01 0x11 0x51 0x2C 0x78 |

Some messages allow PID and SNUM to be zero which indicates a wildcard as follows:

| PID | SNUM | Meaning |
|---|---|---|
| = 0 | = 0 | All devices. |
| = 0 | > 0 | All devices with the specified serial number. |
| > 0 | = 0 | All devices with the specified product ID. |
| > 0 | > 0 | A device that matches the specified product ID and serial number. |

### Session ID

Session ID can be used to uniquely identify messages for a particular host or application. Typically, the host sends a query to one or more devices, each device then sends a response back to the host. A device will return the same session ID in the response as was sent in the query. Session IDs should be a random number generated by the host. Session ID is a 28-bit value split across 4 bytes (28x4 format). Each byte is 0x7F or less as explained in the Data Format section.

### Transaction ID

Transaction ID can be used to uniquely identify messages. Typically, the host sends a query message to one or more devices, each device then sends a response message back to the host. A device will return the same transaction ID in the response as was sent in the query. It is not required that transaction IDs increment for every message sent by a host, they can always be 0 or some other value, it is up to the host whether or not it wants to use transaction ID for managing messages. Transaction ID is a 28-bit value split across 4 bytes (28x4 format). Each byte is 0x7F or less as explained in the Data Format section.

### Message Length

The number of bytes that follow in the message content section. Message length is a 14-bit value split across 2 bytes (14x2 format). Each byte is 0x7F or less as explained in the Data Format section.

### Message Content

The message content field begins with a two-byte header. The first byte of the header indicates the Message Class. The second byte of the header indicates the Data Class. These two bytes combined identify the message type. The remainder of the message content field depends on the message type.

Minimum length of the message content field is two bytes. The host can send a Ping message to a device by using 0 for the Message Length field and leaving the Message Content field empty. The device will respond with a similar message if it supports this protocol. This should not be used as means to discover devices; the HstSesnVal message class should be used for device discovery.

### Checksum

The checksum field contains the 2s complement of the sum of all bytes in the body, excluding the checksum byte. In other words, summing all the bytes in the body should result in 0x00. Note that the checksum byte must be 0x7F or less to be compatible with MIDI system exclusive format (i.e. if the calculated checksum value is 0x83 then the checksum value used in the system exclusive message should be 0x03).

_Example: minimal length message with no message content_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x02                    // message length (2)
0x02, 0x01                    // message class, data class
0xxx                          // checksum
0xF7                          // footer
```

_Example: ping message with zero message length (all products, all serial numbers)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x00                    // product ID
0x00, 0x00, 0x00, 0x00, 0x00  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x00                    // message length (0)
0x00                          // checksum
0xF7                          // footer
```

_Example: message with content (1 data block)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x00                    // product ID (any)
0x00, 0x00, 0x00, 0x00, 0x00  // serial number (any)
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0A                    // message length (10)
0x01, 0x01                    // message class, data class
0x01                          // NumDataBlock: (1)
0x07, 0x03                    // data block #1: size (7), type (ParmVal)
0x01                          // NumParmValBlock: (1)
0x04                          // ParmVal #1: ParmSize: (4)
0x01                          // ParmVal #1: ParmID: HstInSizeMax (0x01)
0x04, 0x00                    // ParmVal #1: ParmVal: (512)
0xxx                          // checksum
0xF7                          // footer
```

## Data Blocks

Data blocks are used in the message content field and have the following format:

| Byte | Description |
|---|---|
| 1 | DataBlockSize: size of this data block (including this byte). |
| 2 | DataBlockType: data block type. |
| 3 – N | DataBlockContent: Depends on the data block type. |

**Data Block Types:**

| Value | Name | Description |
|---|---|---|
| 0x01 | ParmList | Parameter List: Used to hold a list of parameter IDs to be retrieved from a device for a specific data class. |
| 0x02 | ParmDef | Parameter Definition: Used to define which parameters are supported by a device for a specific data class along with their attributes. |
| 0x03 | ParmVal | Parameter Value: Used to define parameter values when reading from a device or writing to a device for a specific data class. |
| 0x04 | ArgVal | Argument Value: Used to define argument values when reading parameters from a device or writing parameters to a device. |
| 0x05 | CmdDef | Command Definition: Used to define which commands and command values a device supports. |
| 0x06 | CmdVal | Command Value: Used to execute commands in a device. |
| 0x70 | BulkHdr | Bulk Header: Used for backup and restore operations. |

ArgVal blocks (if they are used) should always precede ParmList and ParmVal blocks so that the arguments are defined before any parameters are returned or modified.

### ParmList (Data Block Type = 0x01)

This data block is used to hold a list of parameters to be retrieved from a device for a specific data class.

Data Block Content:

Data block content is a single parameter list block.

**Parameter List Block:**

| Byte | Description |
|---|---|
| 3 | NumParmID: number of parameter IDs that follow. |
| 4 – N | ParmID: single byte parameter IDs. |

_Example: ParmList data block_

```text
0x05  // DataBlockSize: (5)
0x01  // DataBlockType: (ParmList)
0x02  // NumParmID: (2)
0x04  // ParmID #1: (4)
0x41  // ParmID #2: (65)
```

### ParmDef (Data Block Type = 0x02)

This data block is used to define which parameters are supported for a specific data class along with their attributes.

Data Block Content:

Byte 3: NumParmDefBlock: number of parameter definition blocks that follow. Bytes 4-N: parameter definition blocks.

**Parameter Definition Block:**

| Byte | Description |
|---|---|
| 1 | ParmID : Parameter ID, specific to each data class. |
| 2 | ParmFlag: Parameter flags:<br>bits 7 - 4: reserved (always 0).<br>bit 3: set if this parameter supports scenes, clear if this parameter does not support scenes.<br>bit 2: set if this parameter is a preset parameter, clear if this is a global parameter.<br>bit 1: for writeable parameters, this bit is set if changing this parameter requires the device to be rebooted for the change to take effect, for read-only parameters this bit is set if the parameter is a constant value that never changes.<br>bit 0: set if this parameter is writeable, clear if this parameter is read-only.<br>The following notation is used for the flags:<br>RD = read-only dynamic parameter (bit 0 = 0, bit 1 = 0)<br>RC = read-only constant parameter (bit 0 = 0, bit 1 = 1)<br>WN = writeable normal parameter (bit 0 = 1, bit 1 = 0)<br>WB = writable reboot parameter (bit 0 = 1, bit 1 = 1)<br>P = preset parameter (bit 2 = 1)<br>G = global parameter (bit 2 = 0)<br>S = scene parameter (bit 3 = 1)<br>T = not a scene parameter (bit 3 = 0)<br>Examples:<br>RCGT = global read-only constant parameter (no scene support)<br>WBPT = preset writeable parameter that requires a reboot to take effect (no scene support)<br>RDPT = preset read-only dynamic parameter (no scene support)<br>WNPS = preset writeable normal scene parameter |

_Example: ParmDef data block_

```text
0x0B        // DataBlockSize: (11)
0x02        // DataBlockType: (ParmDef)
0x04        // NumParmDefBlock: (4)
0x04, 0x00  // block #1: ID = 4, flags = RDGT
0x07, 0x02  // block #2: ID = 7, flags = RCGT
0x09, 0x0D  // block #3: ID = 9, flags = WNPS
0x05, 0x03  // block #4: ID = 5, flags = WBGT
```

### ParmVal (Data Block Type = 0x03)

This data block is used to define parameter values when reading from a device or writing to a device for a specific data class.

Data Block Content:

Byte 3: NumParmValBlock: number of parameter value blocks that follow. Bytes 4-N: parameter value blocks.

**Parameter Value Block:**

| Byte | Description |
|---|---|
| 1 | ParmSize: Size of this block (including this byte). |
| 2 | ParmID: Parameter ID, specific to each data class. |
| 3 – N | ParmVal: Parameter value, specific to the parameter ID. Some parameters consist of a single value, some parameters use bitmaps and contain several values, some parameters use a fixed structure, and some parameters use a variable length structure comprising a number of sub-ID and value pairs. |

_Example: ParmVal data block_

```text
0x12        // DataBlockSize: (18)
0x03        // DataBlockType: (ParmVal)
0x02        // NumParmValBlock: (3)
0x03        // block #1: ParmSize (3)
0x04        // block #1: ParmID (4)
0x09        // block #1: ParmVal (9)
0x04        // block #2: ParmSize (4)
0x41        // block #2: ParmID (65)
0x01, 0x02  // block #2: ParmVal (0x0102)
0x08        // block #3: ParmSize (8)
0x05        // block #3: ParmID (5)
0x01, 0x03  // block #3: ParmVal: sub-ID (1), value (3)
0x02, 0x08  // block #3: ParmVal: sub-ID (2), value (8)
0x04, 0x09  // block #3: ParmVal: sub-ID (4), value (9)
```

### ArgVal (Data Block Type = 0x04)

This data block is used to define argument values when reading parameters from a device or writing parameters to a device. If argument value blocks are used, they should always precede the ParmList and ParmVal blocks to which they apply. Not all arguments are applicable to every data class; see the details for each data class for a list of valid arguments.

Data Block Content:

Byte 3: NumArgValBlock: number of argument value blocks that follow. Bytes 4-N: argument value blocks.

**Argument Value Block:**

| Byte | Description |
|---|---|
| 1 | ArgID: Argument ID, see tables below. |
| 2 | ArgVal: Argument value, specific to the argument ID. |

**All Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x02 | SceneID | Scene selection (0 – N). Only applies to devices and parameters that support scenes. 0 is used for the currently active scene, 1 – N is used to specify a particular scene. Use 0 for globals and parameters that do not support scenes. |
| 0x03 | HWPortType | Hardware port type (1 – N). |
| 0x04 | HWPortID | Hardware port ID (1 – N). |
| 0x05 | MIDIPortID | MIDI port ID (1 – N). |
| 0x06 | MIDIChannel | MIDI channel number (1 – 16). |
| 0x07 | AMPID | AMP ID (1 – N). This specifies the AMP (advanced MIDI processor) algorithm ID, the operatorID, the custom route map ID, or the lookup table ID (depends on the parameter). |
| 0x08 | USBHMIDIID | USB host controller MIDI ID (1 – N). |
| 0x09 | PresetID | Preset ID (1 – N). Can be used to read some preset parameters directly from non-volatile storage without having to load a preset into the working area or the shadow area. |
| 0x0A | MIDICC<br>ProcessorID | MIDI control change processor ID (1 – N). |
| 0x0B | MIDIMonitor<br>EventID | MIDI monitor event ID (0 – N). Supported values:<br>0 = no MIDI events<br>// channel events<br>1 = Note Off (0x8n)<br>2 = Note On (0x9n)<br>3 = Poly Aftertouch (0xAn)<br>4 = Control Change (0xBn)<br>5 = Program Chage (0xCn)<br>6 = Mono Aftertouch (0xDn)<br>7 = Pitch Bend (0xEn)<br>// system common events 1<br>8 = Clock (0xF8)<br>9 = Start (0xFA)<br>10 = Continue (0xFB)<br>11 = Stop (0xFC)<br>12 = MIDI Timecode Quarter Frame (0xF1)<br>13 = Song Position (0xF2)<br>14 = Song Select (0xF3)<br>// system common events 2<br>15 = Tune Request (0xF6)<br>16 = Active Sense (0xFE)<br>17 = Reset (0xFF)<br>18 = System Exclusive (0xF0) |
| 0x0C | MIDIGobID | MIDI-Gob ID (1 – N). |
| 0x0D | KybdRptGrpID | Keyboard report group ID (1 – N). |
| 0x0E | ControlButtonID | Control button ID (1 – N). |
| 0x0F |  |  |
| 0x10 | AudioPortID | Audio port ID (1 – N). |
| 0x11 | AudioChannelNumberDST | Destination audio channel number (1 – N). |
| 0x12 | AudioChannelNumberSRC | Source audio channel number (1 – N). |
| 0x13 | AudioChannelType | Audio channel type (1 – N). |
| 0x14 | AudioMixerSnapshotID | Audio mixer snapshot ID (1 – N). |
| 0x15 | ACAID | Automation control for audio ID (1 – N). |

The following table shows all the values that can be used for a device that supports one shadow area and two scenes:

| AreaID | SceneID | Description |
|---|---|---|
| 0 | 0 | Work area, currently active scene. |
| 0 | 1 | Work area, scene 1. |
| 0 | 2 | Work area, scene 2. |
| 1 | 0 | Shadow area, currently active scene. |
| 1 | 1 | Shadow area, scene 1. |
| 1 | 2 | Shadow area, scene 2. |

_Example: ArgVal data block_

```text
0x07  // DataBlockSize: (7)
0x04  // DataBlockType: (ArgVal)
0x02  // NumArgValBlock: (2)
0x01  // block #1: ArgID: (1)
0x00  // block #1: ArgVal: (0)
0x02  // block #2: ArgID: (2)
0x01  // block #2: ArgVal: (1)
```

### CmdDef (Data Block Type = 0x05)

This data block is used to define which command IDs and command values a device supports.

Data Block Content:

Byte 3: NumCmdDefBlock: number of command definition blocks that follow. Bytes 4-N: command definition blocks.

**Command Definition Block:**

| Byte | Description |
|---|---|
| 1 | CmdSize: Size of this block (including this byte). |
| 2 | CmdID: Command ID (0x01 – 0x7F). |
| 3 – N | CmdVal: Command values supported for this command ID (0x01 – 0x7F). |

_Example: CmdDef data block_

```text
0x0A  // DataBlockSize: (10)
0x05  // DataBlockType: (CmdDef)
0x02  // NumCmdDefBlock: (2)
0x03  // block #1: CmdSize: (3)
0x04  // block #1: CmdID: (4)
0x09  // block #1: CmdVal: (9)
0x04  // block #2: CmdSize: (4)
0x41  // block #2: CmdID: (65)
0x07  // block #2: CmdVal #1: (7)
0x09  // block #2: CmdVal #2: (9)
```

### CmdVal (Data Block Type = 0x06)

This data block is sent from a host to a device to execute commands.

Data Block Content:

Byte 3: NumCmdValBlock: number of command value blocks that follow. Bytes 4-N: command value blocks.

**Command Value Block:**

| Byte | Description |
|---|---|
| 1 | CmdSize: Size of this block (including this byte). |
| 2 | CmdID: Command ID (0x01 – 0x7F). |
| 3 | CmdVal: Command value (0x01 – 0x7F). |
| 4 – N | CmdArg: Command arguments (specific to the command ID and command value). |

_Example: CmdVal data block_

```text
0x0B  // DataBlockSize: (11)
0x06  // DataBlockType: (CmdVal)
0x02  // NumCmdValBlock: (2)
0x03  // block #1: CmdSize: (3)
0x04  // block #1: CmdID: (4)
0x09  // block #1: CmdVal: (9)
0x05  // block #2: CmdSize: (5)
0x41  // block #2: CmdID: (65)
0x07  // block #2: CmdVal: (7)
0x01  // block #2: CmdArg 1: (1)
0x08  // block #2: CmdArg 2: (8)
```

### BulkHdr (Data Block Type = 0x70)

This data block is used for backup and restore operations. See the Bulk Data Transfer section for more information regarding bulk data transfer operations.

## Message Classes

The following Message Classes are defined:

**Message Classes:**

| Value | Name | Description |
|---|---|---|
| 0x01 | HstSesnVal | Sent from host to device to exchange session information. |
| 0x02 | GetParmDef | Sent from host to device to retrieve a list of parameters. |
| 0x03 | GetParmVal | Sent from host to device to retrieve parameter values. |
| 0x04 | GetCmdDef | Sent from host to device to retrieve a list of commands. |
| 0x10 | SetParmVal | Sent from host to device to set parameter values. |
| 0x11 | SetCmdVal | Sent from host to device to execute a command. |
| 0x40 | Ack | Sent from device to host in response to various messages (indicates success or failure). |
| 0x41 | DevSesnVal | Sent from device to host in response to a HstSesnVal message (exchange session information) |
| 0x42 | RetParmDef | Sent from device to host in response to a GetParmDef message (return a parameter list). |
| 0x43 | RetParmVal | Sent from device to host in response to a GetParmVal message (return parameter values). |
| 0x44 | RetCmdDef | Sent from device to host in response to a GetCmdDef message (return a command list). |
| 0x50 | NotParmVal | Sent from a device to a host when parameters are changed on a device that the host did not initiate. |
| 0x70 | BulkTransfer | Sent between a device and a host (or another device) to perform bulk data transfer operations (i.e. backup and restore). |

### HstSesnVal (Message Class = 0x01)

HstSesnVal messages are sent from a host to a device to exchange session information. The device will return either a DevSesnVal message or an Ack message with an error code. The following format is used for all HstSesnVal messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x01) |
| 2 | Data Class (0x01 = SessionInfo) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One or more ParmVal data blocks. |

_Example: HstSesnVal for data class SessionInfo_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0A                    // message length (10)
0x01, 0x01                    // message class, data class
0x01                          // NumDataBlock: (1)
0x07, 0x03                    // data block #1: size (7), type (ParmVal)
0x01                          // NumParmValBlock: (1)
0x04                          // block #1: ParmSize: (4)
0x01                          // block #1: ParmID: (1)
0x02, 0x00                    // block #1: ParmVal: (256)
0xxx                          // checksum
0xF7                          // footer
```

### GetParmDef (Message Class = 0x02)

GetParmDef messages are sent from a host to a device to retrieve a list of parameter definitions from a device for a specific data class. The device will return either a RetParmDef message or an Ack message with an error code. The following format is used for all GetParmDef messages for all data classes:

| Byte | Description |
|---|---|
| 1 | Message Class (0x02) |
| 2 | Data Class (0x01 – 0x7F) No message content for this message class. |

_Example: GetParmDef for data class nn_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x02                    // message length (2)
0x02, 0xnn                    // message class, data class
0xxx                          // checksum
0xF7                          // footer
```

### GetParmVal (Message Class = 0x03)

GetParmVal messages are sent from a host to a device to retrieve parameter values from a device. The device will return either a RetParmVal message or an Ack message with an error code. The following format is used for all GetParmVal messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x03) |
| 2 | Data Class (0x01 – 0x7F) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One ArgVal data block (optional, depends on data class), followed by one or more ParmList data blocks. |

_Example: GetParmVal for data class nn, using ArgVal data block_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0D                    // message length (13)
0x03, 0xnn                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x01                          // block #1: ArgID: (1)
0x01                          // block #1: ArgVal: (1)
0x05, 0x01                    // data block #2: size (5), type (ParmList)
0x02                          // NumParmID: (2)
0x04                          // ParmID #1: (4)
0x41                          // ParmID #2: (65)
0xxx                          // checksum
0xF7                          // footer
```

### GetCmdDef (Message Class = 0x04)

GetCmdDef messages are sent from a host to a device to retrieve a list of executable commands that the device supports. The device will return either a RetCmdDef message or an Ack message with an error code. The following format is used for all GetCmdDef messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x04) |
| 2 | Data Class (0x00) No message content for this message class. |

_Example: GetCmdDef_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x02                    // message length (2)
0x04, 0x00                    // message class, data class
0xxx                          // checksum
0xF7                          // footer
```

### SetParmVal (Message Class = 0x10)

SetParmVal messages are sent from a host to a device to set parameter values in a device. The device will return an Ack message. The following format is used for all SetParmVal messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x10) |
| 2 | Data Class (0x01 – 0x7F) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One ArgVal data block (optional, depends on data class), followed by one or more ParmVal data blocks. |

_Example: SetParmVal for data class nn, using ArgVal data block_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x12                    // message length (18)
0x10, 0xnn                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x01                          // block #1: ArgID: (1)
0x01                          // block #1: ArgVal: (1)
0x0A, 0x03                    // data block #2: size (10), type (ParmVal)
0x02                          // NumParmValBlock: (2)
0x03                          // block #1: ParmSize: (3)
0x04                          // block #1: ParmID: (4)
0x09                          // block #1: ParmVal: (9)
0x04                          // block #2: ParmSize: (4)
0x41                          // block #2: ParmID: (65)
0x01, 0x02                    // block #2: ParmVal: (0x0102)
0xxx                          // checksum
0xF7                          // footer
```

### SetCmdVal (Message Class = 0x11)

SetCmdVal messages are sent from a host to a device to execute a command. The device will return an Ack message. The following format is used for all SetCmdVal messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x11) |
| 2 | Data Class (0x00) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One or more CmdVal data blocks. |

_Example: SetCmdVal_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0D                    // message length (13)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x0A, 0x06                    // data block #1: size (10), type (CmdVal)
0x02                          // NumCmdValBlock: (2)
0x03                          // block #1: CmdSize: (3)
0x04                          // block #1: CmdID: (4)
0x09                          // block #1: CmdVal: (9)
0x04                          // block #2: CmdSize: (4)
0x41                          // block #2: CmdID: (65)
0x07                          // block #2: CmdVal: (7)
0x01                          // block #2: CmdArg 1: (1)
0xxx                          // checksum
0xF7                          // footer
```

### Ack (Message Class = 0x40)

Ack messages are sent from a device to a host in response to various messages sent from the host (returns success or failure). The following format is used for all Ack messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x40) |
| 2 | Data Class (0x00) |
| 3 | Message class of the host message that caused this Ack message to be sent. |
| 4 | Data class of the host message that caused this Ack message to be sent. |
| 5 | ErrorCode: an error code from the Ack Error Codes table. |

**Ack Error Codes:**

| Value |  | Description |
|---|---|---|
| 0x00 | No error. |  |
| 0x01 | Malformed message, message failed | to parse correctly, bad length, bad checksum. |
| 0x02 | Message class not supported. |  |
| 0x03 | Data class not supported. |  |
| 0x04 | Message in too large, message cann | ot be received because it is too large. |
| 0x05 | Message out too large, message can | not be sent because it is too large. |
| 0x06 | Data block length is invalid. |  |
| 0x07 | Data block type is invalid. |  |
| 0x08 | Argument ID is invalid. |  |
| 0x09 | Argument value is invalid. |  |
| 0x0A | Parameter ID is invalid. |  |
| 0x0B | Parameter value is invalid. |  |
| 0x0C | Invalid characters used for name. |  |
| 0x0D | Command ID is invalid. |  |
| 0x0E | Command value is invalid. |  |
| 0x0F | Command argument is invalid. |  |
| 0x10 | Required ArgVal block was not found | or does not occur before other data blocks. |
| 0x11 | Sub ID is invalid. |  |
| 0x12 | Sub ID value is invalid. |  |
| 0x13 | Command failed. |  |

_Example: Ack for data class 0x02 with no error_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x05                    // message length (5)
0x40, 0x00                    // message class, data class of this message
0x01, 0x02                    // message class, data class of host message
0x00                          // ErrorCode: no error
0xxx                          // checksum
0xF7                          // footer
```

### DevSesnVal (Message Class = 0x41)

DevSesnVal messages are sent from a device to a host in response to an HstSesnVal message (exchange session information). The following format is used for all DevSesnVal messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x41) |
| 2 | Data Class (0x01 = SessionInfo) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One or more ParmVal data blocks. |

_Example: DevSesnVal for data class SessionInfo_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x17                    // message length (23)
0x41, 0x01                    // message class, data class
0x01                          // NumDataBlock: (1)
0x14, 0x03                    // data block #1: size (20), type (ParmVal)
0x04                          // NumParmValBlock: (4)
0x04                          // block #1: ParmSize: (4)
0x10                          // block #1: ParmID: (0x10)
0x02, 0x00                    // block #1: ParmVal: (256)
0x04                          // block #2: ParmSize: (4)
0x11                          // block #2: ParmID: (0x11)
0x02, 0x00                    // block #2: ParmVal: (256)
0x03                          // block #3: ParmSize: (3)
0x12                          // block #3: ParmID: (0x12)
0x01                          // block #3: ParmVal: (1)
0x06                          // block #4: ParmSize: (6)
0x13                          // block #4: ParmID: (0x13)
0x01, 0x02, 0x03, 0x04        // block #4: ParmVal: (0x01020304)
0xxx                          // checksum
0xF7                          // footer
```

### RetParmDef (Message Class = 0x42)

RetParmDef messages are sent from a device to a host in response to a GetParmDef message (returns a list of parameters from a device). The following format is used for all RetParmDef messages for all data classes:

| Byte | Description |
|---|---|
| 1 | Message Class (0x42) |
| 2 | Data Class (0x01 – 0x7F) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One or more ParmDef data blocks. |

_Example: RetParmDef for data class nn_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0E                    // message length (14)
0x42, 0xnn                    // message class, data class
0x01                          // NumDataBlock (1)
0x0B, 0x02                    // data block #1: size (11), type (ParmDef)
0x04                          // NumParmDefBlock: (4)
0x04, 0x00                    // block #1: ID = 4, flags = RDGT
0x07, 0x02                    // block #2: ID = 7, flags = RCGT
0x09, 0x0D                    // block #3: ID = 9, flags = WNPS
0x05, 0x03                    // block #4: ID = 5, flags = WBGT
0xxx                          // checksum
0xF7                          // footer
```

### RetParmVal (Message Class = 0x43)

RetParmVal messages are sent from a device to a host in response to a GetParmVal message (return parameter values from a device). The following format is used for all RetParmVal messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x43) |
| 2 | Data Class (0x01 – 0x7F) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One ArgVal data block (optional, depends on data class), followed by one or more ParmVal data blocks. |

_Example: RetParmVal for data class nn, using ArgVal data block_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x12                    // message length (18)
0x43, 0xnn                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x01                          // block #1: ArgID: (1)
0x01                          // block #1: ArgVal: (1)
0x0A, 0x03                    // data block #2: size (10), type (ParmVal)
0x02                          // NumParmValBlock: (2)
0x03                          // block #1: ParmSize: (3)
0x04                          // block #1: ParmID: (4)
0x09                          // block #1: ParmVal: (9)
0x04                          // block #2: ParmSize: (4)
0x41                          // block #2: ParmID: (65)
0x01, 0x02                    // block #2: ParmVal: (0x0102)
0xxx                          // checksum
0xF7                          // footer
```

### RetCmdDef (Message Class = 0x44)

RetCmdDef messages are sent from a device to a host in response to a GetCmdDef message (returns a list of command IDs and command values that the device supports). The following format is used for all RetCmdDef messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x44) |
| 2 | Data Class (0x00) |
| 3 | NumCmdDefBlock: number of command definition blocks that follow. |
| 4 – N | One or more command definition blocks. |

_Example: RetCmdDef, two command IDs supported (one and two command values respectivey)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0D                    // message length (13)
0x44, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x0A, 0x05                    // data block #1: size (10), type (CmdDef)
0x02                          // NumCmdDefBlock: (2)
0x03                          // block #1: CmdSize: (3)
0x04                          // block #1: CmdID: (4)
0x09                          // block #1: CmdVal: (9)
0x04                          // block #2: CmdSize: (4)
0x41                          // block #2: CmdID: (65)
0x07                          // block #2: CmdVal 1: (7)
0x09                          // block #2: CmdVal 2: (9)
0xxx                          // checksum
0xF7                          // footer
```

### NotParmVal (Message Class = 0x50)

NotParmVal messages are sent from a device to a host when parameters are changed on a device that the host did not initiate, but only if the host has registered for automatic updates using the Notification command. The following format is used for all NotParmVal messages (which is identical to the RetParmVal message format):

| Byte | Description |
|---|---|
| 1 | Message Class (0x50) |
| 2 | Data Class (0x01 – 0x7F) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One ArgVal data block (optional, depends on data class), followed by one or more ParmVal data blocks. |

### BulkTransfer (Message Class = 0x70)

BulkTransfer messages are sent between a device and a host (or another device) to perform bulk data transfer operations (i.e. backup and restore). See the Bulk Data Transfer section for more information regarding bulk data transfer operations. The following format is used for all BulkTransfer messages:

| Byte | Description |
|---|---|
| 1 | Message Class (0x70) |
| 2 | Data Class (0x70) |
| 3 | NumDataBlock: number of data blocks that follow. |
| 4 – N | One bulk header data block possibly followed by additional data blocks. |

## Data Classes

The following Data Classes are defined:

**Data Classes:**

| Value | Name | Description |
|---|---|---|
| 0x00 | null | Placeholder value for messages that don’t require a data class. |
| 0x01 | SessionInfo | Used to discover devices and to establish communications parameters. |
| 0x02 | DeviceInfo | Contains many read-only parameters for a device. |
| 0x03 | DeviceFeature | Contains parameters related to optional device features. |
| 0x04 | HardwareInfo | Contains parameters related to hardware ports. |
| 0x05 | MIDIInfo | Contains global parameters related to the MIDI configuration of a device. |
| 0x06 | MIDIPortInfo | Contains parameters related to specific MIDI ports. |
| 0x07 | MIDIFeature | Contains parameters for optional MIDI features. |
| 0x08 | AudioInfo | Contains parameters related to the audio configuration of a device. |
| 0x09 | AudioPortInfo | Contains parameters related to specific audio ports. |
| 0x0A | AudioChannelInfo | Contains parameters related to specific audio channels. |
| 0x0B | AudioAutomationInfo | Contains parameters related to automation controls for audio. |
| 0x70 | BulkData | Used to perform bulk data transfer operations. |

### SessionInfo (Data Class = 0x01)

This data class is used to discover devices and to establish communication parameters for a session. Sessions are stateless but a device will maintain any session values established on a MIDI port until the device is either reset or a new set of session parameters are established on that MIDI port.

The host sends a HstSesnVal message to discover which MIDI ports have devices attached to them and to determine the best MIDI port to establish a communication session. The host can specify a specific product ID (PID), a specific serial number (SNUM), both PID & SNUM, or neither (using wildcards). This message should always be the first query sent to a device because the DevSesnVal response contains useful information about which MIDI port the host should use to establish a communication session with the device.

Different parameters are used for the HstSesnVal and DevSesnVal messages. The host should send all host related parameters in the HstSesnVal message. The device will return all device related parameters in the DevSesnVal message.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| HstSesnVal |  |
| DevSesnVal |  |

**HstSesnVal Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x01 | HstInSizeMax | Maximum length allowed for sysex messages sent to the host (2 bytes, 14x2 format). If the device needs to send a sysex message to the host that is longer than this value the device will return an Ack message with an error code. The host must then either: allocate a larger buffer then send another HstSesnVal message with the larger buffer size and then resend the message that failed, or split the message into several messages so that the device can respond with several smaller messages. A device cannot automatically split a message that is too large into several smaller messages. |

**DevSesnVal Parameters:**

Note that these parameters are a small subset of the DeviceInfo parameters.

| ID | Name | Description |
|---|---|---|
| 0x10 | DevInSizeMax | Maximum length allowed for sysex messages sent to the device (2 bytes, 14x2 format). If the host needs to send a sysex message to the device that is longer than this value the host will need to split the message into multiple smaller messages. |
| 0x11 | DevOutSizeMax | Maximum length for sysex messages that the device can send back to the host (2 bytes, 14x2 format). The host can use this value to allocate a buffer for incoming sysex messages from the device. This value will always be less than or equal to the HstInSizeMax value that the host sent in the HstSesnVal message. |
| 0x12 | DevOpMode | Device’s current operating mode (1 byte):<br>0 = bootloader mode<br>1 = application mode<br>Note that older products (before TNG) use 2 for bootloader mode when using the RetDevice command. |
| 0x13 | DevMIDIPortInfo | Info regarding the MIDI port that this message is sent on (4 bytes):<br>Byte 1: MIDI port ID (1 – N)<br>Byte 2: MIDI port type:<br>0x01 = DIN<br>0x02 = USB device<br>0x03 = USB host<br>0x04 = Ethernet<br>Bytes 3-4: MIDI port detail, depends on MIDI port type:<br>DIN:<br>Byte 3: DIN IN port (1 – N)<br>Byte 4: DIN OUT port (1 – N)<br>USB device:<br>Byte 3: USB device port (1 - N)<br>Byte 4: MIDI port (1 - 16)<br>USB host:<br>Byte 3: USB host controller (1 - N)<br>Byte 4: USB host controller port (1 - N)<br>Ethernet:<br>Byte 3: Ethernet port (1 - N)<br>Byte 4: RTP-MIDI session (1 - N) |

_Example: HstSesnVal (discover devices, any product, any serial number)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x00                    // product ID
0x00, 0x00, 0x00, 0x00, 0x00  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0A                    // message length (10)
0x01, 0x01                    // message class, data class,
0x01                          // NumDataBlock: (1)
0x07, 0x03                    // data block #1: size (7), type (ParmVal)
0x01                          // NumParmValBlock: (1)
0x04                          // block #1: ParmSize: (4)
0x01                          // block #1: ParmID: HstInSizeMax (0x01)
0x04, 0x00                    // block #1: ParmVal: (512)
0xxx                          // checksum
0xF7                          // footer
```

_Example: HstSesnVal (discover all devices that are a specific product ID (5), any serial number)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID (5)
0x00, 0x00, 0x00, 0x00, 0x00  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0A                    // message length (10)
0x01, 0x01                    // message class, data class
0x01                          // NumDataBlock: (1)
0x07, 0x03                    // data block #1: size (7), type (ParmVal)
0x01                          // NumParmValBlock: (1)
0x04                          // block #1: ParmSize: (4)
0x01                          // block #1: ParmID: HstInSizeMax (0x01)
0x02, 0x00                    // block #1: ParmVal: (256)
0xxx                          // checksum
0xF7                          // footer
```

_Example: DevSesnVal (response to HstSesnVal example above)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID (5)
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x17                    // message length (23)
0x41, 0x01                    // message class, data class
0x01                          // NumDataBlock: (1)
0x14, 0x03                    // data block #1: size (20), type (ParmVal)
0x04                          // NumParmValBlock: (4)
0x04                          // block #1: ParmSize: (4)
0x10                          // block #1: ParmID: DevInSizeMax (0x10)
0x02, 0x00                    // block #1: ParmVal: (256)
0x04                          // block #2: ParmSize: (4)
0x11                          // block #2: ParmID: DevOutSizeMax (0x11)
0x02, 0x00                    // block #2: ParmVal: (256)
0x03                          // block #3: ParmSize: (3)
0x12                          // block #3: ParmID: DevOpMode (0x12)
0x01                          // block #3: ParmVal: application mode (1)
0x06                          // block #4: ParmSize: (6)
0x13                          // block #4: ParmID: DevMIDIPortInfo (0x13)
0x05                          // block #4: ParmVal 1: MIDI port ID (5)
0x02                          // block #4: ParmVal 2: MIDI port type: USB device (2)
0x01                          // block #4: ParmVal 3: USB device port number (1)
0x01                          // block #4: ParmVal 4: MIDI port number (1)
0xxx                          // checksum
0xF7                          // footer
```

### DeviceInfo (Data Class = 0x02)

This data class contains many read-only global parameters for a device and is also used to set the device name. This data class should always be one of the first messages sent to a device.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef |  |
| RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is optional. All parameters are global and do not support scenes. |
| RetParmVal |  |
| NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | Area | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x01 | ProductName | Product name (7-bit ASCII string). |
| 0x02 | MfgName | Manufacturer (7-bit ASCII string). |
| 0x03 | ModelNumber | Model number (7-bit ASCII string). |
| 0x04 | SerialNumber | Serial number (7-bit ASCII string). |
| 0x05 | FirmwareVersion | Firmware version (4 bytes) using the format: Major.Minor.Revision.Beta (where 0 is used in the Beta field for final builds). Examples:<br>0x01 0x02 0x03 0x00 = 1.2.3<br>0x02 0x00 0x0B 0x04 = 2.0.11b4 |
| 0x06 | HardwareVersion | Hardware version (2 bytes) using the format: Major.Minor. Examples:<br>0x01 0x02 = 1.2<br>0x02 0x22 = 2.34 |
| 0x07 | DevNameMax | Maximum length allowed for device name (DevIinfo:DevName) (1 byte). |
| 0x08 | DevUserDataMax | Max length allowed for device user data (DeviceInfo:DevUserData) (1 byte). |
| 0x09 | DINInPortCount | Number of DIN IN ports (0 – N) (1 byte). |
| 0x0A | DINOutPortCount | Number of DIN OUT ports (0 – N) (1 byte). |
| 0x0B | USBDPortCount | Number of USB device ports (0 – N) (1 byte). |
| 0x0C | USBHPortCount | Number of USB host controller ports (0 – N) (1 byte). |
| 0x0D | EthPortCount | Number of Ethernet ports (0 – N) (1 byte). |
| 0x0E | CtrlPortCount | Number of control ports (0 – N) (1 byte). |
| 0x0F | HWPortNameMax | Maximum length allowed for hardware port name (HardwareInfo:HWPortName) (1 byte). |
| 0x10 | DevInSizeMax | Maximum length allowed for sysex messages sent to the device (2 bytes, 14x2 format). If the host needs to send a sysex message to the device that is longer than this value the host will need to split the message into multiple smaller messages. |
| 0x11 | DevOutSizeMax | Maximum length for sysex messages that the device can send back to the host (2 bytes, 14x2 format). The host can use this value to allocate a buffer for incoming sysex messages from the device. This value will always be less than or equal to the HstInSizeMax value that the host sent in the most recent HstSesnVal message. |
| 0x12 | DevOpMode | Device’s current operating mode (1 byte):<br>0 = bootloader mode<br>1 = application mode |
| 0x13 | DevMIDIPortInfo | Info regarding the MIDI port that this message is sent on (4 bytes):<br>Byte 1: MIDI port ID (1 – N)<br>Byte 2: MIDI port type:<br>0x01 = DIN<br>0x02 = USB device<br>0x03 = USB host<br>0x04 = Ethernet<br>Bytes 3-4: MIDI port detail, depends on MIDI port type:<br>DIN:<br>Byte 3: DIN IN port (1 – N)<br>Byte 4: DIN OUT port (1 – N)<br>USB device:<br>Byte 3: USB device port (1 - N)<br>Byte 4: MIDI port (1 - 16)<br>USB host:<br>Byte 3: USB host controller (1 - N)<br>Byte 4: USB host controller port (1 - N)<br>Ethernet:<br>Byte 3: Ethernet port (1 - N)<br>Byte 4: RTP-MIDI session (1 - N) |
| 0x14 | PresetMax | Maximum number of presets supported (1 – N)  (1 byte). |
| 0x15 | PresetNameMax | Maximum length allowed for preset names (DeviceFeature:PresetName) (1 byte). |
| 0x16 | PresetUserDataMax | Maximum length allowed for preset user data (DeviceFeature:PresetUserData) (1 byte). |
| 0x17 | SceneMax | Maximum number of scenes supported per preset (1 – N) (1 byte). A value of 1 indicates a device that does not support scene changes. |
| 0x18 | ShadowAreaMax | Maximum number of shadow areas supported (0 – N) (1 byte). |
| 0x19 | NotificationTimeout | Notification timeout, in seconds (1 byte). If a host registers for automatic updates using the Notification command, the host must ensure that it sends a sysex message to the device at regular intervals otherwise the device will assume the host is not there and will stop sending notification messages. Example:<br>Timeout = 5  // host should send a sysex message every 5 seconds (or sooner) to continue receiving automatic updates |
| 0x1A | NotificationClassSupport | Notification classes supported by the device (one byte for each class that the device supports). See Notification command for a list of classes. |
| 0x1B | DisplayLevelMax | Maximum level (pixel value) for display (1 – N) (1 byte). |
| 0x1C | DisplayBrightnessMax | Maximum brightness level for display (0 – N) (1 byte). 0 if display brightness cannot be set using DeviceInfo:DisplayBrightness. |
| 0x1D | DisplayContrastMax | Maximum contrast level for display (0 – N) (1 byte). 0 if display contrast cannot be set using DeviceInfo:DisplayContrast. |
| 0x1E | DisplaySupportFlags | Display support flags (1 byte):<br>bits 7 - 6: reserved (always 0).<br>bit 5: set if audio activity can wake display<br>bit 4: set if MIDI activity can wake display<br>bit 3: reserved (always 0)<br>bit 2: set if display has a contrast control<br>bit 1: set if display has a brightness control<br>bit 0: set if display can be inverted (background = white, foreground = black). |
| 0x1F | SystemEventSupportFlags | System event support flags (2 bytes):<br>Byte 1:<br>bits 7 - 6: reserved (always 0).<br>bit 5: set if audio global mute events are supported (see DeviceFeature:SysEvntAudioGlobalMute parameters).<br>bit 4: set if control button events are supported (see DeviceFeature:SysEvntControlButton parameters).<br>bit 3: set if scene change events are supported (see DeviceFeature:SysEvntScene parameters).<br>bit 2: set if preset load events are supported (see DeviceFeature:SysEvntPreset parameters).<br>bit 1: set if failover events are supported (see DeviceFeature:SysEvntFailover parameters).<br>bit 0: set if hardware control input events are supported (see DeviceFeature:SysEvntHardwareCtrlInput parameters).<br>Byte 2:<br>bits 7 - 0: reserved (always 0). |
| 0x20 | KybdRptGrpMax | Maximum number of keyboard report groups supported by device (1 byte). |
| 0x21 | KybdRptGrpNameMax | Maximum length allowed for keyboard report group name (1 byte). |
| 0x22 | ControlButtonMax | Number of control buttons (0 – N) (1 byte). |
| 0x23 | ControlButtonColourSupportFlags | Control button colour support flags (1 byte):<br>bit 7: reserved (always 0).<br>bit 6: set if control buttons support white (red+green+blue).<br>bit 5: set if control buttons support cyan (green+blue).<br>bit 4: set if control buttons support magenta (red+blue).<br>bit 3: set if control buttons support blue.<br>bit 2: set if control buttons support yellow (red+green).<br>bit 1: set if control buttons support green.<br>bit 0: set if control buttons support red. |
| 0x24 | AnalogAudioPortCount | Number of analog audio ports (0 – N) (1 byte). |
| 0x25 | EthSwitchPortCount | Number of ethernet switch ports (0 – N) (1 byte). |
| 0x26 | AOIPPortCount | Number of audio over IP ports (0 – N) (1 byte). |
| 0x40 | DevName | Device name (7-bit ASCII string). This name will be used for the Bonjour name of the device on a network. Note that the actual name used on the network may be different if there is a name conflict following Bonjour name resolution (see HardwareInfo:EthDevName). Maximum length is given by DeviceInfo:DevNameMax. |
| 0x41 | DevUserData | Device user data. Free field for storing user data (all bytes must be in the range 0x00 – 0x7F).<br>Bytes 1: index (i.e. starting address)<br>Bytes 2 – N: user data<br>The first byte is an index into the user data field (i.e. the starting address).<br>The remaining bytes are the user data. RetParmVal will always return the entire user data field (index is always 0, user data is always DevUserDataMax bytes in length). SetParmVal can use a non-zero value for the offset byte and can write just a portion of the user data field.<br>Maximum length is given by DeviceInfo:DevUserDataMax. |
| 0x42 | DisplayBrightness | Brightness level for display (1 – DeviceInfo:DisplayBrightnessMax) (1 byte). Only used if bit 1 is set in DeviceInfo:DisplaySupportFlags. |
| 0x43 | DisplayContrast | Contrast level for display (1 – DeviceInfo:DisplayContrastMax) (1 byte).<br>Only used if bit 2 is set in DeviceInfo:DisplaySupportFlags. |
| 0x44 | DisplayEnableFlags | Display enable flags (1 byte). These bits can only be set if the matching bit is set in DeviceInfo:DisplaySupportFlags:<br>bits 7 – 6: reserved (always 0)<br>bit 5: set if audio activity will wake display<br>bit 4: set if midi activity will wake display<br>bits 3 – 1: reserved (always 0)<br>bit 0: set if the display is inverted, clear if display is not inverted |
| 0x45 | DisplayTimeout | Timeout for display (0 – 127) (1 byte) in minutes. Use 1-127 to turn off display after 1-127 minutes of inactivity. Use 0 to never turn off display. |

_Example: GetParmVal_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0D                    // message length (13)
0x03, 0x02                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x01                          // block #1: ArgID: (1)
0x01                          // block #1: ArgVal: (1)
0x05, 0x01                    // data block #2: size (5), type (ParmList)
0x02                          // NumParmID: (2)
0x07                          // ParmID #1: (7)
0x40                          // ParmID #2: (64)
0xxx                          // checksum
0xF7                          // footer
```

_Example: RetParmVal_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x14                    // message length (20)
0x43, 0x02                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x01                          // block #1: ArgID: (1)
0x01                          // block #1: ArgVal: (1)
0x0C, 0x03                    // data block #2: size (12), type (ParmVal)
0x02                          // NumParmValBlock: (2)
0x03                          // block #1: size (3)
0x07                          // block #1: parm ID (7)
0x0F                          // block #1: parm value (15)
0x06                          // block #2: size (6)
0x40                          // block #2: parm ID (64)
0x41, 0x42, 0x43, 0x44        // block #2: parm value (ASCII string“ABCD”)
0xxx                          // checksum
0xF7                          // footer
```

_Example: SetParmVal_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x14                    // message length (20)
0x10, 0x02                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x01                          // block #1: ArgID: (1)
0x01                          // block #1: ArgVal: (1)
0x0C, 0x03                    // data block #2: size (12), type (ParmVal)
0x01                          // NumParmValBlock: (2)
0x06                          // block #1: size (6)
0x40                          // block #1: parm ID (64)
0x61, 0x62, 0x63, 0x64        // block #1: parm value (ASCII string“abcd”)
0x03                          // block #2: size (3)
0x07                          // block #2: parm ID (7)
0x0C                          // block #2: parm value (12)
0xxx                          // checksum
0xF7                          // footer
```

### DeviceFeature (Data Class = 0x03)

This data class contains parameters related to device features.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef |  |
| RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is optional. All parameters are global and do not support scenes. |
| RetParmVal |  |
| NotParmVal |  |

#### Presets & Scenes

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | Area | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x09 | PresetID | Preset ID (1 – N). Maximum value is given by DeviceInfo:PresetMax.<br>Required when using the PresetName parameter to read preset names from non-volatile storage. |

**Parameters for Presets & Scenes:**

| ID | Name | Description |
|---|---|---|
| 0x01 | PresetNumber | Currently loaded preset number (1 – N) (1 byte). This is a read-only parameter that cannot be used to change presets; use the SaveLoad commands to change presets. Maximum value is given by DeviceInfo:PresetMax. |
| 0x02 | PresetName | Preset name (7-bit ASCII string). Maximum length is given by DeviceInfo:PresetNameMax. Can be used to read/write the currently loaded preset name in the working area or the shadow area. Can also be used to read (but not write) preset names from non-volatile storage.<br>When reading/writing the preset name in the working area ArgID:Area can be used but is not required. When reading/writing the preset name in the shadow area ArgID:Area is required. In both cases, ArgID:PresetID must not be used.<br>When reading the preset name from non-volatile storage ArgID:Area can be used but will be ignored. However, ArgID:PresetID is required.<br>When reading the preset name, setting ArgID:PresetID to 0, or to the value of the currently loaded preset, will return the preset name from RAM (which may not be the same as is stored in non-volatile storage). Otherwise, the preset name from non-volatile storage is returned. |
| 0x03 | PresetUserData | Currently loaded preset user data. Free field for storing user data (all bytes must be in the range 0x00 – 0x7F).<br>Bytes 1: index (i.e. starting address)<br>Bytes 2 – N: user data<br>The first byte is an index into the user data field (i.e. the starting address).<br>The remaining bytes are the user data. RetParmVal will always return the entire user data field (index is always 0, user data is always PresetUserDataMax bytes in length). SetParmVal can use a non-zero value for the offset byte and can write just a portion of the user data field. Maximum length is given by DeviceInfo:PresetUserDataMax. |
| 0x04 | SceneNumber | Currently active scene number (1 – N) (1 byte). This parameter can be used to change the active scene in the currently loaded preset. Maximum value is given by DeviceInfo:SceneMax. |

_Example: Get PresetName from shadow area_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0C                    // message length (12)
0x03, 0x03                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x01                          // block #1: ArgID: (1)
0x01                          // block #1: ArgVal: (1)
0x04, 0x01                    // data block #2: size (4), type (ParmList)
0x01                          // NumParmID: (1)
0x02                          // ParmID #1: (2)
0xxx                          // checksum
0xF7                          // footer
```

_Example: Get PresetName for preset #4 from non-volatile storage_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0C                    // message length (12)
0x03, 0x03                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x09                          // block #1: ArgID: (9)
0x04                          // block #1: ArgVal: (4)
0x04, 0x01                    // data block #2: size (4), type (ParmList)
0x01                          // NumParmID: (1)
0x02                          // ParmID #1: (2)
0xxx                          // checksum
0xF7                          // footer
```

#### Automatic Failover

Automatic failover is used to switch audio and MIDI data from a primary (master) computer to a secondary (backup) computer automatically should the primary computer fail for some reason. Typically, this would be configured such that the master computer provides audio and MIDI data on USB device port #1 with routing defined by scene A while the backup computer provides an identical synchronized copy of the same audio and MIDI data on USB device port #2 with routing defined by scene B. Switching between master and backup is therefore accomplished simply by switching between scene A and scene B. The failover system has a number of operating modes and parameters to detect when the master computer has failed and automatically switch scenes.

See Automation Control MIDI Port for information on using automation controls to arm/disarm the failover system and clear the failover alarm.

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | Area | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |

**Parameters for Automatic Failover:**

| ID | Name | Description |
|---|---|---|
| 0x10 | FailoverArmStatus | Failover system arm status (1 byte). Set to 1 to enable the failover system.<br>Set to 0 to disable the failover system. Set to 2 to toggle the failover system arm state (arm to disarm, or disarm to arm). Arming the failover system will clear the alarm status (see FailoverAlarmStatus).<br>0 = failover system is not armed (read/write)<br>1 = failover system is armed (read/write)<br>2 = toggle failover arm/disarm state (write only) |
| 0x11 | FailoverAlarmStatus | Failover system alarm status (1 byte). Alarm is set if the failover system has detected a failure in the master computer and has switched to the backup computer. Write 0 to clear the alarm status. Alarm cannot be set by writing 1.<br>0 = alarm is off (no failure on master detected, read/write)<br>1 = alarm is on (failure on master detected, read only) |
| 0x12 | FailoverUSBPortStatus | State of USB device ports (4 bytes):<br>Byte 1: USB device port #1 audio status.<br>Byte 2: USB device port #1 MIDI status.<br>Byte 3: USB device port #2 audio status.<br>Byte 4: USB device port #2 MIDI status.<br>Read-only. Each byte will be one of the following values:<br>0 = Host is not connected or is not sending any USB data.<br>1 = Host has enumerated the device and is sending USB start of frame.<br>2 = Host is sending data to USB audio endpoint or MIDI OUT endpoint.<br>3 = Host is sending non-zero data to the specified audio channel or sending any valid MIDI data to the specified MIDI port. |
| 0x13 | FailoverOpMode | Failover system operating mode (1 byte):<br>1 = Single master mode. The master uses USB port #1 for audio/MIDI data and failover trigger information. The backup uses USB port #2 for audio/MIDI data only. When failure occurs on USB port #1 the backup on USB port #2 becomes active. Switching back to the master computer (manually or automatically) occurs once the master computer returns to normal operating state.<br>2 = Dual master mode. The master and backup alternate between USB #1 and USB port #2. Both USB ports are used for audio/MIDI data and failover trigger information. When failure occurs on USB port #1, the backup on USB port #2 becomes the new master and USB port #1 becomes the new backup. |
| 0x14 | FailoverConfigFlags | Failover system configuration flags (1 byte):<br>bits 7 - 6: reserved (always 0).<br>bit 5: set if failover system should flag USB port status (for any USB port) not meeting the trigger level as an error.<br>bit 4: set if failover system should monitor audio tone channel to detect signal overflow (i.e. white noise that can occur when using bad USB cables).<br>bit 3: set if failover system should send MIDI panic when changing scenes.<br>bit 2: set if failover system should clear the alarm status when arming the system.<br>bit 1: set if the failover system should select scene A when automatically arming itself (only used if bit 0 is also set and FailoverOpMode = 1).<br>bit 0: set if the failover system should automatically arm itself when the master computer is stable. |
| 0x15 | FailoverTriggerMode | Trigger mode for switching from master to backup (1 byte):<br>1: audio only, master is stable if audio meets the failover trigger.<br>2: MIDI only, master is stable if MIDI meets the failover trigger.<br>3: audio or MIDI, master is stable if either audio or MIDI meets the failover trigger.<br>4: audio and MIDI, master is stable only if both audio and MIDI meet the failover trigger. |
| 0x16 | FailoverAudioTrigger | Failover trigger parameter block for audio (see below). |
| 0x17 | FailoverMIDITrigger | Failover trigger parameter block for MIDI (see below). |
| 0x18 | FailoverToneVolumeMax | Maximum level allowed for audio tone channel. Values greater than this will trigger the failover system to automatically mute all line outputs if bit 4 is set in FailoverConfigFlags. Parameter value is in dB using 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). Maximum value is 0 dB (0x0000), minimum value is -64 dB (0xC000) with 0.5 dB (0x0080) resolution. |
| 0x19 | FailoverToneFlags | Failover audio tone channel flags (1 byte):<br>bits 7 - 2: reserved (always 0).<br>bit 1: set if level of audio tone channel on USB port #2 is currently greater than the value set with FailoverToneVolumeMax.<br>bit 0: set if level of audio tone channel on USB port #1 is currently greater than the value set with FailoverToneVolumeMax. |

**Failover Trigger Parameter Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs. SetParmVal messages can include any selection of sub-IDs.

| Sub-ID | Description |
|---|---|
| 0x01 | Trigger level for switching from master to backup (1 byte). If the master fails to meet the data requirements then the automatic failover system will switch to the backup.<br>1: Host has enumerated the device and is sending USB start of frame (SOF).<br>2: Host is sending any data (could be digital silence) to USB audio OUT endpoint or any data to USB MIDI OUT endpoint.<br>3: Host is sending non-zero data to a specific audio channel or any valid MIDI data to a specific MIDI port. |
| 0x02 | Audio trigger channel or MIDI trigger port (1 – N) (1 byte).<br>Must be a valid audio channel number or MIDI port number. Only used if trigger level = 3. |
| 0x03 | Audio or MIDI timeout value (1 – 16363) (2 bytes, 14x2 format). Number of USB frames that must fail to meet the audio or MIDI trigger level before switching from master to backup. One frame is 125 microseconds. |

#### System Events

System events are used to trigger one or more output actions based on a specific input condition. Typically, this is used to connect hardware control ports (inputs and outputs) to the failover system and MIDI-Gobs (custom MIDI messages sent to specific MIDI ports). For example, a hardware control input may be connected to a foot switch, pressing the footswitch can arm the failover system, clear the failover alarm, set a hardware control output to the active state, and send a MIDI-Gob. Many of the parameters are scene specific and can be configured differently for each scene.

The DeviceInfo:SystemEventSupportFlags parameter indicates which system events are supported.

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | Area | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x02 | SceneID | Scene ID (1 – DeviceInfo:SceneMax). Required for most parameters. |
| 0x04 | HWPortID | Hardware port ID (1 – N). Required for some parameters. |
| 0x0E | ControlButtonID | Control button ID (1 – N). Required for some parameters. |

**Parameters for System Events:**

| ID | Name | Description |
|---|---|---|
| 0x20 | SysEvntHardwareCtrlInputActive | System event parameter block for hardware control input going to active state (see below). This parameter requires the HWPortID ArgID to specify the hardware control port and the SceneID ArgID to specify the scene. |
| 0x21 | SysEvntHardwareCtrlInputInactive | System event parameter block for hardware control input going to inactive state (see below). This parameter requires the HWPortID ArgID to specify the hardware control port and the SceneID ArgID to specify the scene. |
| 0x22 | SysEvntFailoverArm | System event parameter block for failover system going to_arm_state (see below). This parameter requires the SceneID ArgID to specify the scene. |
| 0x23 | SysEvntFailoverDisarm | System event parameter block for failover system going to_disarm_state (see below). This parameter requires the SceneID ArgID to specify the scene. |
| 0x24 | SysEvntFailoverAlarmSet | System event parameter block for failover alarm going to_set_state (see below). This parameter requires the SceneID ArgID to specify the scene. |
| 0x25 | SysEvntFailoverAlarmClear | System event parameter block for failover alarm going to_clear_state (see below). This parameter requires the SceneID ArgID to specify the scene. |
| 0x26 | SysEvntPresetLoad | System event parameter block for preset being loaded (see below). |
| 0x27 | SysEvntSceneActive | System event parameter block for scene being made active (see below).<br>This parameter requires the SceneID ArgID to specify the scene. |
| 0x28 | SysEvntControlButtonInactive | System event parameter block for control button going to inactive state (see below). This parameter requires the ControlButtonID ArgID to specify the control button and the SceneID ArgID to specify the scene. |
| 0x29 | SysEvntControlButtonSglClick | System event parameter block for control button going to single-click state (see below). This parameter requires the ControlButtonID ArgID to specify the control button and the SceneID ArgID to specify the scene. |
| 0x2A | SysEvntControlButtonDblClick | System event parameter block for control button going to double-click state (see below). This parameter requires the ControlButtonID ArgID to specify the control button and the SceneID ArgID to specify the scene. |
| 0x2B | SysEvntAudioGlobalMuteActive | System event parameter block for audio global mute going to active/on state (see below). This parameter requires the SceneID ArgID to specify the scene. |
| 0x2C | SysEvntAudioGlobalMuteInactive | System event parameter block for audio global mute going to inactive/off state (see below). This parameter requires the SceneID ArgID to specify the scene. |

**System Event Parameter Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs that are valid to the parameter (not all sub-IDs are valid for all parameters). Send a GetParmVal message to get a list of valid sub-IDs for each parameter. SetParmVal messages can include any selection of valid sub-IDs.

| Sub-ID | Description |
|---|---|
| 0x01<br>0x02<br>0x03<br>0x04 | Hardware control port output state (1 byte):<br>0 = port is not affected by input<br>1 = set port active<br>2 = set port inactive<br>3 = toggle port active/inactive state<br>The hardware control port must be configured as an output in order for it to be set to active or inactive state. Sub-ID 0x01 is used for control port #1, sub-ID 0x02 is used for control port #2, etc. |
| 0x05 | Failover arm state (1 byte):<br>0 = failover arm state is not affected by input<br>1 = failover system is armed<br>2 = failover system is disarmed<br>3 = toggle failover arm/disarm state |
| 0x06 | Failover alarm state (1 byte):<br>0 = failover alarm state is not affected by input<br>1 = failover alarm is cleared |
| 0x07 | Preset load (4 bytes). Loads a preset. Specify a specific preset to load or specify a range of presets to load (increment/decrement with or without wraparound). This 4-byte structure is also used for sub-IDs 0x08 (scene change) and 0x09 (mixer snapshot change).<br>Byte 1 (Mode):<br>0 = preset is not affected by input<br>1 = load specified preset<br>2 = increment preset<br>3 = decrement preset<br>For Mode = 0:<br>Bytes 2 – 4 are not used.<br>For Mode = 1:<br>Byte 2: preset to load<br>1 – DeviceInfo:PresetMax<br>Bytes 3 – 4 are not used.<br>For Mode = 2 or 3<br>Byte 2: minimum preset to load (should be lower than maximum preset)<br>1 – DeviceInfo:PresetMax<br>Byte 3: maximum preset to load (should be higher than minimum preset)<br>1 – DeviceInfo:PresetMax<br>Byte 4: flags<br>bits 7 - 1: reserved (always 0).<br>bit 0: set if preset selection can wraparound between minimum and maximum. |
| 0x08 | Scene change (4 bytes). Change the active scene in the currently loaded preset. See sub-ID<br>0x07 (preset load) for details on the 4-byte structure. Uses DeviceInfo:SceneMax instead of<br>DeviceInfo:PresetMax. |
| 0x09 | Mixer snapshot change (4 bytes). Change the active mixer snapshot in the currently loaded preset. See sub-ID 0x07 (preset load) for details on the 4-byte structure. Uses AudioInfo:AudioMixerSnapshotMax instead of DeviceInfo:PresetMax. |
| 0x0A | MIDI Panic send (1 byte):<br>0 = Panic is not sent by input<br>1 = Panic message is sent |
| 0x0B | MIDI-Gob send (1 byte):<br>0 = MIDI-Gob is not sent by input<br>1 – MIDIInfo:MIDIGobMax = MIDI-Gob to send |
| 0x0C<br>0x0D<br>0x0E<br>0x0F | Automation control for audio (ACA) state (2 bytes). These sub-IDs allow up to four ACAs to be affected by a single input. Only switch type controls are supported (e.g. mute, solo enable, dim enable).<br>Byte 1 (Mode):<br>0 = ACA is not affected by input<br>1 = ACA is set active<br>2 = ACA is set inactive<br>3 = ACA is toggled between active/inactive state<br>Byte 2: automation control for audio ID (ACAID)<br>1 – AudioInfo:AudioAutomationControlMax |
| 0x10 | Keyboard report group send (1 byte).<br>0 = Keyboard report group is not sent by input<br>1 – DeviceInfo:KybdRptGrpMax = keyboard report group to send |
| 0x11 | Audio global mute state (1 byte):<br>0 = mute state is not affected by input<br>1 = set mute active/on<br>2 = set mute inactive/off<br>3 = toggle mute active/inactive state |

#### Keyboard Reports

Keyboard reports are used to send virtual QWERTY keypresses to computer hosts connected to the USB device ports. Each keyboard report closely follows the format of standard USB HID keyboard reports and contains:

1. A modifier keys value (e.g. shift, control, alt, and GUI keys).

2. Six keycode values (e.g. letter keys, number keys, delete, enter, escape, page up, arrow keys).

3. Port route indicating which USB device ports to send the keyboard reports to.

4. A flag indicating if the keyboard report is enabled for sending.

5. A flag indicating if the keyboard report should be followed by a NULL keyboard report (all fields containing zeros). A NULL keyboard report must be sent at the end of one or more keyboard reports to avoid the auto-repeat feature that you get if you press and hold a key on a QWERTY keyboard (a NULL keyboard report is similar to sending a MIDI All Notes Off message to stop stuck MIDI notes).

Four keyboard reports are contained within a keyboard report group. Keyboard report groups can be linked to create longer keypress sequences. Keyboard report groups can be sent using sysex messages, automation controls, and in response to various system events (e.g. hardware control inputs, failover alarm set, scene change and preset load). Keyboard report groups have a name field to make them easier to manage when assigning them to system events and automation controls.

The HardwareInfo:USBDFlags (bit 3) and HardwareInfo:USBDEnableHIDKybd parameters are used to indicate if a USB HID keyboard is supported and enabled on each USB device port.

Keyboard reports have a number of rules (dictated by USB consortium, not by iConnectivity):

1. A keycode cannot appear more than once in a report.

2. The keycodes in a report indicate which keys are currently pressed.

3. A new report is generated whenever a change is made to the currently pressed keys (e.g. pressing additional keys or releasing pressed keys).

4. The order of keycodes in the report does not necessarily reflect the order in which the keys were pressed.

5. The same keycode appearing in consecutive reports indicates that the key is still pressed and has not been released.

6. The only way to indicate releasing a key is to send a report that does not contain the keycode of the released key.

See the following documents from www.usb.org for more info:

1. Device Class Definition for Human Interface Devices

2. HID Usage Tables For Universal Serial Bus (esp. Keyboard/Keypad Page [0x07])

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x0D | KybdRptGrpID | Keyboard report group ID (1 – DeviceInfo:KybdRptGrpMax). Required for all parameters. |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x30 | KRGName | Name to use for keyboard report group (7-bit ASCII string). Maximum length is given by DeviceInfo:KybdRptGrpNameMax. |
| 0x31 | KRGData | Keyboard report group data. See Keyboard Report Format for more info. |
| 0x32 | KRGNext | Pointer to next keyboard report group in linked list (1 – DeviceInfo:<br>KybdRptGrpMax) (1 byte). Use 0 if not using links. |

##### Keyboard Report Format

A keyboard report group contains four keyboard reports:

1. Keyboard report #1

2. Keyboard report #2

3. Keyboard report #3

4. Keyboard report #4

The keyboard reports are sent in order from 1 through 4, if enabled. Keyboard reports that are not enabled are not sent. The data for the four keyboard reports is in the KRGData parameter.

The KRGData parameter contains a set of sub-ID & value pairs. The most significant nibble of each subID defines the keyboard report number (1 – 4), the least significant nibble defines the sub-parameter (PortRoute, Flags, Modifiers, Keycodes). RetParmVal messages always include all sub-IDs. SetParmVal messages can include any selection of sub-IDs.

| Sub-ID | Name | Description |
|---|---|---|
| 0x11<br>0x21<br>0x31<br>0x41 | KybdRptPortRoute | Port route (1 byte):<br>bits 7 - 2: always 0.<br>bit 1: set if keyboard report should be sent to USB device port #2<br>bit 0: set if keyboard report should be sent to USB device port #1 |
| 0x12<br>0x22<br>0x32<br>0x42 | KybdRptFlags | Flags (1 byte):<br>bits 7 - 2: always 0.<br>bit 1: set if a NULL keyboard report should be sent after sending keyboard report<br>bit 0: set if keyboard report is enabled for sending |
| 0x13<br>0x23<br>0x33<br>0x43 | KybdRptModifiers | Modifier flags (2 bytes):<br>Byte 1: left modifiers<br>bits 7 - 4: always 0<br>bit 3: set if left GUI key is pressed (command key on Mac)<br>bit 2: set if left alt key is pressed (option key on Mac)<br>bit 1: set if left shift key is pressed<br>bit 0: set if left control key is pressed<br>Byte 2: right modifiers<br>bits 7 - 4: always 0<br>bit 3: set if right GUI key is pressed (command key on Mac)<br>bit 2: set if right alt key is pressed (option key on Mac)<br>bit 1: set if right shift key is pressed<br>bit 0: set if right control key is pressed |
| 0x14<br>0x24<br>0x34<br>0x44 | KybdRptKeycodes | Keycodes, 6 single-byte values encoded into 12 bytes (BAx2 format).<br>These are not ASCII codes, these are Usage IDs as defined in HID Usage Tables For Universal Serial Bus (esp. Keyboard/Keypad Page [0x07]). Use 0x00 for no keycode. |

_Example: Keyboard report group data (KRGData)_

Keyboard report #1: press left shift, a/A and b/B keys, send to USB device ports 1 and 2, do not send NULL report

```text
0x11, 0x03        // KybdRptPortRoute: send to USB device ports 1 & 2
0x12, 0x01        // KybdRptFlags: enabled, no NULL report
0x13, 0x02, 0x00  // KybdRptModifiers: left shift key
0x14,             // KybdRptKeycodes: BAx2 format
0x04, 0x00        // Keycode: keyboard a and A (0x04)
0x05, 0x00        // Keycode: keyboard b and B (0x05)
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
```

Keyboard report #2: release left shift and a/A, still pressing b/B key, send to USB device ports 1 and 2, send NULL report after this report (i.e. release all keys)

```text
0x21, 0x03        // KybdRptPortRoute: send to USB device ports 1 & 2
0x22, 0x03        // KybdRptFlags: enabled, send NULL report
0x23, 0x00, 0x00  // KybdRptModifiers: none
0x24,             // KybdRptKeycodes: BAx2 format
0x05, 0x00        // Keycode: keyboard b and B (0x05)
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
```

Keyboard report #3: press left control, right alt and delete keys, send to USB device port 1, send NULL report after this report (i.e. release all keys)

```text
0x31, 0x01        // KybdRptPortRoute: send to USB device port 1
0x32, 0x03        // KybdRptFlags: enabled, send NULL report
0x33, 0x01, 0x04  // KybdRptModifiers: left control, right alt
0x34,             // KybdRptKeycodes: BAx2 format
0x0A, 0x02        // Keycode: delete (0x2A)
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
0x00, 0x00        // Keycode: none
```

Keyboard report #4: press all left modifier keys, keypads A through F, send to USB device port 2, send NULL report after this report (i.e. release all keys)

```text
0x41, 0x02        // KybdRptPortRoute: send to USB device port 2
0x42, 0x03        // KybdRptFlags: enabled, send NULL report
0x43, 0x0F, 0x00  // KybdRptModifiers: left control, left alt, left shift, left GUI
0x44,             // KybdRptKeycodes: BAx2 format
0x0C, 0x0B        // Keycode: keypad A (0xBC)
0x0D, 0x0B        // Keycode: keypad B (0xBD)
0x0E, 0x0B        // Keycode: keypad C (0xBE)
0x0F, 0x0B        // Keycode: keypad D (0xBF)
0x00, 0x0C        // Keycode: keypad E (0xC0)
0x01, 0x0C        // Keycode: keypad F (0xC1)
```

#### Control Buttons

Control buttons support three states: inactive, single-click and double-click. All three states can generate system events and light the underlying control button LEDs. The double-click state is optional and can be enabled/disabled for each control button. Two control button modes are supported: momentary and latching. In momentary mode, the single-click and double-click states are attained by pressing the control button once or twice in rapid succession, respecively; releasing the control button returns to the inactive state. In latching mode, the single-click or double-click states are similarly attained by pressing the control button once or twice in rapid sucession, respectively; however, the state is then held until the control button is pushed once again before returning to the inactive state.

The double-click state can be either inclusive with the single-click state or exclusive to the single-click state. When set to inclusive, the first click attains the single-slick state before the second click attains the double-click state. When set to exclusive, a double-click on a control button does not first attain the single-click state before attaining the double-click state, only the double-click state is attained. Exclusive requires a short delay after the first click to determine if this is a single-click or a double-click.

Each state supports flashing between two colours at several frequencies and duty cycles. The _main_ colour is used if there is no flashing. The _alt_ colour is only used when flashing is enabled (flashing toggles between the _main_ and _alt_ colours).

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x0E | ControlButtonID | Control button ID (1 – DeviceInfo:ControlButtonMax). Required for all parameters. |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x35 | ControlButtonConfigFlags | Control button configuration flags (1 byte):<br>bits 7 – 6: reserved (always 0)<br>bits 5 – 4: control button state when preset is loaded (only used if mode is latching, always inactive when mode is momentary)<br>0 = inactive state<br>1 = single-click state<br>2 = double-click state<br>3 = no change<br>bit 3: set if mode is momentary, clear if mode is latching<br>bit 2: set if double-click state is exclusive to single-click state, clear if double-click state is inclusive with single-click state<br>bit 1: set if double-click is enabled, clear if only single-click is enabled<br>bit 0: set if control button is enabled, clear if control button is disabled |
| 0x36 | ControlButtonColourFlags | Control button colour flags, 2 bytes for each state (6 bytes total). Bytes 1-2 are for inactive state, bytes 3-4 are for single-click state, bytes 5-6 are for double-click state.<br>Byte #1 specifies main and alt colours:<br>bit 7: reserved (always 0)<br>bit 6: set if alt should light blue LED<br>bit 5: set if alt should light green LED<br>bit 4: set if alt should light red LED<br>bit 3: reserved (always 0)<br>bit 2: set if main should light blue LED<br>bit 1: set if main should light green LED<br>bit 0: set if main should light red LED<br>The DeviceInfo:ControlButtonColourSupportFlags parameter specifies which colours are supported.<br>Byte #2 specifies flashing behaviour:<br>bits 7 – 5: reserved (always 0)<br>bit 4 – 3: flashing duty cycle (main/alt)<br>0 = 50/50<br>1 = 75/25<br>2 = 88/12<br>3 = 94/6<br>bits 2 – 1: flashing frequency:<br>0 = 1x (slowest)<br>1 = 2x<br>2 = 4x<br>3 = 8x (fastest)<br>bit 0: set if flashing is enabled, clear if flashing is disabled (disabled = main colour only) |
| 0x37 | ControlButtonState | Control button state (1 byte):<br>0 = inactive state<br>1 = single-click state<br>2 = double-click state<br>Read this parameter to get the current state of the control button. Write this parameter to set the current state of the control button (equivalent to tapping the control button). |

### HardwareInfo (Data Class = 0x04)

This data class contains parameters related to hardware ports. The DeviceInfo data class returns information regarding the number and types of hardware ports that the device supports. Hardware port type and port ID are required arguments for all parameters in this data class. MIDIPortID is required for USBH-MIDI parameters.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef<br>RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is required. All parameters are global and do not support scenes. |
| RetParmVal |  |
| NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | Area | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x03 | HWPortType | Hardware port type (1 – N), see table below. Required for all parameters. |
| 0x04 | HWPortID | Hardware port ID (1 – N). Required for all parameters. |
| 0x05 | MIDIPortID | MIDI port ID (1 – N). Required for some USB host controller MIDI port parameters. |
| 0x08 | USBHMIDIID | USB host controller MIDI ID (1 – N). Required for the USB host controller MIDI port parameters. |

**Hardware Port Types:**

| Value | Description |
|---|---|
| 0x01 | DIN In |
| 0x02 | DIN Out |
| 0x03 | USB Device |
| 0x04 | USB Host Controller |
| 0x05 | Ethernet |
| 0x06 | Control |
| 0x07 | Analog Audio |
| 0x08 | Ethernet Switch |
| 0x09 | Audio Over IP |

**Parameters for All Hardware Port Types:**

| ID | Name | Description |
|---|---|---|
| 0x01 | HWPortName | Hardware port name (7-bit ASCII string). Maximum length is given by DeviceInfo:HWPortNameMax. |

**Parameters for DIN In Ports:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

**Parameters for DIN Out Ports:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

**Parameters for USB Device Ports:**

| ID | Name | Description |
|---|---|---|
| 0x10 | USBDFlags | USB device port flags (1 byte):<br>bits 7 - 4: reserved (always 0).<br>bit 3: set if this port supports HID keyboard<br>bit 2: set if this port supports audio and MIDI, clear if this port only supports MIDI<br>bit 1: set if this port supports high-speed (480 Mbit), clear if this port only supports full-speed (12 Mbit)<br>bit 0: set if this port supports firmware updates when the device is operating in bootloader mode |
| 0x11 | USBDConnect | Indicates if a host is connected to this USB device port and has enumerated the device (1 byte):<br>0 = no host connected or device not enumerated<br>1 = host connected and device is enumerated |
| 0x12 | USBDMIDI<br>PortMax | Maximum number of MIDI ports supported on this USB device port (1 – 16) (1 byte). |
| 0x13 | USBDMIDI<br>PortCount | Number of MIDI ports to use on this USB device port (1 – USBDMIDIPortMax) (1 byte). |
| 0x14 | USBDEnable<br>Audio | Flag to indicate if audio is enabled on this USB device port (1 byte). Is only used, and can only be enabled, if bit 2 is set in USBDFlags.<br>0 = audio is disabled<br>1 = audio is enabled |
| 0x15 | USBDEnable<br>HIDKybd | Flag to indicate if HID keyboard is enabled on this USB device port (1 byte).<br>Is only used, and can only be enabled, if bit 3 is set in USBDFlags.<br>0 = HID kybd is disabled<br>1 = HID kybd is enabled |

**Parameters for USB Host Controller Ports:**

| ID | Name | Description |
|---|---|---|
| 0x20 | USBHJackCount | The number of USB jacks on the interface for this host controller (1 – N) (1 byte). |
| 0x21 | USBHMIDI<br>PortMax | Maximum number of MIDI ports supported on this host controller (1 – N) (1 byte). |
| 0x22 | USBHMIDI<br>PortCount | Number of MIDI ports to use on this host controller (1 – USBHMIDIPortMax) (1 byte). |
| 0x23 | USBHMIDI<br>MultiMax | Maximum number of ports to host on multi-port MIDI devices connected to this host controller (1 – USBHMIDIPortCount) (1 byte). |
| 0x24 | USBHMIDI<br>MultiRoute | Flag to enable routing MIDI events between ports on multi-port MIDI devices connected to this host controller (1 byte).<br>0 = routing is disabled (MIDI routing matrix is not used for these ports)<br>1 = routing is enabled (MIDI routing matrix is used for these ports) |

**Parameters for USB Host Controller MIDI Ports:**

These parameters provide information about all USB-MIDI devices on a USB host controller port even if they are not connected to a MIDI port. USBHMIDIID is a required ArgID (1 – USBHMIDIPortCount) but this is not the same ID as used with the MIDIPortInfo data class. Some parameters also require MIDIPortID ArgID to specify the MIDI port on the hosted device. Note that similar parameters are present in the MIDIPortInfo section for USB host controller MIDI ports.

These parameters are read-only. Use the parameters for USB Host Controller Ports in the MIDIPortInfo section to determine which ports on hosted USB-MIDI devices are currently connected to a device’s MIDI ports.

| ID | Name | Description |
|---|---|---|
| 0x26 | USBHMIDI<br>PortNameIn | Hosted device’s MIDI IN port name (7-bit ASCII string). Devices that do not support names for MIDI ports in their USB device descriptor will return an empty string. This parameter requires the MIDIPortID ArgID to specify the port number of the device (1 – USBHMIDIPortCountIn). |
| 0x27 | USBHMIDI<br>PortNameOut | Hosted device’s MIDI IN port name (7-bit ASCII string). Devices that do not support names for MIDI ports in their USB device descriptor will return an empty string. This parameter requires the MIDIPortID ArgID to specify the port number of the device (1 – USBHMIDIPortCountOut). |
| 0x28 | USBHMIDI<br>VID | Hosted device’s USB vendor ID (16-bit value encoded in 3 bytes, 16x3 format). 0 if no device is hosted. |
| 0x29 | USBHMIDI<br>PID | Hosted device’s USB product ID (16-bit value encoded in 3 bytes, 16x3 format). 0 if no device is hosted. |
| 0x2A | USBHMIDI<br>VName | Hosted device’s vendor name from USB descriptor (7-bit ASCII string). |
| 0x2B | USBHMIDI<br>PName | Hosted device’s product name from USB descriptor (7-bit ASCII string). |
| 0x2C | USBHMIDI<br>SerialNum | Hosted device’s serial number from USB descriptor (7-bit ASCII string). Most USB-MIDI devices do not have a serial number in the USB descriptor. |
| 0x2D | USBHMIDI<br>PortCountIn | Number of MIDI IN ports supported by the hosted device (1 – 16) (1 byte). 0 if no device is hosted. |
| 0x2E | USBHMIDI<br>PortCountOut | Number of MIDI OUT ports supported by the hosted device (1 – 16) (1 byte).<br>0 if no device is hosted. |
| 0x2F | USBHMIDI<br>Identifier | A unique ID assigned to each hosted device (1 – N) (1 byte). This can be used to distinguish between multiple devices that have the same USBHMIDIVID and USBHMIDIPID with a blank USBHMIDISerialNum. 0 if no device is hosted. |

**Parameters for Ethernet Ports:**

| ID | Name | Description |
|---|---|---|
| 0x30 | EthMACAddress | Ethernet MAC address, 48-bit value encoded in 12 bytes using 4 bits per byte, least significant 4 bits first, most significant four bits last. Example (MAC address = AC:7A:42:12:34:56):<br>0x06, 0x05, 0x04, 0x03, 0x02, 0x01, 0x02, 0x04, 0x0A, 0x07, 0x0C, 0x0A |
| 0x31 | EthConnect | Indicates if Ethernet port is connected to a network (1 byte):<br>0 = not connected<br>1 = connected |
| 0x32 | EthCurrentIP | Current (active) IP address, subnet mask, and gateway address (15 bytes).<br>All are four-byte big-endian values encoded in five-bytes (32x5 format).<br>These values are only valid if the Ethernet port is connected to a network, is active, and (if EthIPMode = dynamic) has obtained values from a DHCP server or AutoIP has finished negotiation. Example:<br>0x0A, 0x4F, 0x78, 0x00, 0x08 // current IP address (169.254.0.8)<br>0x0F, 0x7F, 0x7C, 0x00, 0x00 // current subnet mask (255.255.0.0)<br>0x0A, 0x4F, 0x78, 0x00, 0x01 // current gateway address (169.254.0.1) |
| 0x33 | EthDevName | Device name on network (Bonjour name), 7-bit ASCII string, taken from DeviceInfo:DevName parameter. EthDevName is only valid if the Ethernet port is connected to a network and has finished Bonjour name resolution. |
| 0x34 | EthConfigFlags | Ethernet configuration flags (1 byte):<br>bits 7 - 2: reserved (always 0).<br>bit 1: EthMulticastMode: set if multicast packets (MDNS) should be ignored, clear if multicast packets should be accepted.<br>bit 0: EthIPMode: set if using dynamic IP addresses (DHCP or AutoIP if DHCP server is not available), clear if using static IP addresses. |
| 0x35 | EthStaticIP | Device IP address, subnet mask, and gateway address to use when<br>EthIPMode = static (15 bytes). All are four-byte big-endian values encoded in five-bytes (32x5 format). Example:<br>0x0C, 0x05, 0x20, 0x02, 0x64 // static IP address (192.168.1.100)<br>0x0F, 0x7F, 0x7F, 0x7E, 0x00 // static subnet mask (255.255.255.0)<br>0x0C, 0x05, 0x20, 0x02, 0x01 // static gateway address (192.168.1.1) |
| 0x36 | EthMIDIPortMax | Maximum number of RTP-MIDI sessions supported on this Ethernet port (1 – N) (1 byte). |
| 0x37 | EthMIDIPortCount | Number of RTP-MIDI sessions to use on this Ethernet port (1 – EthMIDIPortMax) (1 byte). |

**Parameters for Control Ports:**

| ID | Name | Description |
|---|---|---|
| 0x40 | CtrlLocation | Control location on product (2 bytes):<br>Byte 1: control jack number (1 – N).<br>Byte 2: jack pin number: 1 = tip, 2 = ring. |
| 0x41 | CtrlSupportFlags | Control support flags (1 byte):<br>bits 7 - 2: reserved (always 0).<br>bit 1: set if this control can be used as an output.<br>bit 0: set if this control can be used as an input. |
| 0x42 | CtrlConfigFlags | Control configuration flags (1 byte):<br>bits 7 - 5: reserved (always 0).<br>bit 4: set if control state should be read and applied on power up (inputs only).<br>bit 3: set if control is momentary, clear if control is latching.<br>bit 2: set if control is inverted (normally closed), clear if control is not inverted (normally open).<br>bit 1: set if this control is an input, clear if this control is an output.<br>bit 0: set if control is enabled, clear if control is disabled. |
| 0x43 | CtrlDbncHoldTime | Control debounce time (inputs) or hold time (outputs, if momentary flag is set) in milliseconds (1 – 6000) (2 bytes 14x2 format). Can also use 0 to indicate “minimum time supported by device” which may be less than 1 ms. |
| 0x44 | CtrlState | Control state (1 byte):<br>0 = control is inactive/off (read/write)<br>1 = control is active/on (read/write)<br>2 = toggle control state (write only)<br>Outputs support read and write. Inputs are read-only (writing has no effect).<br>Reading returns the control’s active/inactive status. For latching controls this is the same state as seen at the control’s input pin. For momentary controls this is the control’s runtime state, not the state as seen at the control’s input pin. |

**Parameters for Analog Audio Ports:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

**Parameters for Ethernet Switch Ports:**

| ID | Name | Description |
|---|---|---|
| 0x50 | EthSwitchPortTypeLocation | Ethernet switch port type and location (1 byte):<br>The least significant nibble contains the port type:<br>1 = external RJ45 jack<br>2 = internal RTP-MIDI port<br>3 = internal digital audio port<br>The most significant nibble contains the port location on product:<br>0 = internal port<br>1 – N = RJ45 jack number |
| 0x51 | EthSwitchPortConnect | Indicates if Ethernet switch port is connected to a network (1 byte). Only significant for external ports; internal ports always return 0.<br>0 = not connected<br>1 = connected |
| 0x52 | EthSwitchPortForward | Ethernet switch port forwarding flags (1 byte). Used to restrict which output ports this input port can forward frames to. It is not possible for a port to forward frames to itself.<br>bit 7: reserved (always 0).<br>bit 6: set if this port forwards to port 7.<br>bit 5: set if this port forwards to port 6.<br>bit 4: set if this port forwards to port 5.<br>bit 3: set if this port forwards to port 4.<br>bit 2: set if this port forwards to port 3<br>bit 1: set if this port forwards to port 2.<br>bit 0: set if this port forwards to port 1. |

**Parameters for Audio Over IP Ports:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

_Example: GetParmVal (Ethernet port 1)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0E                    // message length (15)
0x03, 0x04                    // message class, data class
0x02                          // NumDataBlock: (2)
0x07, 0x04                    // data block #1: size (7), type (ArgVal)
0x02                          // NumArgValBlock: (2)
0x03                          // block #1: ArgID: hardware port type (3)
0x05                          // block #1: ArgVal: Ethernet (5)
0x04                          // block #2: ArgID: hardware port ID (4)
0x01                          // block #2: ArgVal: (1)
0x05, 0x01                    // data block #3: size (5), type (ParmList)
0x02                          // NumParmID: (2)
0x30                          // ParmID #1: EthMACAddress
0x32                          // ParmID #2: EthCurrentIP
0xxx                          // checksum
0xF7                          // footer
```

_Example: RetParmVal (Ethernet port 1)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x2C                    // message length (44)
0x43, 0x04                    // message class, data class
0x02                          // NumDataBlock: (2)
0x07, 0x04                    // data block #1: size (7), type (ArgVal)
0x02                          // NumArgValBlock: (2)
0x03                          // block #1: ArgID: hardware port type (3)
0x05                          // block #1: ArgVal: Ethernet (5)
0x04                          // block #2: ArgID: hardware port ID (4)
0x01                          // block #2: ArgVal: (1)
0x22, 0x03                    // data block #2: size (34), type (ParmVal)
0x02                          // NumParmValBlock: (2)
0x0E                          // block #1: size (14)
0x30                          // block #1: parm ID (EthMACAddress)
0x06, 0x05, 0x04, 0x03,       // Ethernet MAC address AC:7A:42:12:34:56
0x02, 0x01, 0x02, 0x04, 0x0A, 0x07, 0x0C, 0x0A
0x11                          // block #2: size (17)
0x32                          // block #2: parm ID (EthCurrentIP)
0x0A, 0x4F, 0x78, 0x00, 0x08  // current IP address (169.254.0.8)
0x0F, 0x7F, 0x7C, 0x00, 0x00  // current subnet mask (255.255.0.0)
0x0A, 0x4F, 0x78, 0x00, 0x01  // current gateway address (169.254.0.1)
0xxx                          // checksum
0xF7                          // footer
```

_Example:SetParmVal (Ethernet port 1)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x21                    // message length (33)
0x10, 0x04                    // message class, data class
0x02                          // NumDataBlock: (2)
0x07, 0x04                    // data block #1: size (7), type (ArgVal)
0x02                          // NumArgValBlock: (2)
0x03                          // block #1: ArgID: hardware port type (3)
0x05                          // block #1: ArgVal: Ethernet (5)
0x04                          // block #2: ArgID: hardware port ID (4)
0x01                          // block #2: ArgVal: (1)
0x17, 0x03                    // data block #2: size (23), type (ParmVal)
0x02                          // NumParmValBlock: (2)
0x03                          // block #1: size (3)
0x34                          // block #1: parm ID (EthIPMode)
0x00                          // IPMode: static
0x11                          // block #2: size (17)
0x35                          // block #2: parm ID (EthStaticIP)
0x0C, 0x05, 0x20, 0x02, 0x64  // static IP address (192.168.1.100)
0x0F, 0x7F, 0x7F, 0x7E, 0x00  // static subnet mask (255.255.255.0)
0x0C, 0x05, 0x20, 0x02, 0x01  // static gateway address (192.168.1.1)
0xxx                          // checksum
0xF7                          // footer
```

### MIDIInfo (Data Class = 0x05)

This data class contains global parameters related to the MIDI capabilities and configuration of a device.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef<br>RetParmDef |  |
| GetParmVal<br>SetParmVal | No argument value block. All parameters are global and do not support scenes. |
| RetParmVal<br>NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | Area | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x0B | MIDIMonitor<br>EventID | MIDI monitor event ID (0 – N). Required only for the PortMonitorDetailDisplayIn/Out parameters. MIDIMonitorEventID 0 is used if this argument is not supplied. Supported values:<br>0 = no MIDI events<br>// channel events<br>1 = Note Off (0x8n)<br>2 = Note On (0x9n)<br>3 = Poly Aftertouch (0xAn)<br>4 = Control Change (0xBn)<br>5 = Program Chage (0xCn)<br>6 = Mono Aftertouch (0xDn)<br>7 = Pitch Bend (0xEn)<br>// system common events 1<br>8 = Clock (0xF8)<br>9 = Start (0xFA)<br>10 = Continue (0xFB)<br>11 = Stop (0xFC)<br>12 = MIDI Timecode Quarter Frame (0xF1)<br>13 = Song Position (0xF2)<br>14 = Song Select (0xF3)<br>// system common events 2<br>15 = Tune Request (0xF6)<br>16 = Active Sense (0xFE)<br>17 = Reset (0xFF)<br>18 = System Exclusive (0xF0) |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x01 | PortCount | Total number of MIDI ports supported by device (1 – N) (1 byte). |
| 0x02 | DINPortCount | Number of DIN MIDI ports supported by device (0 – N) (1 byte). |
| 0x03 | CtrlPortCount | Number of (internal) MIDI control ports supported by device (0 – N) (1 byte). |
| 0x04 | USBDPortCount | Number of USBD MIDI ports supported by device (0 – N) (1 byte). |
| 0x05 | USBHPortCount | Number of USBH MIDI ports supported by device (0 – N) (1 byte). |
| 0x06 | EthPortCount | Number of RTP-MIDI sessions supported by device (0 – N) (1 byte). |
| 0x07 | MIDIPortNameMax | Maximum length allowed for MIDI port name (MIDIPortInfo:PortNameIn & PortNameOut) (1 byte). |
| 0x08 | USBDPortNameMax | Maximum length allowed for USB device MIDI port name (MIDIPortInfo:USBDPortName) (1 byte). |
| 0x09 | EthSesnNameMax | Maximum length allowed for Ethernet session name (MIDIPortInfo:EthSesnName) and remote session name (MIDIPortInfo:EthSesnNameR) (1 byte). |
| 0x0A | PortFeatureFlags | MIDI port feature flags (1 byte):<br>bits 7 - 5: reserved (always 0).<br>bit 4: set if AMP is supported.<br>bit 3: set if CCProcessor is supported.<br>bit 2: set if MIDI remap for channel messages is supported.<br>bit 1: set if MIDI filter for channel messages is supported.<br>bit 0: set if MIDI filter for system messages is supported. |
| 0x0B | AMPAlgMax | Maximum number of AMP algorithms supported by device (1 byte). |
| 0x0C | AMPOpMax | Maximum number of AMP operators supported by device (1 byte). |
| 0x0D | AMPCRMMax | Maximum number of AMP custom route maps supported by device (1 byte). |
| 0x0E | AMPLUTMax | Maximum number of AMP lookup tables supported by device (1 byte). |
| 0x0F | AMPOPAMax | Maximum number of AMP operators-per-algorithm supported by device (1 byte). |
| 0x10 | AMPAlg<br>NameMax | Maximum length allowed for AMP algorithm name (1 byte). |
| 0x11 |  |  |
| 0x12 | CCProcessorMax | Maximum number of control change processors supported by device for each MIDI port (1 byte). |
| 0x13 | PortMonitorDetailIconMax | Maximum number of MIDI event icons supported by device (1 byte). |
| 0x14 | PanicSupportFlags | Panic support flags (1 byte):<br>bits 7 - 2: reserved (always 0).<br>bit 1: set if Panic message can be sent automatically when changing presets.<br>bit 0: set if Panic message can be sent manually from front panel.<br>Panic message can also be automatically sent when changing scenes (for devices that support failover). See the Automatic Failover section for more info. |
| 0x15 | AutomationControlPortSupportFlags | Automation Control Port support flags (2 bytes):<br>Byte 1:<br>bit 7: reserved (always 0).<br>bit 6: set if MIDI-Gobs can be sent using automation control port.<br>bit 5: set if Panic message can be sent using automation control port.<br>bit 4: set if Failover arm/disarm state can be changed and Failover Alarm can be cleared using automation control port.<br>bit 3: set if audio controls can be changed using automation control port (see AudioAutomationInfo data class).<br>bit 2: set if mixer snapshots can be changed using automation control port (see MIDIPortInfo:MixerSnapshotSelector).<br>bit 1: set if scenes can be changed using automation control port (see MIDIPortInfo:SceneSelector).<br>bit 0: set if presets can be changed using automation control port (see MIDIPortInfo:PresetSelector).<br>Byte 2:<br>bits 7 - 1: reserved (always 0).<br>bit 0: set if hardware control port outputs can be changed using automation control port. |
| 0x16 |  |  |
| 0x17 | MIDIGobMax | Maximum number of MIDI-Gobs supported by device (1 byte). |
| 0x18 | MIDIGobNameMax | Maximum length allowed for MIDI-Gob name (1 byte). |
| 0x19 | MIDIGobDataMax | Maximum length allowed for data in one MIDI-Gob (1 byte). |
| 0x1A | RouteProcessBlockMax | Maximum number of route processor blocks supported by device (1 byte). |
| 0x1B | RouteProcessBlockNameMax | Maximum length allowed for route processor block name (1 byte). |
| 0x20 | PortMonitorIn | Bitmap indicating activity on all MIDI inputs (length depends on the number of MIDI ports). See below for more info. Also see MIDIInfo:PortMonitorDetailEnableIn and MIDIPortInfo:PortMonitorDetailIn. |
| 0x21 | PortMonitorOut | Bitmap indicating activity on all MIDI outputs (length depends on the number of MIDI ports). See below for more info. Also see MIDIInfo:PortMonitorDetailEnableOut and MIDIPortInfo:PortMonitorDetailOut. |
| 0x40 | PortMonitorDetailEnableIn | Bitmap indicating which types of MIDI events are monitored for all MIDI inputs. See below for more info. Also see MIDIInfo:PortMonitorIn and MIDIPortInfo:PortMonitorDetailIn. |
| 0x41 | PortMonitorDetailEnableOut | Bitmap indicating which types of MIDI events are monitored for all MIDI outputs. See below for more info. Also see MIDIInfo:PortMonitorOut and MIDIPortInfo:PortMonitorDetailOut. |
| 0x42 | PortMonitorDetailDisplayIn | Display settings for MIDI events on all MIDI inputs. See below for more info.<br>MIDIMonitorEventID is a required ArgID for this parameter. |
| 0x43 | PortMonitorDetailDisplayOut | Display settings for MIDI events on all MIDI outputs. See below for more info.<br>MIDIMonitorEventID is a required ArgID for this parameter. |

**PortMonitorIn and PortMonitorOut Parameter Block:**

Bitmap indicating MIDI activity on all ports (BAx2 format). Bit 0 is port #1, bit 1 is port #2, etc. Least significant byte is first, most significant byte is last. The most significant 4 bits of each byte must be 0 which allows 4 ports to be specified per byte. Unused bits should be set to 0. The number of ports is given in the MIDIInfo:PortCount parameter. The number of bytes (in the bitmap) is an even value so that the bitmap (when unpacked) is a multiple of 8 bits:

number of bytes (using INTEGER math) = (((number of MIDI ports - 1) / 8) + 1) x 2

_Example (device has 20 ports):_

```text
0x01  // MIDI activity for ports 4 through 1 (port 1 has activity)
0x04  // MIDI activity for ports 8 through 5 (port 7 has activity)
0x0C  // MIDI activity for ports 12 through 9 (ports 12 and 11 have activity)
0x03  // MIDI activity for ports 16 through 13 (ports 14 and 13 have activity)
0x08  // MIDI activity for ports 20 through 17 (port 20 has activity)
0x00  // MIDI activity for ports 24 through 21 (padding)
```

**PortMonitorDetailEnableIn and PortMonitorDetailEnableOut Parameter Block:**

Each block contains a set of sub-ID & value pairs that enable or disable the MIDI monitor for specific types of MIDI events. The sub-ID and value pairs are the same as those used in MIDIPortInfo:PortMonitorDetailIn/Out parameter blocks. RetParmVal messages always return all sub-IDs (6 bytes). SetParmVal messages can include one sub-ID, two sub-IDs, or all three sub-IDs (2, 4, and 6 bytes, respectively).

| Sub-ID | Description |
|---|---|
| 0x01 | MIDI channel message event flags (1 byte):<br>bit 7: always 0.<br>bit 6: set if monitor should be enabled for MIDI Pitch Bend events (0xEn).<br>bit 5: set if monitor should be enabled for MIDI Channel Pressure events (0xDn).<br>bit 4: set if monitor should be enabled for MIDI Program Change events (0xCn).<br>bit 3: set if monitor should be enabled for MIDI Control Change events (0xBn).<br>bit 2: set if monitor should be enabled for MIDI Poly Key Pressure events (0xAn).<br>bit 1: set if monitor should be enabled for MIDI Note On events (0x9n).<br>bit 0: set if monitor should be enabled for MIDI Note Off events (0x8n). |
| 0x02 | MIDI system message event flags #1 (1 byte):<br>bit 7: always 0.<br>bit 6: set if monitor should be enabled for MIDI Song Select events (0xF3).<br>bit 5: set if monitor should be enabled for MIDI Song Position Pointer events (0xF2).<br>bit 4: set if monitor should be enabled for MIDI Time Code Quarter Frame events (0xF1).<br>bit 3: set if monitor should be enabled for MIDI Stop events (0xFC).<br>bit 2: set if monitor should be enabled for MIDI Continue events (0xFB).<br>bit 1: set if monitor should be enabled for MIDI Start events (0xFA).<br>bit 0: set if monitor should be enabled for MIDI Timing Clock events (0xF8). |
| 0x03 | MIDI system message event flags #2 (1 byte):<br>bits 7 - 4: always 0.<br>bit 3: set if monitor should be enabled for MIDI System Exclusive events (0xF0).<br>bit 2: set if monitor should be enabled for MIDI Reset events (0xFF).<br>bit 1: set if monitor should be enabled for MIDI Active Sensing events (0xFE).<br>bit 0: set if monitor should be enabled for MIDI Tune Request events (0xF6). |

**PortMonitorDetailDisplayIn and PortMonitorDetailDisplayOut Parameter Block:**

Each block contains a set of sub-ID & value pairs that configure the MIDI monitor display for a specific type of MIDI event. MIDIMonitorEventID ArgID is used to specify the MIDI event (use 0 for no MIDI events). RetParmVal messages always return all sub-IDs (6 bytes). SetParmVal messages can include one sub-ID, two sub-IDs, or all three sub-IDs (2, 4, and 6 bytes, respectively).

| Sub-ID | Description |
|---|---|
| 0x01 | Icon ID (0 – MIDIInfo:PortMonitorDetailIconMax) (1 byte). Use 0 for blank icon. |
| 0x02 | Level (0 – DeviceInfo:DisplayBrightnessMax) (1 byte). Use 0 for off. Use DeviceInfo:DisplayBrightnessMax for maximum brightness. Level is ignored if Icon ID = 0 (blank icon). |
| 0x03 | Priority (0 – 127) (1 byte). Sets the priority for this MIDI event: 0 = lowest priority, 127 = highest priority. If different types of MIDI events occur within a display update interval the MIDI event with the highest priority will be displayed. If there are multiple types of MIDI events with the same highest priority, the MIDI event with the lowest MIDIMonitorEventID will be used. Priority is ignored for MIDIMonitorEventID = 0 (no MIDI events). |

### MIDIPortInfo (Data Class = 0x06)

This data class contains parameters related to specific MIDI ports. MIDI port ID is a required argument for all parameters in this data class. MIDI channel number is a required argument for the FilterChannelIn, FilterChannelOut, RemapChannelIn, and RemapChannelOut parameters.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef<br>RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is required. Some parameters can be specified for individual scenes. |
| RetParmVal<br>NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x02 | SceneID | Scene selection (0 – N). Only applies to devices and parameters that support scenes. 0 is used for the currently active scene, 1 – N is used to specify a particular scene. Use 0 for globals and parameters that do not support scenes. |
| 0x04 | HWPortID | Hardware port ID (1 – N). Required for some parameters. Hardware port ID 1 is used if this argument is not supplied. |
| 0x05 | MIDIPortID | MIDI port ID (1 – N). Required for all parameters. |
| 0x06 | MIDIChannel | MIDI channel number (1 – 16).  Required only for the FilterChannelIn, FilterChannelOut, RemapChannelIn, and RemapChannelOut parameters.<br>Channel 1 is used if this argument is not supplied. |
| 0x0A | MIDICC<br>ProcessorID | MIDI control change processor ID (1 – N). Required only for the CCProcessorIn and CCProcessorOut parameters. MIDICCProcessor 1 is used if this argument is not supplied. |
| 0x0C | MIDIGobID | MIDI-Gob ID (1 – N). Required for some Automation Control port parameters.<br>MIDI-Gob 1 is used if this argument is not supplied. |
| 0x0D | KybdRptGrpID | Keyboard report group ID (1 – N). Required for some Automation Control port parameters. Keyboard report group 1 is used if this argument is not supplied. |

**MIDI Port Types:**

| Value | Description |
|---|---|
| 0x01 | DIN |
| 0x02 | USB Device |
| 0x03 | USB Host Controller |
| 0x04 | Ethernet |
| 0x05 | Control (see Control Port Types table below) |

**Control Port Types:**

| Value | Description |
|---|---|
| 0x01 | Automation Control. Used to change presets, scenes, mixer snapshots, analog audio controls, mixer controls, and failover operation using MIDI continuous controller and program change messages. |

**Parameters for All MIDI Port Types:**

| ID | Name | Description |
|---|---|---|
| 0x01 | PortType | MIDI port type (1 byte), see MIDI Port Types table. |
| 0x02 | PortIdentifier | MIDI port identifier (2 bytes), depends on PortType:<br>DIN:<br>Byte 1: DIN IN port ID (1 – N, 0 if port has no hardware DIN IN port).<br>Byte 2: DIN OUT port ID (1 – N, 0 if port has no hardware DIN OUT port).<br>USB Device:<br>Byte 1: USB device port ID (1 – N).<br>Byte 2: MIDI port number (1 – 16).<br>USB Host Controller :<br>Byte 1: USB host controller ID (1 – N).<br>Byte 2: USB host controller port (1 – N).<br>Ethernet :<br>Byte 1: Ethernet port (1 - N).<br>Byte 2: RTP-MIDI session number (1 - N).<br>Control:<br>Byte 1: Control Port ID (1 – N).<br>Byte 2: Control Port Type (1 – N), see Control Port Types table. |
| 0x03 | PortConnectFlags | MIDI port connection flags (1 byte):<br>bits 7 - 1: reserved (always 0).<br>bit 0: set if this port has an active connection.<br>Active USB device ports return 1 for bit 0 if a host is connected. Active USB host ports return 1 for bit 0 if a device is connected. Active Ethernet ports return 1 for bit 0 if a session is connected. Active DIN and Control ports always return 1 for bit 0. Inactive ports (all types) always return 0 for bit 0. |
| 0x04 | PortActiveFlags | MIDI port active flags (1 byte):<br>bits 7 - 1: reserved (always 0).<br>bit 0: set if this port is active.<br>The number of MIDI ports available on USB device, USB host controller and Ethernet ports can be set using the HardwareInfo parameters USBDMIDIPortCount, USBHMIDIPortCount and EthMIDIPortCount respectively. |
| 0x05 | PortSupportFlags | MIDI port support flags (1 byte):<br>bits 7 - 3: reserved (always 0).<br>bit 2: set if this port supports running status on MIDI output.<br>bit 1: set if this port supports MIDI output.<br>bit 0: set if this port supports MIDI input. |
| 0x06 | PortEnableFlags | MIDI port enable flags (1 byte). These bits can only be set if the matching bit is set in PortSupportFlags:<br>bits 7 - 3: reserved (always 0).<br>bit 2: set if running status is enabled on MIDI output.<br>bit 1: set if the MIDI output port is enabled.<br>bit 0: set if the MIDI input port is enabled. |
| 0x07 | PortRoutePreset | Bitmap indicating preset routing for MIDI events entering at the input for this port (BAx2 format). Each bit represents a MIDI port output. Bit 0 is routing from this port to port #1, bit 1 is routing from this port to port #2, etc. Least significant byte is first, most significant byte is last. The most significant 4 bits of each byte must be 0 which allows 4 ports to be specified per byte. Unused bits should be set to 0. The number of MIDI ports is given in the MIDIInfo:PortCount parameter. The number of bytes (in the bitmap) is an even value so that the bitmap (when unpacked) is a multiple of 8 bits:<br>number of bytes (using INTEGER math) = (((number of MIDI ports - 1) / 8 ) + 1) x 2.<br>Example: device has 20 ports, route MIDI events coming into this port to ports 2, 3, 7, 11-14, 20:<br>0x06<br>// routing for ports 4 through 1 (ports 3 and 2 are selected)<br>0x04<br>// routing for ports 8 through 5 (port 7 is selected)<br>0x0C<br>// routing for ports 12 through 9 (ports 12 and 11 are selected)<br>0x03<br>// routing for ports 16 through 13 (ports 14 and 13 are selected)<br>0x08<br>// routing for ports 20 through 17 (port 20 is selected)<br>0x00<br>// routing for ports 24 through 21 (padding)<br>Also see MIDIPortInfo:PortRouteGlobal. Preset and global routing bitmaps are OR’d together. |
| 0x08 | PortFeatureFlagsIn | MIDI port feature enable flags for MIDI input (1 byte). These bits can only be set if the matching bit is set in MIDIInfo:PortFeatureFlags:<br>bits 7 - 5: reserved (always 0).<br>bit 4: set if AMP is enabled.<br>bit 3: set if CCProcessor is enabled.<br>bit 2: set if MIDI remap for channel messages is enabled.<br>bit 1: set if MIDI filter for channel messages is enabled.<br>bit 0: set if MIDI filter for system messages is enabled. |
| 0x09 | PortFeatureFlagsOut | MIDI port feature enable flags for MIDI output (1 byte). These bits can only be set if the matching bit is set in MIDIInfo:PortFeatureFlags:<br>bits 7 - 5: reserved (always 0).<br>bit 4: set if AMP is enabled.<br>bit 3: set if CCProcessor is enabled.<br>bit 2: set if MIDI remap for channel messages is enabled.<br>bit 1: set if MIDI filter for channel messages is enabled.<br>bit 0: set if MIDI filter for system messages is enabled. |
| 0x0A | PortNameIn | MIDI port name to use for incoming MIDI events (7-bit ASCII string).<br>Maximum length is given by MIDIInfo:MIDIPortNameMax. |
| 0x0B | PortNameOut | MIDI port name to use for outgoing MIDI events (7-bit ASCII string).<br>Maximum length is given by MIDIInfo:MIDIPortNameMax. |
| 0x0C | FilterSystemIn | Filter settings for MIDI system messages on MIDI input (2-4 bytes). See below. |
| 0x0D | FilterSystemOut | Filter settings for MIDI system messages on MIDI output (2-4 bytes). See below. |
| 0x0E | FilterChannelIn | Filter settings for MIDI channel messages on MIDI input (2 bytes). See below. MIDI Channel is a required ArgID for this parameter |
| 0x0F | FilterChannelOut | Filter settings for MIDI channel messages on MIDI output (2 bytes). See below. MIDI Channel is a required ArgID for this parameter |
| 0x10 | RemapChannelIn | Remap settings for MIDI channel message on MIDI input (2-14 bytes). See below. MIDI Channel is a required ArgID for this parameter. |
| 0x11 | RemapChannelOut | Remap settings for MIDI channel message on MIDI output (2-14 bytes). See below. MIDI Channel is a required ArgID for this parameter. |
| 0x12 | CCProcessorIn | Settings for control change processor on MIDI input (10 bytes). See below for more info. MIDICCProcessorID is a required ArgID for this parameter. |
| 0x13 | CCProcessorOut | Settings for control change processor on MIDI output (10 bytes). See below for more info. MIDICCProcessorID is a required ArgID for this parameter. |
| 0x14 | AMPAlgorithmIn | AMP algorithm ID for MIDI input (0 – N) (1 byte). Use 0 for no algorithm. |
| 0x15 | AMPAlgorithmOut | AMP algorithm ID for MIDI output (0 – N) (1 byte). Use 0 for no algorithm. |
| 0x16 | PortMonitorDetailIn | Bitmap containing MIDI data details for a specific MIDI input port (6 bytes).<br>See below for more info. Also see MIDIInfo:PortMonitorIn and MIDIInfo:PortMonitorDetailEnableIn. |
| 0x17 | PortMonitorDetailOut | Bitmap containing MIDI data details for a specific MIDI output port (6 bytes).<br>See below for more info. Also see MIDIInfo:PortMonitorOut and MIDIInfo:PortMonitorDetailEnableOut. |
| 0x18 | PortRouteGlobal | Bitmap indicating global routing for MIDI events entering at the input for this port (BAx2 format). See MIDIPortInfo:PortRoutePreset for more info. |
| 0x19 | PortRouteProcessBlockID | _This is not yet implemented._ |

**FilterSystemIn and FilterSystemOut Parameter Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs (4 bytes total). SetParmVal messages can include both sub-IDs (4 bytes total) or just one sub-ID (2 bytes total).

| Sub-ID | Description |
|---|---|
| 0x01 | MIDI system message filter flags #1 (1 byte):<br>bit 7: always 0.<br>bit 6: set if port should filter MIDI Song Select events (0xF3).<br>bit 5: set if port should filter MIDI Song Position Pointer events (0xF2).<br>bit 4: set if port should filter MIDI Time Code Quarter Frame events (0xF1).<br>bit 3: set if port should filter MIDI Stop events (0xFC).<br>bit 2: set if port should filter MIDI Continue events (0xFB).<br>bit 1: set if port should filter MIDI Start events (0xFA).<br>bit 0: set if port should filter MIDI Timing Clock events (0xF8). |
| 0x02 | MIDI system message filter flags #2 (1 byte):<br>bits 7 - 4: always 0.<br>bit 3: set if port should filter MIDI System Exclusive events (0xF0).<br>bit 2: set if port should filter MIDI Reset events (0xFF).<br>bit 1: set if port should filter MIDI Active Sensing events (0xFE).<br>bit 0: set if port should filter MIDI Tune Request events (0xF6). |

**FilterChannelIn and FilterChannelOut Parameter Block:**

Each block contains a set of sub-ID & value pairs for MIDI channel message filtering. One block is used for each of the 16 MIDI channels (using the MIDIChannel ArgID). RetParmVal messages always return all sub-IDs for the selected MIDI channel (always 2 bytes since there is only one sub-ID). SetParmVal messages can send any subset of the sub-IDs for the selected MIDI channel (always 2 bytes since there is only one sub-ID).

| Sub-ID | Description |
|---|---|
| 0x01 | MIDI channel message filter flags (1 byte):<br>bit 7: always 0.<br>bit 6: set if port should filter MIDI Pitch Bend events (0xEn).<br>bit 5: set if port should filter MIDI Channel Pressure events (0xDn).<br>bit 4: set if port should filter MIDI Program Change events (0xCn).<br>bit 3: set if port should filter MIDI Control Change events (0xBn).<br>bit 2: set if port should filter MIDI Poly Key Pressure events (0xAn).<br>bit 1: set if port should filter MIDI Note On events (0x9n).<br>bit 0: set if port should filter MIDI Note Off events (0x8n).<br>Note that enabling the filter for Control Change messages will filter out all controls on that MIDI channel; to filter out specific controls use CCProcessorIn/Out or AMP. |

**RemapChannelIn and RemapChannelOut Parameter Block:**

Each block contains a set of sub-ID & value pairs to select which MIDI channel messages are to be remapped and the MIDI channel to use for the modified message. One block is used for each of the 16 MIDI channels (using the MIDIChannel ArgID). RetParmVal messages always return all sub-IDs for the selected MIDI channel (14 bytes). SetParmVal messages can send any subset of the sub-IDs for the selected MIDI channel (e.g. 2 bytes for one remap, 14 bytes for all remaps).

| Sub-ID | Description |
|---|---|
| 0x01 | MIDI channel number for remapping MIDI Note Off events (0x8n) (1 – 16) (1 byte). |
| 0x02 | MIDI channel number for remapping MIDI Note On events (0x9n) (1 – 16) (1 byte). |
| 0x03 | MIDI channel number for remapping MIDI Poly Key Pressure events (0xAn) (1 – 16) (1 byte). |
| 0x04 | MIDI channel number for remapping MIDI Control Change events (0xBn) (1 – 16) (1 byte). Note that enabling remapping for Control Change messages will remap all controls on that MIDI channel; to remap specific controls use AMP. |
| 0x05 | MIDI channel number for remapping MIDI Program Change events (0xCn) (1 – 16) (1 byte). |
| 0x06 | MIDI channel number for remapping MIDI Channel Pressure events (0xDn) (1 – 16) (1 byte). |
| 0x07 | MIDI channel number for remapping MIDI Pitch Bend events (0xEn) (1 – 16) (1 byte). |

**CCProcessorIn and CCProcessorOut Parameter Block:**

Each block contains a set of sub-ID & value pairs for one CCProcessor (use the MIDICCProcessorID ArgID to select the CCProcessor). RetParmVal messages always return all sub-IDs for the selected CCProcessor (10 bytes). SetParmVal messages can send any subset of the sub-IDs for the selected CCProcessor.

| Sub-ID | Description |
|---|---|
| 0x01 | Mode: Operating mode of this CCProcessor (1 byte):<br>0x00: Disable. Processor is inactive.<br>0x01: Stop Filter. MIDI CC events that match the MIDI channel and whose CC ID is between CCID1 and CCID2 (inclusive) are deleted. Non-matching MIDI CC events are sent to the next CCProcessor.<br>0x02: Pass Filter. MIDI CC events that match the MIDI channel and whose CC ID is between CCID1 and CCID2 (inclusive) are passed to the next CCProcessor that is not a Pass Filter. Non-matching MIDI CC events are sent to the next CCProcessor if that<br>CCProcessor is a Pass Filter. If the next processor is not a CCProcessor, then non-matching MIDI CC events are deleted. Adjacent Pass Filters effectively operate together as a group that will pass any MIDI CC event that matches one of the CCProcessors and deletes any MIDI CC event that does not match any CCProcessor.<br>0x03: Remap. MIDI CC events that match the MIDI channel and whose CC ID equals<br>CCID1 are modified such that the event’s CC ID is set to CCID2. The matching event Is then sent to the next CCProcessor that is not a Remap. Non-matching MIDI CC events are sent to the next CCProcessor unchanged. |
| 0x02 | MIDI Channel: Bitmask indicating which MIDI channels to use for matching the Stop, Pass, or Remap condition (0x0001 = MIDI channel 1, 0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x03 | CCID1: MIDI control change number, from 0 to 127 (1 byte). For Stop Filter and Pass Filter processing, this value should be less than or equal to CCID2. For Remap processing this should be set to the CC ID that is to be replaced. |
| 0x04 | CCID2: MIDI control change number, from 0 to 127 (1 byte). For Stop Filter and Pass Filter processing, this value should be greater than or equal to CCID1. For Remap processing this should be set to the replacement CC ID. |

**PortMonitorDetailIn and PortMonitorDetailOut Parameter Block:**

Each block contains a set of sub-ID & value pairs indicating MIDI activity on a specific port. The sub-ID and value pairs are the same as those used in MIDIInfo:PortMonitorDetailEnableIn/Out parameter blocks. RetParmVal messages always return all sub-IDs for the selected MIDI channel (6 bytes).

| Sub-ID | Description |
|---|---|
| 0x01 | MIDI channel message event flags (1 byte):<br>bit 7: always 0.<br>bit 6: set if port passed any MIDI Pitch Bend events (0xEn).<br>bit 5: set if port passed any MIDI Channel Pressure events (0xDn).<br>bit 4: set if port passed any MIDI Program Change events (0xCn).<br>bit 3: set if port passed any MIDI Control Change events (0xBn).<br>bit 2: set if port passed any MIDI Poly Key Pressure events (0xAn).<br>bit 1: set if port passed any MIDI Note On events (0x9n).<br>bit 0: set if port passed any MIDI Note Off events (0x8n). |
| 0x02 | MIDI system message event flags #1 (1 byte):<br>bit 7: always 0.<br>bit 6: set if port passed any MIDI Song Select events (0xF3).<br>bit 5: set if port passed any MIDI Song Position Pointer events (0xF2).<br>bit 4: set if port passed any MIDI Time Code Quarter Frame events (0xF1).<br>bit 3: set if port passed any MIDI Stop events (0xFC).<br>bit 2: set if port passed any MIDI Continue events (0xFB).<br>bit 1: set if port passed any MIDI Start events (0xFA).<br>bit 0: set if port passed any MIDI Timing Clock events (0xF8). |
| 0x03 | MIDI system message event flags #2 (1 byte):<br>bits 7 - 4: always 0.<br>bit 3: set if port passed any MIDI System Exclusive events (0xF0).<br>bit 2: set if port passed any MIDI Reset events (0xFF).<br>bit 1: set if port passed any MIDI Active Sensing events (0xFE).<br>bit 0: set if port passed any MIDI Tune Request events (0xF6). |

**Parameters for DIN Ports:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

**Parameters for USB Device Ports:**

| ID | Name | Description |
|---|---|---|
| 0x1E | USBDPortNameIn | USBD name for MIDI IN port (MIDI data sent from host to device, 7-bit ASCII string). Maximum length is given by MIDIInfo:USBDPortNameMax. |
| 0x1F | USBDPortNameOut | USBD name for MIDI OUT port (MIDI data sent from device to host, 7-bit ASCII string). Maximum length is given by MIDIInfo:USBDPortNameMax. |

**Parameters for USB Host Controller Ports:**

These parameters provide information about USB-MIDI devices on a USB host controller port that are connected to a MIDI port. To find all USB-MIDI devices connected to a host comtroller port, regardless of whether or not they are connected to a MIDI port, use the parameters in the HardwareInfo section for USB host controller MIDI ports.

The only parameter that is read/write is USBHReserve, all other parameters are read-only. Use the MIDIOperation:USBHMIDIReserve command to set and clear USB host reservations.

| ID | Name | Description |
|---|---|---|
| 0x20 | USBHVID | Hosted device’s USB vendor ID (16-bit value encoded in 3 bytes, 16x3 format). 0 if no device is hosted. |
| 0x21 | USBHPID | Hosted device’s USB product ID (16-bit value encoded in 3 bytes, 16x3 format). 0 if no device is hosted. |
| 0x22 | USBHVName | Hosted device’s vendor name from USB descriptor (7-bit ASCII string). |
| 0x23 | USBHPName | Hosted device’s product name from USB descriptor (7-bit ASCII string). |
| 0x24 | USBHSerialNum | Hosted device’s serial number from USB descriptor (7-bit ASCII string).<br>Most USB-MIDI devices do not have a serial number in the USB descriptor. |
| 0x25 | USBHPort<br>CountIn | Number of MIDI IN ports supported by the hosted device (1 – 16) (1 byte). 0 if no device is hosted. |
| 0x26 | USBHPort<br>CountOut | Number of MIDI OUT ports supported by the hosted device (1 – 16) (1 byte). 0 if no device is hosted. |
| 0x27 | USBHIdentifier | A unique ID assigned to each hosted device (1 – N) (1 byte). This can be used to distinguish between multiple devices that have the same USBHVID and USBHPID with a blank USBHSerialNum. 0 if no device is hosted. |
| 0x28 | USBHPortNum | MIDI port number on the hosted device (1 – 16) (1 byte). 0 if no device is hosted. |
| 0x29 | USBHReserve | Indicates if the port is reserved for a specific USB-MIDI device (1 byte).<br>Devices are reserved using USBHVIDR, USBHPIDR, USBHPortNumR, and (optionally) USBHSerialNumR.<br>0 = port is not reserved<br>1 = port is reserved |
| 0x2A | USBHVIDR | Reserved device’s USB vendor ID (16-bit value encoded in 3 bytes, 16x3 format). This parameter is used when reserving a device. |
| 0x2B | USBHPIDR | Reserved device’s USB vendor ID (16-bit value encoded in 3 bytes, 16x3 format). This parameter is used when reserving a device. |
| 0x2C | USBHVNameR | Reserved device’s vendor name from USB descriptor (7-bit ASCII string).<br>Value should always be copied from USBHVName. This parameter is information only and is not used for reserving a device. |
| 0x2D | USBHPNameR | Reserved device’s product name from USB descriptor (7-bit ASCII string).<br>Value should always be copied from USBHPName. This parameter is information only and is not used for reserving a device. |
| 0x2E | USBHSerialNumR | Reserved device’s serial number from USB descriptor (7-bit ASCII string).<br>Most USB-MIDI devices do not have a serial number in the USB descriptor.<br>Set this parameter to an empty string to not use serial number when searching for reserved devices. This parameter is used when reserving a device. |
| 0x2F | USBHPortNumR | MIDI port number on the reserved device (1 – 16) (1 byte). This parameter is used when reserving a device. |
| 0x3C | USBHPortNameIn | Hosted device’s MIDI IN port name from USB descriptor (7-bit ASCII string) for the selected MIDI port number (USBHPortNum). Devices that do not support names for MIDI ports in their USB device descriptor will return an empty string. |
| 0x3D | USBHPortNameOut | Hosted device’s MIDI OUT port name from USB descriptor (7-bit ASCII string) for the selected MIDI port number (USBHPortNum). Devices that do not support names for MIDI ports in their USB device descriptor will return an empty string. |
| 0x3E | USBHPortNameInR | Reserved device’s MIDI IN port name from USB descriptor (7-bit ASCII string). Value should always be copied from USBHPortNameIn. This parameter is information only and is not used for reserving a device. |
| 0x3F | USBHPortNameOutR | Reserved device’s MIDI OUT port name from USB descriptor (7-bit ASCII string). Value should always be copied from USBHPortNameOut. This parameter is information only and is not used for reserving a device. |

**Parameters for Ethernet Ports:**

| ID | Name | Description |
|---|---|---|
| 0x30 | EthSesnFlags | Session configuration flags (1 byte):<br>bits 7 - 4: reserved (always 0).<br>bit 3: timestamp mode: 0 = use timestamps on incoming MIDI events,<br>1 = ignore timestamps on incoming MIDI events<br>bit 2: initiator mode: 0 = use IP address & port number (EthIPAddressR & EthPortNumberR), 1 = use session name (EthSesnNameR).<br>bit 1: role: 0 = responder, 1 = initiator.<br>bit 0: enable: 0 = disabled, 1 = enabled. |
| 0x31 | EthPortNumber | Port number used for this session (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x32 | EthSesnName | Session name (Bonjour name) to use for this session on the network. Actual name used on the network (EthSesnNameN) may be different following Bonjour name resolution. Maximum length is given by MIDIInfo:EthSesnNameMax. |
| 0x33 | EthSesnNameN | Actual session name (Bonjour name) for this session on the network (7-bit ASCII string). Only valid if the Ethernet port is connected to a network, is active, and has finished Bonjour name resolution. |
| 0x34 | EthIPAddressX | IP address of the device on the other side of this connection (four-byte big- endian value encoded in five-bytes, 32x5 format). Only valid if the session has an active connection. Example:<br>0x0C, 0x05, 0x20, 0x02, 0x64<br>// IP address (192.168.1.100) |
| 0x35 | EthPortNumberX | Port number for the device on the other side of this connection (16-bit value encoded in 3 bytes, 16x3 format). Only valid if the session has an active connection. |
| 0x36 | EthSesnNameX | Session name (Bonjour name) for the device on the other side of this connection (7-bit ASCII string). Only valid if the session has an active connection. |
| 0x37 | EthIPAddressR | IP address to use when initiating a connection (four-byte big-endian value encoded in five-bytes, 32x5 format). Only used when EthSesnFlags role = 1 and initiator mode = 0. Example:<br>0x0C, 0x05, 0x20, 0x02, 0x64 // IP address (192.168.1.100) |
| 0x38 | EthPortNumberR | Port number to use when initiating a connection (16-bit value encoded in 3 bytes, 16x3 format). Only used when EthSesnFlags role = 1 and initiator mode = 0. |
| 0x39 | EthSesnNameR | Session name (Bonjour name) to use when initiating a connection (7-bit ASCII string). Only used when EthSesnFlags role = 1 and initiator mode = 1.<br>Maximum length is given by MIDIInfo:EthSesnNameMax. |

**Parameters for Automation Control Ports:**

| ID | Name | Description |
|---|---|---|
| 0x40 | PresetSelector | Program selector parameter block for presets. See notes below. |
| 0x41 | SceneSelector | Program selector parameter block for scenes. See notes below. |
| 0x42 | MixerSnapshotSelector | Program selector parameter block for mixer snapshots. See notes below. |
| 0x43 | FailoverArmControl | Automation control parameter block for arming failover. See notes below. |
| 0x44 | FailoverDisarmControl | Automation control parameter block for disarming failover. See notes below. |
| 0x45 | FailoverClearAlarmControl | Automation control parameter block for clearing failover alarm. See notes below. |
| 0x46 | PanicControl | Automation control parameter block for sending Panic message. See notes below. |
| 0x47 | MIDIGob<br>Control | Automation control parameter block for sending MIDI-Gobs. This parameter requires the MIDIGobID ArgID (1 – MIDIInfo:MIDIGobMax). See notes below. |
| 0x48 | HardwareOutputControl | Automation control parameter block for hardware control outputs (not used by inputs). This parameter requires the HWPortID ArgID (1 – DeviceInfo:CtrlPortCount). See notes below. |
| 0x49 | KybdRptGrpControl | Automation control parameter block for sending keyboard report groups. This parameter requires the KybdRptGrpID ArgID (1 – DeviceInfo:KybdRptGrpMax). See notes below. |
| 0x4A | AudioGlobalMuteControl | Automation control parameter block for audio global mute. See notes below. |

See AudioAutomationInfo data class for additional information on using the Automation Control Port with audio controls.

**Program Selector Parameter Block:**

Automation control ports can be used to change presets, scenes, and mixer snapshots using MIDI events. The preset, scene, and mixer snapshot selectors can be configured to support either MIDI Program Change (0xCn) or MIDI Control Change (0xBn) events. Both Program Change and Control Change events support selecting a preset/scene/snapshot using any of the 16 MIDI channels. When using Program Change events, byte #2 of the MIDI event is used to select the preset/scene/snapshot number. When using Control Change events, byte #3 of the MIDI event is used to select the preset/scene/snapshot number. For Control Change events, the CC mode sub-id selects from several different types of behaviour as discussed below.

For Continuous mode, the values between _Minimum CC Value_ and _Maximum CC Value_ are equally distributed among the subset of presets/scenes/snapshots so that the _Minimum CC Value_ selects the lowest numbered preset/scene/snapshot of the subset and the _Maximum CC Value_ selects the highest numbered preset/scene/snapshot of the subset. An invert flag can be used to reverse the logic. This mode works well with both fader/volume controls and pushbutton controls.

For Increment/Decrement mode, CCID1 is used to advance upwards through the subset of presets/scenes/snapshots and CCID2 is used to advance downwards through the subset of presets/scenes/snapshots. A wraparound flag can be used to loop through the subset of presets/scenes/snapshots. This mode works with pushbutton controls.

For Toggle mode, two presets/scenes/snapshots are selected for the OFF and ON state of a pushbutton. The pushbutton is used to toggle between the two presets/scenes/snapshots. This mode works best with pushbutton controls but also works with fader/volume controls.

| Sub-ID | Description |
|---|---|
| 0x01 | Selector configuration flags (1 byte):<br>bits 7 - 4: reserved (always 0).<br>bit 3: set if selector should invert the controller values when in Continuous mode.<br>bit 2: set if selector should allow preset/scene/snapshot selection to wraparound when in<br>Increment/Decrement mode.<br>bit 1: set if selector should use MIDI Control Change event (0xBn), clear if selector should use MIDI Program Change event (0xCn).<br>bit 0: set if selector is enabled, clear if selector is disabled. |
| 0x02 | MIDI channel number bitmask indicating which MIDI channels to use for selector (0x0001 = MIDI channel 1, 0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x03 | Minimum Program Number (1 – N) (1 byte). Used to select a subset of all presets/scenes. Value should be less than Maxiumum Program Number. |
| 0x04 | Maximum Program Number (1 – N) (1 byte). Used to select a subset of all presets/scenes/snapshots. Value should be greater than Miniumum Program Number. |
| 0x05 | CC Mode (1 byte). Sets the operating mode when using MIDI Control Change events.<br>1 = Continuous mode, use fader or volume control to cycle through presets/scenes/snapshots, or use pushbuttons to select specific presets/scenes/snapshots.<br>2 = Increment/Decrement mode, use two pushbuttons to cycle through presets/scenes/snapshots.<br>3 = Toggle mode, use one pushbutton to toggle between two presets/scenes/snapshots (OFF and ON). |
| 0x06 | CC ID 1 (0 – 127) (1 byte). The controller ID to use for Continuous and Toggle modes and the Increment controller ID for Increment/Decrement mode. |
| 0x07 | CC ID 2 (0 – 127) (1 byte). The Decrement controller ID for Increment/Decrement mode. |
| 0x08 | Minimum CC Value (0 – 127) (1 byte). The minimum CC value to be used when in Continuous mode. Values less than this are ignored. Value should be less than Maximum CC Value. |
| 0x09 | Maximum CC Value (0 – 127) (1 byte). The maximum CC value to be used when in Continuous mode. Values greater than this are ignored. Value should be greater than Minimum CC Value. |
| 0x0A | On CC Value (0 – 127) (1 byte). Minimum controller value to be used as ON value when in Increment/Decrement or Toggle modes. Values less than this are considered OFF, values greater or equal to this are considered ON. |
| 0x0B | OFF Program Number (1 – N) (1 byte). Sets the preset/scene/snapshot to use when selector is OFF in Toggle mode. |
| 0x0C | ON Program Number (1 – N) (1 byte). Sets the preset/scene/snapshot to use when selector is ON in Toggle mode. |

**Automation Control Parameter Block:**

Automation control ports can be used to arm/disarm failover, clear failover alarms, send Panic message, send MIDI-Gobs, activate hardware control outputs, send keyboard reports and enable/disable audio global mute using MIDI events. These automation controls use MIDI Control Change (0xBn) events and work best with pushbutton controls but also works with fader/volume controls.

| Sub-ID | Description |
|---|---|
| 0x01 | Automation control configuration flags (1 byte):<br>bits 7 - 4: reserved (always 0).<br>bit 3: set if using a momentary switch to toggle between ON and OFF states (automation control changes state every time the MIDI value goes to ON state).<br>bit 2: set if MIDI value must change between ON and OFF states to trigger the automation control (makes a fader/volume control act like a pushbutton control).<br>bit 1: set if MIDI values should be inverted so that 127 is lowest value and 0 is highest value.<br>bit 0: set if automation control is enabled, clear if automation control is disabled. |
| 0x02 | MIDI channel number bitmask indicating which MIDI channels to use for automation control (0x0001 = MIDI channel 1, 0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x03 | MIDI continuous controller number to use for automation control (0 – 127) (1 byte). |
| 0x04 | CC value to use for ON state (0 – 127) (1 byte). Values at this level and above are treated as ON state. Values below this level are treated as OFF state. Use the invert bit in configuration flags sub-ID to reverse the logic. |

### MIDIFeature (Data Class = 0x07)

This data class contains parameters for advanced MIDI features. All parameters in this data class require an argument value block.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef<br>RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is required. All parameters are global and do not support scenes. |
| RetParmVal<br>NotParmVal |  |

#### Advanced MIDI Processor (AMP)

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x07 | AMPID | AMP ID (1 – N). This specifies the algorithm ID, the operator ID, the custom route map ID, or the lookup table ID (depends on the parameter ID). |

**Parameters for AMP Algorithms:**

AMP algorithm parameters require an AMPID argument value block to specify the algorithm ID.

| ID | Name | Description |
|---|---|---|
| 0x01 | AMPAlgName | AMP algorithm name (7-bit ASCII string). |

**Parameters for AMP Operators:**

AMP operator parameters require an AMPID argument value block to specify the operator ID.

AMP operators have one input pin and three output pins (Y, N, M). The Y and N output pins are from the output of the Match function. The M output pin is from the output of the Modify function. The Match function input is always connected to the input pin. The Modify function input can be connected to the Y or N output pins. Operators are assigned to algorithms, up to a maximum of N operators per algorithm (see MIDIInfo:AMPOPAMax).

If the Match function uses 2x8 mode (see Match Header Flags in AMPOpMatchHeaderBlock) then two instances of the Match function 2x8 block are used: one for byte 2 in the MIDI event (AMPOpMatch2x8b2), and another for byte 3 in the MIDI event (AMPOpMatch2x8b3). If the Match function uses 1x16 mode then one instance of the Match function 1x16 block is used (AMPOpMatch1x16).

If the Modify function uses 2x8 mode (see Modify Header Flags in AMPOpModifyHeaderBlock) then two instances of the Modify function 2x8 block are used: one for byte 2 in the MIDI event (AMPOpModify2x8b2), and another for byte 3 in the MIDI event (AMPOpModify2x8b3). If the Modify function uses 1x16 mode then one instance of the Modify function 1x16 block is used (AMPOpModify1x16).

| ID | Name | Description |
|---|---|---|
| 0x03 | AMPOp<br>Connection | Operator connection block. See notes below. |
| 0x04 | AMPOpMatchHeader | Operator match function header block. See notes below. |
| 0x05 | AMPOpMatch2x8b2 | Operator match function 2x8 block for byte 2. See notes below. |
| 0x06 | AMPOpMatch2x8b3 | Operator match function 2x8 block for byte 3. See notes below. |
| 0x07 | AMPOpMatch1x16 | Operator match function 1x16 block. See notes below. |
| 0x08 | AMPOpModifyHeaderBlock | Operator modify function header block. See notes below. |
| 0x09 | AMPOpModify2x8b2 | Operator modify function 2x8 block for byte 2. See notes below. |
| 0x0A | AMPOpModify2x8b3 | Operator modify function 2x8 block for byte 3. See notes below. |
| 0x0B | AMPOpModify1x16 | Operator modify function 1x16 block. See notes below. |

**Operator Connection Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs SetParmVal messages can include any selection of sub-IDs.

| Sub-ID | Description |
|---|---|
| 0x01 | Algorithm ID (1 – N) that this operator is assigned to (1 byte). Use 0 if this operator is not assigned to any algorithm. |
| 0x02 | Connection Flags (1 byte):<br>bits 7 - 2: always zero<br>bit 1: set if the N output pin is connected to the Modify function input.<br>bit 0: set if the Y output pin is connected to the Modify function input. |
| 0x03 | Input pin connection for this operator (1 byte):<br>0: input pin is not connected to anything.<br>1 – N: input pin connects to operator output with this ID.<br>0x7F: input pin connects to algorithm Entry point. |
| 0x04 | Pin ID of the operator output that connects to the input of this operator (only used if sub-ID 0x03 is not 0 or 0x7F) (1 byte):<br>0: no operator output pin is connected to the input of this operator.<br>1: the Y output pin of an operator is connected to the input of this operator.<br>2: the N output pin of an operator is connected to the input of this operator.<br>3: the M output pin of an operator is connected to the input of this operator. |
| 0x05 | MIDI channel to use for incoming MIDI event when the M output of another operator is connected to the input of this operator (1 byte). Only used if sub-ID 0x03 is a valid operator ID and sub-ID<br>0x04 has the value 3.<br>0: use the same MIDI channel as the original event.<br>1: MIDI channel 1.<br>2: MIDI channel 2.<br>…<br>16: MIDI channel 16. |
| 0x06 | Output Order (1 byte). Sets the order of MIDI events from Y, N, and M outputs to algorithm Exit point.<br>0: YNM (Y output first, N output second, M output third).<br>1: YMN (Y output first, M output second, N output third).<br>2: NYM (N output first, Y output second, M output third).<br>3: NMY (N output first, M output second, Y output third).<br>4: MYN (M output first, Y output second, N output third).<br>5: MNY (M output first, N output second, Y output third). |
| 0x07 | Y output pin connection of this operator to algorithm Exit point (1 byte):<br>0: Y output pin does not connect to algorithm Exit point.<br>1-N: Y output pin connects to algorithm Exit point via custom route map with this ID.<br>0x7F: Y output pin connects to algorithm Exit point via default route map. |
| 0x08 | N output pin connection of this operator to algorithm Exit point (1 byte):<br>0: N output pin does not connect to algorithm Exit point.<br>1-N: N output pin connects to algorithm Exit point via custom route map with this ID.<br>0x7F: N output pin connects to algorithm Exit point via default route map. |
| 0x09 | M output pin connection of this operator to algorithm Exit point (1 byte):<br>0: M output pin does not connect to algorithm Exit point.<br>1-N: M output pin connects to algorithm Exit point via custom route map with this ID.<br>0x7F: M output pin connects to algorithm Exit point via default route map. |

**Operator Match Function Header Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs SetParmVal messages can include any selection of sub-IDs.

| Sub-ID | Description |
|---|---|
| 0x01 | Match Header Flags (1 byte):<br>bits 7 - 1: always zero<br>bit 0: set if Match function uses 1x16 mode, clear if Match function uses 2x8 mode. |
| 0x02 | Match Event Type (1 byte). Bitmask indicating which MIDI event types to match. More than 1 bit can be set:<br>bit 7: always 0<br>bit 6: Pitch Bend (0xEn, 3-byte MIDI event)<br>bit 5: Mono Aftertouch (0xDn, 2-byte MIDI event)<br>bit 4: Program Change (0xCn, 2-byte MIDI event)<br>bit 3: Control Change (0xBn, 3-byte MIDI event)<br>bit 2: Poly Aftertouch (0xAn, 3-byte MIDI event)<br>bit 1: Note On (0x9n, 3-byte MIDI event)<br>bit 0: Note Off (0x8n, 3-byte MIDI event) |
| 0x03 | Match Channel: Bitmask indicating which MIDI channels to match (0x0001 = MIDI channel 1,<br>0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). |

**Operator Match Function 2x8 Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs SetParmVal messages can include any selection of sub-IDs. If Match 2x8 Flags (sub-ID 0x01) is included, it should always be the first sub-ID in the message. This parameter is only used if Match Header Flags bit 0 is clear.

| Sub-ID | Description |
|---|---|
| 0x01 | Match 2x8 Flags (1 byte):<br>bits 7 - 2: always zero<br>bit 1: set if a lookup table is used, clear if minimum value and maximum value are used.<br>bit 0: set if Match function logic is inverted. |
| 0x02 | Lookup table ID (1 – N) (1 byte). Only used if bit 1 is set in Match 2x8 Flags. Set to 0 if unused. |
| 0x03 | Lookup table value that is used for matching (0x00 – 0x7F) (1 byte). Only used if bit 1 is set in Match 2x8 Flags. |
| 0x04 | Minimum value that is used for matching (0x00 – 0x7F) (1 byte). Must be less than or equal to the maximum value (sub-ID 0x05). Only used if bit 1 is clear in Match 2x8 Flags. |
| 0x05 | Maximum value that is used for matching (0x00 – 0x7F) (1 byte). Must be greater than or equal to the minimum value (sub-ID 0x04). Only used if bit 1 is clear in Match 2x8 Flags. |

**Operator Match Function 1x16 Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs SetParmVal messages can include any selection of sub-IDs. If Match 1x16 Flags (sub-ID 0x01) is included, it should always be the first sub-ID in the message. This parameter is only used if Match Header Flags bit 0 is set.

| Sub-ID | Description |
|---|---|
| 0x01 | Match 1x16 Flags (1 byte):<br>bits 7 - 3: always zero<br>bit 2: set if byte-3 (MSB) in MIDI event is used for lookup table offset, clear if byte-2 (LSB) in<br>MIDI event is used for lookup table offset.<br>bit 1: set if a lookup table is used, clear if minimum value and maximum value are used.<br>bit 0: set if Match function logic is inverted. |
| 0x02 | Lookup table ID (1 – N) (1 byte). Only used if bit 1 is set in Match 1x16 Flags. Set to 0 if unused. |
| 0x03 | Lookup table value that is used for matching (0x00 – 0x7F) (1 byte). Only used if bit 1 is set in Match 1x16 Flags. |
| 0x04 | Minimum value that is used for matching (0x0000 – 0x3FFF, 14x2 format, 2 bytes). Must be less than or equal to the maximum value (sub-ID 0x05). Only used if bit 1 is clear in Match 1x16 Flags. |
| 0x05 | Maximum value that is used for matching (0x0000 – 0x3FFF, 14x2 format, 2 bytes). Must be greater than or equal to the minimum value (sub-ID 0x04). Only used if bit 1 is clear in Match 1x16 Flags. |

**Operator Modify Function Header Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs SetParmVal messages can include any selection of sub-IDs.

| Sub-ID | Description |
|---|---|
| 0x01 | Modify Header Flags (1 byte):<br>bits 7 - 1: always zero<br>bit 0: set if Modify function uses 1x16 mode, clear if Modify function uses 2x8 mode. |
| 0x02 | Modify In Event Type (1 byte). Bitmask indicating which MIDI event types to modify. More than 1 bit can be set:<br>bit 7: always 0<br>bit 6: Pitch Bend (0xEn, 3-byte MIDI event)<br>bit 5: Mono Aftertouch (0xDn, 2-byte MIDI event)<br>bit 4: Program Change (0xCn, 2-byte MIDI event)<br>bit 3: Control Change (0xBn, 3-byte MIDI event)<br>bit 2: Poly Aftertouch (0xAn, 3-byte MIDI event)<br>bit 1: Note On (0x9n, 3-byte MIDI event)<br>bit 0: Note Off (0x8n, 3-byte MIDI event) |
| 0x03 | Modify In Channel: Bitmask indicating which MIDI channels to modify (0x0001 = MIDI channel 1,<br>0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x04 | Modify Out Event Type (1 byte). Sets the output MIDI event type. Only 1 bit can be set. Use 0x00 to select the same event type as the input MIDI event.<br>bit 7: always 0<br>bit 6: Pitch Bend (0xEn, 3-byte MIDI event)<br>bit 5: Mono Aftertouch (0xDn, 2-byte MIDI event)<br>bit 4: Program Change (0xCn, 2-byte MIDI event)<br>bit 3: Control Change (0xBn, 3-byte MIDI event)<br>bit 2: Poly Aftertouch (0xAn, 3-byte MIDI event)<br>bit 1: Note On (0x9n, 3-byte MIDI event)<br>bit 0: Note Off (0x8n, 3-byte MIDI event) |
| 0x05 | Modify Out Channel: Bitmask indicating which MIDI channels to use for outgoing MIDI events (0x0001 = MIDI channel 1, 0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). Use 0 to select the same channel as the input MIDI event. |

**Operator Modify Function 2x8 Block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs SetParmVal messages can include any selection of sub-IDs. If Modify 2x8 Flags (sub-ID 0x01) is included, it should always be the first sub-ID in the message. This parameter is only used if Modify Header Flags bit 0 is clear.

| Sub-ID | Description |
|---|---|
| 0x01 | Modify 2x8 Flags (1 byte):<br>bits 7 - 3: always zero<br>bit 2: set if Modify Default Value should be used when input is a 2-byte MIDI event.<br>bit 1: set if this output byte (either byte 2 or byte 3) uses input byte 3, clear if this output byte (either byte 2 or byte 3) uses input byte 2.<br>bit 0: set if a lookup table is used. |
| 0x02 | Lookup table ID (1 – N) (1 byte). Only used if bit 0 is set in Modify 2x8 Flags. Set to 0 if not used. |
| 0x03 | Minimum in value that is used by Modify function (0x00 – 0x7F) (1 byte). Must be less than or equal to the maximum in value (sub-ID 0x04). |
| 0x04 | Maximum in value that is used by Modify function (0x00 – 0x7F) (1 byte). Must be greater than or equal to the minimum in value (sub-ID 0x03). |
| 0x05 | Minimum out value that is used by Modify function (0x00 – 0x7F) (1 byte). Can be greater than maximum out value (sub-ID 0x06) to allow for inversion. |
| 0x06 | Maximum out value that is used by Modify function (0x00 – 0x7F) (1 byte). Can be less than minimum out value (sub-ID 0x05) to allow for inversion. |
| 0x07 | Minimum Default Mode (1 byte). Selects the processing behaviour for when the input byte value is less than Minimum In:<br>0: output value is set to input value, lookup table is not used.<br>1: output value is set to input value, lookup table is used if enabled.<br>2: output value is set to Minimum In value, lookup table is used if enabled.<br>3: output value is set to Minimum Default Value (sub-ID 0x08), lookup table is not used. |
| 0x08 | Minimum Default Value (1 byte). Value to use when Minimum Default Mode (sub-ID 0x07) is set to 3 (0x00 – 0x7F). |
| 0x09 | Maximum Default Mode (1 byte). Selects the processing behaviour for when the input byte value is greater than Maximum In:<br>0: output value is set to input value, lookup table is not used.<br>1: output value is set to input value, lookup table is used if enabled.<br>2: output value is set to Maximum In value, lookup table is used if enabled.<br>3: output value is set to Maximum Default Value (sub-ID 0x0A), lookup table is not used. |
| 0x0A | Maximum Default Value (1 byte). Value to use when Maximum Default Mode (sub-ID 0x09) is set to 3 (0x00 – 0x7F). |
| 0x0B | Modify Default Value (1 byte). Value to use when output is a 3-byte MIDI event but input is a 2- byte MIDI event (0x00 – 0x7F). |

**Operator Modify function 1x16 block:**

Each block contains a set of sub-ID & value pairs. RetParmVal messages always include all sub-IDs SetParmVal messages can include any selection of sub-IDs. If Modify 1x16 Flags (sub-ID 0x01) is included, it should always be the first sub-ID in the message. This parameter is only used if Modify Header Flags bit 0 is set.

| Sub-ID | Description |
|---|---|
| 0x01 | Modify 1x16 Flags (1 byte):<br>bit 7: always zero.<br>bit 6: set if MSB of the result is used for byte-2 of output, clear if LSB of the result is used for byte-2 of output (only used when output is 2-byte MIDI event).<br>bit 5: set if byte-2 of input is used for MSB and 0x00 is used for LSB, clear if byte-2 of input is used for LSB and 0x00 is used for MSB (only used when input is 2-byte MIDI event).<br>bit 4: set if using a fixed value for byte-3 (MSB) rather than a lookup table value.<br>bit 3: set if using a fixed value for byte-2 (LSB) rather than a lookup table value.<br>bit 2: set if input byte-3 (MSB) value is used, clear if 0x00 is used for input byte-3.<br>bit 1: set if input byte-2 (LSB) value is used, clear if 0x00 is used for input byte-2.<br>bit 0: set if a lookup table is used. |
| 0x02 | Lookup table ID (1 – N) for byte-2 (LSB) (1 byte). Only used if bit 0 is set and bit 3 is clear in Modify 1x16 Flags. Set to 0 if not used. |
| 0x03 | Lookup table ID (1 – N) for byte-3 (MSB) (1 byte). Only used if bit 0 is set and bit 4 is clear in Modify 1x16 Flags. Set to 0 if not used. |
| 0x04 | Byte-2 (LSB) fixed value to use instead of lookup table value (0x00 – 0x7F) (1 byte). Only used if<br>bit 0 and bit 3 are both set in Modify 1x16 Flags. |
| 0x05 | Byte-3 (MSB) fixed value to use instead of lookup table value (0x00 – 0x7F) (1 byte). Only used if bit 0 and bit 4 are both set in Modify 1x16 Flags. |
| 0x06 | Minimum in value that is used by Modify function (0x0000 – 0x3FFF, 14x2 format, 2 bytes). Must be less than or equal to the maximum in value (sub-ID 0x07). |
| 0x07 | Maximum in value that is used by Modify function (0x0000 – 0x3FFF, 14x2 format, 2 bytes). Must be greater than or equal to the minimum in value (sub-ID 0x06). |
| 0x08 | Minimum out value that is used by Modify function (0x0000 – 0x3FFF, 14x2 format, 2 bytes). Can be greater than maximum out value (sub-ID 0x09) to allow for inversion. |
| 0x09 | Maximum out value that is used by Modify function (0x0000 – 0x3FFF, 14x2 format, 2 bytes).<br>Can be less than minimum out value (sub-ID 0x08) to allow for inversion. |
| 0x0A | Minimum Default Mode (1 byte). Selects the processing behaviour for when the input value is less than minimum in value:<br>0: output value is set to input value, lookup table is not used.<br>1: output value is set to input value, lookup table is used if enabled.<br>2: output value is set to Minimum in value, lookup table is used if enabled.<br>3: output value is set to Minimum Default Value (sub-ID 0x0B), lookup table is not used. |
| 0x0B | Minimum Default Value, value to use when Minimum Default Mode (sub-ID 0x0A) is set to 3 (0x0000 – 0x3FFF, 14x2 format, 2 bytes). |
| 0x0C | Maximum Default Mode. Selects the processing behaviour for when the input value is greater than maximum in value:<br>0: output value is set to input value, lookup table is not used.<br>1: output value is set to input value, lookup table is used if enabled.<br>2: output value is set to Maximum in value, lookup table is used if enabled.<br>3: output value is set to Maximum Default Value (sub-ID 0x0D), lookup table is not used. |
| 0x0D | Maximum Default Value, value to use when Maximum Default Mode (sub-ID 0x0C) is set to 3 (0x0000 – 0x3FFF, 14x2 format, 2 bytes). |

_Example: single operator algorithm, Match and Modify functions both use 2x8 mode_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x01, 0x01                    // message length (129)
0x43, 0x07                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x07                          // ArgID: AMPID (7)
0x09                          // ArgVal: operator ID (9)
0x79, 0x03                    // data block #2: size (121), type (ParmVal)
0x07                          // NumParmValBlock: (7)
                              // ParmVal 1: Operator Connection Block
0x14                          // ParmSize: (20)
0x03                          // ParmID: (3)
0x01, 0x01                    // algorithm ID (1)
0x02, 0x01                    // flags: Y output pin is connected to Modify input
0x03, 0x7F                    // operator input is connected to algorithm Entry point (0x7F)
0x04, 0x00                    // no operator output pin is connected to the input of this operator
0x05, 0x00                    // MIDI channel for incoming event (not used)
0x06, 0x00                    // output order is YNM
0x07, 0x00                    // Y output pin does not connect to Exit point (0)
0x08, 0x04                    // N output pin connects to Exit point via custom route map #4
0x09, 0x7F                    // M output pin connects to Exit point via default route map (0x7F)
                              // ParmVal 2: Match Function Header Block
0x0A                          // ParmSize: (10)
0x04                          // ParmID: (4)
0x01, 0x00                    // flags: use 2x8 mode
0x02, 0x03                    // match event types: note on and note off
0x03, 0x02, 0x01, 0x05        // match channels: 1, 3, 8, 16
                              // ParmVal 3: Match Function 2x8 Block for byte 2
0x0C                          // ParmSize: (12)
0x05                          // ParmID: (5)
0x01, 0x02                    // flags: a lookup table is used
0x02, 0x08                    // lookup table ID (8)
0x03, 0x40                    // lookup table value used for matching (64)
0x04, 0x00                    // minimum value (not used)
0x05, 0x00                    // maximum value (not used)
                              // ParmVal 4: Match Function 2x8 Block for byte 3
0x0C                          // ParmSize: (12)
0x06                          // ParmID: (6)
0x01, 0x01                    // flags: logic is inverted
0x02, 0x00                    // lookup table ID (not used)
0x03, 0x00                    // lookup table value used for matching (not used)
0x04, 0x20                    // minimum value (32)
0x05, 0x80                    // maximum value (96)
                              // ParmVal 5: Modify Function Header Block
0x10                          // ParmSize: (16)
0x08                          // ParmID: (8)
0x01, 0x00                    // flags: use 2x8 mode
0x02, 0x08                    // modify in event types: control change
0x03, 0x03, 0x7F, 0x7F        // modify in channels: all
0x04, 0x00                    // modify out event type: same as input (0)
0x05, 0x00, 0x00, 0x00        // modify out channel: same as input (0)
                              // ParmVal 6: Modify Function 2x8 Block for byte 2
0x18                          // ParmSize: (24)
0x09                          // ParmID: (9)
0x01, 0x01                    // flags: a lookup table is used
0x02, 0x07                    // lookup table ID (7)
0x03, 0x00                    // minimum in value (0)
0x04, 0x7F                    // maximum in value (127)
0x05, 0x00                    // minimum out value (0)
0x06, 0x7F                    // maximum out value (127)
0x07, 0x00                    // minimum default mode (0)
0x08, 0x00                    // minimum default value (not used)
0x09, 0x00                    // maximum default mode (0)
0x0A
0x00,                         // maximum default value (not used)
0x0B, 0x00                    // modify default value (0)
                              // ParmVal 7: Modify Function 2x8 Block for byte 3
0x18                          // ParmSize: (24)
0x0A                          // ParmID: (10)
0x01, 0x02                    // flags: use input byte 3
0x02, 0x00                    // lookup table ID (not used)
0x03, 0x0A                    // minimum in value (10)
0x04, 0x64                    // maximum in value (100)
0x05, 0x00                    // minimum out value (0)
0x06, 0x7F                    // maximum out value (127)
0x07, 0x03                    // minimum default mode (3)
0x08, 0x05                    // minimum default value (5)
0x09, 0x00                    // maximum default mode (2)
0x0A, 0x00                    // maximum default value (not used)
0x0B, 0x00                    // modify default value (0)
0xxx                          // checksum
0xF7                          // footer
```

_Example: multi-operator algorithm, Match and Modify functions both use 1x16 mode_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x69                    // message length (105)
0x43, 0x07                    // message class, data class
0x02                          // NumDataBlock: (2)
0x05, 0x04                    // data block #1: size (5), type (ArgVal)
0x01                          // NumArgValBlock: (1)
0x07                          // ArgID: AMPID (7)
0x09                          // ArgVal: operator ID (9)
0x61, 0x03                    // data block #2: size (97), type (ParmVal)
0x05                          // NumParmValBlock: (5)
                              // ParmVal 1: Operator Connection Block
0x14                          // ParmSize: (20)
0x03                          // ParmID: (3)
0x01, 0x02                    // algorithm ID (2)
0x02, 0x02                    // flags: N output pin is connected to Modify input
0x03, 0x03                    // ID of operator that connects to this input pin of this operator (3)
0x04, 0x03                    // M output pin of operator #3 is connected to the input of this operator
0x05, 0x07                    // MIDI channel to use for MIDI event from operator 3 M output (7)
0x06, 0x05                    // output order is MNY
0x07, 0x00                    // Y output pin does not connect to Exit point (0)
0x08, 0x00                    // N output pin does not connect to Exit point (0)
0x09, 0x00                    // M output pin does not connect to Exit point (0)
                              // ParmVal 2: Match Function Header Block
0x0A                          // ParmSize: (10)
0x04                          // ParmID: (4)
0x01, 0x01                    // flags: use 1x16 mode
0x02, 0x40                    // match event types: pitch bend
0x03, 0x03, 0x7F, 0x7F        // match channels: all
                              // ParmVal 3: Match Function 1x16 Block
0x0E                          // ParmSize: (14)
0x07                          // ParmID: (7)
0x01, 0x01                    // flags: logic is inverted
0x02, 0x00                    // lookup table ID (not used)
0x03, 0x00                    // lookup table value used for matching (not used)
0x04, 0x20, 0x00              // minimum value (0x1000)
0x05, 0x60, 0x00              // maximum value (0x3000)
                              // ParmVal 4: Modify Function Header Block
0x10                          // ParmSize: (16)
0x08                          // ParmID: (8)
0x01, 0x01                    // flags: use 1x16 mode
0x02, 0x40                    // modify in event types: pitch bend
0x03, 0x03, 0x7F, 0x7F        // modify in channels: all
0x04, 0x00                    // modify out event type: same as input (0)
0x05, 0x00, 0x00, 0x20        // modify out channel: 6
                              // ParmVal 5: Modify Function 1x16 Block
0x22                          // ParmSize: (34)
0x0B                          // ParmID: (11)
0x01, 0x01                    // flags: a lookup table is used
0x02, 0x05                    // lookup table ID for LSB (5)
0x03, 0x06                    // lookup table ID for MSB (6)
0x04, 0x00                    // byte-2 fixed value (not used)
0x05, 0x00                    // byte-3 fixed value (not used)
0x06, 0x00, 0x00              // minimum in value (0x0000)
0x07, 0x7F, 0x7F              // maximum in value (0x3FFF)
0x08, 0x60, 0x00              // minimum out value (0x3000)
0x09, 0x20, 0x00              // maximum out value (0x1000)
0x0A, 0x02                    // minimum default mode (2)
0x0B, 0x00, 0x00              // minimum default value (not used)
0x0C, 0x02                    // maximum default mode (2)
0x0D, 0x00, 0x00              // maximum default value (not used)
0xxx                          // checksum
0xF7                          // footer
```

**Parameters for AMP Custom Route Maps:**

AMP custom route map parameters require an AMPID argument value block to specify the custom route map ID.

| ID | Name | Description |
|---|---|---|
| 0x0C | AMPCRMRoute | Bitmap indicating routing for MIDI events entering at the input for this port (BAx2 format). Bit 0 is routing from this port to port #1, bit 1 is routing from this port to port #2, etc. Least significant byte is first, most significant byte is last. The most significant 4 bits of each byte must be 0 which allows 4 ports to be specified per byte. Unused bits should be set to 0. The number of MIDI ports is given in the MIDIInfo:PortCount parameter. The number of bytes (in the bitmap) is an even value so that the bitmap (when unpacked) is a multiple of 8 bits: number of bytes (using INTEGER math) = (((number of MIDI ports - 1) / 8 ) + 1) x 2. |

_Example: device has 20 ports, route MIDI events coming into this port to ports 2, 3, 7, 11-14, 20:_

```text
0x06  // routing for ports 4 through 1 (ports 3 and 2 are selected)
0x04  // routing for ports 8 through 5 (port 7 is selected)
0x0C  // routing for ports 12 through 9 (ports 12 and 11 are selected)
0x03  // routing for ports 16 through 13 (ports 14 and 13 are selected)
0x08  // routing for ports 20 through 17 (port 20 is selected)
0x00  // routing for ports 24 through 21 (padding)
```

**Parameters for AMP Lookup Tables:**

AMP lookup table parameters require an AMPID argument value block to specify the lookup table ID. Lookup tables are always 128 bytes in length which is too long to be sent in a single parameter. Thus, the first 64 bytes of the lookup table are accessed with parameter ID = 0x0D and the last 64 bytes of the lookup table are accessed with parameter ID = 0x0E.

| ID | Name | Description |
|---|---|---|
| 0x0D<br>0x0E | AMPLUTData1<br>AMPLUTData2 | AMP lookup table data (all bytes must be in the range 0x00 – 0x7F).<br>Bytes 1: index (i.e. starting address) (0 – 63).<br>Bytes 2-N: lookup table data (0x00 – 0x7F).<br>The first byte is an index into this portion of the lookup table (i.e. the starting address for this portion). The remaining bytes are the lookup table data.<br>RetParmVal will always return all 64 bytes for this portion of the lookup table (index is always 0, lookup table data is always 64 bytes in length).<br>SetParmVal can use a non-zero value for the offset byte and can write just a portion of the lookup table. |

#### MIDI Panic

There is no MIDI Panic message that can be used to immediately silence stuck notes. There are several MIDI messages that can be strung together to handle most, if not all, stuck note scenarios. The Panic parameters are used configure a bank of MIDI messages into a Panic message that can be sent to any number of MIDI ports on any number of MIDI channels to deal with stuck note scenarios. The Panic message can be sent: in response to an incoming sysex message, manually from the front panel, automatically when changing presets, when failover occurs, using automation controls, and using system events. The MIDIInfo:PanicSupportFlags parameter indicates which mechanisms are available on each product (this sysex message is always available).

See Automation Control MIDI Port for information on using automation controls to send Panic message. See Automatic Failover section for information on sending Panic message when failover occurs. See System Events section for additional methods to send Panic message.

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x10 | PanicEnable | Panic enable flags (1 byte):<br>bits 7 - 2: reserved (always 0).<br>bit 1: set if Panic message is sent automatically when changing presets.<br>bit 0: set if Panic message can be sent manually from front panel.<br>Panic message can also be automatically sent when changing scenes (for devices that support failover). See the Automatic Failover section for more info. Panic message can also be sent using System Events. |
| 0x11 | PanicMessages | Panic message flags (1 byte):<br>bits 7 - 5: reserved (always 0).<br>bit 4: set if Pitch Bend Off message is sent (0xEn 00 40)<br>bit 3: set if All Notes Off message is sent (0xBn 0x7B 0x00).<br>bit 2: set if Reset Controls messages is sent (0xBn 0x79 0x00).<br>bit 1: set if All Sound Off message is sent (0xBn 0x78 0x00).<br>bit 0: set if Sustain Pedal Off message is sent (0xBn 0x40 0x00). |
| 0x12 | PanicChannels | Bitmask indicating which MIDI channels to use for sending messages (0x0001 = MIDI channel 1, 0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x13 | PanicRoute | Bitmap indicating which MIDI ports to route Panic message to (BAx2 format).<br>Each bit represents a MIDI port output. Bit 0 is routing to port #1, bit 1 is routing to port #2, etc. Least significant byte is first, most significant byte is last. The most significant 4 bits of each byte must be 0 which allows 4 ports to be specified per byte. Unused bits should be set to 0. The number of MIDI ports is given in the MIDIInfo:PortCount parameter. The number of bytes (in the bitmap) is an even value so that the bitmap (when unpacked) is a multiple of 8 bits: number of bytes (using INTEGER math) = (((number of MIDI ports - 1) / 8 ) + 1) x 2.<br>Example: device has 20 ports, route Panic message to ports 2, 3, 7, 11-14,<br>20:<br>0x06<br>// routing for ports 4 through 1 (ports 3 and 2 are selected)<br>0x04<br>// routing for ports 8 through 5 (port 7 is selected)<br>0x0C<br>// routing for ports 12 through 9 (ports 12 and 11 are selected)<br>0x03<br>// routing for ports 16 through 13 (ports 14 and 13 are selected)<br>0x08<br>// routing for ports 20 through 17 (port 20 is selected)<br>0x00<br>// routing for ports 24 through 21 (padding) |

#### MIDI-Gob

A MIDI-Gob (group of bytes) is a buffer containing user configurable MIDI messages. Buffers are a fixed size and have a length field that indicates how many bytes in the buffer are used by the MIDI-Gob. MIDIGobs can be linked to create longer buffers. The MIDI messages in a MIDI-Gob can be sent using sysex messages, automation controls, and in response to various system events (e.g. hardware control inputs, failover alarm set, scene change and preset load). MIDI-Gobs have a name field to make them easier to manage when assigning them to system events and automation controls.

##### MIDI-Gob Data Format

Each buffer contains one or more MIDIRoute and MIDIData blocks with optional MIDIChannel blocks. MIDIRoute and MIDIChannel blocks contain information needed by MIDIData blocks and must precede the MIDIData blocks they apply to. MIDIRoute blocks apply to all MIDIData blocks that follow it. MIDIChannel blocks only apply to the first MIDIData block that follows it. For example:

MIDI-Gob #1: MIDIRoute (applies to MIDIData blocks that follow) MIDIData

MIDI-Gob #2: MIDIRoute (applies to both MIDIData-1 and MIDIData-2 blocks) MIDIChannel (applies to MIDIData-1 block only) MIDIData-1 MIDIData-2 MIDI-Gob #3: MIDIRoute-1 (applies to MIDIData-1 block only) MIDIData-1 MIDIRoute-2 (applies to MIDIData-2 block only) MIDIData-2

MIDI-Gobs can be linked to create longer buffers. For example:

MIDI-Gob #1 (linked to MIDI-Gob #2): MIDIRoute-1 MIDIData-1 MIDIRoute-2 MIDIData-2 (beginning) MIDI-Gob #2 (linked to MIDI-Gob #3): MIDIData-2 (continuation)

MIDI-Gob #3: MIDIData-2 (end) MIDIRoute-3 MIDIData-3

The first byte of each block contains a block type ID. The remaining bytes in the block depend on the block type:

| ID | Name | Description |
|---|---|---|
| 0x01 | MIDIRoute | Bitmap indicating which MIDI ports to route MIDI messages to. Each bit represents a MIDI port output. Bit 0 is routing to port #1, bit 1 is routing to port #2, etc. Least significant byte is first, most significant byte is last. Unused bits should be set to 0. The number of MIDI ports is given in the MIDIInfo:PortCount parameter. The number of bytes (in the bitmap) is (using INTEGER math) = (((number of MIDI ports - 1) / 8 ) + 1).<br>Example: device has 20 ports, route MIDI messages to ports 2, 3, 7, 11-14, 20:<br>0x46<br>// routing for ports 8 through 1 (ports 7, 3 and 2 are selected)<br>0x3C<br>// routing for ports 16 through 9 (ports 14 - 11 are selected)<br>0x08<br>// routing for ports 20 through 17 (port 20 is selected)<br>This block is required before any MIDIData blocks and applies to all MIDIData blocks that follow it. |
| 0x02 | MIDIChannel | Bitmask indicating which MIDI channels to use for channel messages (0x0001 = MIDI channel 1, 0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value, 2 bytes, MSB first). This is used to override the channel info in MIDI channel messages so that a single MIDI channel message can be sent multiple times using several MIDI channels. This field is ignored for non-channel messages (i.e. MIDI messages that begin with 0xFn).<br>This block is optional and only applies to the first MIDIData block that follows it. |
| 0x03 | MIDIData | MIDI messages to send to the MIDI ports specified in the preceding MIDIRoute block.<br>Byte 1: number of bytes in the MIDI messages that follow<br>Bytes 2 – N: MIDI messages<br>If a MIDIChannel block precedes this MIDIData block then the channel nibble in each MIDI channel message is not used and the MIDI message will be sent once for each channel specified in the MIDIChannel block. |

_Example: MIDI-Gob data for a device with 20 MIDI ports. A MIDIChannel block precedes the first MIDIData block so the channel nibble (in each channel message) in the first MIDIData block will be replaced with channels from the MIDIChannel block. Two MIDI messages are defined in the first MIDIData block. These two messages are sent on MIDI channels 1, 2, 13-16 to MIDI ports 2, 3, 7, 11-14 and 20 (2 messages x 6 channels x 8 ports = 96 MIDI messages total). One MIDI message is defined in the second MIDIData block and is sent once to MIDI ports 2, 3, 7, 11-14 and 20 (1 message x 8 ports = 8 MIDI messages total)._

```text
0x01,             // MIDIRoute block
0x46
0x3C, 0x08,       // MIDIRoute bitmap (route MIDI messages to ports 2, 3, 7, 11-14, 20)
0x02,             // MIDIChannel block
0xF0, 0x03,       // MIDIChannel bitmap (use MIDI channels 1, 2, 13-16)
0x03,             // MIDIData block #1 (uses MIDIChannel defined above)
0x06,             // number of bytes in the MIDI messages that follow
0xBn, 0x01, 0x00  // MIDI message #1.1: set modulation to 0 (channel nibble not used)
0xBn, 0x07, 0x00  // MIDI message #1.2: set volume to 0 (channel nibble not used)
0x03,             // MIDIData block #2 (does not use MIDIChannel defined above)
0x03,             // number of bytes in the MIDI messages that follow
0xB7, 0x01, 0x40  // MIDI message #2.1: set modulation to 0x40 on MIDI channel 8
```

See Automation Control MIDI Port for information on using automation controls to send MIDI-Gobs.

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x0C | MIDIGobID | MIDI-Gob ID (1 – N). Required for all parameters. |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x14 | GobName | Name to use for MIDI-Gob (7-bit ASCII string). Maximum length is given by MIDIInfo:MIDIGobNameMax. |
| 0x15 | GobData | MIDI data for MIDI-Gob (BAx2 format). Length is always 2 x MIDIInfo:MIDIGobDataMax after BAx2 encoding even if GobLength is less than MIDIInfo:MIDIGobDataMax when reading/writing this parameter. |
| 0x16 | GobLength | Length of valid data in GobData parameter (0 – MIDIInfo:MIDIGobDataMax) (1 byte). MIDI-Gobs can contain up to MIDIInfo:MIDIGobDataMax bytes of data but they may use less than that. |
| 0x17 | GobNext | Pointer to next MIDI-Gob in linked list (1 – MIDIInfo:MIDIGobMax) (1 byte).<br>Use 0 if not using links. |

#### MIDI Route Processor

_This is not yet implemented._

### AudioInfo (Data Class = 0x08)

This data class contains global parameters related to the audio capabilities and configuration of a device. Almost everything is an audio port, the exception being clock sources. See the Audio Port Types table regarding the different types of audio ports.

#### Audio Clock

A product will typically support several clock sources, only one clock source can be selected at any time to be the clock master. The Audio Clock Types table defines the supported clock types.

#### Input Streams & Output Streams (Sources & Destinations)

Rather than using the terms _input_ and _output_, the terms _source_ and _destination_ are used to refer to the two directions of audio streams (abbreviated as SRC and DST respectively). This is done to avoid the usual confusion where an input stream on one device is an output stream on another device (i.e. input vs. output depends on where you are viewing the data stream).

SRC refers to all audio streams that are either received by a device from the outside world or generated by an audio port inside of a device. DST refers to all audio streams that are either sent from a device to the outside world or consumed by an audio port inside of a device.

SRC and DST have the following rules:

1. SRC: A source can be connected to one or more destinations (or none).

2. DST: A destination can be connected to, at most, one source (or none).

#### Audio Channel Type

Each audio port on a device has one or more audio channels, each channel is of a specific audio channel type. The types of audio channels supported by a product can be retrieved using the AudioChannelTypeDetail parameter. It is not safe to assume that all products support the same audio channel types with the same capabilities. Examples of common audio channel types and whether they are SRC or DST are given below:

| Port Type | SRC/DST | Description |
|---|---|---|
| USB Device | SRC | USB audio stream sent from a computer host to a device. This is an output from the computer’s viewpoint but is an input from the device’s viewpoint. |
| USB Device | DST | USB audio stream sent to a computer host from a device. This is an input from the computer’s viewpoint but is an output from the device’s viewpoint. |
| Analog | SRC | Analog audio stream sent from an external source to a mic/line input jack on a device. |
| Analog | DST | Analog audio stream sent to an external sink from a line output jack or a headphone jack on a device. |
| Mixer | DST | Internal audio stream feeding a mixer input on a device. |
| Mixer | SRC | Internal audio stream generated by a mixer output on a device. |
| Tone<br>Generator | SRC | Internal audio stream generated by a tone generator output on a device. |
| AOIP | SRC | Audio over IP stream sent from network to device. This is an input (receive channels) as seen by the network. |
| AOIP | DST | Audio over IP stream sent from device to network. This is an output (transmit channels) as seen by the network. |

#### Current_Value & Restart_Value

Some AudioInfo parameters require that the SaveGlobal or SaveGP command be issued and the device restarted before any changes to that parameter become active. For these parameters, the device supports reading the _current_value_ (displayed in BLUE) and reading/writing the _restart_value_ (displayed in RED). The _current_value_ is always what the device is currently using and may be different than the _restart_value_ which will be used after restarting the device.

For example:

_1. Powerup Device_

```text
Read AudioConfigCurrentValue (0x10): current_value = 1
Read AudioConfigRestartValue (0x40): restart_value = 1
Write AudioConfigRestartValue (0x40): restart_value = 2
Read AudioConfigCurrentValue (0x40): restart_value = 2
Read AudioConfigCurrentValue (0x10): current_value = 1 (value not changed until after SaveGlobal or SaveGP commands are sent and the device is restarted)
```

_2. Issue SaveGlobal or SaveGP Command_

_3. Restart Device_

```text
Read AudioConfigCurrentValue (0x10): current_value = 2
Read AudioConfigRestartValue (0x40): restart_value = 2
```

Changing the audio configuration affects how many audio channels are supported by some audio ports. When changing from a single-speed to a double-speed audio configuration, the number of audio channels will be automatically reduced, as necessary, which may affect audio routings (e.g. audio patchbay assignments). Audio patchbay assighments are not changed when the number of audio channels are automatically reduced; any audio patchbay assignments that refer to unavailable channels will behave as if they are not connected to anything. It is therefore recommended that the number of audio channels be reviewed and changed (if necessary) before issuing the SaveGlobal or SaveGP command and restarting the device. After restarting the device, the audio patchbay settings should then be reviewed and changed (if necessary) in accordance with the new number of audio channels for each audio port.

A similar situation can arise when the device is operating at a double-speed and presets are recalled that were created when the device was operating at a single-speed. The number of audio channels used by the device (as returned by reading the _current_value_) may be less than the number of audio channels set by the user (as returned by reading the _restart_value_). Thus, there may be audio patchbay assignments that refer to unavailable channels and these will behave as if they are unconnected.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef |  |
| RetParmDef |  |
| GetParmVal<br>SetParmVal<br>RetParmVal | All parameters are global and do not support scenes. Most parameters do not require an Argument Value block, the few that do are documented in the Parameters section below. |
| NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | Area | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x13 | AudioChannelType | Audio channel type (1 – N). Only used by AudioChannelTypeDetail parameter. |

**Audio Sample Rates:**

| Value | Description |
|---|---|
| 0x01 | 44,100 Hz |
| 0x02 | 48,000 Hz |
| 0x03 | 88,200 Hz |
| 0x04 | 96,000 Hz |

**Audio Clock Types:**

| Value | Description |
|---|---|
| 0x01 | Internal (device is clock master). |
| 0x02 | USB Device, clock is derived from audio data sent by USB host (USB host is clock master). |
| 0x03 | AOIP, clock is generated by audio over IP system (AOIP module or network is clock master). |

**Audio Port Types:**

| Value | Description |
|---|---|
| 0x01 | USB Device |
| 0x02 | Analog |
| 0x03 | Mixer |
| 0x04 | Tone Generator |
| 0x05 | Audio Over IP |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x01 | PortCount | Total number of audio ports supported by device (1 – N) (1 byte). |
| 0x02 | USBDPortCount | Number of USBD audio ports supported by device (0 – N) (1 byte). |
| 0x03 | AnalogPortCount | Number of analog audio ports supported by device (0 – N) (1 byte). |
| 0x04 | MixerPortCount | Number of mixer audio ports supported by device (0 – N) (1 byte). |
| 0x05 | ToneGeneratorPortCount | Number of mixer tone generator ports supported by device (0 – N) (1 byte). |
| 0x06 | AOIPPortCount | Number of network audio ports supported by device (0 – N) (1 byte). |
| 0x07 | AudioPortNameMax | Maximum length allowed for audio port name (1 byte). |
| 0x08 | AudioChannelNameMax | Maximum length allowed for audio channel name (1 byte). |
| 0x09 | USBDChannel<br>NameMax | Maximum length allowed for USB device audio channel name (1 byte). |
| 0x0A | AudioConfigCount | The number of audio configurations supported by device (1 – N) (1 byte). |
| 0x0B | AudioConfigSupport | The audio configurations supported by device. Each audio configuration structure consists of two bytes:<br>Byte 1: configuration ID (1 – N)<br>Byte 2: audio sample rate (1 – N), value from Audio Sample Rates table<br>The number of structures is given by AudioInfo:AudioConfigCount. For example, a product that supports four sample rates will return four two-byte audio configuration structures:<br>0x01, 0x01: configuration #1 is 44,100 Hz<br>0x02, 0x02: configuration #2 is 48,000 Hz<br>0x03, 0x03: configuration #3 is 88,200 Hz<br>0x04, 0x04: configuration #4 is 96,000 Hz |
| 0x0C | AudioClockCount | The number of audio clocks supported by device (1 – N) (1 byte). |
| 0x0D | AudioClockSupport | The audio clock sources supported by device. Each audio clock source structure consists of three bytes:<br>Byte 1: clock ID (1 – N)<br>Byte 2: clock type (1 – N), value from Audio Clock Types table<br>Byte 3: instance number (1 – N).<br>The number of structures is given by AudioInfo:AudioClockCount. For example, a product that has one internal clock and two USB device ports (each of which can use the connected USB host as a clock master) will return three three-byte audio clock source structures:<br>0x01, 0x01, 0x01: clock #1 is internal #1<br>0x02, 0x02, 0x01: clock #2 is USB device #1<br>0x03, 0x02, 0x02: clock #3 is USB device #2<br>0x04, 0x03, 0x01: clock #4 is AOIP #1 |
| 0x0E | AudioChannelTypeCount | The number of audio channel types supported by device (1 – N) (1 byte). To be used with AudioChannelTypeDetail parameter below. |
| 0x0F | AudioChannelTypeDetail | Parameter block containing details about a specific audio channel type that is supported by the device. This parameter requires the AudioChannelTypeID ArgID to be set to a value from 1 to AudioInfo:AudioChannelTypeCount. See AudioChannelTypeDetail parameter block below. |
| 0x10 | AudioConfigCurrentValue | The audio configuration that is currently active for the device (1 – AudioInfo:AudioConfigCount) (1 byte). Read-only and may not be the same as AudioInfo:AudioConfigRestartValue. Only supports Area 0 (work area). |
| 0x11 | AudioClockCurrentValue | The audio clock source that is currently active for the device (1 – AudioInfo:AudioClockCount) (1 byte). Read-only and may not be the same as AudioInfo:AudioClockRestartValue. Only supports Area 0 (work area). |
| 0x12 | AudioFrameWaitCurrentValue | The number of audio frame buffers that audio system currently uses (1 byte). Read-only and may not be the same as AudioInfo:AudioFrameWaitRestartValue. Only supports Area 0 (work area). |
| 0x13 | AudioMixerSnapshotMax | The number of mixer snapshots supported by device (1 – N) (1 byte). |
| 0x14 | AudioAutomationControlMax | The number of automation controls for audio (ACA) supported by device (0 – N) (1 byte). |
| 0x15 | AudioChannelMax | The maximum number of audio channels the device supports on a single audio stream (1 – N) (1 byte). |
| 0x16 | AudioFrameWaitSupport | The range of values supported for audio frame rate (2 bytes). Used by AudioFrameWaitCurrentValue and AudioFrameWaitRestartValue parameters.<br>Byte 1: minimum frame wait value<br>Byte 2: maximum frame wait value |
| 0x40 | AudioConfigRestartValue | The selected audio configuration for the device (1 – AudioInfo:AudioConfigCount) (1 byte). Changing the audio configuration requires rebooting the device before the new value becomes active. Read- write and may not be same as AudioInfo:AudioConfigCurrentValue. |
| 0x41 | AudioClockRestartValue | The selected audio clock source for the device (1 – AudioInfo:AudioClockCount) (1 byte). Changing the audio clock source requires rebooting the device before the new value becomes active. Read- write and may not be same as AudioInfo:AudioClockCurrentValue. |
| 0x42 | AudioFrameWaitRestartValue | The selected number of audio frame buffers for the device (1 byte).<br>Changing this parameter requires rebooting the device before the new value becomes active. Read-write and may not be same as AudioInfo:AudioFrameWaitCurrentValue. |
| 0x43 | AudioGlobalMute | Audio global mute state (1 byte):<br>0 = off/inactive (read/write)<br>1 = on/active (read/write)<br>2 = toggle the current state (write only) |

**AudioChannelTypeDetail Parameter Block:**

This block comprises a set of sub-ID & value pairs that define the controls for an audio channel type that is supported by the device. Not all sub-IDs are used with every audio channel type. Length depends on the capabilities of the audio channel type. Use the AudioChannelInfo:ChannelControlValue parameter to read and write control values.

Many controls comprise a simple on/off switch that uses a single byte ON/Enabled (0x01) or OFF/Disabled (0x00) value.

Volume, Solo Volume and Dim Level minimum/maximum controls use 16 bit 2’s complement numbers (8.8 value encoded in 3 bytes, 16x3 format) that can range from +127.9961 dB (0x7FFF) down to - 127.9961 dB (0x8001) in steps of 1/256 dB or 0.00390625 dB (0x0001). Volume, Solo Volume and Dim Level resolution can only have positive values and range from 1/256 dB (0x0001) to +127.9961 dB (0x7FFF). In addition, value 0x8000, representing silence (i.e. -∞ dB), may be implemented in some controls. However, 0x8000 is never reported as the minimum value.

Pad control also uses 16 bit 2’s complement numbers (8.8 value encoded in 3 bytes, 16x3 format) with the same dB range as Volume controls but only for the pad value; the pad is enabled or disabled with a single byte ON/Enabled (0x01) or OFF/Disabled (0x00) value and may apply to Mic inputs and/or Line inputs.

Pan controls use 16 bit 2’s complement numbers (encoded in 3 bytes, 16x3 format) that can range from +32767 (0x7FFF) down to - 32767 (0x8001). Positive numbers are panning to the right. Negative numbers are panning to the left. 0 means fully centered. The maximum pan value representing a signal panned fully right is specified in the control definition. The minimum pan value representing a signal panned fully left is the negative of this value. For example, if the maxium pan value is 0x007F (+127) then the minimum pan value is 0xFF81 (-127).

Frequency controls use unsigned 28-bit numbers (16.12 value encoded in 4 bytes, 28x4 format) that can range from 65,535.9997558594 Hz (0xFFFFFFF) down to 0 Hz (0x0000000) in steps of 1/4096 Hz or 0.000244140625 Hz (0x0000001).

Audio channels can be linked to form a stereo pair (left channel and right channel). Stereo pairs can only be created from adjacent audio channels on the same audio port and of the same direction (SRC/DST). A stereo pair must have the left channel be an odd-numbered channel and the right channel be an even-numbered channel (e.g. channel 1&2 or 7&8 can be stereo linked but not channels 2&3 or 6&7). When linked, stereo pairs share many, but not necessarily all, controls. For shared controls, only the left channel control is used by the device, any changes to the right channel controls are ignored by the device. For controls that do not support stereo-linking, both left and right controls are used to make changes to the respective audio channels on the device.

#### Pan Controls & Balance Controls

Pan controls, when available, should only be used when it makes sense to do so. Pan controls are only used with analog and mixer audio ports.

For analog input channels, the pan control is a balance control and is only used when two channels form a stereo pair. For a mono input, there is a single gain/volume control and no pan/balance control. For a stereo pair there is one gain/volume control and a balance control that modifies the input gain on the left or right channel. At the centre position, both left and right input levels are set according to the gain/volume fader. As the balance control is rotated to the right the gain/volume of the left channel is lowered and as the balance control is rotated to the left the gain/volume of the right channel is lowered. These gain/volume adjustments are made at the hardware level, in 1 dB or 0.5 dB steps depending on the product, which can result in some fairly coarse stepping depending on the gain/volume of the input. For example, if the gain/volume range is from 0 dB to 60 dB with a 1 dB resolution and the gain/volume is set to 6 dB then rotating the balance control will lower the left or right gain/volume from 6 dB down to 0 dB in 1 dB steps.

For analog output controls, the same rules apply as for analog input controls: pan is really balance and is only used when two channels are stereo linked. For mono outputs there is a single attenuation/volume control and no pan/balance control. For stereo outputs there is a single attenuation/volume control and a balance control that modifies the output attenuation on the left or right channels. Attenuation/volume adjustments are made at the hardware level and may exhibit some coarse stepping depending on the attenuation/volume of the output.

For mixer input and output controls, the pan control may be a true pan control or a balance control, depending on whether or not the inputs and/or the outputs are stereo linked. The mixer uses floating point numbers so the pan/balance control can be fairly smooth with no coarse stepping. The pan/balance control uses equal power panning (-3 dB at centre).

For a mono mixer input going to a mono mixer output there is a single volume fader and no pan/balance control.

For a mono mixer input going to a stereo mixer output there is a single volume fader and a true pan control. At the centre position the input is sent equally to both left and right outputs. As the pan control is rotated to the right the amount of input appearing in the left channel is attenuated. As the pan control is rotated to the left the amount of input appearing in the right channel is attenuated.

For a stereo mixer input to a mono mixer output there is a single volume fader and a balance control. At the centre position both left and right inputs are mixed equally to the mono output. As the balance control is rotated to the right, the left channel input is gradually attenuated. As the balance control is rotated to the left, the right channel is gradually attenuated.

For a stereo mixer input to a stereo mixer output there is a single volume fader and a balance control. The left channel input is connected to the left channel output. The right channel input is connected to the right channel output. There is no cross-mixing between left and right channels. At the centre position the left and right channel gains are equal (as set by the volume fader). As the balance control is rotated to the right the volume is decreased in the left channel. As the balance control is rotated to the left the volume is decreased in the right channel.

| Sub-ID | Description |
|---|---|
| 0x00 | Flags: This sub-ID is always present (1 byte).<br>bits 7 - 3: always 0<br>bit 2: set if controls require mixer snapshot number when reading or writing values<br>bit 1: set if controls require SRC channel number when reading or writing values<br>bit 0: set if controls require DST channel number when reading or writing values<br>Most controls require either SRC or DST channel number to be specified when reading or writing values. However, some require both SRC and DST channel. Some controls require mixer snapshot number to be specified. See AudioChannelInfo data class for more information. |
| 0x01 | Mute: This sub-ID is present if the audio channel type supports a mute control (on/off switch) (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x02 | High Z: This sub-ID is present if the audio channel type supports a high impedance control (on/off switch) (1 byte). This feature may be fixed in hardware and thus not editable.<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x03 | Phantom Power: This sub-ID is present if the audio channel type supports a phantom power control (on/off switch) (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x04 | Phase Inversion: This sub-ID is present if the audio channel type supports a phase inversion control (on/off switch) (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x05 | Stereo Link: This sub-ID is present if the audio channel type supports a stereo link control (on/off switch) (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x06 | Solo Enable: This sub-ID is present if the audio channel type supports a solo enable control (on/off switch) (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x07 | Solo PFL: This sub-ID is present if the audio channel type supports a solo pre-fader-listen control (on/off switch) (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x08 | Dim Enable: This sub-ID is present if the audio channel type supports a dim enable control (on/off switch) (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable |
| 0x0F | Pad: This sub-ID is present if the audio channel type supports a pad for inputs (on/off switch) (4 bytes). This feature may be fixed in hardware and is thus not editable.<br>Byte 1:<br>bits 7 - 4: always 0<br>bit 3: set if pad applies to Mic input<br>bit 2: set if pad applies to Line input<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable<br>Bytes 2 – 4: Pad value (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x10 | Volume: This sub-ID is present if the audio channel type supports a volume control (continuous control) (10 bytes).<br>Byte 1:<br>bits 7 - 3: always 0<br>bit 2: set if control supports 0x8000 value (mute)<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable<br>Bytes 2 – 4: Minimum volume value (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format).<br>Bytes 5 – 7: Maximum volume value (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format).<br>Bytes 8 – 10: Resolution of volume control (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x11 | Solo Volume: This sub-ID is present if the audio channel type supports a solo volume control (continuous control) (10 bytes).<br>Byte 1:<br>bits 7 - 3: always 0<br>bit 2: set if control supports 0x8000 value (mute)<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable<br>Bytes 2 – 4: Minimum solo volume value (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format).<br>Bytes 5 – 7: Maximum solo volume value (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format).<br>Bytes 8 – 10: Resolution of volume level control (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x12 | Dim Level: This sub-ID is present if the audio channel type supports a dim level control (10 bytes).<br>Byte 1:<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable<br>Bytes 2 – 4: Minimum dim level (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format).<br>Bytes 5 – 7: Maximum dim level (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format).<br>Bytes 8 – 10: Resolution of dim level (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x13 | Pan: This sub-ID is present if the audio channel type supports a pan (or balance) control (continuous control) (4 bytes).<br>Byte 1:<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable<br>Bytes 2 – 4: Maximum value of pan control representing a signal panned fully right (a positive 16 bit 2’s complement value encoded in 3 bytes, 16x3 format). The negative of this value represents a signal panned fully left. |
| 0x14 | Frequency: This sub-ID is present if the audio channel type supports a frequency control (continuous control) (13 bytes).<br>Byte 1:<br>bits 7 - 2: always 0<br>bit 1: set if control supports stereo link<br>bit 0: set if control is editable<br>Bytes 2 – 5: Minimum frequency value (Hz) in 16.12 format (28-bit value encoded in 4 bytes, 28x4 format).<br>Bytes 6 – 9: Maximum frequency value (Hz) in 16.12 format (28-bit value encoded in 4 bytes, 28x4 format).<br>Bytes 10 – 13: Resolution of frequency control (Hz) in 16.12 format (28-bit value encoded in 4 bytes, 28x4 format). |

### AudioPortInfo (Data Class = 0x09)

This data class contains parameters related to specific audio ports. Audio port ID is a required argument for all parameters in this data class.

#### Current_Value & Restart_Value

Some AudioPortInfo parameters require that the SaveGlobal or SaveGP command be issued and the device restarted before any changes to that parameter become active. For these parameters, the device supports reading the _current_value_ (displayed in BLUE) and reading/writing the _restart_value_ (displayed in RED). The _current_value_ is always what the device is currently using and may be different than the _restart_value_ which will be used after restarting the device. However, if the _restart_value_ is invalid for the selected audio configuration then the _current_value_ will automatically be adjusted to a valid value.

#### Source (SRC) & Destination (DST) Channel Count

Most audio ports support multiple SRC and DST channels. The maximum number of channels supported can vary depending on the sample rate (the configuration ID). The reason for this is that a product operating at a 96 kHz sample rate usually requires twice as much processing time to deal with the audio data as it does at a 48 kHz sample rate. Some audio ports have the same number of channels regardless of sample rate.

One channel count structure is used for each configuration ID:

1. **Configuration ID**: From 1 to AudioInfo:AudioConfigCount (i.e. the sample rate used for this configuration).

2. **Channel Sum Max**: The maximum value allowed for SRC and DST channels summed together for this audio port at this sample rate.

3. **Channel Product Max**: The maximum value allowed for SRC and DST channels multiplied together for this audio port at this sample rate.

4. **DST Minimum**: The minimum number of DST channels that can be selected for this audio port at this sample rate.

5. **DST Maximum**: The maximum number of DST channels that can be selected for this audio port at this sample rate.

6. **SRC Minimum**: The minimum number of SRC channels that can be selected for this audio port at this sample rate.

7. **SRC Maximum**: The maximum number of SRC channels that can be selected for this audio port at this sample rate.

The following rules are applied to the number of DST and SRC channels:

1. If Channel Sum Max is not zero, the number of SRC and DST channels summed together cannot exceed Channel Sum Max.

2. If Channel Product Max is not zero, the number of SRC and DST channels multiplied together cannot exceed Channel Product Max.

3. The number of DST channels cannot be less than DST Minimum or greater than DST Maximum.

4. The number of SRC channels cannot be less than SRC Minimum or greater than SRC Maximum.

For example, given the following values for a USB device port:

```
// cfgID, chnSumMax, chnProdMax, dstMin, dstMax, srcMin, srcMax
1, 20, 100, 2, 18, 2, 18
```

Both SRC and DST channel counts cannot be set lower than 2 or greater than 18. Also, the sum of SRC and DST channel counts must not be greater than 20 and the product of the SRC and DST channel counts must not be greater than 100. Therefore, you cannot set SRC=12 and DST=10 because it breaks the Channel Sum Max rule (it also breaks the Channel Product Max rule). But you can set SRC=4 and DST=6 or SRC=12 and DST=8.

The values in the above structure will always be the same for 44.1 kHz and 48 kHz sample rates (these sample rates are called _single-speed_). Similarly, the values for 88.2 kHz and 96 kHz sample rates will always be the same (these sample rates are called _double-speed_).

#### Audio Meters

Most audio ports have a single meter value for each input and output channel. For example, an analog audio port with 4 mic inputs, 4 mono line outputs and a stereo headphone output will have 4 input meters and 6 output meters. To get all 10 meter values for this analog port it is necessary to send 2 sysex messages:

1. Get 4 input meters (SRC).

2. Get 6 output meters (DST).

However, some audio ports (i.e. mixers) have a separate input meter for each output channel. When getting meter values for these types of ports it is necessary to specify either the associated DST or SRC channel when getting the meter values for the inputs and outputs. The reason is that the mixer sends each input to multiple outputs and each path has a unique meter value. For example, a mixer with six (mono) input channels and four (mono) output channels can be visualized as a matrix having six columns for the inputs and four rows for the outputs as shown below. Each of the six inputs (along the top) are summed together along each row before passing to each output (at the right side).

| **Input 1** | **Input 2** | **Input 3** | **Input 4** | **Input 5** | **Input 6** |  |  |
|---|---|---|---|---|---|---|---|
| Input Meter 1.1 | Input Meter 2.1 | Input Meter 3.1 | Input Meter 4.1 | Input Meter 5.1 | Input Meter 6.1 | Output Meter 1 | **Output 1** |
| Input Meter 1.2 | Input Meter 2.2 | Input Meter 3.2 | Input Meter 4.2 | Input Meter 5.2 | Input Meter 6.2 | Output Meter 2 | **Output 2** |
| Input Meter 1.3 | Input Meter 2.3 | Input Meter 3.3 | Input Meter 4.3 | Input Meter 5.3 | Input Meter 6.3 | Output Meter 3 | **Output 3** |
| Input Meter 1.4 | Input Meter 2.4 | Input Meter 3.4 | Input Meter 4.4 | Input Meter 5.4 | Input Meter 6.4 | Output Meter 4 | **Output 4** |

Each cell of the matrix below the inputs has a set of controls and a meter. For example, _Input Meter 1.2_ is the meter for column 1 (input 1) row 2 (output 2) and displays the contribution of input 1 to output 2 based on the controls in that cell (e.g. volume control). Also, each output has its own meter. Thus, there are a total of 28 meters for this mixer:

1. 24 input meters (6 input meters for each of the 4 output channels, 6 x 4 = 24).

2. 4 output meters (1 output meter for each of the 4 output channels).

When getting the input meters for this mixer it is necessary to specify the output channel (SRC: 1 – 4) which will return 6 meter values (one for each of the 6 inputs) for the specified output channel. When getting the output meters for this mixer it is not necessary to specify the input channel; 4 meter values (one for each of the 4 outputs) will be returned. Therefore, to get all 28 meter values for this mixer it is necessary to send 5 sysex messages:

1. Get 6 input meters (DST) for output 1 (specify SRC = 1).

2. Get 6 input meters (DST) for output 2 (specify SRC = 2).

3. Get 6 input meters (DST) for output 3 (specify SRC = 3).

4. Get 6 input meters (DST) for output 4 (specify SRC = 4).

5. Get 4 output meters (SRC).

The AudioPortInfo:PortFlags parameter defines which ports require DST or SRC audio channels be specified when getting meter values. The audio channel is specified using the AudioChannelNumberDST and AudioChannelNumberSRC ArgIDs.

#### Mixer Snapshots

Mixers may support multiple snapshots for each preset. Snapshots are used to rapidly change between different mixer configurations without having to change presets. The parameters that can be specified for each mixer snapshot are:

1. Mixer input assignments.

2. Mixer input controls.

3. Mixer output controls.

Snapshots cannot be used to change mixer output assignments. Mixer snapshots can be given names so that is easier to distinguish between them. Snapshot #1 is always selected as the active mixer snapshot when changing presets.

The AudioMixerSnapshotID ArgVal is used to specify the mixer shapshot ID when reading/writing mixer parameters.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef<br>RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is required. Some parameters can be specified for individual scenes. |
| RetParmVal |  |
| NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x02 | SceneID | Scene selection (0 – N). Only applies to devices and parameters that support scenes. 0 is used for the currently active scene, 1 – N is used to specify a particular scene. Use 0 for globals and parameters that do not support scenes. |
| 0x10 | AudioPortID | Audio port ID (1 – N). Required for all parameters. |
| 0x11 | AudioChannelNumberDST | Destination audio channel number (1 – N).  Required for some parameters depending on the audio port type. |
| 0x12 | AudioChannelNumberSRC | Source audio channel number (1 – N).  Required for some parameters depending on the audio port type. |
| 0x14 | AudioMixerSnapshotID | Audio mixer snapshot ID (1 – N). Required for some parameters if the audio port is a mixer. |

**Parameters for All Audio Port Types:**

| ID | Name | Description |
|---|---|---|
| 0x01 | PortType | Audio port type (1 byte), see Audio Port Types table. |
| 0x02 | PortIdentifier | Audio port identifier (1 byte) (1 – N). Used to differentiate between multiple instances of the same type of audio port on a product. For most audio port types there will only ever be one instance of that type (i.e. analog, mixer and tone generator). For other types of ports:<br>USB Device: USB device port ID (1 – N). |
| 0x03 | PortFlags | Audio port flags (1 byte).<br>bits 7 - 2: always 0<br>bit 1: set if getting SRC meter values requires that the DST channel number be specified using AudioChannelNumberDST ArgID<br>bit 0: set if getting DST meter values requires that the SRC channel number be specified using AudioChannelNumberSRC ArgID<br>Most ports do not require SRC or DST channel number to be specified when getting meter values. Mixers are different. |
| 0x04 | PortChannelCountSupport | The channel count supported for each audio configuration. Each channel count structure consists of 9 values (10 bytes):<br>Byte 1: configuration number (1 – AudioInfo:AudioConfigCount)<br>Byte 2: maximum value for sum of SRC and DST channels<br>Bytes 3-4: maximum value for product of SRC and DST channels (14x2 format)<br>Byte 5: minimum number of DST channels allowed<br>Byte 6: maximum number of DST channels allowed<br>Byte 7: minimum number of SRC channels allowed<br>Byte 8: maximum number of SRC channels allowed<br>Byte 9: default number of DST channels<br>Byte 10: default number of SRC channels<br>The number of structures is given by AudioInfo:AudioConfigCount. For example, a product that supports four sample rates will return four channel count structures:<br>1, 20, 100, 2, 18, 2, 18, 10, 10 // channel count for configuration #1 (44.1 kHz) 2, 20, 100, 2, 18, 2, 18, 10, 10 // channel count for configuration #2 (48 kHz) 3, 12, 50, 2, 10, 2, 10, 6, 6 // channel count for configuration #3 (88.2 kHz) 4, 12, 50, 2, 10, 2, 10, 6, 6 // channel count for configuration #4 (96 kHz) |
| 0x08 | PortMeterDST | Audio meter values for the DST channels for this port. Each meter value is 2 bytes, 14x2 format. 0 dB is 8192 (0x2000) and is the maximum value that can be returned (representing digital full scale). The first value is channel 1, the second value is channel 2, etc. To convert to decibels (dB) use the formula:<br>dB = 20 log (value/8192) The minimum meter value of 0x0001 corresponds to -78.27 dB but resolution drops to 1 dB when meter value is 0x0008 (-60 dB). |
| 0x09 | PortMeterSRC | Audio meter values for the SRC channels for this port. Each meter value is 2 bytes, 14x2 format. 0 dB is 8192 (0x2000) and is the maximum value that can be returned (representing digital full scale). The first value is channel 1, the second value is channel 2, etc. To convert to decibels (dB) use the formula:<br>dB = 20 log (value/8192) The minimum meter value of 0x0001 corresponds to -78.27 dB but resolution drops to 1 dB when meter value is 0x0008 (-60 dB). |
| 0x0A | PortChannelTypeDST | Audio channel type, one byte for each DST channel for this port. The audio channel type defines which Audio Controls are available for each DST channel. See AudioInfo:AudioChannelTypeDetail. Use the AudioChannelInfo:ChannelTypeDST parameter to get the channel type for a single DST channel. |
| 0x0B | PortChannelTypeSRC | Audio channel type, one byte for each SRC channel for this port. The audio channel type defines which Audio Controls are available for each SRC channel. See AudioInfo:AudioChannelTypeDetail. Use the AudioChannelInfo:ChannelTypeSRC parameter to get the channel type for a single SRC channel. |
| 0x0C | PortChannelLink | Link connections for all DST audio channels for this port (2 bytes for each DST audio channel). For example, an audio port with two DST channels would contain four bytes:<br>Byte 1: Link audio port ID (1 – N) for DST channel 1<br>Byte 2: Link audio channel number (1 – N) for DST channel 1<br>Byte 3: Link audio port ID (1 – N) for DST channel 2<br>Byte 4: Link audio channel number (1 – N) for DST channel 2<br>See the notes for AudioChannelInfo:ChannelLink parameter. |
| 0x10 | PortChannelCountCurrentValue | The channel counts that are currently active for this audio port (2 bytes):<br>Byte 1: DST channel count<br>Byte 2: SRC channel count<br>Read-only and may not be the same as AudioPortInfo:PortChannelCountRestartValue. Only supports Area 0 (work area). |
| 0x40 | PortChannelCountRestartValue | The selected channel counts for this audio port (2 bytes):<br>Byte 1: DST channel count<br>Byte 2: SRC channel count<br>Changing the channel counts requires rebooting the device before the new values become active. Read-write and may not be same as AudioPortInfo:PortChannelCountCurrentValue. |
| 0x41 | PortName | Audio port name (7-bit ASCII string). Maximum length is given by AudioInfo:AudioPortNameMax. |
| 0x42 | PortPatchbay | Audio patchbay connections for all DST audio channels for this port (2 bytes for each DST audio channel). For example, an audio port with two DST channels would contain four bytes:<br>Byte 1: SRC audio port ID (1 – N) for DST channel 1<br>Byte 2: SRC audio channel number (1 – N) for DST channel 1<br>Byte 3: SRC audio port ID (1 – N) for DST channel 2<br>Byte 4: SRC audio channel number (1 – N) for DST channel 2<br>A DST audio channel can be connected to (at most) one SRC audio channel. Use 0x00 for both the port ID and channel number bytes to disconnect a DST audio channel from all SRC audio channels. Use the AudioChannelInfo:ChannelPatchbay parameter to read/write a single patchbay DST channel. AudioMixerSnapshotID is a required ArgID for mixer ports. |

**Parameters for Mixer Ports:**

| ID | Name | Description |
|---|---|---|
| 0x50 | MixerSnapshotNumber | Currently active mixer snapshot (1 – N) (1 byte). This parameter can be used to change the active mixer snapshot in the currently loaded preset. Maximum value is given by AudioInfo:AudioMixerSnapshotMax. |
| 0x51 | MixerSnapshotName | Mixer snapshot name (7-bit ASCII string). Maximum length is given by AudioInfo:AudioPortNameMax. AudioMixerSnapshotID and SceneID are required ArgIDs for this parameter. |

### AudioChannelInfo (Data Class = 0x0A)

This data class contains parameters related to specific audio channels. Audio port ID and audio channel number DST/SRC are required arguments for all parameters in this data class. Most parameters apply to to both DST and SRC audio channels but some parameters only apply to DST channels and some parameters only apply to SRC channels (indicated in the tables below in square brackets below the parameter name).

#### Audio Controls

Most audio controls require either DST or SRC channel to be specified when reading or writing values. However, some controls (i.e. mixer input controls) require both DST and SRC channel to be specified when reading or writing values. The reason is that the mixer sends each input to multiple outputs and each path requires a unique set of controls. For example, a mixer with six (mono) input channels and four (mono) output channels can be visualized as a matrix having six columns for the inputs and four rows for the outputs as shown below. Each of the six inputs (along the top) are summed together along each row before passing to each output (at the right side).

| **Input 1** | **Input 2** | **Input 3** | **Input 4** | **Input 5** | **Input 6** |  |  |
|---|---|---|---|---|---|---|---|
| Input<br>Controls 1.1 | Input<br>Controls 2.1 | Input<br>Controls 3.1 | Input<br>Controls 4.1 | Input<br>Controls 5.1 | Input<br>Controls 6.1 | Output Controls 1 | **Output 1** |
| Input<br>Controls 1.2 | Input<br>Controls 2.2 | Input<br>Controls 3.2 | Input<br>Controls 4.2 | Input<br>Controls 5.2 | Input<br>Controls 6.2 | Output Controls 2 | **Output 2** |
| Input<br>Controls 1.3 | Input<br>Controls 2.3 | Input<br>Controls 3.3 | Input<br>Controls 4.3 | Input<br>Controls 5.3 | Input<br>Controls 6.3 | Output Controls 3 | **Output 3** |
| Input<br>Controls 1.4 | Input<br>Controls 2.4 | Input<br>Controls 3.4 | Input<br>Controls 4.4 | Input<br>Controls 5.4 | Input<br>Controls 6.4 | Output Controls 4 | **Output 4** |

Each cell of the matrix below the inputs has a set of input controls. For example, _Input Controls 1.2_ are the input controls for column 1 (input 1) row 2 (output 2) and determine the contribution of input 1 to output 2. Also, each output has its own set of controls to set the total volume for that output. Thus, there are a total of 28 sets of controls for this mixer:

1. 24 input controls that require both DST (input channel number) and SRC (output channel number) to be specified.

2. 4 output controls that require SRC (output channel number) to be specified.

The Flags sub-ID in the AudioInfo:AudioChannelTypeDetail parameter defines which controls require DST or SRC, or both SRC and DST. The audio channel is specified using the AudioChannelNumberDST and AudioChannelNumberSRC ArgIDs.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef |  |
| RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is required. Some parameters can be specified for individual scenes. |
| RetParmVal |  |
| NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x02 | SceneID | Scene selection (0 – N). Only applies to devices and parameters that support scenes. 0 is used for the currently active scene, 1 – N is used to specify a particular scene. Use 0 for globals and parameters that do not support scenes. |
| 0x10 | AudioPortID | Audio port ID (1 – N). Required for all parameters. |
| 0x11 | AudioChannelNumberDST | Destination audio channel number (1 – N). Required for DST audio channels. |
| 0x12 | AudioChannelNumberSRC | Source audio channel number (1 – N). Required for SRC audio channels. Also required for mixer inputs (mixer DST channels). |
| 0x14 | AudioMixerSnapshotID | Audio mixer snapshot ID (1 – N). Required for some parameters if the audio port is a mixer. |

**Parameters for All Audio Channel Types:**

| ID | Name | Description |
|---|---|---|
| 0x01 | ChannelTypeDST | Audio channel type (1 byte). The audio channel type defines which Audio Controls are available for this DST channel. See AudioInfo:AudioChannelTypeDetail. Use the AudioPortInfo:PortChannelTypeDST parameter to get the channel type for all channels of a port. |
| 0x02 | ChannelTypeSRC | Audio channel type (1 byte). The audio channel type defines which Audio Controls are available for this SRC channel. See AudioInfo:AudioChannelTypeDetail. Use the AudioPortInfo:PortChannelTypeSRC parameter to get the channel types for all channels of a port. |
| 0x03 | ChannelNameDST | Audio channel name for DST channel (7-bit ASCII string). Maximum length is given by AudioInfo:AudioChannelNameMax. |
| 0x04 | ChannelNameSRC | Audio channel name for SRC channel (7-bit ASCII string). Maximum length is given by AudioInfo:AudioChannelNameMax. |
| 0x05 | ChannelControlValueDST | Parameter block containing audio control values for this DST channel. Most channel types require that AudioChannelNumberDST be specified but some require that both AudioChannelNumberDST and AudioChannelNumberSRC be specified. See AudioChannelControlValue parameter block below. This parameter supports the ChannelBitmapDST parameter with SetParmVal messages to set the control values for multiple channels to the same value using one message. |
| 0x06 | ChannelControlValueSRC | Parameter block containing audio control values for this SRC channel. Most channel types require that AudioChannelNumberSRC be specified but some require that both AudioChannelNumberDST and AudioChannelNumberSRC be specified. See AudioChannelControlValue parameter block below. This parameter supports the ChannelBitmapSRC parameter with SetParmVal messages to set the control values for multiple channels to the same value using one message. |
| 0x07 | ChannelPatchbay | Audio patchbay connection for DST audio channel (2 bytes):<br>Byte 1: SRC audio port ID (1 – N)<br>Byte 2: SRC audio channel number (1 – N)<br>A DST audio channel can be connected to (at most) one SRC audio channel.<br>Use 0x00 for both bytes to disconnect a DST audio channel from all SRC audio channels. Use the AudioPortInfo:PortPatchbay parameter to read/write all patchbay connections for a port. AudioMixerSnapshotID is a required ArgID for mixer channels. |
| 0x08 | ChannelLink | Link connection for DST audio channel (2 bytes):<br>Byte 1: Link audio port ID (1 – N)<br>Byte 2: Link audio channel number (1 – N)<br>DST audio channels on one port may be linked to DST audio channels on another audio port. This as a master-slave relationship where the linked channel is the slave. A master may have multiple slaves. A master cannot be a slave to another master (no recursion). Linking only affects patchbay connections. For linked channels (i.e. slave channels), reading and writing the AudioPortInfo:PortPatchbay or AudioChannelInfo:ChannelPatchbay parameters always reads/writes the master’s patchbay. It is not possible to set the slave channel’s patchbay to different values than the master channel’s patchbay. Non-linked and master channels always return 0x00 for both bytes. |
| 0x09 | ChannelBitmapDST | Audio channel bitmap (BAx2 format). Each bit represents one audio channel,<br>bit 0 is channel 1, bit 1 is channel 2, etc. Least significant byte is first, most significant byte is last. The most significant 4 bits of each byte must be 0 which allows 4 channels to be specified per byte. Unused bits should be set to 0. The number of bytes (in the bitmap) is an even value so that the bitmap (when unpacked) is a multiple of 8 bits: number of bytes (using INTEGER math) = (((number of audio channels - 1) / 8 ) + 1) x 2.<br>Example: device has 20 audio channels on the selected DST audio stream, select channels 2, 3, 7, 11-14, 20:<br>0x06<br>// channels 3 and 2<br>0x04<br>// channel 7<br>0x0C<br>// channels 12 and 11<br>0x03<br>// channels 14 and 13<br>0x08<br>// channel 20 is selected<br>0x00<br>// channels 24 through 21 (padding)<br>This parameter is used with the ChannelControlValueDST parameter to set the control values for multiple audio channels using a single SetParmVal message (e.g. mute all analog outputs, or a subset of analog outputs). This parameter is to be used instead of the AudioChannelNumberDST ArgID and must always be specified before the ChannelControlValueDST parameter.<br>Notification messages also use this parameter (instead of using AudioChannelNumberDST ArgID) when audio control values are changed by automation controls because multiple audio channels may be affected by a single automation control.<br>The maximum number of audio channels for all audio streams is given by AudioInfo:AudioChannelMax. |
| 0x0A | ChannelBitmapSRC | This parameter is used with the ChannelControlValueSRC parameter to set the control values for multiple audio channels using a single SetParmVal message. This parameter is to be used instead of the AudioChannelNumberSRC ArgID and must always be specified before the ChannelControlValueSRC parameter.<br>Notification messages also use this parameter (instead of using AudioChannelNumberSRC ArgID) when audio control values are changed by automation controls because multiple audio channels may be affected by a single automation control.<br>See ChannelBitmapDST parameter for more details. |

**Parameters for USBD Channels:**

| ID | Name | Description |
|---|---|---|
| 0x10 | USBDChannel<br>NameDST | USBD audio channel name for DST channels (7-bit ASCII string). Maximum length is given by AudioInfo:USBDChannelNameMax. |
| 0x11 | USBDChannel<br>NameSRC | USBD audio channel name for SRC channels (7-bit ASCII string). Maximum length is given by AudioInfo:USBDChannelNameMax. |

**Parameters for Analog Channels:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

**Parameters for Mixer Channels:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

**Parameters for Tone Generator Channels:**

| ID | Name | Description |
|---|---|---|
|  | none |  |

**Parameters for AOIP Channels:**

| ID | Name<br>none | Description |
|---|---|---|

**AudioChannelControlValue Parameter Block:**

This block comprises a set of sub-ID & value pairs that contain control values for a specific audio channel. Not all sub-IDs are used with every audio channel type. Length depends on the capabilities of the audio channel type. The audio controls available for each audio channel are given by AudioInfo:AudioChannelTypeDetail parameter.

RetParmVal messages always return all sub-IDs for the selected audio channel. The SetParmVal message is used to write one or more control values for the selected audio channel using one or more sub-IDs.

Many controls comprise a simple on/off switch that uses a single byte ON/Enabled (0x01) or OFF/Disabled (0x00) value.

Volume, Solo Volume and Dim Level controls use 16 bit 2’s complement numbers (8.8 value encoded in 3 bytes, 16x3 format) that can range from +127.9961 dB (0x7FFF) down to -127.9961 dB (0x8001) in steps of 1/256 dB or 0.00390625 dB (0x0001). In addition, value 0x8000, representing silence (i.e. -∞ dB), may be implemented in some controls. The AudioInfo:AudioChannelTypeDetail parameter defines the minimum, maximum, and resolution for Volume, Solo Volume and Dim Level values.

Pan controls use 16 bit 2’s complement numbers (encoded in 3 bytes, 16x3 format) that can range from +32767 (0x7FFF) down to - 32767 (0x8001). Positive numbers are panning to the right. Negative numbers are panning to the left. 0 means fully centered. The maximum pan value representing a signal panned fully right is specified in the control definition. The minimum pan value representing a signal panned fully left is the negative of this value. For example, if the maxium pan value is 0x007F (+127) then the minimum pan value is 0xFF81 (-127). The AudioInfo:AudioChannelTypeDetail parameter defines the maximum pan value.

Frequency controls use unsigned 28-bit numbers (16.12 value encoded in 4 bytes, 28x4 format) that can range from 65,535.9997558594 Hz (0xFFFFFFF) down to 0 Hz (0x0000000) in steps of 1/4096 Hz or 0.000244140625 Hz (0x0000001). The AudioInfo:AudioChannelTypeDetail parameter defines the minimum, maximum, and resolution for frequency value.

| Sub-ID | Description |
|---|---|
| 0x01 | Mute: This sub-ID is present if the audio channel type supports a mute control (on/off switch) (1 byte).<br>bits 7 - 1: always 0<br>bit 0: set if mute is enabled (channel is muted) |
| 0x02 | High Z: This sub-ID is present if the audio channel type supports a high impedance control (on/off switch) (1 byte). This feature may be fixed in hardware.<br>bits 7 - 1: always 0<br>bit 0: set if high Z is enabled (high impedance is on) |
| 0x03 | Phantom Power: This sub-ID is present if the audio channel type supports a phantom power control (on/off switch) (1 byte).<br>bits 7 - 1: always 0<br>bit 0: set if phantom power is enabled (phantom power is on) |
| 0x04 | Phase Inversion: This sub-ID is present if the audio channel type supports a phase inversion control (on/off switch) (1 byte).<br>bits 7 - 1: always 0<br>bit 0: set if channel is inverted (phase inversion is on) |
| 0x05 | Stereo Link: This sub-ID is present if the audio channel type supports a stereo link control (on/off switch) (1 byte).<br>bits 7 - 1: always 0<br>bit 0: set if stereo link is enabled (adjacent channels are stereo linked) |
| 0x06 | Solo Enable: This sub-ID is present if the audio channel type supports a solo enable control (on/off switch) (1 byte).<br>bits 7 - 1: always 0<br>bit 0: set if solo is enabled (solo is on) |
| 0x07 | Solo PFL: This sub-ID is present if the audio channel type supports a solo pre-fader-listen control (on/off switch) (1 byte).<br>bits 7 - 1: always 0<br>bit 0: set if solo PFL is enabled (solo PFL is on) |
| 0x08 | Dim Enable: This sub-ID is present if the audio channel type supports a dim enable control (on/off switch) (1 byte).<br>bits 7 - 1: always 0<br>bit 0: set if dim is enabled (dim is on) |
| 0x0F | Pad: This sub-ID is present if the audio channel type supports a pad for inputs (on/off switch) (1 byte). This feature may be fixed in hardware.<br>bits 7 - 1: always 0<br>bit 0: set if pad is enabled (pad is on) |
| 0x10 | Volume: This sub-ID is present if the audio channel type supports a volume control (continuous control) (3 bytes).<br>Bytes 1 – 3: Volume value (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x11 | Solo Volume: This sub-ID is present if the audio channel type supports a solo volume control (continuous control) (3 bytes).<br>Bytes 1 – 3: Solo volume (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x12 | Dim Level: This sub-ID is present if the audio channel type supports a dim level control (continuous control) (3 bytes).<br>Bytes 1 – 3: Dim level (dB) in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x13 | Pan: This sub-ID is present if the audio channel type supports a pan (or balance) control (continuous control) (3 bytes).<br>Bytes 1 – 3: Pan value (16-bit 2’s complement value encoded in 3 bytes, 16x3 format). |
| 0x14 | Frequency: This sub-ID is present if the audio channel type supports a frequency control (continuous control) (4 bytes).<br>Bytes 1 – 4: Frequency value (Hz) in 16.12 format (28-bit value encoded in 4 bytes, 28x4 format). |

### AudioAutomationInfo (Data Class = 0x0B)

This data class contains parameters related to automation controls for audio (ACA). Automation controls are used to modify a device’s audio controls (e.g. fader, pan, mute) using standard MIDI continuous controller messages. Typically, this is performed by connecting a suitable MIDI controller to a USB host port or DIN port on a device and sending MIDI continuous controller messages to the device’s Automation Control MIDI port. A MIDI controller can be used to modify multiple controls (of the same type) on a device so that a single pushbutton or fader on the MIDI controller can mute or change the volume of several audio channels on the device.

Note that for mixer inputs it is necessary to set the AudioChannelNumberSRC parameter to a non-zero value to indicate the mixer output channel. Although mixer inputs have a unique set of audio controls for each mixer snapshot, automation always affects the currently active snapshot.

Only analog and mixer audio ports support automation controls.

Also see the MIDIOperation:AudioAutomationControlAssign command.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| GetParmDef<br>RetParmDef |  |
| GetParmVal<br>SetParmVal | Argument value block is required. Some parameters can be specified for individual scenes. |
| RetParmVal |  |
| NotParmVal |  |

**Applicable Arguments:**

| ArgID | Name | Description |
|---|---|---|
| 0x01 | AreaID | Area selection (0 – N), optional. Only applies to devices that support areas. 0 is used for the work area, 1 – N is used for the shadow areas. Work area is used if this argument is not supplied. |
| 0x15 | ACAID | Automation control for audio ID (1 – N). Required for all parameters. |

**Parameters:**

| ID | Name | Description |
|---|---|---|
| 0x01 | AudioStream | Audio stream (2 bytes):<br>Byte 1: audio port ID (1 – N), use 0 to unassign this automation control<br>Byte 2: destination (DST = 0) or source (SRC = 1) of audio port<br>Only analog and mixer audio ports are supported.<br>For mixer input streams (DST = 0) it is necessary to set the AudioChannelNumberSRC parameter to a non-zero value to indicate the mixer output channel. |
| 0x02 | AudioChannelBitmap | Audio channel bitmap (BAx2 format). Each bit represents one audio channel,<br>bit 0 is channel 1, bit 1 is channel 2, etc. Least significant byte is first, most significant byte is last. The most significant 4 bits of each byte must be 0 which allows 4 channels to be specified per byte. Unused bits should be set to 0. The number of bytes (in the bitmap) is an even value so that the bitmap (when unpacked) is a multiple of 8 bits: number of bytes (using INTEGER math) = (((number of audio channels - 1) / 8 ) + 1) x 2.<br>Example: device has 20 audio channels on the selected audio stream, select channels 2, 3, 7, 11-14, 20:<br>0x06<br>// channels 3 and 2<br>0x04<br>// channel 7<br>0x0C<br>// channels 12 and 11<br>0x03<br>// channels 14 and 13<br>0x08<br>// channel 20 is selected<br>0x00<br>// channels 24 through 21 (padding)<br>The maximum number of audio channels for all audio streams is given by AudioInfo:AudioChannelMax. |
| 0x03 | AudioChannelNumberSRC | Source audio channel number (1 – N). Used to specify mixer output (SRC) channel for automation controls assigned to mixer input (DST) channels.<br>Only used if AudioStream is mixer DST. |
| 0x04 | AudioControlID | Audio control ID from AudioChannelTypeDetail parameter block:<br>0x00: none<br>0x01: Mute<br>0x02: High Z<br>0x03: Phantom Power<br>0x04: Phase Inversion<br>0x05: Stereo Link<br>0x06: Solo Enable<br>0x07: Solo PFL<br>0x08: Dim Enable<br>0x0F: Pad<br>0x10: Volume<br>0x11: Solo Volume<br>0x12: Dim Level<br>0x13: Pan<br>Only the controls that both exist and are editable for the selected audio port’s channel type can be used with automation control. |
| 0x05 | AudioControlMinValue | Minimum audio control value to be used for automation control. For volume controls this is dB in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). Value must be within range of the values supported by the control, not including 0x8000 (mute) value. Value should be less than AudioControlMaxValue. Not applicable to switch type controls (mute, dim, etc). Not used by pan controls; pan controls use negative of AudioControlMaxValue for AudioControlMinValue. |
| 0x06 | AudioControlMaxValue | Maximum audio control value to be used for automation control. For volume controls this is dB in 8.8 format (16-bit value encoded in 3 bytes, 16x3 format). For pan controls this is an integer (16 bit 2’s complement value encoded in 3 bytes, 16x3 format). Value must be within range of the values supported by the control, not including 0x8000 (mute) value. Value should be greater than AudioControlMinValue. Not applicable to switch type controls (mute, dim, etc). |
| 0x07 | ACAFlags | ACA configuration flags (1 byte):<br>bits 7 - 3: reserved (always 0).<br>bit 2: set if lowest MIDI value should be used to mute volume control, only applicable to volume controls that support 0x8000 (mute) value.<br>bit 1: set if MIDI values should be inverted so that 127 is lowest value and 0 is highest value.<br>bit 0: set if control is enabled, clear if control is disabled. |
| 0x08 | MIDIChannels | Bitmask indicating which MIDI channels to use for automation control (0x0001 = MIDI channel 1, 0x0002 = MIDI channel 2 … 0x8000 = MIDI channel 16). More than 1 bit can be set (16-bit value encoded in 3 bytes, 16x3 format). |
| 0x09 | MIDICCID | MIDI continuous controller number to use for automation control (0 – 127) (1 byte). |
| 0x0A | MIDIMinCCValue | Minimum CC Value (0 – 127) (1 byte). The minimum CC value to be used for automation control. Values less than this are treated as minimum value.<br>Value should be less than MIDIMaxCCValue. Only applicable to continuous type controls (volume, pan, etc). |
| 0x0B | MIDIMaxCCValue | Maximum CC Value (0 – 127) (1 byte). The maximum CC value to be used for automation control. Values greater than this are treated as maximum value. Value should be greater than MIDIMinCCValue. Only applicable to continuous type controls (volume, pan, etc). |
| 0x0C | MIDIOnCCValue | CC value to use for ON state (0 – 127) (1 byte). Values at this level and above are treated as ON state. Values below this level are treated as OFF state. Use the invert flag in ACAFlags parameter to reverse the logic. Only applicable to switch type controls (mute, dim, etc). |

### BulkData (Data Class = 0x70)

This data class is used to perform bulk data transfer operations (i.e. backup and restore) between a device and a host (or between two devices where one device acts as a host). See the Bulk Data Transfer section for more information regarding bulk data transfer operations.

**Message Class Support:**

| Message Class | Notes |
|---|---|
| BulkTransfer |  |

## Commands

The following command IDs are defined:

**Command IDs:**

| ID | Name | Description |
|---|---|---|
| 0x01 | DeviceMode | Used to change a device’s operating mode. |
| 0x02 | SaveLoad | Used to manage globals and presets between RAM and non-volatile storage. |
| 0x03 | SetGroup | Used to set a group of parameters to a predefined state. |
| 0x04 | BulkRequest | Used to start a bulk data transfer operation. |
| 0x05 | Notification | Used to register for automatic parameter updates. |
| 0x06 | MIDIOperation | Used to trigger MIDI operations, functions, and events. |
| 0x07 | DeviceOperation | Used to trigger device operations, functions, and events. |

### DeviceMode (Command ID = 0x01)

DeviceMode commands are used to change a device’s operating mode. Note that all mode changes will result in a loss of any global and preset parameters residing in volatile storage (RAM).

| Val | Name | Description |
|---|---|---|
| 0x01 | RebootApp | Reboot device into application mode. No command arguments. |
| 0x02 | RebootBL | Reboot device into bootloader mode. No command arguments. |
| 0x03 | EraseRebootApp | Erase all user settings (set globals and presets to factory defaults) and reboot device into application mode. No command arguments. |
| 0x04 | EraseRebootBL | Erase all user settings (set globals and presets to factory defaults) and reboot device into bootloader mode. No command arguments. |
| 0x05 | Shutdown | Power down device. Device can only be restarted by toggling the power supply or by pressing the power button on the device. No command arguments. |

_Example: EraseRebootApp_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x09                    // message length (9)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x06, 0x06                    // data block #1: size (6), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x03                          // block #1: CmdSize: (3)
0x01                          // block #1: CmdID: DeviceMode (1)
0x03                          // block #1: CmdVal: EraseRebootApp (3)
0xxx                          // checksum
0xF7                          // footer
```

### SaveLoadInit (Command ID = 0x02)

The Save, Load and Init commands are used to manage globals and presets between volatile (RAM) and non-volatile storage (FLASH, EEPROM). For the area argument, 0 always refers to the work area and 1 – N refers to one of the shadow areas. The preset number is also a required argument whenever presets are saved or loaded.

| Val | Name | Description |
|---|---|---|
| 0x01 | SaveGP | Save global and preset parameters (for a single preset) in RAM to non-volatile storage. The source area and destination preset are required arguments.<br>CmdArg 1: source area in RAM (0 – N) (1 byte).<br>CmdArg 2: destination preset number in non-volatile storage (1 – N) (1 byte). |
| 0x02 | SaveGlobal | Save global parameters in RAM to non-volatile storage. Preset parameters are not saved to non-volatile storage. The source area is a required argument.<br>CmdArg 1: source area in RAM (0 – N) (1 byte). |
| 0x03 | SavePreset | Save preset parameters (for a single preset) in RAM to non-volatile storage.<br>Global parameters are not saved to non-volatile storage. The source area and destination preset are required arguments.<br>CmdArg 1: source area in RAM (0 – N) (1 byte).<br>CmdArg 2: destination preset number in non-volatile storage (1 – N) (1 byte). |
| 0x11 | InitGlobalRAM | Initialize all global parameters in RAM to factory defaults. This is equivalent to sending a LoadGlobal command after initializing the device. This does not affect the global parameters in non-volatile storage. Send a SaveGlobal command to save the initialized global parameters to non-volatile storage.<br>CmdArg 1: source area in RAM (0 – N) (1 byte). |
| 0x12 | InitPresetRAM | Initialize all preset parameters in RAM to factory defaults. This is equivalent to sending a LoadPreset command after initializing the device. This does not affect the preset parameters in non-volatile storage. Send a SavePreset command to save the initialized preset parameters to non-volatile storage.<br>CmdArg 1: source area in RAM (0 – N) (1 byte). |
| 0x13 | InitGlobalNV | Initialize all global parameters in non-volatile storage to factory defaults. This does not affect the global parameters in RAM. |
| 0x14 | InitPresetNV | Initialize all preset parameters (for a single preset) in non-volatile storage to factory defaults. This does not affect the preset parameters in RAM.<br>CmdArg 1: preset number in non-volatile storage (1 – N) (1 byte). |
| 0x41 | LoadGP | Load all global and preset parameters (for a single preset) from non-volatile storage into RAM. The destination area and source preset are required arguments.<br>CmdArg 1: destination area in RAM (0 – N) (1 byte).<br>CmdArg 2: source preset number in non-volatile storage (1 – N) (1 byte). |
| 0x42 | LoadGlobal | Load all globals from non-volatile storage into RAM. Presets are not loaded from non-volatile storage. The destination area is a required argument.<br>CmdArg 1: destination area in RAM (0 – N) (1 byte). |
| 0x43 | LoadPreset | Load preset parameters (for a single preset) from non-volatile storage into RAM.<br>Globals are not loaded from non-volatile storage. The destination area and source preset are required arguments.<br>CmdArg 1: destination area in RAM (0 – N) (1 byte).<br>CmdArg 2: source preset number in non-volatile storage (1 – N) (1 byte). |
| 0x44 | LoadPresetNP | Load next preset (increment) or previous preset (decrement) from non-volatile storage into RAM. The destination area and direction are required arguments. If wraparound is used, incrementing from the highest numbered preset will load the lowest numbered preset and decrementing from the lowest numbered preset will load the highest numbered preset. If not using wraparound, incrementing from the highest numbered preset or decrementing from the lowest numbered preset will not change the preset.<br>CmdArg 1: destination area in RAM (0 – N) (1 byte).<br>CmdArg 2: direction (1 byte):<br>0 = load next preset (increment preset), no wraparound<br>1 = load next preset (increment preset), with wraparound<br>2 = load previous preset (decrement preset), no wraparound<br>3 = load previous preset (decrement preset), with wraparound |

_Example: SaveGP (save from work area to preset #8)_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0B                    // message length (11)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x08, 0x06                    // data block #1: size (8), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x05                          // block #1: CmdSize: (5)
0x02                          // block #1: CmdID: SaveLoad (2)
0x01                          // block #1: CmdVal: SaveGP (1)
0x00                          // block #1: CmdArg 1: source area (0)
0x08                          // block #1: CmdArg 2: destination preset (8)
0xxx                          // checksum
0xF7                          // footer
```

### SetGroup (Command ID = 0x03)

SetGroup commands are used to set a group of parameters to a predefined state. For the area argument, 0 always refers to the work area and 1 – N refers to one of the shadow areas.

| Val | Name | Description |
|---|---|---|
| 0x01 | Reset | Resets global and preset parameters (for a single preset) in RAM to default values. The destination area is a required argument. CmdArg 2 (also required) is one or more SetGroupReset parameter blocks, each block contains 2 bytes:<br>CmdArg 1: destination area in RAM (0 – N) (1 byte).<br>CmdArg 2: SetGroupReset Parameter Blocks (each block is 2 bytes). |

**SetGroupReset Parameter Block:**

Each block contains one or more sub-ID & value pairs (always 2 bytes). Not all sub-IDs need to be included in the command. Any sub-ID not recognized is ignored (no error code returned). Any bits set to undefined values are also ignored (no error code returned).

| Sub-ID | Description |
|---|---|
| 0x01 | MIDI preset parameters (1 byte):<br>bits 7 - 6: always 0.<br>bit 5: if set, all route processing block parameters are set to default values.<br>bit 4: if set, all AMP parameters are set to default values.<br>bit 3: if set, all output processing is disabled (all filters except Active Sense, remapping,<br>CC processor, AMP assignment).<br>bit 2: if set, all input processing is disabled (all filters except Active Sense, remapping,<br>CC processor, AMP assignment).<br>bit 1: if set, all preset port routing is cleared.<br>bit 0: if set, input and output ports are enabled and running status is disabled. |
| 0x02 | MIDI global parameters (1 byte):<br>bits 7 - 5: always 0.<br>bit 4: if set, MIDI and USB device port names are set to default values.<br>bit 3: if set, all global port routing is cleared.<br>bit 2: if set, input and output ports are enabled.<br>bit 1: if set, all RTP-MIDI session information is reset (all sessions set to responder, all reservations are cleared, all session names set to default values).<br>bit 0: if set, all USB Host Controller information is reset (all reservations cleared). |
| 0x03 | System events for hardware control inputs (1 byte). This affects all states (active and inactive) for all scenes.<br>bits 7 - 4: always 0.<br>bit 3: if set, all system events for hardware control input #4 are cleared.<br>bit 2: if set, all system events for hardware control input #3 are cleared.<br>bit 1: if set, all system events for hardware control input #2 are cleared.<br>bit 0: if set, all system events for hardware control input #1 are cleared. |
| 0x04 | System events for failover, preset/scene selection and audio global mute (1 byte). This affects all scenes.<br>bits 7 - 5: always 0.<br>bit 4: if set, all system events for audio global mute input are cleared.<br>bit 3: if set, all system events for scene change input are cleared.<br>bit 2: if set, all system events for preset load input are cleared.<br>bit 1: if set, all system events for failover alarm set/clear inputs are cleared.<br>bit 0: if set, all system events for failover arm/disarm inputs are cleared. |
| 0x05 | System events for control button inputs (1 byte). This affects all states (inactive, single-click, double-click) for all scenes.<br>bits 7 - 4: always 0.<br>bit 3: if set, all system events for control button input #4 are cleared.<br>bit 2: if set, all system events for control button input #3 are cleared.<br>bit 1: if set, all system events for control button input #2 are cleared.<br>bit 0: if set, all system events for control button input #1 are cleared. |

_Example: reset all parameters_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0E                    // message length (14)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x0B, 0x06                    // data block #1: size (11), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x08                          // block #1: CmdSize: (8)
0x03                          // block #1: CmdID: SetGroup (3)
0x01                          // block #1: CmdVal: Reset (1)
0x00                          // block #1: CmdArg 1: Area (0)
0x01, 0x0F                    // block #1: CmdArg 2a: sub-ID (1), value (0x0F)
0x02, 0x07                    // block #1: CmdArg 2b: sub-ID (2), value (0x07)
0xxx                          // checksum
0xF7                          // footer
```

_Example: reset all the USB host controller reservations_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0C                    // message length (12)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x09, 0x06                    // data block #1: size (9), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x06                          // block #1: CmdSize: (6)
0x03                          // block #1: CmdID: SetGroup (3)
0x01                          // block #1: CmdVal: Reset (1)
0x00                          // block #1: CmdArg 1: Area (0)
0x02, 0x01                    // block #1: CmdArg 2: sub-ID (2), value (0x01)
0xxx                          // checksum
0xF7                          // footer
```

### BulkRequest (Command ID = 0x04)

Bulk request commands are used to start a backup operation. A host sends a bulk request message to a device and the device returns an Ack message with an error code. If the Ack message returns with no error the device will begin the backup operation after a brief delay. The backup data is always read from non-volatile storage (not from RAM). See the Bulk Data Transfer section for more information regarding bulk data transfer operations.

| Val | Name | Description |
|---|---|---|
| 0x01 | BackupAll | Backup globals and all presets from non-volatile storage.<br>CmdArg 1: MIDI port to use for backup operation (0 – MIDIInfo:PortCount) (1 byte). Use 0 to use the same MIDI port that the BulkRequest message is sent on. |
| 0x02 | BackupPresetAll | Backup all presets from non-volatile storage. Globals are not backed up.<br>CmdArg 1: MIDI port to use for backup operation (0 – MIDIInfo:PortCount) (1 byte). Use 0 to use the same MIDI port that the BulkRequest message is sent on. |
| 0x03 | BackupGlobal | Backup globals from non-volatile storage. Presets are not backed up.<br>CmdArg 1: MIDI port to use for backup operation (0 – MIDIInfo:PortCount) (1 byte). Use 0 to use the same MIDI port that the BulkRequest message is sent on. |
| 0x04 | BackupPreset | Backup one preset from non-volatile storage. Globals are not backed up.<br>CmdArg 1: MIDI port to use for backup operation (0 – MIDIInfo:PortCount) (1 byte). Use 0 to use the same MIDI port that the BulkRequest message is sent on.<br>CmdArg 2: preset number (1 – N) (1 byte). |
| 0x05 | BackupGlobalPreset | Backup globals and one preset from non-volatile storage.<br>CmdArg 1: MIDI port to use for backup operation (0 – MIDIInfo:PortCount) (1 byte). Use 0 to use the same MIDI port that the BulkRequest message is sent on.<br>CmdArg 2: preset number (1 – N) (1 byte). |

_Example: Backup preset number 7 to MIDI port 3_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0B                    // message length (11)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x08, 0x06                    // data block #1: size (8), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x05                          // block #1: CmdSize: (5)
0x04                          // block #1: CmdID: BulkRequest (4)
0x04                          // block #1: CmdVal: BackupPreset (4)
0x03                          // block #1: CmdArg 1: MIDI port (3)
0x07                          // block #1: CmdArg 2: preset number (7)
0xxx                          // checksum
0xF7                          // footer
```

### Notification (Command ID = 0x05)

Notification commands are used to register for automatic parameter updates. This allows a host to be informed about parameter changes without having to constantly poll a device. Once registered, a device will send NotParmVal messages to the host at either regular intervals (e.g. MIDI and audio meters) or when changes occur that the host did not initiate (e.g. changing preset or headphone volume using front panel of device). A host should never respond to NotParmVal messages. The device always returns the session ID and transaction ID of the message used to register the notification class in each NotParmVal message. The device requires that the host communicate with the device regularly (via sysex messages) otherwise the device will automatically cancel all notifications (even a ping message will suffice). The timeout interval is given in DeviceInfo:NotificationTimeout. The notification classes supported by a device are given in DeviceInfo:NotificationClassSupport.

Notifications are not supported on DIN ports.

| Val | Name | Description |
|---|---|---|
| 0x01 | Register | Register one or more notification classes for automatic updates. CmdArg is a list of notification class IDs from the table below (1 byte each). |
| 0x02 | Unregister | Unregister one or more notification classes for automatic updates. CmdArg is a list of notification class IDs from the table below (1 byte each). |

**Notification Classes:**

| Arg | Name | Description |
|---|---|---|
| 0x00 | NotifyAll | All notification classes. Shortcut to register or unregister all notification classes supported by a device. |
| 0x01 | NotifyProgramSelector | Send notification messages whenever preset or mixer snapshot changes.<br>This is equivalent to polling the device for DeviceFeature:PresetNumber and AudioPortInfo:MixerSnapshotNumber parameters. Use NotifiyFailoverStatus to send notification messages that include the current scene number. |
| 0x02 | NotifyMIDIMeter | Send notification messages containing MIDI port monitor data at regular intervals. This is equivalent to polling the device for MIDIInfo:PortMonitorIn and MIDIInfo:PortMonitorOut parameters. |
| 0x03 | NotifyAudioMeter | Send notification messages containing audio channel meter data at regular intervals. This is equivalent to polling the device for AudioPortInfo:PortMeterDST and AudioPortInfo:PortMeterSRC parameters for all audio ports except the tone generator port. |
| 0x04 | NotifyAudioControl | Send notification messages whenever audio channel channel control values are changed. This is equivalent to polling the device for AudioChannelInfo:ChannelControlValueDST, AudioChannelInfo:ChannelControlValueSRC and AudioInfo:AudioGlobalMute parameters. |
| 0x05 | NotifyAutomationControlAssign | Send notification messages containing MIDI channel and MIDI continuous controller parameters when automation control assignments are made. See MIDIOperation:AutomationControlAssign commands. |
| 0x06 | NotifyFailoverStatus | Send notification messages containing failover status information and current scene number whenever any of these have changed. This is equivalent to polling the device for DeviceFeature:FailoverArmStatus, DeviceFeature:FailoverAlarmStatus, DeviceFeature:FailoverUSBPortStatus, DeviceFeature:FailoverToneFlags and DeviceFeature:SceneNumber parameters. |
| 0x07 | NotifyHardwareControlState | Send notification messages whenever hardware control state has changed.<br>This is equivalent to polling the device for HardwareInfo:CtrlState parameters. |
| 0x08 | NotifyUSBDeviceConnectState | Send notification messages when USB device port connection status changes. This is equivalent to polling the device for HardwareInfo:USBDConnect parameter. |
| 0x09 | NotifyUSBHostConnectState | Send notification messages when USB host port connection status changes.<br>This is equivalent to polling the device for HardwareInfo:USBHMIDIIdentifier and MIDIPortInfo:PortConnectFlags (for USB Host Controller ports only) parameters. |
| 0x0A | NotifyEthernetConnectState | Send notification messages when Ethernet port connection status changes.<br>This is equivalent to polling the device for the HardwareInfo parameters:<br>EthConnect, EthCurrentIP, EthDevName and the MIDIPortInfo parameters:<br>EthSesnNameN, EthIPAddressX, EthPortNumberX, EthSesnNameX, EthSwitchPortConnect. |
| 0x0B | NotifyControlButtonState | Send notification messages whenever control button state has changed. This is equivalent to polling the device for HardwareInfo:ControlButtonState parameters. |

_Example: Register for notifications for MIDI meters, audio meters and failover status_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0C                    // message length (12)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x09, 0x06                    // data block #1: size (9), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x06                          // block #1: CmdSize: (6)
0x05                          // block #1: CmdID: Notification (5)
0x01                          // block #1: CmdVal: Register (1)
0x02                          // block #1: CmdArg 1: NotifyMIDIMeter (2)
0x03                          // block #1: CmdArg 2: NotifyAudioMeter (3)
0x06                          // block #1: CmdArg 3: NotifyFailoverStatus (6)
0xxx                          // checksum
0xF7                          // footer
```

_Example: Unregister all notifications_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0A                    // message length (10)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x07, 0x06                    // data block #1: size (7), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x04                          // block #1: CmdSize: (4)
0x05                          // block #1: CmdID: Notification (5)
0x02                          // block #1: CmdVal: Unregister (2)
0x00                          // block #1: CmdArg 1: NotifyAll (0)
0xxx                          // checksum
0xF7                          // footer
```

### MIDIOperation (Command ID = 0x06)

MIDIOperation commands are used to trigger MIDI operations, functions, and events.

| Val | Name | Description |
|---|---|---|
| 0x01 | PanicSend | Send Panic message. No command arguments. |
| 0x02 | AudioAutomationControlAssign | Use this command to automatically assign MIDI channel number and continuous controller ID to an automation control for audio (ACA). After receiving this command, the MIDI channel number and controller ID of the next MIDI continuous controller (CC) message sent to the Automation Control port will be assigned to the specified automation control for audio.<br>CmdArg 1: Automation control for audio ID (ACAID) (1 – AudioInfo:AudioAutomationControlMax) (1 byte). Use 0 to cancel a previous AudioAutomationControlAssign command.<br>CmdArg 2: Automatic port route value (see below).<br>When the assignment is made, MIDI channel number and continuous controller ID are sent to the host in a NotParmVal message containing AudioAutomationInfo:MIDIChannels and AudioAutomationInfo:MIDICCID parameters if NotifyAutomationControlAssign notifications are registered. |
| 0x03 | FailoverAutomationControlAssign | Use this command to automatically assign MIDI channel number and continuous controller ID to an automation control for failover (ACF). After receiving this command, the MIDI channel number and controller ID of the next MIDI continuous controller (CC) message sent to the Automation Control port will be assigned to the specified automation control for failover.<br>CmdArg 1: Automation control for failover ID (1 byte):<br>0x01: arm failover system<br>0x02: disarm failover system<br>0x03: clear failover alarm<br>Use 0 to cancel a previous FailoverAutomationControlAssign command.<br>CmdArg 2: Automatic port route value (see below).<br>When the assignment is made, MIDI channel number and continuous controller ID are sent to the host in a NotParmVal message containing MIDIPortInfo:FailoverArmControl, MIDIPortInfo:FailoverDisarmControl or MIDIPortInfo:FailoverClearAlarmControl parameters if NotifyAutomationControlAssign notifications are registered. |
| 0x04 | PanicAutomationControlAssign | Use this command to automatically assign MIDI channel number and continuous controller ID to an automation control for sending Panic message. After receiving this command, the MIDI channel number and controller ID of the next MIDI continuous controller (CC) message sent to the Automation Control port will be assigned to automation control for Panic.<br>CmdArg 1: Automation control for panic ID (1 byte):<br>0x01: send MIDI panic<br>Use 0 to cancel a previous PanicAutomationControlAssign command.<br>CmdArg 2: Automatic port route value (see below).<br>When the assignment is made, MIDI channel number and continuous controller ID are sent to the host in a NotParmVal message containing a MIDIPortInfo:PanicControl parameter if NotifyAutomationControlAssign notifications are registered. |
| 0x05 | MIDIGob<br>AutomationControlAssign | Use this command to automatically assign MIDI channel number and continuous controller ID to an automation control for sending MIDI-Gobs.<br>After receiving this command, the MIDI channel number and controller ID of the next MIDI continuous controller (CC) message sent to the Automation Control port will be assigned to the automation control for specified MIDI- Gob.<br>CmdArg 1: MIDI-Gob ID (1 – MIDIInfo:MIDIGobMax) (1 byte). Use 0 to cancel a previous MIDIGobAutomationControlAssign command.<br>CmdArg 2: Automatic port route value (see below).<br>When the assignment is made, MIDI channel number and continuous controller ID are sent to the host in a NotParmVal message containing a MIDIPortInfo:MIDIGobControl parameter if NotifyAutomationControlAssign notifications are registered. |
| 0x06 | HardwareOutputAutomationControlAssign | Use this command to automatically assign MIDI channel number and continuous controller ID to an automation control for setting hardware control outputs. After receiving this command, the MIDI channel number and controller ID of the next MIDI continuous controller (CC) message sent to the Automation Control port will be assigned to the automation control for specified hardware control output.<br>CmdArg 1: HWPortID (1 – DeviceInfo:CtrlPortCount) (1 byte). Use 0 to cancel a previous HardwareOutputAutomationControlAssign command.<br>CmdArg 2: Automatic port route value (see below).<br>When the assignment is made, MIDI channel number and continuous controller ID are sent to the host in a NotParmVal message containing a MIDIPortInfo:HardwareOutputControl parameter if NotifyAutomationControlAssign notifications are registered. |
| 0x07 | MIDIGobSend | Send MIDI-Gob.<br>CmdArg 1: MIDI-Gob ID (1 – MIDIInfo:MIDIGobMax) (1 byte). |
| 0x08 | USBHMIDIReserve | Use this command to connect USB Host Controller ports to specific MIDI ports on hosted USB-MIDI devices.<br>CmdArg1: MIDI port ID on device (1 – MIDInfo:PortCount) (1 byte).<br>Must be a USB Host Controller port.<br>CmdArg2: USB Host controller ID (1 – DeviceInfo: USBHPortCount) (1 byte).<br>CmdArg3: USBHMIDIID (1–USBHMIDIPortCount) (1 byte).<br>CmdArg4: MIDI port on hosted device (1 – 16) (1 byte).<br>Must be a valid port number for the hosted device.<br>Use 0x00 for CmdArgs 2, 3, and 4 to delete a reservation. Creating a reservation will set the MIDIPortInfo:USBHReserve parameter. Deleting a reservation will clear the MIDIPortInfo:USBHReserve parameter. |
| 0x09 | KybdRptGrpAutomationControlAssign | Use this command to automatically assign MIDI channel number and continuous controller ID to an automation control for sending keyboard report groups. After receiving this command, the MIDI channel number and controller ID of the next MIDI continuous controller (CC) message sent to the Automation Control port will be assigned to the automation control for specified keyboard report group.<br>CmdArg 1: KybdRptGrpID (1 – DeviceInfo:KybdRptGrpMax) (1 byte). Use 0 to cancel a previous KybdRptGrpAutomationControlAssign command.<br>CmdArg 2: Automatic port route value (see below).<br>When the assignment is made, MIDI channel number and continuous controller ID are sent to the host in a NotParmVal message containing a MIDIPortInfo:KybdRptGrpControl parameter if NotifyAutomationControlAssign notifications are registered. |
| 0x0A | AudioGlobalMuteAutomationControlAssign | Use this command to automatically assign MIDI channel number and continuous controller ID to an automation control for enabling/disabling audio global mute. After receiving this command, the MIDI channel number and controller ID of the next MIDI continuous controller (CC) message sent to the Automation Control port will be assigned to automation control for audio global mute.<br>CmdArg 1: Automation control for audio global mute ID (1 byte):<br>0x01: enable/disable audio global mute<br>Use 0 to cancel a previous AudioGlobalMuteAutomationControlAssign command.<br>CmdArg 2: Automatic port route value (see below).<br>When the assignment is made, MIDI channel number and continuous controller ID are sent to the host in a NotParmVal message containing a MIDIPortInfo:AudioGlobalMuteControl parameter if NotifyAutomationControlAssign notifications are registered. |

**Automatic Port Route:**

Commands that set automation control assignments require that MIDI input ports receiving the continuous controller (CC) message be routed to the Automation Control port before receiving the CC message. This routing can be done automatically using the automatic port route command argument. When set to a non-zero value, all MIDI inputs ports will be temporarily routed to the Automation Control port until the assignment is made or cancelled. After making the assignment, the MIDI input port that received the CC message will be routed to the Automation Control port.

| Value | Description |
|---|---|
| 0x00 | Do not add MIDI port to Automation Control port route. |
| 0x01 | Add MIDI port to Automation Control port to preset route for scene A only. |
| 0x02 | Add MIDI port to Automation Control port to preset route for scene B only. For products that do not support multiple scenes this will have no affect. |
| 0x03 | Add MIDI port to Automation Control port to preset route for both scenes A and B. For products that do not support multiple scenes this will affect scene A only. |
| 0x04 | Add MIDI port to Automation Control port global route. |

_Example: Panic_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x09                    // message length (9)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x06, 0x06                    // data block #1: size (6), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x03                          // block #1: CmdSize: (3)
0x06                          // block #1: CmdID: MIDIOperation (6)
0x01                          // block #1: CmdVal: Panic (1)
0xxx                          // checksum
0xF7                          // footer
```

_Example: AudioAutomationControlAssign_

```text
0xF0, 0x00, 0x01, 0x73, 0x7D  // header
0x00, 0x05                    // product ID
0x01, 0x02, 0x03, 0x04, 0x05  // serial number
0x00, 0x00, 0x00, 0x00        // session ID
0x00, 0x00, 0x00, 0x00        // transaction ID
0x00, 0x0B                    // message length (11)
0x11, 0x00                    // message class, data class
0x01                          // NumDataBlock: (1)
0x08, 0x06                    // data block #1: size (8), type (CmdVal)
0x01                          // NumCmdValBlock: (1)
0x05                          // block #1: CmdSize: (5)
0x06                          // block #1: CmdID: MIDIOperation (6)
0x02                          // block #1: CmdVal: AudioAutomationControlAssign (2)
0x01                          // block #1: CmdArg: ACAID (1)
0x00                          // block #1: CmdArg: AutomaticPortRoute (0)
0xxx                          // checksum
0xF7                          // footer
```

### DeviceOperation (Command ID = 0x07)

DeviceOperation commands are used to trigger device operations, functions, and events.

| Val | Name | Description |
|---|---|---|
| 0x01 | KybdRptGrpSend | Send keyboard report group.<br>CmdArg 1: Keyboard report group ID (1 – DeviceInfo:KybdRptGrpMax) (1 byte). |

## Bulk Data Transfer

The BulkTransfer message and BulkData data class are used to perform bulk data transfer operations (i.e. backup and restore) between a device and a host (or between two devices where one device acts as a host). During a backup, the device being backed up sends BulkTransfer messages containing an image of the global or preset settings. During a restore, the device being restored receives BulkTransfer messages containing an image of the global or preset settings. Bulk data transfer operations support handshaking using the BulkAck parameter. Handshaking can speed up the data transfer but handshaking is not a requirement. Transaction ID (in the sysex header) is not incremented with each message; the entire bulk data transfer is considered to be a single transaction. The SequenceNumber parameter is used to ensure that all BulkTransfer messages are present in a bulk data transfer operation. The SequenceNumber is 1 on the first message and increments by 1 on each subsequent message. In handshaking mode, the receiver sends a BulkTransfer message to the sender with a BulkAck parameter after each BulkTransfer message is received. If handshaking is used the receiver can request a resend of the last message by setting the BulkAck parameter value to Resend. The receiver can cancel the bulk data transfer operation by setting the BulkAck parameter value to Abort. Aborting works regardless of whether or not handshaking is used for the bulk data transfer operation.

### BulkHdr (Data Block Type = 0x70)

This data block is used to define different packet types used in a bulk data transfer operation. This data block is always the first data block in a BulkTransfer message. There is only ever one BulkHdr data block.

Data Block Content:

Byte 3: Bulk packet type, see table below

Bytes 4-7: Sequence number (4-bytes, 28x4 format). Value is always 1 for the first message and increments by 1 on every subsequent message. Bytes 8-N: Additional data, depends on bulk packet type, see table below.

**Bulk Packet Types:**

| Type | Description |
|---|---|
| 0x01 | BulkStart: Always the first message in a bulk data transfer operation.<br>Additional data:<br>Bytes 8 – 9: device product ID (14x2 format)<br>Bytes 10 – 14: device serial number (32x5 format)<br>Bytes 15 – 18: device firmware version (28x4 format)<br>Byte 19: number of chapters that are used in this bulk data transfer operation |
| 0x02 | BulkEnd: Always the last message in a bulk data transfer operation. No additional data bytes. |
| 0x03 | ChapterStart: Indicates the start of a new chapter.<br>Additional data:<br>Byte 8: chapter number (1 – N)<br>Byte 9: preset number for this chapter (1 – N). Use 0 for globals. |
| 0x04 | ChapterEnd: Indicates the end of the current chapter. No additional data bytes. |
| 0x05 | PageData: Indicates that this message contains page data. No additional data bytes are included in this data block but additional bytes are included in this message containing page data. |
| 0x40 | BulkAck: Used during handshaking mode or to cancel a bulk data transfer operation.<br>Additional data:<br>Byte 8: error code:<br>0 = OK: no error<br>1 = Resend: request resend of last BulkTransfer message<br>2 = Abort: abort bulk data transfer |

_Example: BulkAck data block (no error)_

```text
0x08                    // DataBlockSize: (8)
0x70                    // DataBlockType: (BulkHdr)
0x40                    // Bulk Packet Type: (BulkAck)
0x00, 0x00, 0x00, 0x02  // Sequence Number: (2)
0x00                    // Error code (no error)
```

### Backup Operation

A backup operation begins when a device receives a BulkRequest command. The device will acknowledge the command by sending an Ack message as a response. The device will then pause for several seconds before sending BulkTransfer messages. The first message contains a BulkStart packet to delineate the start of the backup operation. This is then followed by additional BulkTransfer messages containing ChapterStart, ChapterEnd, and PageData packets. A backup operation concludes with a BulkTransfer message containing a BulkEnd packet. While the backup operation is commencing, the host can send BulkTransfer messages with a BulkAck packet to the device but this is not a requirement. After sending the first BulkTransfer message (the one containing the BulkStart packet), the device will wait a short time for a BulkTransfer message with a BulkAck packet set to OK (or Resend) to continue the backup operation in handshaking mode. If no handshake message is received the device will not use handshaking and will continue the backup operation at a leisurely pace. If handshaking is used, the host must send a BulkTransfer message with a BulkAck packet after every BulkTransfer message that it receives, failure to do so will result in the device aborting the backup operation. A backup operation can be cancelled by sending a BulkTransfer message to the device with a BulkAck packet set to Abort. Aborting works regardless of whether or not handshaking is used.

Even though there are multiple messages involved with a backup operation, the transaction ID in the sysex header does not increment; the device will use the transaction ID from the BulkRequest command in all BulkTransfer messages. If the host uses handshaking, the host should not increment the transaction ID in the handshaking messages. The host can return the SequenceNumber of the most recently received BulkTransfer message in the BulkAck packet but this is not required (any value can be used for SequenceNumber).

The host does not need to be concerned about parsing the content of the BulkTransfer messages; the host only needs to be able to save the entire stream of BulkTransfer messages to a file which can then be sent back to the device in a restore operation.

_Example: Backup, no handshaking_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkRequest: Trans ID = 37 |  |
| 2 |  | Device returns Ack message (no error).<br>Device enters backup mode. |
| 3 |  | Send BulkStart: TransID = 37, SeqNum = 1 |
| 4 |  | Wait a short time for BulkAck (none received).<br>Handshaking is not used. |
| 5 |  | Send ChapterStart: TransID = 37, SeqNum = 2 |
| 6 |  | Send PageData: TransID = 37, SeqNum = 3 |
| 7 |  | Send ChapterEnd: TransID = 37, SeqNum = 4 |
| 8 |  | Send BulkEnd: TransID = 37, SeqNum = 5 Device exits backup mode. |

_Example: Backup with handshaking, no errors_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkRequest: Trans ID = 37 |  |
| 2 |  | Device returns Ack message (no error).<br>Device enters backup mode. |
| 3 |  | Send BulkStart: TransID = 37, SeqNum = 1 |
| 4 | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK | Wait a short time for BulkAck (OK received).<br>Handshaking is used. |
| 5 |  | Send ChapterStart: TransID = 37, SeqNum = 2 |
| 6 | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |  |
| 7 |  | Send PageData: TransID = 37, SeqNum = 3 |
| 8 | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = OK |  |
| 9 |  | Send ChapterEnd: TransID = 37, SeqNum = 4 |
| 10 | Send BulkAck: TransID = 37, SeqNum = 4<br>ErrCode = OK |  |
| 11 |  | Send BulkEnd: TransID = 37, SeqNum = 5 |
| 12 | Send BulkAck: TransID = 37, SeqNum = 5<br>ErrCode = OK |  |
| 13 |  | Device exits backup mode. |

_Example: Backup with handshaking, host requests a resend_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkRequest: Trans ID = 37 |  |
| 2 |  | Device returns Ack message (no error).<br>Device enters backup mode. |
| 3 |  | Send BulkStart: TransID = 37, SeqNum = 1 |
| 4 | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK | Wait a short time for BulkAck (rOK eceived).<br>Handshaking is used. |
| 5 |  | Send ChapterStart: TransID = 37, SeqNum = 2 |
| 6 | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |  |
| 7 |  | Send PageData: TransID = 37, SeqNum = 3 |
| 8 | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = Resend |  |
| 9 |  | Resend PageData: TransID = 37, SeqNum = 3 |
| 10 | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = OK |  |
| 11 |  | Send ChapterEnd: TransID = 37, SeqNum = 4 |
| 12 | Send BulkAck: TransID = 37, SeqNum = 4<br>ErrCode = OK |  |
| 13 |  | Send BulkEnd: TransID = 37, SeqNum = 5 |
| 14 | Send BulkAck: TransID = 37, SeqNum = 5<br>ErrCode = OK |  |
| 15 |  | Device exits backup mode. |

_Example: Backup with handshaking, host aborts backup_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkRequest: Trans ID = 37 |  |
| 2 |  | Device returns Ack message (no error).<br>Device enters backup mode. |
| 3 |  | Send BulkStart: TransID = 37, SeqNum = 1 |
| 4 | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK | Wait a short time for BulkAck (OK received).<br>Handshaking is used. |
| 5 |  | Send ChapterStart: TransID = 37, SeqNum = 2 |
| 6 | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |  |
| 7 |  | Send PageData: TransID = 37, SeqNum = 3 |
| 8 | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = Abort |  |
| 9 |  | Device exits bulk backup mode. |

### Restore Operation

A restore operation begins when the device receives a BulkTransfer message with a BulkStart packet. The device will then send a BulkTransfer message to the host containing a BulkAck packet for every BulkTransfer message that it receives. If handshaking is used (by the host) the host should parse the received BulkAck packet and resend messages or abort as requested by the device. If handshaking is not used (by the host) the BulkTransfer messages should be sent at a leisurely rate so as to give the device enough time to deal with the incoming data. If the message stream is interrupted or if there is too long a delay between messages the device will terminate the restore operation. The restore operation can also be terminated by the host sending a BulkAck packet set to Abort, the device will respond with a BulkAck packet also set to Abort.

Even though there are multiple messages involved with a restore operation, the transaction ID in the sysex header does not increment; the host should use the same transaction ID in all BulkTransfer messages and use the SequenceNumber parameter to differentiate between the messages.

The host does not need to be concerned about parsing the content of the BulkTransfer messages that it sends to the device; the host only needs to be able to send each message to the device and respond to BulkAck errors reported from the device.

_Example: Restore with handshaking, no errors_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkStart: TransID = 37, SeqNum = 1 | Device enters restore mode. |
| 2 |  | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK |
| 3 | Send ChapterStart: TransID = 37, SeqNum = 2 |  |
| 4 |  | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |
| 5 | Send PageData: TransID = 37, SeqNum = 3 |  |
| 6 |  | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = OK |
| 7 | Send ChapterEnd: TransID = 37, SeqNum = 4 |  |
| 8 |  | Send BulkAck: TransID = 37, SeqNum = 4<br>ErrCode = OK |
| 9 | Send BulkEnd: TransID = 37, SeqNum = 5 |  |
| 10 |  | Send BulkAck: TransID = 37, SeqNum = 5<br>ErrCode = OK<br>Device exits restore mode. |

_Example: Restore with handshaking, device requests a resend_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkStart: TransID = 37, SeqNum = 1 | Device enters restore mode. |
| 2 |  | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK |
| 3 | Send ChapterStart: TransID = 37, SeqNum = 2 |  |
| 4 |  | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |
| 5 | Send PageData: TransID = 37, SeqNum = 3 |  |
| 6 |  | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = Resend |
| 7 | Resend PageData: TransID = 37, SeqNum = 3 |  |
| 8 |  | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = OK |
| 9 | Send ChapterEnd: TransID = 37, SeqNum = 4 |  |
| 10 |  | Send BulkAck: TransID = 37, SeqNum = 4<br>ErrCode = OK |
| 11 | Send BulkEnd: TransID = 37, SeqNum = 5 |  |
| 12 |  | Send BulkAck: TransID = 37, SeqNum = 5<br>ErrCode = OK<br>Device exits restore mode. |

_Example: Restore with handshaking, device aborts transfer_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkStart: TransID = 37, SeqNum = 1 | Device enters restore mode. |
| 2 |  | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK |
| 3 | Send ChapterStart: TransID = 37, SeqNum = 2 |  |
| 4 |  | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |
| 5 | Send PageData: TransID = 37, SeqNum = 3 |  |
| 6 |  | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = Abort<br>Device exits restore mode. |
| 7 | Host aborts restore operation. |  |

_Example: Restore with handshaking, device is not able to perform a restore operation at this time_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkStart: TransID = 37, SeqNum = 1 | Device is not able to perform a restore operation. |
| 2 |  | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = Abort |
| 3 | Host aborts restore operation. |  |

_Example: Restore with handshaking, host aborts transfer_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkStart: TransID = 37, SeqNum = 1 | Device enters restore mode. |
| 2 |  | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK |
| 3 | Send ChapterStart: TransID = 37, SeqNum = 2 |  |
| 4 |  | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |
| 5 | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = Abort |  |
| 6 |  | Send BulkAck: TransID = 37, SeqNum = 3<br>ErrCode = Abort<br>Device exits restore mode. |
| 7 | Host aborts restore operation. |  |

_Example: Restore with handshaking, host sends wrong sequence number_

| step | Host | Device |
|---|---|---|
| 1 | Send BulkStart: TransID = 37, SeqNum = 1 | Device enters restore mode. |
| 2 |  | Send BulkAck: TransID = 37, SeqNum = 1<br>ErrCode = OK |
| 3 | Send ChapterStart: TransID = 37, SeqNum = 2 |  |
| 4 |  | Send BulkAck: TransID = 37, SeqNum = 2<br>ErrCode = OK |
| 5 | Send ChapterEnd: TransID = 37, SeqNum = 4 |  |
| 6 |  | Send BulkAck: TransID = 37, SeqNum = 4<br>ErrCode = Abort<br>Device exits restore mode. |
| 7 | Host aborts restore operation. |  |

## Frequently Used Data Conversions

System exclusive messages require that the most significant bit of each byte be clear. Since many of the parameter values do not meet this requirement, some parameters need to be converted into a system exclusive friendly format. The following encodings are frequently used.

### 32x5: 32-bit value encoded into 5 bytes

Value as seen in memory (big endian, b31 is the most significant bit):

```text
Byte #1: b31, b30, b29, b28, b27, b26, b25, b24
Byte #2: b23, b22, b21, b20, b19, b18, b17, b16
Byte #3: b15, b14, b13, b12, b11, b10, b09, b08
Byte #4: b07, b06, b05, b04, b03, b02, b01, b00
```

Value as seen in sysex message (the most significant four bits of byte #1 are clear, the most significant bit of bytes #2-5 are clear):

```text
Byte #1: 000, 000, 000, 000, b31, b30, b29, b28
Byte #2: 000, b27, b26, b25, b24, b23, b22, b21
Byte #3: 000, b20, b19, b18, b17, b16, b15, b14
Byte #4: 000, b13, b12, b11, b10, b09, b08, b07
Byte #5: 000, b06, b05, b04, b03, b02, b01, b00
```

_Example: IP address = 192.168.1.100_

Value as seen in memory: 0xC0A80164 Value as seen in sysex message: 0x0C, 0x05, 0x20, 0x02, 0x64

### 28x4: 28-bit value encoded into 4 bytes

Value as seen in memory (big endian, value is stored as a 32-bit value with the most significant four bits clear, b27 is the most significant bit):

```text
Byte #1: 000, 000, 000, 000, b27, b26, b25, b24
Byte #2: b23, b22, b21, b20, b19, b18, b17, b16
Byte #3: b15, b14, b13, b12, b11, b10, b09, b08
Byte #4: b07, b06, b05, b04, b03, b02, b01, b00
```

Value as seen in sysex message (the most significant bit of bytes #1-4 are clear):

```text
Byte #1: 000, b27, b26, b25, b24, b23, b22, b21
Byte #2: 000, b20, b19, b18, b17, b16, b15, b14
Byte #3: 000, b13, b12, b11, b10, b09, b08, b07
Byte #4: 000, b06, b05, b04, b03, b02, b01, b00
```

_Example: Session ID = 19,088,743 (decimal)_

Value as seen in memory (big endian): 0x01234567 Value as seen in sysex message: 0x09, 0x0D, 0x0A, 0x67

### 16x3: 16-bit value encoded into 3 bytes

Value as seen in memory (big endian, b15 is the most significant bit):

```text
Byte #1: b15, b14, b13, b12, b11, b10, b09, b08
Byte #2: b07, b06, b05, b04, b03, b02, b01, b00
```

Value as seen in sysex message (the most significant six bits of byte #1 are clear, the most significant bit of bytes #2-3 are clear):

```text
Byte #1: 000, 000, 000, 000, 000, 000, b15, b14
Byte #2: 000, b13, b12, b11, b10, b09, b08, b07
Byte #3: 000, b06, b05, b04, b03, b02, b01, b00
```

_Example: Headphone attenuation of -2.5 dB (in 8.8 format):_

Value as seen in memory: 0xFD80 Value as seen in sysex message: 0x03, 0x7B, 0x00

### 14x2: 14-bit value encoded into 2 bytes

Value as seen in memory (big endian, value is stored as a 16-bit value with the most significant two bits clear, b13 is the most significant bit):

```text
Byte #1: 000, 000, b13, b12, b11, b10, b09, b08
Byte #2: b07, b06, b05, b04, b03, b02, b01, b00
```

Value as seen in sysex message (the most significant bit of bytes #1-2 are clear):

```text
Byte #1: 000, b13, b12, b11, b10, b09, b08, b07
Byte #2: 000, b06, b05, b04, b03, b02, b01, b00
```

_Example: Meter value of +1.5 dB (9736 where 0 dB = 8192):_

Value as seen in memory: 0x2608 Value as seen in sysex message: 0x4C, 0x08

### BAx2: byte array of length N encoded into byte array of length 2N

Each byte of the input array is split into two bytes such that the most significant nibble of each byte is 0 and the least significant nibble contains either the least significant nibble of the input byte, or the most significant nibble of the input byte. The least significant nibble occurs in the first byte, the most significant nibble occurs in the second byte.

Value as seen in memory (lowest address first):

```text
Byte #1: b07, b06, b05, b04, b03, b02, b01, b00
Byte #2: b15, b14, b13, b12, b11, b10, b09, b08
```

Value as seen in sysex message (the most significant four bits of bytes #1-4 are clear):

```text
Byte #1: 000, 000, 000, 000, b03, b02, b01, b00
Byte #2: 000, 000, 000, 000, b07, b06, b05, b04
Byte #3: 000, 000, 000, 000, b11, b10, b09, b08
```

```text
Byte #4: 000, 000, 000, 000, b15, b14, b13, b12
```

_Example: one byte encoded into two bytes_ Value as seen in memory: 0xDA Value as seen in sysex message: 0x0A, 0x0D

_Example: two bytes encoded into four bytes_

Value as seen in memory (lowest address first): 0xDA, 0x95 Value as seen in sysex message: 0x0A, 0x0D, 0x05, 0x09

_Example: 6 bytes encoded into 12 bytes:_

Value as seen in memory (lowest address first): 0xAC, 0x7A, 0x42, 0x12, 0x34, 0x56 Value as seen in sysex message: 0x0C, 0x0A, 0x0A, 0x07, 0x02, 0x04, 0x02, 0x01, 0x04, 0x03, 0x06, 0x05

## Product IDs

| PID | Device |
|---|---|
| 1 (0x0001) | iConnectMIDI (does not support the commands in this document) |
| 2 (0x0002) | mio10 (does not support the commands in this document*) |
| 3 (0x0003) | mio (does not support the commands in this document*) |
| 4 (0x0004) | iConnectMIDI1 (does not support the commands in this document*) |
| 5 (0x0005) | iConnectMIDI2+ (does not support the commands in this document*) |
| 6 (0x0006) | iConnectMIDI4+ (does not support the commands in this document*) |
| 7 (0x0007) | iConnectAUDIO4+ (does not support the commands in this document*) |
| 8 (0x0008) | iConnectAUDIO2+ (does not support the commands in this document*) |
| 9 (0x0009) | mio2 (does not support the commands in this document*) |
| 10 (0x000A) | mio4 (does not support the commands in this document*) |
| 11 (0x000B) | PlayAUDIO12 (does not support the commands in this document*) |
| 12 (0x000C) | AUDIO4c (does not support the commands in this document*) |
| 13 (0x000D) | ConnectAUDIO2/4 (does not support the commands in this document*) |
| 14 (0x000E) | mioXL |
| 15 (0x000F) | mioXM |
| 16 (0x0010) | mioXC |
| 17 (0x0011) | PlayAUDIO1U |
| 18 (0x0012) | PlayAUDIO2U |
| 19 (0x0013) | PlayAUDIO2D |

*These products are covered by the companion doc **_Common System Exclusive Commands_** (the non-TNG doc for products before mioXL).

## History

| # | Date/Author | Changes |
|---|---|---|
| 5 | 2026-11-20<br>S. Juskiw | 1. Fixed several errors in the examples. |
| 4 | 2026-02-04<br>S. Juskiw | 1. Added AOIP audio clock type (0x03), AOIP audio port type (0x05),<br>AudioInfo:AOIPPortCount parameter (0x06), AOIP hardware port type (0x09) and DeviceInfo:AOIPPortCount parameter (0x26).<br>2. Added AudioChannelInfo:ChannelLink parameter (0x08).<br>3. Added AudioPortInfo:PortChannelLink parameter (0x0C). |
| 3 | 2025-11-21<br>S. Juskiw | 1. Added toggle failover arm/disarm state (2) to<br>DeviceFeature:FailoverArmStatus (0x10). |
| 2 | 2025-09-04<br>S. Juskiw | 1. Add timestamp mode (bit 3) to MIDIPortInfo:EthSesnFlags (0x30).<br>2. HardwareInfo:EthIPMode (0x34) has been changed to EthConfigFlags. Bit 0 configures IP mode, bit 1 configures multicast mode.<br>3. Added SaveLoadInit:LoadPresetNP (0x44), InitGlobalRAM (0x11),<br>InitPresetRAM (0x12), InitGlobalNV (0x13) and InitPresetNV (0x14) commands. |
| 1 | 2025-07-01<br>S. Juskiw | 1. Cleanup and first official non-beta release. |
| 1b37 | 2025-06-01<br>S. Juskiw | 1. Added AudioInfo:AudioGlobalMute parameter (0x43).<br>2. Added MIDIPortInfo:AudioGlobalMuteControl parameter (0x4A).<br>3. Added MIDIOperation:AudioGlobalMuteAutomationControlAssign command (0x0A).<br>4. Added bit 3 (momentary switch) to sub-ID 0x01 of Automation Control<br>Parameter Block.<br>5. Added bit 5 for DeviceInfo:SystemEventSupportFlags parameter (0x1F).<br>6. Added DeviceFeature:SysEvntAudioGlobalMuteActive and Inactive parameters (0x2B and 0x2C).<br>7. Added sub-id 0x11 to System Event Parameter Block (audio global mute).<br>8. Added bit 4 to sub-id 0x05 of SetGroupReset Parameter Block (clear system events for audio global mute).<br>9. Added toggle option (2) to HardwareInfo:CtrlState parameter (0x44). |
| 1b36 | 2025-04-16<br>S. Juskiw | 1. Added DeviceInfo:EthSwitchPortCount parameter (0x25).<br>2. Added Ethernet Switch (0x08) to Hardware Port Types.<br>3. Added HardwareInfo:Ethernet Switch Port parameters 0x50 – 0x52. |
| 1b35 | 2025-04-09<br>S. Juskiw | 1. Added bit 5 to DeviceFeature:FailoverConfigFlags parameter.<br>2. The “Panel Buttons” are now called “Control Buttons”. |
| 1b34 | 2024-08-28<br>S. Juskiw | 1. Updated AudioPortInfo:MixerSnapshotName parameter.<br>2. Updated MIDIPortInfo:PortActiveFlags parameter.<br>3. Updated the SetGroup:Reset command bitmaps for MIDI preset and global parameters (sub-ids 0x01 and 0x02).<br>4. Updated DeviceFeature:PresetName parameter.<br>5. Added DeviceFeature:FailoverToneFlags parameter (0x19). |
| 1b33 | 2024-03-02<br>S. Juskiw | 1. Added DeviceInfo:AnalogAudioPortCount parameter (0x24).<br>2. Updated the notes for Ethernet connection notifications.<br>3. Updated DeviceInfo:PanelButtonColourSupportFlags parameter. |
| 1b32 | 2023-11-18<br>S. Juskiw | 1. Updates to the DeviceInfo display parameters: 0x1B through 0x1E and 0x42 through 0x45. |
| 1b31 | 2023-11-13<br>S. Juskiw | 1. Added sub-ids 0x03 through 0x05 to SetGroupReset parameter block for clearing system events. |
| 1b30 | 2023-11-10<br>S. Juskiw | 1. Added PanelButtonMax and PanelButtonColourSupportFlags parameters to<br>DeviceInfo section.<br>2. Added PanelButtonID ArgID.<br>3. Added Panel Buttons section to DeviceFeature data class.<br>4. Added NotifyPanelButtonState notification class to Notification section.<br>5. Added SysEvntPanelButtonInactive, SysEvntPanelButtonSglClick and<br>SysEvntPanelButtonDblClick parameters to DeviceFeature:SystemEvents section.<br>6. Added bit 4 (SystemEventPanelButton) to<br>DeviceInfo:SystemEventSupportFlags parameter. |
| 1b29 | 2023-09-28<br>S. Juskiw | 1. Added HardwareInfo:USBDEnableHIDKybd parameter.<br>2. Added bit 3 (HID keyboard) to HardwareInfo:USBDFlags parameter.<br>3. Added DevinceInfo:KybdRptGrpMax and DeviceInfo:KybdRptGrpNameMax parameters.<br>4. Added Keyboard Reports to DeviceFeature section.<br>5. Added DeviceOperation to Commands section.<br>6. Changed the nomenclature for control ports from “momentary/toggle” to “momentary/latching” to avoid confusion with other sections that use toggle to indicate “change to opposite state”.<br>7. Fix errors in BAx2 examples (Frequently Used Data Conversions section) and<br>HardwareInfo:EthMACAddress parameter.<br>8. Added MIDIPortInfo:KybdRptGrpControl parameter and<br>MIDIOperation:KybdRptGrpAutomationControlAssign command for sending keyboard report groups using MIDI controllers.<br>9. Added sub-ID 0x10 to system event parameter block for sending keyboard report groups.<br>10. Added KybdRptGrpID ArgID. |
| 1b28 | 2023-06-18<br>S. Juskiw | 1. Added additional bits to SetGroup subid 0x01 for both preset and global MIDI settings.<br>2. Changed all the green text to black text (some audio parameters and args were using green text to make them easier to find).<br>3. Added Automatic Port Route to MIDIOperation automation control assignment commands.<br>4. Added Pitch Bend Off to MIDIPanic:PanicMessages. |
| 1b27 | 2023-03-11<br>S. Juskiw | 1. Added toggle flag to automation control parameter block (sub-ID 0x01, bit 2).<br>2. Removed MIDIInfo:AMPAlgUserDataMax and MIDIFeature:AMPAlgUserData parameters.<br>3. Added DeviceFeature:FailoverToneVolumeMax parameter and added bit 4 to<br>DeviceFeature:FailoverConfigFlags. |
| 1b26 | 2022-11-10<br>S. Juskiw | 1. Added AudioInfo:AudioFrameWaitSupport parameter. Clean up other<br>FrameWait parameters.<br>2. Added Notification classes NotifyUSBDeviceConnectState,<br>NotifyUSBHostConnectState and NotifyEthernetConnectState.<br>3. Added MIDIOperation:USBHMIDIReserve command to set USB host port reservations. |
| 1b25 | 2022-10-17<br>S. Juskiw | 1. Rewrite of Notification commands section.<br>2. Added Failover Selector parameters to MIDI automation control port.<br>3. Added MIDI-Gob parameters, data blocks and ArgID.<br>4. Added several automation control assignment commands for MIDIOperation.<br>5. Added System Events section. |
| 1b24 | 2022-08-26<br>S. Juskiw | 1. Added DeviceInfo parameters: DisplayBrightnessMax, DisplayContrastMax,<br>DisplaySupportFlags, DisplayBrightness, DisplayContrast, DisplayEnableFlags, DisplayTimeout.<br>2. Added HardwareInfo parameters: USBHMIDIPortNameIn,<br>USBHMIDIPortNameOut.<br>3. Added MIDIPortInfo parameters: USBHPortNameIn, USBHPortNameOut,<br>USBHPortNameInR, USBHPortNameOutR.<br>4. Added AudioAutomationInfo data classvand ACAID ArgID.<br>5. Added AudioInfo parameters: AudioAutomationControlMax and<br>AudioChannelMax.<br>6. Program Selector (control port) is now called Automation Control and is used for selecting presets, scenes, mixer snapshots, and changing audio controls for analog and mixer ports using the AudioAutomationInfo data class.<br>7. Added MIDIOperation:AudioAutomationControlAssign command. |
| 1b23 | 2022-07-30<br>S. Juskiw | 1. Preset/Scene selector is now called Program selector and can select presets, scenes and mixer snapshots.<br>2. Added MIDIInfo:ProgramSelectorSupportFlags parameter.<br>3. Added AudioChannelInfo:ChannelBitmapDST/SRC parameters.<br>4. Added parameters for hardware control ports.<br>5. Added MIDIPortInfo:USBDPortNameIn (0x1E) parameter.<br>MIDIPortInto:USBDPortName (0x1F) has been renamed USBPortNameOut. |
| 1b22 | 2022-01-24<br>S. Juskiw | 1. Added support for audio mixer snapshots. |
| 1b21 | 2021-12-16<br>S. Juskiw | 1. Added MIDIFeature:Panic parameters.<br>2. Added MIDIOperation:Panic command.<br>3. Added MIDIInfo:PanicSupportFlags parameter. |
| 1b20 | 2021-11-18<br>S. Juskiw | 1. Various changes to audio data classes and parameters. |
| 1b19 | 2021-04-08<br>S. Juskiw | 1. Complete rewrite of MIDIPortInfo:Preset/SceneSelector for control ports.<br>2. Changed name of MIDIPortInfo:PortRoute parameter to<br>MIDIPortInfo:PortRoutePreset.<br>3. Added MIDIPortInfo:PortRouteGlobal parameter. |
| 1b18 | 2021-03-30<br>S. Juskiw | 1. Added ArgIDs: MIDICCProcessorID, MIDIMonitorEventID.<br>2. Added DeviceInfo:DisplayLevelMax parameter.<br>3. Added MIDIInfo parameters: PortMonitorDetailIconMax,<br>PortMonitorDetailEnableIn/Out, PortMonitorDetailDisplayIn/Out, CCProcessorMax.<br>4. Added MIDIPortInfo parameters: PortMonitorDetailIn/Out, CCProcessorIn/Out.<br>5. Changed IDs of MIDIInfo:PortMonitorIn/Out parameters.<br>6. Added CCProcessor flag to MIDInfo:PortFeature and<br>MIDIPortInfo:PortFeatureIn/Out parameters. Changed location of AMP flag so that is after CCProcessor flag.<br>7. Moved some of the parameter IDs for AMP so that they are after<br>CCProcessorIDs. |
| 1b17 | 2021-02-24<br>S. Juskiw | 1. Various changes to audio data classes and parameters. |
| 1b16 | 2021-01-08<br>S. Juskiw | 1. Added support for audio data classes and parameters. |
| 1b15 | 2020-06-27<br>S. Juskiw | 1. Changed SaveRestore commands to SaveLoad to avoid confusion with the<br>Backup/Restore operations. |
| 1b14 | 2020-04-23<br>S. Juskiw | 1. Complete rewrite of everything related to Bulk Data Transfer. |
| 1b13 | 2020-02-12<br>S. Juskiw | 1. Added SetGroup command. |
| 1b12 | 2019-09-26<br>S.Juskiw | 1. Fixed errors in the bitmap for MIDI system message filter flags #1. |
| 1b11 | 2019-09-12<br>S.Juskiw | 1. Added ArgID:PresetID. Modified DeviceFeature:PresetName to allow using<br>PresetID to get preset names from non-volatile storage without having to load each preset into the working area or shadow area. |
| 1b10 | 2019-06-07<br>S.Juskiw | 1. Added additional comments to several parameters for clarification.<br>2. Added USBHMIDIID ArgID for USB host controller MIDI ports (was previously using MIDIPortID which is a different argument with different values). |
| 1b9 | 2019-05-06<br>S.Juskiw | 1. Added USBH-MIDI parameters to HardwareInfo. |
| 1b8 | 2019-05-03<br>S.Juskiw | 1. Various changes to MIDIInfo Preset/Scene Selector parameters.<br>2. General cleanup of MIDIFeature:AMP parameters. |
| 1b7 | 2019-04-22<br>S.Juskiw | 1. Modifed the text descriptions for some of the MIDIPortInfo parameters. |
| 1b6 | 2019-04-13<br>S.Juskiw | 1. Various changes to MIDIPortInfo.<br>2. Added MIDIChannel ArgID and renumbered the ArgIDs.<br>3. Consolidated AMP ArgIDs and changed structures to use sub-IDs.<br>4. Added additional Ack message error codes. |
| 1b5 | 2019-03-29<br>S.Juskiw | 1. Moved several of the MIDIInfo parameters to HardwareInfo.<br>2. Changed the parameter IDs for HardwareInfo.<br>3. Added several parameters to MIDIInfo. |
| 1b4 | 2019-02-19<br>S. Juskiw | 1. Many changes to presets, scenes, areas and how these are managed.<br>2. Removed segment designator data block, merged into argument list.<br>3. Removed iOS related flags and parameters.<br>4. Added flags for USB device port speed and parameter for USB host jacks.<br>5. Changes to BAx2 data format. |
| 1b3 | 2018-10-18<br>S. Juskiw | 1. Removed CC filter and CC remap parameters (use AMP instead).<br>2. Changed system filters, channel filters, channel remaps.<br>3. Added USBDPortNameMax and EthSesnNameMax parameters.<br>4. Created PortConnectFlags and PortActiveFlags parameters and eliminated<br>USBHConnect, EthConnect parameters.<br>5. Changes to PortSupportFlags and PortEnableFlags.<br>6. Changes to segment designator data block.<br>7. Removed version bytes from everywhere.<br>8. Complete rewrite of the memory model.<br>9. Changes to SaveRestore command. |
| 1b2 | 2018-04-06<br>S. Juskiw | 1. Added support for parameter sub-IDs (break up some of the fixed structures). |
| 1b1 | 2018-03-25<br>S. Juskiw | 1. First version (RFC). No support for audio commands. |
