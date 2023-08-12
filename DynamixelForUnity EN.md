# Table of Contents
[TOC]

# 1. Overview
Dynamixel for Unity is an asset to easily implement Dynamixel motor control apps in Unity.

- Automatically load the Control Table of the motor and control multiple models at once 
- Control Dynamixel Motor with declarative code 
- Reduce redundant code 
- Use Dynamixel API compatible classes

Reduction of redundant code - Use of Dynamixel API compatible classes

Other useful functions such as batch control of multiple models and unit conversion are also included.

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
With the Unity project you want to use open, double-click DynamixelForUnity.unitypackage, or drag and drop it into the Project window of the Unity project you want to use.

Import with all selected. 

# 3. Quick Start (Explanation of Example Scene)
HatsuMuv/DynamixelForUnity/Example/ExampleScene_x64.unity is a sample GUI application using DynamixelForUnity.

## Screen Explanation

1. enter the USB serial port to which the Dynamixel motor is connected in the Device Name field. Check the port name from the device manager, e.g.) "COM3". 2.

If you know the baud rate of the motor you want to operate, set the Baud Rate in the inspector of DynamixelForUnity beforehand.

:::info
If you cannot connect to a motor due to different baud rates, etc., scan for all baud rates and connect at the first baud rate at which a motor is found. Note, however, that this process can be very time consuming.
:::

Automatically connects to Dynamixel motor when Play is pressed.


The screen lists the connected Dynamixel motors. The connected baud rate is displayed at the top of the screen.

The read-only data items at the bottom are constantly updated.

The other items are updated when there is a change in the data.

If the value deviates from the actual value, press the Reflesh button to retrieve the data again.

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

    // Varificate your licence here
    d4u.Varificate(new DevTest.HatsuVarif(licenceFilePath: "some", userID: "some", email: "example@sample.com", phonenumber: "123456789"));

    cancellationTokenSource = new CancellationTokenSource();

    var connectResult = await Task.Run(() => InitializeDynamixel(cancellationTokenSource.Token));
    if (connectResult) StartMotorMetrics();

    NeedWait = false;
}
```

The NeedWait property is externally referenced to play the screen load animation.

DynamixelForUnity d4u expects to be assigned by Inspector.

License authentication using the d4u.Validate method.

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

The d4u.ConnectDynamixel method connects to the motor. You can also specify a port name and baud rate as arguments. In this case, the values entered by Inspector are ignored.

The SetUpAsync method calls two methods to scan the motor and hold the IDs found at once.

1. Call the ScanInBaudRateAsync method to scan for Dynamixel motors at the specified port name and baud rate. If the motor is not found, it reruns the scan at all baud rates and sets the port to the baud rate at which the motor was first found.
2. Retains the ID and model number of the motor found by calling the SetIDsForUse method.
:::info
By having DynamixelForUnity retain the motor ID and model number, it is possible to send commands using only the data field names, and some functions can be written to be concise when sending commands to all motors.
:::

In creating an ExampleScene, the DynamixelMotor class is implemented when handling motor information. Add properties according to the application you are creating.

Get all motor properties by the GetAllMotorProperty method and create a DynamixelMotor instance.

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

# 4. Script Reference

Select a namespace from the following according to your CPU configuration.

```
HatsuMuv.DynamixelForUnity.x64
HatsuMuv.DynamixelForUnity.x86
```


## DynamixelForUnity

DynamixelForUnity is a class provided to control Dynamixel motors with declarative code. It manages with one component per port, implementing IControlTableUser. 

Flow of Use
1. Open a port with ConnectDynamixel.
2. (Optional) Keep the motor ID and model number used in SetUp or SetIDsForUse. This procedure allows for concise writing when sending instructions using only data field names or when sending instructions to all motors.
3. Control the motor with methods such as SetData, etc. Note that data in the EEPROM area such as OperatingMode can only be written when the torque is OFF.
5. At the end of the oparation, perform a termination procedure such as torque disable in the OnApplicationQuit or OnDisable. If the DynamixelForUnity is connected at the time when OnDestroy is called, it will call DisconnectDynamixel.

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
        <dd>Verification object. Created from the license file path and verification information.</dd>
    </dl>

- Returns
    License validity.

- Description
:::info
Please run this method and validate your license before use. 
:::



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
Reset the Dynamixel motor to the factory defaults.


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
Set the baud rate of the port.



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
Check the Contorol Table from the e-Manual of each Dynamitel motor for the address and length of the data field.
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
SetData(ushort ADDR, ushort LEN, uint data, byte id) is used to write data to the Dynamixel motor with the specified ID. If the data field is read-only, it is not executed.

---
- Declaration
`
public bool SetData(string dataName, uint data, byte id)
`

- Argument
    <dl>
        <dt>string dataName</dt>
        <dd>Control Table data field name.</dd>
        <dt>uint data</dt>
        <dd>Data to be written.</dd>
        <dt>byte id</dt>
        <dd>Target ID.</dd>
    </dl>

- Returns
Returns true if successful, false otherwise. 

- Description
SetData(ushort ADDR, ushort LEN, uint data, byte id) is used to write data to the Dynamixel motor with the specified ID. If the data field is read-only, it is not executed.

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
Check the Contorol Table from the e-Manual of each Dynamitel motor for the address and length of the data field.
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
GetData(ushort ADDR, ushort LEN, byte id) is used to read the data of the Dynamixel motor with the specified ID.

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
GetData(ushort ADDR, ushort LEN, byte id) is used to read the data of the Dynamixel motor with the specified ID.

:::info
The ID and model number must be retained by the SetUp or SetIDsForUse method to be used.
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
Writes data to multiple motors simultaneously using GroupSyncWrite. 

### SyncSetDataGroupMultiModels
- Declaration
`public bool SyncSetDataGroupMultiModels(string dataName, uint[] data, byte[] ids = null)`

- Description
Writes data to multiple motors at the same time using GroupSyncWrite. Send data to different addresses as long as the data field names are the same. In that case, call SyncSetDataGroup for each unique address; see Description of SyncSetDataGroup.

:::info
The ID and model number must be retained by the SetUp or SetIDsForUse method to be used.
:::


### SyncSetSignedDataGroup
- Declaration
`public bool SyncSetSignedDataGroup(ushort ADDR, ushort LEN, int[] data, byte[] ids = null)`
`public bool SyncSetSignedDataGroup(ControlTableAddressInfo addressInfo, int[] data, byte[] ids = null)
`

- Description
Writes signed data to multiple motors simultaneously using GroupSyncWrite; see SyncSetDataGroup Description.

### SyncSetSignedDataGroupMultiModels
- Declaration
`public bool SyncSetSignedDataGroupMultiModels(string dataName, int[] data, byte[] ids = null)
`

- Description
Write signed data to multiple motors simultaneously using GroupSyncWrite, see Description of SyncSetDataGroupMultiModels.

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
Data array read.

- Description
Read data from multiple motors simultaneously using GroupSyncRead.


### SyncGetDataGroupMultiModels
- Declaration
`public uint[] SyncGetDataGroupMultiModels(string dataName, byte[] ids = null)
`

- Description
Read data to multiple motors simultaneously using GroupSyncRead. Send data to different addresses as long as the data field names are identical. In that case, call SyncSetDataGroup for each unique address; see Description of SyncGetDataGroup.
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
Returns a BulkWritePacket object for writing data to multiple motors at once using GroupBulkWrite.

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



## BulkWritePacket
Object for writing data to multiple motors at once using GroupBulkWrite, described by a method chain mediated by BulkWritePacket.

Example:
```Csharp!
dynamixelForUnity.BulkSetDataGroup()
    .AddParam(addressInfo1, idArray1, data1)
    .AddParam(addressInfo2, idArray2, data2)
    .Send();
```

### Property
|Name|Description|
|--|--|
|IDParamPair|Dictionary<byte, ParamInfo> type. Holds information about each ID and data field to be written; ParamInfo holds address, length, success or failure, and data to be sent. Read-only.|
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
        <dd>Target ID; if null is specified, the ID held in DynamixelForUnity is targeted.</dd>
    </dl>

- Returns
Myself.

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
Myself.

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
Myself.

- Description
Send GroupBulkWrite command to Dynamxiel motor based on IDParamPair. 

### ResetStatus
- Declaration
`public void ResetStatus()`


- Description
Reset communication results and transmission flags. The data to be written is not deleted. 

## BulkReadPacket
Object for reading data from multiple motors at once using GroupBulkRead, used in a method chain mediated by BulkReadPacket. 

Example:
```Csharp!
int[] data = dynamixelForUnity.BulkGetDataGroup()
    .AddParam(addressInfo1, idArray1)
    .AddParam(addressInfo2, idArray2)
    .Send()
    .GetSignedData();
```

### Property
|Name|Description|
|--|--|
|IDParamPair|Dictionary<byte, ParamInfo> type. ParamInfo has address, length, success or read data. Read only.|
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
Myself.

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
Myself.

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
Myself.

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

## ParamResult
Class representing data fields in BulkWritePacket and BulkReadPacket. 

### Variable
| Name|Description|
|--|--|
|address|Data address.|
|length|Length of data.|
|isSuccess|Whether the communication was successful or not.|
|data|In BulkWritePacket, data to be written; in BulkReadPacket, data to be read.|



## ControlTables

A class that stores the Control Table for each motor; the Control Table is initialized by putting text written in the specified format into the constructor argument. The constructor takes either string[] or TextAsset[] as an argument.

Text files of Control Table for DynamxielForUnity compatible models are located in Assets/HatsuMuv/DynamixelForUnity/Resources/ControlTables. See ControlTableInjectorFromResources for usage examples.

ControlTables is the ControlTable of the motor to be used during initialization. The first line describes the model name and model number with ":" in between. The other lines describe the data name, address, length, and access. For access, specify "R" for read-only and "RW" for write-enabled. Lines beginning with "#" or "//" are ignored as comments.
```=
<ModelName>:<ModelName>
<ItemName>, <Address>, <Length>, <Access>
<ItemName>, <Address>, <Length>, <Access>
...
```

### Property

| Name | Description |
| -------- | -------- |
| ModelNames | Dictionary<int, string> type that represents the model name corresponding to the model number.|



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
A structure representing information on data fields in the Control Table. 

### Variable
| Name|Description|
|--|--|
|address|Data address.|
|length|Length of data.|
|readWrite|True if writable, false if read-only.|

## IControlTableUser
Interface to inject control tables, implemented by DynamixelForUnity. 

### SetControlTables
- Declaration
`public void SetControlTables(ControlTables controlTables)`


## Core

Dynamixel SDK compatible class. Put a HatsuVerif object in the constructor and create an instance.
