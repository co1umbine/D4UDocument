# Table of Contents
[TOC]

# 1. Overview
DynamixelforUnity (D4U) is an asset to easily implement Dynamixel motor control within in Unity3D. D4U provides numerous methods to interface with Dynamixel servomotors, enabling you to efficiently and easily connect, control, and read data from a variety of Dynamixel servomotors. 

Specificially, D4U focuses on the following feature sets:
- Automatically load the Control Table of the servomotor 
-  Enable simultaneous control of multiple servomotor models. 
- Enable Controlling Dynamixel servomotor with simple declarative code. 
- Abstracts low level servomotor and provides a number of classes to simplify servomotor controls.
- Enable advanced functions like, batch control of multiple models and unit conversion.
- Provide low-level communication and servomotor controls  within Unity3D (when required).


Therefore, users are able to completely focus on writing all robot control code from within Unity3D, using simple code in C# that can easily be integrated with a variety of applications, games, simulations or other experiences and use cases that leverage Unity3D.

## Supported OS
- Windows

## Supported protocol
- [DYNAMIXEL Protocol 2.0](https://emanual.robotis.com/docs/en/dxl/protocol2/)


## Supported Dynamixel models
Model compatible with DYNAMIXEL Protocol 2.0. 
- MX-28
- MX-64
- MX-106
- X Series (2X Series included)
- PRO Series
- P Series

# 2. Installation
With the Unity3D project you want to use, double-click DynamixelForUnity.unitypackage, or drag and drop it into the Project window of the Unity3D project. A pop-up will appear, please choose to improt the package and all its contents to your project. 

# 3. License Purchase and Activation

A license must be purchased from [HatsuMuv site](https://hatsumuv.myshopify.com/) in order to use D4U.


After the purcahse, you will recieve a link to a form, where you are required to provide various information, including device ID, which is a unique ID number that is used to link your license to the machine D4U will be used on: 
![License Registration Form](https://hackmd.io/_uploads/B19vxZZn3.jpg)

To generate your device ID and insert it in the above form, click **Get DeviceID for Registration** in your toolbar. It will automatically copy your device ID to your clip board and shows in consle window. Please follow the instructions shown in the screenshot below:
![Getting Device ID](https://hackmd.io/_uploads/r1W30l-2n.png)


After your purchase D4U, you will recieve an SN by email. To use your SN within D4U, please insert your Email and SN in the D4U verificate method or through the Dynamixel For Unity Component in Unity3D UI as shown below: 

![](https://hackmd.io/_uploads/Skw6tax3n.jpg)

To check your verify your activation, simply run your scene, and you can check if the activation is correct or not in consle. 




# 4. Quick Start 

Here is a demo that gets and sets the positions of two Dynamixel motors.

## Setting up a new scene
 
1. Open a new scene and drag and drop HatsuMuv/DynamixelForUnity/Prefabs/DynamixelForUnity into the hierarchy window.

![](https://hackmd.io/_uploads/SyBq4DX3n.png)

2. Select **DynamixelForUnity** gameobject in Hierarchy menu

![](https://hackmd.io/_uploads/Skw6tax3n.jpg)

3. In Inspector window, you can find a component called **DynamixelForUnity**. Enter your **Email** and **SN**, and the USB serial port which connected to your Dynamixel motor in the **Device Name** field. (You can find the port name from your **Device manager**, e.g. "COM3"). 

4. (Optional) If you know the baudrate of the motor you want to operate, set the Baud Rate in the inspector of DynamixelForUnity beforehand. If you don't know, leave it blank, D4U SetUp method will scan for you. (Which can be time comsuming)

5. "Click Add Component" Button, and input script name as you like (For this time, "Demo"). Then click "New script".

![](https://hackmd.io/_uploads/rkolrDX32.png)


6. Double click on the script called Demo written in gray text and wait for a while. Visual studio will open.

![](https://hackmd.io/_uploads/Bk0wIdNh2.png)


## Coding C# script

When Visual Studio opens, write as below. This code prints the current positions of two servo motors to Debug.Log and then sets their goal positions to 1024 each.
```csharp!
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using HatsuMuv.DynamixelForUnity.x64;

public class Demo : MonoBehaviour
{
    DynamixelForUnity d4u;
    
    void Start()
    {
        d4u = GetComponent<DynamixelForUnity>();
        d4u.ConnectDynamixel();
        d4u.SetUp();
       
        // Set operating mode to Position Control Mode
        d4u.SyncSetDataGroupMultiModels("OperatingMode", new uint[] { 3, 3 });
        
        // Torque enable
        d4u.SyncSetDataGroupMultiModels("TorqueEnable", new uint[] { 1, 1 });
        
        // Get and print present position
        int[] presentPosition = d4u.SyncGetSignedDataGroupMultiModels("PresentPosition");
        Debug.Log("PresentPosition: " + string.Join(", ", presentPosition));
    
        // Set goal position
        d4u.SyncSetDataGroupMultiModels("GoalPosition", new uint[] { 1024, 1024 });
        
    }
    
    void OnApplicationQuit()
    {
        // Torque disable
        uint[] torqueEnableCommand = new uint[] { 0, 0 };
        d4u.SyncSetDataGroupMultiModels("TorqueEnable", torqueEnableCommand);    
    }
}

```

## Execution
Return to the Unity screen and click the Play button.

The position of the servomotor will be printed on the console.
![](https://hackmd.io/_uploads/B1eOQU_423.png)



# 5. Script Reference

Select a namespace from the following according to your system configuration.

```
HatsuMuv.DynamixelForUnity.x64
HatsuMuv.DynamixelForUnity.x86
```


## DynamixelForUnity

DynamixelForUnity is a class provided to control Dynamixel motors with declarative program.

*Declarative programming: Describe what you want, not how to do it. For more details, please search for 'Declarative Programming'.*

One DynamixelForUnity component manage one port. If you want to control multiple ports simultaneously, please add as many components as the number of ports.

DynamixelForUnity is implementing IControlTableUser. 

Flow of Use:
1. Open a port with ConnectDynamixel.
2. (Optional) Keep the motor ID and model number used in SetUp or SetIDsForUse. This procedure allows for concise writing when sending instructions using only data field names or when sending instructions to all motors.
3. Control the motor with methods such as SetData, etc. Note that data in the EEPROM area such as OperatingMode can only be written when the torque is OFF.
5. At the end of the oparation, perform a termination procedure such as torque disable in the **OnApplicationQuit** or **OnDisable**. If the DynamixelForUnity is connected at the time when OnDestroy is called, it will call DisconnectDynamixel.

Here is Example.
```C#
using HatsuMuv.DynamixelForUnity.x64;

public class ExampleClass : MonoBehaviour
{
    DynamixelForUnity d4u;

    void Start() {
        d4u = GetComponent<DynamxielForUnity>();

        // If the SN code and email address are not set in the inspector, this code is necessary.
        // d4u.Verificate();
        
        d4u.ConnectDynamixel();
        d4u.SetUp();
        
       // something you want to do.
       d4u.SetData("TorqueEnable", 1, 1);
    }
    
    void OnApplicationQuit() {
       
       // Termination procedure
       d4u.SetData("TorqueEnable", 0, 1)
    }
}
```


### Constant

| Constant             | Value  | Description                                                  |
| ---------------- | --- | ----------------------------------------------------- |
| PROTOCOL_VERSION | 2   | DynamixelForUnity supports protocol version 2.0 only.     |


### Variables

| Name        | Description                       |
| ------------- | ----------------------------- |
| controlTables | An object that contains controlTable of multiple models. Functions that send commands from data field names refer to this. |
| showMessage   | If this is set to true, all execution logs of DynamixelForUnity will be output to <br>Debug.Log. Errors and warnings are not affected.  |

### Propertyies
|Name |　Description     |
|----|----|
| BaudRate   | Returns the current baud rate. Read only.|
| IDsForUse | ID array held in DynamixelForUnity. SetUp method and SetIDsForUse method. Read-only.
| ModelNumbers | Dictionary<byte, int> type. Represents the model number of the motor for which the ID is being held, set by the SetIDsForUse and SetUp methods. Read-only.|
| ConnectStatus | True/false value indicating whether or not there is a connection with the motor. Read only. |
| Varification  | True/False value indicating whether the license is authenticated. Read-only. |


### Verificate
- Declaration
`public bool Verificate(HatsuVerif verif)`

- Argument
    <dl>
        <dt>HatsuVerif verif</dt>
        <dd>Verification class object. Created from the license file path and verification information.</dd>
    </dl>

- Returns
    License validity.

- Description
This function is used for validating your license, which will be automatically called (Awake function) if you put DynamxielForUnity in your gameobject as component from beginning. If you new a instant after pressing play, you will need to run this method to validate your license before you start to use D4U's functions. 

----

- Declaration
`public bool Verificate(string userEmail, string userSN, bool forceRenew = false)
`

- Returns
    License validity.


### ConnectDynamixel
- Declaration
`public bool ConnectDynamixel()`
`public bool ConnectDynamixel(string deviceName)`
`public bool ConnectDynamixel(BaudRate baudRate)`
`public bool ConnectDynamixel(string deviceName, BaudRate baudRate)`

- Argument
    <dl>
        <dt>string deviceName</dt>
        <dd>Port name to connect. ex.) "COM3". </dd>
        <dt>BaudRate baudRate</dt>
        <dd>Baud rate to be connected.</dd>
    </dl>

- Returns
    Returns true if the connection succeeds, false otherwise.

- Description
Opens a port with the specified baud rate and connects to the Dynamixel motor. If no port or baud rate is specified, the value specified in the inspector is used.


### ScanInBaudRate
- Declaration
`public byte[] ScanInBaudRate(BaudRate baudRate, byte[] idsForScan = null)`

- Argument
    <dl>
        <dt>BaudRate baudRate</dt>
        <dd>Baud rate used for scanning.</dd>
        <dt>byte[] idsForScan</dt>
        <dd>The ID of the target to be scanned, used to search only for motors with a specific ID; if null is specified, all IDs are scanned.</dd>
    </dl>
    
- Returns
    Byte array of available motor IDs.

- Description
Ping the Dynamixel motor at the specified baud rate and scan it; use the SetBaudRate method to change to the specified baud rate.
---

- Declaration
`public byte[] ScanInBaudRate(BaudRate baudRate, byte minIDsForScan, byte maxIDsForScan)`

- Argument
    <dl>
        <dt>byte minIDsForScan</dt>
        <dd>Lower limit of the range for the ID of the object to be scanned. (Included in the range)</dd>
        <dt>byte maxIDsForScan</dt>
        <dd>Upper limit of the range for the ID of the object to be scanned. (Included in the range)</dd>
    </dl>
    
- Returns
    Byte array of available motor IDs. 
    
- Description
Ping the Dynamixel motor at the specified baud rate and scan it, using the SetBaudRate method to change to the specified baud rate. The scan target can be specified in a range. 

### ScanInBaudRateAsync
- Declaration
`public async Task<byte[]> ScanInBaudRateAsync(BaudRate baudRate, byte[] idsForScan = null, CancellationToken cancellationToken = default(CancellationToken))`

- Argument
    <dl>
        <dt>BaudRate baudRate</dt>
        <dd>Baud rate used for scanning.</dd>
        <dt>byte[] idsForScan</dt>
        <dd>The ID of the target to be scanned, used to search only for motors with a specific ID; if null is specified, all IDs are scanned.</dd>
        <dt>CancellationToken cancellationToken</dt>
        <dd>Used to cancel a process.</dd>
    </dl>
    
- Returns
    Byte array of available motor IDs.
    
- Description
Asynchronously pings and scans Dynamixel motors at the specified baud rate; changes to the specified baud rate using the SetBaudRate method. 
---

- Declaration
`public async Task<byte[]> ScanInBaudRateAsync(BaudRate baudRate, byte minIDsForScan, byte maxIDsForScan, CancellationToken cancellationToken = default(CancellationToken))`

- Argument
    <dl>
        <dt>byte minIDsForScan</dt>
        <dd>Lower limit of the range for the ID of the object to be scanned. (Included in the range)</dd>
        <dt>byte maxIDsForScan</dt>
        <dd>Upper limit of the range for the ID of the object to be scanned. (Included in the range)</dd>
        <dt>CancellationToken cancellationToken</dt>
        <dd>Used to cancel a process.</dd>
    </dl>
    
- Returns
    Byte array of available motor IDs. 
    
- Description
- 
Asynchronously pings and scans Dynamixel motors at a specified baud rate, using the SetBaudRate method to change to the specified baud rate. The scan target can be specified in a range.



### Ping
- Declaration
`public bool Ping(byte ID, bool hideErrorMsg = true)`

- Argument
    <dl>
        <dt>byte ID</dt>
        <dd>ID of the target to ping.</dd>
        <dt>bool hideErrorMsg</dt>
        <dd>Disable error output on connection failure.</dd>
    </dl>

- Returns
Whether they are connected.

- Description
Ping the specified ID to check if it is connected. 

### PingGetModelNumber
- Declaration
`public int PingGetModelNumber(byte ID)`

- Argument
    <dl>
        <dt>byte ID</dt>
        <dd>ID of the target to ping.</dd>
    </dl>

- Returns
Model number. When communication fails, -1 is returned.

- Description
Ping the specified ID to obtain the model number.


### GetAllModelNumber
- Declaration
`public int[] GetAllModelNumber(byte[] ids = null)`

- Argument
    <dl>
        <dt>byte[] ids</dt>
        <dd>The target ID from which the model number is obtained; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
An array of model numbers. Items for which communication failed are marked with -1. 

- Description
Ping the specified multiple IDs and return an array of model numbers. 

### SetControlTable
- Declaration
`public void SetControlTable(ControlTables controlTables)`

- Argument
    <dl>
        <dt>ControlTables controlTables</dt>
        <dd>The target ID from which the model number is obtained; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Description
Implementation of IControlTableUser. Sets the control table. 

### SetUp
- Declaration
`public byte[] SetUp(BaudRate? baudRate = null, byte[] idsForScan = null, bool scanAllBaudRateIfNotFound = true)
`

- Argument
    <dl>
    <dt>BaudRate? baudRate</dt>
    <dd>The baud rate to be scanned first; if null is specified, the BaudRate previously set by Inspector or other means is used.</dd>
        <dt>byte[] idsForScan</dt>
        <dd>ID for setup target; if null is specified, scan is performed for all IDs.</dd>
    <dt>bool scanAllBaudRateIfNotFound</dt>
        <dd>If no Dynamixel motors are found at the first baud rate scanned, scan at all baud rates.</dd>
    </dl>

- Returns
Byte array of available motor IDs. 

- Description
Scan the dynamixel motor with the ScanInBaudRate method and hold the scanned ID as the motor to be used with the SetIDsForUse method. 

### SetUpAsync
- Declaration
`public async Task<byte[]> SetUpAsync(BaudRate? baudRate = null, byte[] idsForScan = null, bool scanAllBaudRateIfNotFound = true, CancellationToken cancellationToken = default(CancellationToken))
`

- Argument
    <dl>
    <dt>BaudRate? baudRate</dt>
    <dd>The baud rate to be scanned first; if null is specified, the BaudRate previously set by Inspector or other means is used.</dd>
        <dt>byte[] idsForScan</dt>
        <dd>ID for setup target; if null is specified, scan is performed for all IDs.</dd>
    <dt>bool scanAllBaudRateIfNotFound</dt>
        <dd>If no Dynamixel motors are found at the first baud rate scanned, scan at all baud rates.</dd>
        <dt>CancellationToken cancellationToken</dt>
        <dd>Used to cancel a process.</dd>
    </dl>

- Returns
Byte array of available motor IDs.


- Description
Asynchronously scan the dynamixel motor with the ScanInBaudRateAsync method and hold the scanned ID as the motor to be used with the SetIDsForUse method.




### SetIDsForUse
- Declaration
`
public void SetIDsForUse(byte[] idsForUse)
`

- Argument
    <dl>
        <dt>byte[] idsForUse</dt>
        <dd>Target ID.</dd>
    </dl>


- Description
Keep the ID as a target for use and associate it with the model number.
:::info
By having DynamixelForUnity retain the motor ID and model number, it is possible to send commands using only the data field names, or to write concisely when sending commands to all motors.
:::



### DisconnectDynamixel
- Declaration
`
public void DisconnectDynamixel()
`

- Description
Close the port.


### CloseAndReopenPort
- Declaration
`public void CloseAndReopenPort(bool withSetBaudRate = true)
`

- Argument
    <dl>
        <dt>bool withSetBaudRate</dt>
        <dd>Set again to the immediately previous baud rate.</dd>
    </dl>


- Description
Close and reopen the port. 


### RebootDynamixel
- Declaration
`
public void RebootDynamixel(byte[] ids = null)
`

- Argument
    <dl>
        <dt>byte[] idsForScan = null</dt>
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Description
Reboot the Dynamixel motor. 

### FactoryResetDynamixel
- Declaration
`
public void FactoryResetDynamixel(DynamixelFactoryResetMode mode, byte[] ids = null)
`

- Argument
    <dl>
        <dt>DynamixelFactoryResetMode mode</dt>
        <dd>Select whether to reset all values, all values except ID, or all values except ID and baud rate.</dd>
        <dt>byte[] idsForScan</dt>
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Description
Reset the Dynamixel servomotor to the factory defaults.


### SetBaudRate
- Declaration
`
public bool SetBaudRate(BaudRate baudRate)
`

- Argument
    <dl>
        <dt>BaudRate baudRate</dt>
        <dd>The baud rate of the target.</dd>
    </dl>

- Returns
Returns true if successful, false otherwise.

- Description
Set the baudrate of the port.



### SetData
- Declaration
`
public bool SetData(ushort ADDR, ushort LEN, uint data, byte id)
`

- Argument
    <dl>
        <dt>ushort ADDR</dt>
        <dd>Address of the data field to be written. See Control Table of the model.</dd>
        <dt>ushort LEN</dt>
        <dd>Size of the data field to be written. See Control Table in the model.</dd>
        <dt>uint data</dt>
        <dd>Data to be written.</dd>
        <dt>byte id</dt>
        <dd>Target ID.</dd>
    </dl>

- Returns
Returns true if successful, false otherwise.

- Description
Writes data to the Dynamixel motor with the specified ID. 
:::info
Check the Contorol Table from the [ROBOTIS e-Manual](https://emanual.robotis.com/) of each Dynamixel servomotor for the address and length of the data field.
:::

---
- Declaration
`
public bool SetData(ControlTableAddressInfo addressInfo, uint data, byte id)
`

- Argument
    <dl>
        <dt>ControlTableAddressInfo addressInfo</dt>
        <dd>A structure with the address, length, and access for each data field in the Control Table.</dd>
        <dt>uint data</dt>
        <dd>Data to be written.</dd>
        <dt>byte id</dt>
        <dd>Target ID.</dd>
    </dl>

- Returns
Returns true if successful, false otherwise. 

- Description
SetData(ushort ADDR, ushort LEN, uint data, byte id) is used to write data to the Dynamixel servomotor with the specified ID. If the data field is read-only (e.g. present position), this method will not execute the command.

---
- Declaration
`
public bool SetData(string dataName, uint data, byte id)
`

- Argument
    <dl>
        <dt>string dataName</dt>
        <dd>Servomotor control Table data field name. Please refer to using the control table section for more information.
    
    </dd>
        <dt>uint data</dt>
        <dd>Data to be written.</dd>
        <dt>byte id</dt>
        <dd>Target ID.</dd>
    </dl>

- Returns
Returns true if successful, false otherwise. 

- Description
SetData(ushort ADDR, ushort LEN, uint data, byte id) is used to write data to the Dynamixel servomotor with the specified ID. If the data field is read-only, it is not executed.



```Csharp!
        d4u.SyncGetSignedDataGroupMultiModels("GoalPosition").ToActualNumber(UNIT.POSITION).ToFloatArray(), ct);


```

:::info
The ID and model number must be retained by the SetUp or SetIDsForUse method to be used.
:::



### SetSignedData
- Declaration
`public bool SetSignedData(ushort ADDR, ushort LEN, uint data, byte id)`
`public bool SetSignedData(ControlTableAddressInfo addressInfo, int data, byte id)`
`public bool SetSignedData(string dataName, int data, byte id)`


- Description
Writes signed data to the Dynamixel motor with the specified ID, see Description of SetData.


### GetData
- Declaration
`
public uint GetData(ushort ADDR, ushort LEN, byte id)
`

- Argument
    <dl>
        <dt>ushort ADDR</dt>
        <dd>Address of the data field to be rewritten. See Control Table of the model.</dd>
        <dt>ushort LEN</dt>
        <dd>Size of the data field to be rewritten. See Control Table in the model.</dd>
        <dt>byte id</dt>
        <dd>Target ID.</dd>
    </dl>

- Returns
Data read. Returns 0 if an error occurs.

- Description
Reads the data of the Dynamixel motor with the specified ID.

:::info
Check the Contorol Table from the [ROBOTIS e-Manual](https://emanual.robotis.com/) of each Dynamixel servomotor for the address and length of the data field.
:::

---
- Declaration
`
public uint GetData(ControlTableAddressInfo addressInfo, byte id)
`

- Argument
    <dl>
        <dt>ControlTableAddressInfo addressInfo</dt>
        <dd>A structure containing the address, length, and access for each data field in the Control Table.</dd>
        <dt>byte id</dt>
        <dd>Target ID.</dd>
    </dl>


- Returns
Data read. Returns 0 if an error occurs. 

- Description
GetData(ushort ADDR, ushort LEN, byte id) is used to read data of a servomotor by specifying it's ID.

---
- Declaration
`
public uint GetData(string dataName, byte id)
`

- Argument
    <dl>
        <dt>string dataName</dt>
        <dd>Control Table data field name.</dd>
        <dt>byte id</dt>
        <dd>Target ID.</dd>
    </dl>

- Returns
Data read. Returns 0 if an error occurs.

- Description
GetData(ushort ADDR, ushort LEN, byte id) is used to read  data from a servomotor by specifying it's ID.

:::info
The ID and model number must be created in using either the SetUp method or SetIDsForUse method to be used.
:::



### GetSignedData
- Declaration
`public int GetSignedData(ushort ADDR, ushort LEN, byte id)`
`public int GetSignedData(ControlTableAddressInfo addressInfo, byte id)`
`public int GetSignedData(string dataName, byte id)`


- Description
Reads the signed data of the Dynamixel motor with the specified ID, see Description of GetData.


### SyncSetDataGroup
- Declaration
`
public bool SyncSetDataGroup(ushort ADDR, ushort LEN, uint[] data, byte[] ids = null)
`
`
public bool SyncSetDataGroup(ControlTableAddressInfo addressInfo, uint[] data, byte[] ids = null)
`

- Argument
    <dl>
        <dt>ushort ADDR</dt>
        <dd>Address of the data field to be rewritten. See Control Table of the model.</dd>
        <dt>ushort LEN</dt>
        <dd>Size of the data field to be rewritten. See Control Table in the model.</dd>
        <dt>ControlTableAddressInfo addressInfo</dt>
        <dd>A structure containing the address, length, and access for each data field in the Control Table.</dd>
        <dt>uint[] data</dt>
        <dd>Data array to be written.</dd>
        <dt>byte[] ids = null</dt>
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
Returns true if successful, false otherwise. 

- Description
Writes data to multiple servomotors simultaneously using GroupSyncWrite. 

### SyncSetDataGroupMultiModels
- Declaration
`public bool SyncSetDataGroupMultiModels(string dataName, uint[] data, byte[] ids = null)`

- Description
Writes data to multiple servomotors at the same time using GroupSyncWrite method. If the item names on the ControlTable are the same, the data will be sent to different addresses. In that case, call SyncSetDataGroup for each unique address; see Description of SyncSetDataGroup.

:::info
The ID and model number must be retained by the SetUp or SetIDsForUse method to be used.
:::


### SyncSetSignedDataGroup
- Declaration
`public bool SyncSetSignedDataGroup(ushort ADDR, ushort LEN, int[] data, byte[] ids = null)`
`public bool SyncSetSignedDataGroup(ControlTableAddressInfo addressInfo, int[] data, byte[] ids = null)
`

- Description
Writes signed data to multiple servomotors simultaneously using GroupSyncWrite; see SyncSetDataGroup Description.

### SyncSetSignedDataGroupMultiModels
- Declaration
`public bool SyncSetSignedDataGroupMultiModels(string dataName, int[] data, byte[] ids = null)
`

- Description
Write signed data to multiple motors simultaneously using GroupSyncWrite method, see Description of SyncSetDataGroupMultiModels.

:::info
The ID and model number must be retained by the SetUp or SetIDsForUse method to be used.
:::

### SyncGetDataGroup
- Declaration
`public uint[] SyncGetDataGroup(ushort ADDR, ushort LEN, byte[] ids = null)`
`public uint[] SyncGetDataGroup(ControlTableAddressInfo addressInfo, byte[] ids = null)`


- Argument
    <dl>
        <dt>ushort ADDR</dt>
        <dd>Address of the data field to be read. See Control Table of the model.</dd>
        <dt>ushort LEN</dt>
        <dd>Size of the data field to be read. See Control Table in the model.</dd>
        <dt>ControlTableAddressInfo addressInfo</dt>
        <dd>A structure containing the address, length, and access for each data field in the Control Table.</dd>
        <dt>byte[] ids = null</dt>
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
uint array containing the read after executing the SyncGetDataGroup.

- Description
Read data from multiple servomotors simultaneously using GroupSyncRead.


### SyncGetDataGroupMultiModels
- Declaration
`public uint[] SyncGetDataGroupMultiModels(string dataName, byte[] ids = null)
`

- Description
Simultaneously read data from multiple servomotors using GroupSyncRead method. If the item names on the ControlTable are the same, the data will be sent to different addresses. In that case, call SyncSetDataGroup for each unique address; see Description of SyncGetDataGroup.
:::info
The ID and model number must be retained by the SetUp or SetIDsForUse method to be used.
:::


### SyncGetSignedDataGroup
- Declaration
`public int[] SyncGetSignedDataGroup(ushort ADDR, ushort LEN, byte[] ids = null)`
`public int[] SyncGetSignedDataGroup(ControlTableAddressInfo addressInfo, byte[] ids = null)
`

- Description
Read signed data to multiple motors simultaneously using GroupSyncRead, see Description of SyncGetDataGroup.

### SyncGetSignedDataGroupMultiModels
- Declaration
`public int[] SyncGetSignedDataGroupMultiModels(string dataName, byte[] ids = null)
`

- Description
Read signed data to multiple motors simultaneously using GroupSyncRead, see Description of SyncGetDataGroupMultiModels.

:::info
The ID and model number must be retained by the SetUp or SetIDsForUse method to be used.
:::


### BulkSetDataGroup
- Declaration
`public BulkWritePacket BulkSetDataGroup() 
`

- Returns
BulkWritePacket object.

- Description
Returns a BulkWritePacket object for writing data to multiple servomotors at once using GroupBulkWrite. 

:::info
GroupBulkWrite is described in a method chain mediated by BulkWritePacket.
:::



### BulkGetDataGroup
- Declaration
`public BulkReadPacket BulkGetDataGroup()
`

- Returns
BulkReadPacket object.

- Description
Returns a BulkReadPacket object for reading data in batches to multiple motors using GroupBulkRead.
:::info
GroupBulkRead is described in a method chain mediated by BulkReadPacket.
:::

## Using the Control Table
We created a text file to represent the control table of each Dynamixel servomotor. The control tables map each property name to the corresponding register number within for  servomotor model.

The control tables correspond to the servomotor control tables found in [ROBOTIS e-Manual](https://emanual.robotis.com/).


Please check the control table data under D4U HatsuMuv/DynamixelForUnity/Resources/ControlTables/ and find the corresponding motor, as shown in the screenshot below:
![ControlTable](https://hackmd.io/_uploads/r1lCvg-n3.jpg)

The control tables can be used as parameters in a variety of methods in D4U, where you can pass the parameter name as a string to control or get feedback from various servomotors, as shown in the example below

```Csharp!
// you may pass a control table parameter to read in the below method, where the control table parameter corresponds to the string representation of the register you need to set/read.  

// This example uses wet a group of data through sync method
SyncGetDataGroupMultiModels("GoalPosition");

```

## Using BulkWritePacket

Executing bulk write includes the following steps:
1- Initiate struct to hold bulk write parameters.
2- Add parameters to the struct.
3- Send the bulkwrite command.
4- (Options) read result of executing the command.

The BulkWritePacket object is used to execute the above four steps by settings its properties and calling its methods as highlighting explained below.



#### Step 1. Initiate Struct
To initiate the struct to hold the parameters for the writing packet, you must first declare and initialize the relevant BulkWritePacket objects as follows:

```Csharp!
// bulk write packet
BulkWritePacket writePacket = dynamixelForUnity.BulkSetDataGroup();
```
#### Step 2. Adding Parameters
Upon creating the BulkWritePacket object, the data to be written should be added to the created object as parameters. This can be done as follows:

```Csharp!
writePacket.AddParam(addressInfo1, idArray1, data1);
// you may add as many parameters as needed
writePacket.AddParam(addressInfo2, idArray2, data2);
writePacket.AddParam(addressInfo3, idArray3, data3);
```
#### Step 3. Sending BulkWrite Command
Upon adding the parameters, the command can be executed by calling the BulkWritePacket object's Send() method, as shown below:

```Csharp!
writePacket.Send();
```

#### Step 4. (Optional) Checking results of BulkWrite Command Execution
You may optionally check the status of the command, whether it was successfully executed or not. You can check the property "AllSuccess" which is set to true if the command is successfully executed.

```Csharp!
if(writePacket.AllSuccess)
    Debug.Log("Successful");
```
Alternatively, you can directly place the BulkPacket object in an 'if' conditional to check if the command was successful.

```Csh    !
if(writePacket)
    Debug.Log("Successful");
```


Overall, you may execute the four steps in one line efficiently as follows:
Example:
```Csharp!
if(dynamixelForUnity
    .BulkSetDataGroup()
    .AddParam(addressInfo1, idArray1, data1)
    .AddParam(addressInfo2, idArray2, data2)
    .AddParam(addressInfo3, idArray3, data3)
    .Send())
    Debug.Log("Successful");
```

### Properties
|Name|Description|
|--|--|
|IDParamPair|It is of type Dictionary<byte, ParamInfo>. This holds parameters for each ID. ParamInfo contains address, length, communication success, and data to be written. Read-only.|
|AllSuccess|A boolean value indicating whether communication of all parameters was successful. Read only.|
|HasSent|A boolean value indicating whether the command was sent. Read only.|



### AddParam
- Declaration
`public BulkWritePacket AddParam(ushort ADDR, ushort LEN, uint[] data, byte[] ids = null)`

- Argument
    <dl>
        <dt>ushort ADDR</dt>
        <dd>Address of the data field to be written. See Control Table of the model.</dd>
        <dt>ushort LEN</dt>
        <dd>Size of the data field to be written. See Control Table in the model.</dd>
        <dt>uint[] data</dt>
        <dd>Data to be written.</dd>
        <dt>byte[] id</dt>
        <dd>Target ID; if null is specified, the IDs held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
Itself.

- Description
The information and data of the data field to be written are stored in IDParamPair.
---
- Declaration
`public BulkWritePacket AddParam(ControlTableAddressInfo addressInfo, uint[] data, byte[] ids = null)`

- Argument
    <dl>
        <dt>ControlTableAddressInfo addressInfo</dt>
        <dd>A structure containing the address, length, and access for each data field in the Control Table.</dd>
    <dt>uint[] data</dt>
        <dd>Data to be written.</dd>
        <dt>byte[] id</dt>
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
Itself.

- Description
AddParam(ushort ADDR, ushort LEN, uint[] data, byte[] ids = null) is used to store the information and data of the data field to be written in IDParamPair. If the data field is read-only, it is not executed.




### AddParamSigned
- Declaration
`public BulkWritePacket AddParamSigned(ushort ADDR, ushort LEN, int[] data, byte[] ids = null)`
`public BulkWritePacket AddParamSigned(ControlTableAddressInfo addressInfo, int[] data, byte[] ids = null)
`


- Description
Stores information and signed data of the data field to be written in IDParamPair; see Description in AddParam.


### Send
- Declaration
`public BulkWritePacket Send(bool discardAllWhenErrorHappen = false)`

- Argument
    <dl>
        <dt>bool discardAllWhenErrorHappen = false</dt>
        <dd>If set to ture, if an error occurs during the AddParam operation, the process is aborted without sending a command.</dd>
    </dl>

- Returns
Itself.

- Description
Send GroupBulkWrite command to servomotors based on IDParamPair. 

### ResetStatus
- Declaration
`public void ResetStatus()`


- Description
Reset communication results and transmission flags. The data to be written is not deleted. 

## Using BulkReadPacket
Executing bulk read includes the following steps:
1- Initiate struct to hold bulkread parameters.
2- Add parameters to the struct.
3- Send the bulkread command.
4- (Options) read result of executing the command.
5- Reading returned values from the servomotors.


The BulkReadPacket object is used to execute the above four steps by settings its properties and calling its methods as highlighting explained below.




#### Step 1. Initiate Struct
To initiate the struct to hold the parameters for the reading packet, you must first declare and initialize the relevant BulkdReadPacket object as follows:

```Csharp!
// bulk read packet
BulkReadPacket readPacket = dynamixelForUnity.BulkGetDataGroup();
```
#### Step 2. Adding Parameters
Upon creating the BulkReadPacket object, the data to be read should be added to the created object as parameters. This can be done as follows:

```Csharp!
readPacket.AddParam(addressInfo1, idArray1);
// you may add as many parameters as needed
readPacket.AddParam(addressInfo2, idArray2);
readPacket.AddParam(addressInfo3, idArray3);
```
#### Step 3. Sending BulkRead Command
Upon adding the parameters, the command can be executed by calling the BulkReadPacket object's Send() method, as shown below:

```Csharp!
readPacket.Send();
```

#### Step 4. (Optional) Checking results of BulkRead Command Execution
You may optionally check the status of the command, whether it was successfully executed or not. You can check the property "AllSuccess" which is set to true if the command is successfully executed.

```Csharp!
if(readPacket.AllSuccess)
    Debug.Log("Successful");
```
Alternatively, you can directly place the BulkPacket object in an 'if' conditional to check if the command was successful.

```Csh    !
if(readPacket)
    Debug.Log("Successful");
```
#### Step 5. Reading returned values from the servomotors.
To read the data that was read from the servomotors after executing the bulkread command, you can use the method GetData() of the BulkReadPacket as follows:

```csharp!
// create a uint array to store the results read from the servomotors
uint[] data = readPacket.GetData();
```

Overall, you may execute the five steps and print out the results as follows:
Example:
```Csharp!
if(var readPacket = dynamixelForUnity
    .BulkGetDataGroup()
    .AddParam(addressInfo1, idArray1)
    .AddParam(addressInfo2, idArray2)
    .AddParam(addressInfo3, idArray3)
    .Send())
{
    Debug.Log("Successful");
    uint[] data = readPacket.GetData();
    Debug.Log("Data: " + string.Join(", ", data));
}
```

### Property
|Name|Description|
|--|--|
|IDParamPair|It is of type Dictionary<byte, ParamInfo>. This holds parameters for each ID. ParamInfo contains address, length, communication success, and data from servomotor. Read only.|
|AllSuccess|A boolean value indicating whether communication of all parameters was successful. Read only.|
|HasSent|A boolean value indicating whether the command was sent. Read only.|



### AddParam
- Declaration
`public BulkReadPacket AddParam(ushort ADDR, ushort LEN, byte[] ids = null)`

- Argument
    <dl>
        <dt>ushort ADDR</dt>
        <dd>Address of the data field to be read. See Control Table of the model.</dd>
        <dt>ushort LEN</dt>
        <dd>Size of the data field to be read. See Control Table in the model.</dd>
        <dt>byte[] id</dt>
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
Itself.

- Description
The information and data of the data field to be read are stored in IDParamPair.
---
- Declaration
`public BulkReadPacket AddParam(ControlTableAddressInfo addressInfo, byte[] ids = null)`

- Argument
    <dl>
        <dt>ControlTableAddressInfo addressInfo</dt>
        <dd>A structure containing the address, length, and access for each data field in the Control Table.</dd>
        <dt>byte[] id</dt>
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
Itself.

- Description
AddParam(ushort ADDR, ushort LEN, byte[] ids = null) is used to store information about the data field to be read in IDParamPair.

### Send
- Declaration
`public BulkReadPacket Send(bool discardAllWhenErrorHappen = false)`

- Argument
    <dl>
        <dt>bool discardAllWhenErrorHappen = false</dt>
        <dd>If set to ture, if an error occurs during the AddParam operation, the process is aborted without sending a command.</dd>
    </dl>

- Returns
Itself.

- Description
Send GroupBulkRead command to Dynamxiel motor based on IDParamPair.

### GetData
- Declaration
`public uint[] GetData(bool resetStatus = false)`

- Argument
    <dl>
        <dt>bool resetStatus</dt>
        <dd>If true, the ResetStatus method is called upon exit.</dd>
    </dl>

- Returns
Data array read.

- Description
Retrieve the read data array.


### GetSignedData
- Declaration
`public int[] GetSignedData(bool resetStatus = false)`

- Argument
    <dl>
        <dt>bool resetStatus</dt>
        <dd>If true, the ResetStatus method is called upon exit.</dd>
    </dl>

- Returns
Data array read.

- Description
Retrieve the signed data array that was read.


### ResetStatus
- Declaration
`public void ResetStatus()`

- Description
Reset communication results and transmission flags. The read data is also deleted. 

## ParamInfo
Class representing data fields in BulkWritePacket and BulkReadPacket. 

### Variable
| Name|Description|
|--|--|
|address|Data address.|
|length|Length of data.|
|isSuccess|Whether the communication was successful or not.|
|data|In case of BulkWritePacket, data to be written; in case of BulkReadPacket, data to be read.|



## ControlTables

A class that stores the Control Table for each motor; the Control Table is initialized by putting text written in the specified format into the constructor argument. The constructor takes either string[] or TextAsset[] as an argument.

Text files of Control Table for DynamxielForUnity compatible models are located in Assets/HatsuMuv/DynamixelForUnity/Resources/ControlTables. See ControlTableInjectorFromResources for usage examples.

ControlTables is the ControlTable of the motor to be used during initialization. The first line describes the model name and model number with ":" in between. The other lines describe the itemName, address, length, and access.

ItemName is derived from the item name in the ControlTable on Dynamixel's model page, with spaces removed.

If the access is read-only, specify 'R'; if writable, specify 'RW'.

Lines beginning with "#" or "//" are ignored as comments.
```=
<ModelName>:<ModelName>
<ItemName>, <Address>, <Length>, <Access>
<ItemName>, <Address>, <Length>, <Access>
...
```

### Property

| Name | Description |
| -------- | -------- |
| ModelNames | This is of type Dictionary<int, string>. It records the correspondence between model numbers and model names from the read text data.|


### GetAddrssInfo
- Declaration
`public ControlTableAddressInfo GetAddrssInfo(int modelNumber, string dataName)`
`public ControlTableAddressInfo this[int modelNumber, string dataName]
`

- Argument
    <dl>
        <dt>int modelNumber</dt>
        <dd>Model Number.</dd>
        <dt>string dataName</dt>
        <dd>Data Name.</dd>
    </dl>

- Returns
ControlTableAddressInfo type representing data field information.

- Description
Returns a ControlTableAddressInfo type representing the data field information from the specified model number and data name. The indexer [int modelNubmer, string dataName] also produces the same result.


### GetControlTable
- Declaration
`public Dictionary<string, ControlTableAddressInfo> GetControlTable(int modelNumber)`
`public Dictionary<string, ControlTableAddressInfo> this[int modelNumber]
`

- Argument
    <dl>
        <dt>int modelNumber</dt>
        <dd>Model number of the subject.</dd>
    </dl>

- Returns
Dictionary<string, ControlTableAddressInfo> type representing the control table. 

- Description
Returns a Dictionary<string, ControlTableAddressInfo> type representing the control table from the specified model number. The indexer [int modelNubmer] produces the same result.



### Contains
- Declaration
`public bool Contains(int modelNumber)`

- Argument
    <dl>
        <dt>int modelNubmer</dt>
        <dd>The model number to look up.</dd>
    </dl>

- Returns
Returns true if the Control Table with the specified model number exists, otherwise returns false.

- Description
Returns whether the Control Table with the specified model number exists.
---

- Declaration
`public bool Contains(int modelNumber, string dataName)`

- Argument
    <dl>
        <dt>int modelNubmer</dt>
        <dd>The model number to look up.</dd>
        <dt>string dataName</dt>
        <dd>Name of the data to be examined.</dd>
    </dl>

- Returns
Returns true if the Control Table with the specified model number exists and the data field with the specified data name exists, otherwise returns false.

- Description
Returns whether a data field with a data name of the specified model number exists. 

### GetModelsContains
- Declaration
`public int[] GetModelsContains(string dataName)`

- Argument
    <dl>
        <dt>string dataName</dt>
        <dd>Name of the data to be examined.</dd>
    </dl>

- Returns
An array of model numbers with data fields of the specified data name in the Control Table. 

- Description
Returns an array of model numbers that have data fields with the specified data name in the Control Table.


## ControlTableAddressInfo
A structure representing information about data fields in the Control Table. 

### Variable
| Name|Description|
|--|--|
|address|Data address.|
|length|Length of data.|
|readWrite|True if writable, false if read-only.|

## IControlTableUser
An Interface to inject control tables, implemented by DynamixelForUnity. 

### SetControlTables
- Declaration
`public void SetControlTables(ControlTables controlTables)`


## Core

Dynamixel SDK compatible class. Put a HatsuVerif object in the constructor and create an instance.

# 6. Explanation of Example Scene
A sample scene is located within: HatsuMuv/DynamixelForUnity/Example/ExampleScene_x64.unity 

This scene has a sample graphical-user interface (GUI) application using that uses DynamixelForUnity to open a com port, scan for servomotors, connect to servomotors, control servomotors and read various information from servomotors that are all displayed on the GUI.

## Screen Explanation
![](https://hackmd.io/_uploads/Sylmk0x2n.jpg)
1. Select **DynamixelForUnity** gameobject in Hierarchy menu

![](https://hackmd.io/_uploads/Skw6tax3n.jpg)
2. In Inspector window, you can find a component called **DynamixelForUnity**. Enter your **Email** and **SN**, and the USB serial port which connected to your Dynamixel motor in the **Device Name** field. (You can find the port name from your **Device manager**, e.g. "COM3"). 

3. (Optional) If you know the baudrate of the motor you want to operate, set the Baud Rate in the inspector of DynamixelForUnity beforehand. If you don't know, leave it blank, D4U will scan for you. (Which can be time comsuming)


After setup DynamxielForUnity components, click play, Dynamixel4Unity will automatically attempt to open the designated port and connect to available servomotors.

![](https://hackmd.io/_uploads/BkQmC6g2n.png)

The screen lists the connected Dynamixel motors. The connected baud rate is displayed at the top of the screen.

The read-only data items at the bottom are constantly updated (such as present position) .

The other items are updated when there is a change in the inserted data (for example, to set a new servomotor position).

**Hint: If values differ from the actual value, try press the Refresh button to retrieve the data again.**

## Implementation Explanation
### D4UExampleController
#### Start()

```Csharp!
private async void Start()
{
    NeedWait = true;
    
    if (d4u == null)
    {
        Debug.LogError("[DynamixelForUnitySample] DynamixelForUnity is not assigned!");
        return;
    }

    cancellationTokenSource = new CancellationTokenSource();

    var connectResult = await Task.Run(() => InitializeDynamixel(cancellationTokenSource.Token));
    if (connectResult) StartMotorMetrics();

    NeedWait = false;
}
```

The NeedWait property is externally referenced to play the screen load animation.

DynamixelForUnity (d4u) is expected to be assigned by Inspector.

The cancellationTokenSource is created to interrupt asynchronous tasks.

Connect to the motor and execute the initialization method InitializeDynamixel asynchronously and wait for it.


Once the Dynamixel motor is connected by InitializeDynamixel, the StartMotorMetricsm method starts a loop that continuously measures the state of the motor.

---
#### InitializeDynamixel(CancellationToken ct)
```Csharp!
private async Task<bool> InitializeDynamixel(CancellationToken ct)
{
    if (d4u == null)
    {
        Debug.LogError("[DynamixelForUnitySample] DynamixelForUnity is not assigned!");
        return false;
    }

    if (!d4u.ConnectStatus)
    {
        var connectResult = d4u.ConnectDynamixel();
        if (!connectResult)
        {
            Debug.LogError("[DynamixelForUnitySample] Dynamixel is not connected. Connect it first.");
            return false;
        }

        await d4u.SetUpAsync(cancellationToken:ct);
    }

    motors = new List<DynamixelMotor>();
    ids = d4u.IDsForUse;

    if (ids.Length == 0) return false;

    await GetAllMotorProperty(ct);
    return true;
}
```

The d4u.ConnectDynamixel method connects to the servomotor. You can also specify a port name and baudrate as arguments. In this case, the values entered in the Unity3D Inspector are ignored.

The SetUpAsync method calls two methods to scan the motor and hold the IDs found at once.

1. Call the ScanInBaudRateAsync method to scan for Dynamixel servomotors at the specified port name and baudrate. If the motor is not found, it will start a scan in all baudrates and sets the port to the baudrate at which the motor is first found.
2. Retains the ID and model number of the motor found by calling the SetIDsForUse method.

:::info
DynamixelForUnity retain servomotor IDs and model numbers, it is possible to send commands using only the data field names, and functions can be written to be concise when sending commands to all the servomotors.
:::

Next, we use get all servomotor properties by calling GetAllMotorProperty method and create a DynamixelMotor instance.

---
#### GetAllMotorProperty(CancellationToken ct)
```Csharp!
private async Task GetAllMotorProperty(CancellationToken ct)
{
    OperatingMode[] operatingModes = 
        await Task.Run(() =>
            d4u.SyncGetDataGroupMultiModels("OperatingMode")
                .Select(d => (OperatingMode)d)
                .ToArray(),
            ct);

    float[] homingOffsets = await Task.Run(() => d4u.SyncGetSignedDataGroupMultiModels("HomingOffset").ToActualNumber(UNIT.POSITION).ToFloatArray(), ct);
    ...

    for (int i = 0; i < ids.Length; i++)
    {
        DynamixelMotor m;
        if (motors.Count < i + 1) 
        {
            m = new DynamixelMotor(ids[i]);
            motors.Add(m);
        }
        else
        {
            m = motors[i];
        }
        m.OperatingMode = operatingModes[i];
        m.HomingOffset = homingOffsets[i];
        ...
    }
}
```

Each and every property of the DynamixelMotor class is obtained by the method SyncGetSignedDataGroupMultiModels. One property is explained using an example. 
```cs    !
float[] homingOffsets = 
    await Task.Run(
        () => d4u.SyncGetSignedDataGroupMultiModels("HomingOffset")
            .ToActualNumber(UNIT.POSITION)
            .ToFloatArray(),
        ct
    );
```

The line that initializes homingOffsets has a line break for clarity.

The second line, Task.Run method, executes the function of the first argument asynchronously in a separate thread.

From the third line, it is the function you want to execute in a separate thread.

d4u.SyncGetSignedDataGroupMultiModels("HomingOffset") retrieves the value of the "HomingOffset" data field for all motors stored in DynamixelForUnity.

ToActualNumber(UNIT.POSITION) applies units to the raw data obtained from the SyncGetSignedDataGroupMultiModels.

ToFloatArray() converts a double[] to a float[].

:::info
MotorMetrics is a similar method that strips some data from GetAllMotorProperty. 
:::

---
#### StartMotorMetrics()
#### MotorMetricsLoop(CancellationToken ct)
#### StopMotorMetrics()
```cs    !
private void StartMotorMetrics()
{
    if (!d4u.ConnectStatus)
        return;

    cancellationTokenSource = cancellationTokenSource ?? new CancellationTokenSource();
    var cancellationToken = cancellationTokenSource.Token;
    metricsLoop = MotorMetricsLoop(cancellationToken);
}

private async Task MotorMetricsLoop(CancellationToken ct)
{
    while (true)
    {
        if (ct.IsCancellationRequested)
            return;

        Debug.Log("[SampleController] Metrics");
        try
        {
            await MotorMetrics(ct);
        }
        catch (OperationCanceledException)
        {
            Debug.Log("[SampleController] Metrics canceled");
            return;
        }
        catch (Exception e)
        {
            Debug.LogError(e);
            return;
        }
    }
}

private async Task StopMotorMetrics()
{
    if (cancellationTokenSource != null && !cancellationTokenSource.IsCancellationRequested)
    {
        cancellationTokenSource.Cancel();
        cancellationTokenSource.Dispose();
        cancellationTokenSource = null;

        if(metricsLoop != null)
            await metricsLoop;
    }
}
```
MotorMetricsLoop continues to call MotorMetrics until it is canceled by CancellationToken. StopMotorMetrics cancels MotorMetrics and waits for the process to finish.

### D4UExampleUI
Generate UI elements. Generates a panel for each motor from D4UExampleController and updates those data.

### D4UExampleUIElement
It directly controls the fields in each panel; when it receives input from a UI element, it passes the value to D4UExampleContorller. 

