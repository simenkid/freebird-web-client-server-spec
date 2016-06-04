Freebird Client/Server Message Formats (through websocket)
===============

version: v0.1.3 (latest updated: 2016/6/4)

## Table of Contents

1. [Overiew](#Overiew)  
2. [Interfaces](#Interfaces)  
    - [Request](#Request)  
    - [Response](#Response)  
    - [Indication](#Indication)  
3. [Data Model](#DataModel)  
    - [Request](#RequestData)  
    - [Response](#ResponseData)  
    - [Indication](#IndicationData)  
4. [RSP Status Codes](#RspCodes)  
5. [Appendix](#Appendix)  
    - [Device Information (devInfo) Object](#devInfoObj)  
    - [Gadget Information (gadInfo) Object](#gadInfoObj)  
    - [Netcore Information (ncInfo) Object](#ncInfoObj)  
    - [Gadget Classes](#gadClasses)  
    - [ Attribute Report Configuration Object](#reportCfg)  
  
### Change Logs  

####2016/03/29:  

* 3.Data Model >> Indication  
    * Indication Example: 'devIncoming'  
        - change `enable: true` to `enabled: true`  
    * Indication Example: 'gadIncoming'  
        - change `enable: true` to `enabled: true`  

* 5.Appendix >>  
    * Device Information (devInfo) Object  
        - Properties: Change Property 'enable' to 'enabled'  
        - Example: Change Property 'enable' to 'enabled'  
    * Gadget Information (gadInfo) Object  
        - Properties: Change Property 'enable' to 'enabled'  
        - Example: Change Property 'enable' to 'enabled'  
    * Netcore Information (ncInfo) Object  
        - Properties: Change Property 'enable' to 'enabled'  
        - Example: Change Property 'enable' to 'enabled'  

####2016/03/30

* 3.Data Model >> 
    * Request
        - Subsystem: change namespace 'dev' to 'net' of command 'ban'
        - Subsystem: change namespace 'dev' to 'net' of command 'unban'
    * Response
        - Response Data Type: change table title **Response Data Type** to **Data Key:Type**
        - Example: change all Response `data` type to an object  
        - Take command 'getAllDevIds' for example: **Data Key:Type** is _ids:Number[]_ and **Example** shows the data object given with `{ ids: [ 1, 2, 3, 8, 12 ] }` which has a key of `'ids'` and value of an array of numbers.  
    * Indication
        - Response Data Type: change table tile **Response Data Type** to **Data Key:Type**  
        - Indication Example: change all Indication `data` type to an object  
            - For example, `data` changes to `{ status: 'online' }` in Indication Example: 'statusChanged'

* Change indication type of 'attrChanged' to 'attrsChanged' 
* Change key `'attributes'` of gadInfo to `'attrs'`

####2016/4/1

* 5.Appendix >> 
    * Gadget Information (gadInfo) Object  
        - Change Property 'owner' to 'dev'  

####2016/4/2

* 5.Appendix >> 
    * Netcore Information (ncInfo) Object  
        - Typo: Change data type of startTime from _String_ to _Number_  

####2016/4/6

* Property `traffic` of devInfo and ncInfo objects is re-defined, the unit is changed from kBytes to bytes
    * OrIginal format: { in: 16, out: 72 }  (unit: kBytes)
    * New Format:

    ```js
        {
            in: {
                hits: 6,    // how many message received
                bytes: 48   // (unit: bytes)
            },
            out: {
                hits: 8,    // how many message transmitted
                bytes: 96   // (unit: bytes)
            }
        }
    ```

####2016/4/26

* 3.Data Model >> 
    * Request  
        - Move `remove` and `ping` form namespace 'dev' to 'net'  
        - Move `ban` and `unban` form namespace 'dev' to 'net'  

    * Response  
        - Move `remove` and `ping` form namespace 'dev' to 'net'  
        - Move `ban` and `unban` form namespace 'dev' to 'net'  


####2016/4/29 (Major Changes) - spec version upgrades to v0.1.0

* 2.Interfaces >> 
    * Indication  
        - **Indication types**: Re-organize all indications

* 3.Data Model >> 
    * Request  
        - Add commands 'getProps' and 'setProps' of 'dev' subsys
        - Add commands 'getProps' and 'setProps' of 'gad' subsys

    * Response  
        - Add response from commands 'getProps' and 'setProps' of 'dev' subsys
        - Add response from commands 'getProps' and 'setProps' of 'gad' subsys

    * Indication  
        - Re-organize all indications and their messages  

* 5.Appendix >> 
    * Device Information (devInfo) Object
        - devInfo turned into a new format

    * Gadget Information (gadInfo) Object
        - gadInfo turned into a new format

    * Netcore Information (ncInfo) Object
        - ncInfo turned into a new format

####2016/5/18

* 5.Appendix >> 
    * Gadget Information (gadInfo) Object  
        - Add Property 'netcore' to gadInfo object  

####2016/6/2

* 3.Data Model >> 
    * Request  
        - Add `enable` and `disable` APIs in namespace 'dev'  
        - Add `enable` and `disable` APIs in namespace 'gad'  
        - Add an `id` field to `args` for `getProps` and `setProps` APIs in namespace 'dev'  
        - Add an `id` field to `args` for `getProps` and `setProps` APIs in namespace 'gad'  

    * Response  
        - Add `enable` and `disable` reponses in namespace 'dev'  
        - Add `enable` and `disable` reponses in namespace 'gad'  

* 5.Appendix >> 
    * Gadget Information (gadInfo) Object  
        - Add Property 'auxId' to gadInfo object  

####2016/6/4

* 3.Data Model >> 
    * Request
        - Modify `exec` arguments example in namespace 'gad'
    

<br />
  
<a name="Overiew"></a>  
## 1. Overview  
  
This document describes the APIs of how a freebird web Client can communicate with the freebird Server through [websocket](http://www.websocket.org/). The APIs are based on a _**Request**_, _**Response**_, and _**Indication**_ messages model. The message object (JSON) has a `__intf` field to denote which type a message is. The message type can be **'REQ'**, **'RSP'**, or **'IND'**.  
  
The freebird framework has _**net**_, _**dev**_, and _**gad**_ subsystems responsible for network, device, and gadget management, respectively. In brief, a network is formed with many devices, and each device may have some gadgets on it. A gadget is the real application in a manchine network.  
  
Let's take a wifi weather station in a machine network for example. The weather station is made up of temperature, humidity, and pm2.5 sensors, where each sensor is a gadget. A device, such as Arduino(with wifi connection), ESP8266, MT7688, RaspberryPi or Beaglebone, is the carrier of applications(gadgets). Now we know, this weather station has 3 gadgets on it, but only has a single device in it. Here is another example, we have a bluetooth low-energy (BLE) light switch in the network. This is a simple one-device-with-one-gadget machine, we can say "there is only one gadget, a light switch, implemented on a TI CC2540 BLE SoC device."  
  
The concept of _**net**_, _**dev**_, and _**gad**_ subsystems in freebird framework well separates the machine management and application management from each other. This brings developers a more clear, convenient and flexible way in building up a IoT machine network.  
  
********************************************
  
<br />
  
<a name="Interfaces"></a>  
## 2. Interfaces  
  
In freebird, the web-Client and Server communicates with each other through websocket along with a JSON message. 
The `__intf` field in a message can be 'REQ', 'RSP' or 'IND' to denote the interface.  
  
<a name="Request"></a>
### Request  
  
- **Direction**:  
    Client sends to Server to request something or to ask the server to perform an operation.  
- **Interface**:  
    __intf = 'REQ'  
- **Message keys**:  
    { __intf, subsys, seq, id, cmd, args }  
  
    | Property | Type            | Description                                                                                                               |
    |----------|-----------------|---------------------------------------------------------------------------------------------------------------------------|
    | __intf   | String          | 'REQ'                                                                                                                     |
    | subsys   | String          | Only 3 types accepted. They are 'net', 'dev', and 'gad' to denote which subsystem is this message going to                |
    | seq      | Number          | Sequence number of this REQ/RSP transaction                                                                               |
    | id       | Number          | Id of the sender. id is meaningless if `subsys === 'net'`. id is **device id** if `subsys === 'dev'`. id is **gadget id** if `subsys === 'gad'`. It is noticed that id = 0 is reserved for the freebird web-Client and Server |
    | cmd      | String          | Command Identifier corresponding to the API name                                                                          |
    | args     | Object          | A value-object that contains command arguments. Please see section [Request Data Model](#RequestData) to learn more about the `args` data object |
  
- **Message Example**:  
  
    ```js
    { 
        __intf: 'REQ',
        subsys: 'net',
        seq: 3,
        id: 0,
        cmd: 'getDevs',
        args: {
            ids: [ 2, 4, 18, 61 ]
        }
    }
    ```
********************************************
  
<br />
  
<a name="Response"></a>
### Response  
  
- **Direction**:  
    Server responds to Client with the results of the client asking for.  
- **Interface**:  
    __intf = 'RSP'  
- **Message keys**:  
    { __intf, subsys, seq, id, cmd, status, data }  
  
    | Property | Type            | Description                                                                                                                        |
    |----------|-----------------|------------------------------------------------------------------------------------------------------------------------------------|
    | __intf   | String          | 'RSP'                                                                                                                              |
    | subsys   | String          | Only 3 types accepted. They are 'net', 'dev', and 'gad' to denote which subsystem is this message coming from                      |
    | seq      | Number          | Sequence number of this REQ/RSP transaction                                                                                        |
    | id       | Number          | Id of the sender. id is meaningless if `subsys === 'net'`. id is **device id** if `subsys === 'dev'`. id is **gadget id** if `subsys === 'gad'`. It is noticed that id = 0 is reserved for the freebird web-Client and Server  |
    | cmd      | String          | Command Identifier corresponding to the API name                                                                                   |
    | status   | Number          | [Status code of the response](#RspCodes)                                                                                           |
    | data     | Depends         | Data along with the response. To learn more about the data format corresponding to each command, please see section [Response Data Model](#ResponseData). |
  
- **Message Example**:  
  
    ```js
    { 
        __intf: 'RSP',
        subsys: 'net',
        seq: 17,
        id: 0,
        cmd: 'getAllDevIds',
        status: 0,
        data: {
            ids: [ 2, 4, 18, 61 ]
        }
    }
    ```
********************************************
  
<br />
  
<a name="Indication"></a>
### Indication  
  
- **Direction**:  
    Server indicates Client  
- **Interface**:  
    __intf = 'IND'  
- **Message keys**:  
    { __intf, subsys, type, id, data }  

    | Property | Type            | Description                                                                                                                                 |
    |----------|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------|
    | __intf   | String          | 'IND'                                                                                                                                       |
    | subsys   | String          | Only 3 types accepted. They are 'net', 'dev', and 'gad' to denote which subsystem is this indication coming from                            |
    | type     | String          | There are few types of indication accepted, such as 'attrsChanged'. Please see section [Indication types](#IndTypes) for details            |
    | id       | Number          | Id of the sender. id is meaningless if `subsys === 'net'`. id is **device id** if `subsys === 'dev'`. id is **gadget id** if `subsys === 'gad'` |
    | data     | Depends         | Data along with the indication. Please see section [Indication Data Model](#IndicationData) to learn more about the indication data format  |
  
<a name="IndTypes"></a>
- **Indication types**:  
   
    | subsys  | Indication Type      | Description                                                                                    |
    |---------|----------------------|------------------------------------------------------------------------------------------------|
    | net     | 'error'              | Netcore error occurs                                                                           |
    | net     | 'started'            | A netcore is started                                                                           |
    | net     | 'stopped'            | A netcore is stopped                                                                           |
    | net     | 'enabled'            | A netcore is enabled                                                                           |
    | net     | 'disabled'           | A netcore is disabled                                                                          |
    | net     | 'permitJoining'      | A netcore is now allowing devices to join the network                                          |
    | net     | 'bannedDevIncoming'  | A banned device is trying to join the network                                                  |
    | net     | 'bannedGadIncoming'  | A banned gadget is trying to join the network                                                  |
    | net     | 'bannedDevReporting' | A banned device is trying to report its attributes                                             |
    | net     | 'bannedGadReporting' | A banned gadget is trying to report its attributes                                             |
    | dev     | 'error'              | Device error occurs                                                                            |
    | dev     | 'devIncoming'        | A new device is incoming                                                                       |
    | dev     | 'devLeaving'         | A device is leaving                                                                            |
    | dev     | 'netChanged'         | Network information of a device has changed                                                    |
    | dev     | 'statusChanged'      | Status of a device has changed. The status can be 'online', 'sleep', 'offline', and 'unknown'  |
    | dev     | 'propsChanged'       | Meta-property(ies) of a device has changed                                                     |
    | dev     | 'attrsChanged'       | Attribue(s) on a device has changed                                                            |
    | dev     | 'attrsReport'        | A report message of certain attribute(s) on a device.                                          |
    | gad     | 'error'              | Gadget error occurs                                                                            |
    | gad     | 'gadIncoming'        | A new gadget is incoming                                                                       |
    | gad     | 'gadLeaving'         | A gadget is leaving                                                                            |
    | gad     | 'panelChanged'       | Panel information of a gadget has changed                                                      |
    | gad     | 'propsChanged'       | Meta-property(ies) of a gadget has changed                                                     |
    | gad     | 'attrsChanged'       | Attribue(s) on a gadget has changed                                                            |
    | gad     | 'attrsReport'        | A report message of certain attribute(s) on a gadget                                           |


- **Message Example**:  
  
    ```js
    { 
        __intf: 'IND',
        subsys: 'gad',
        type: 'attrsChanged',
        id: 147,            // sender of this indication is a gadget with id = 147
        data: {
            sensorValue: 24
        }
    }
    ```
  
********************************************
  
<br />
  
<a name="DataModel"></a>  
## 3. Data Model  
  
The data model presents the `args` and `data` formats in the REQ/RSP/IND messages.  
  
<a name="RequestData"></a>
### Request  
  
The request message is an object with keys { __intf, subsys, seq, id, cmd, args }. The following table gives the details of each API.  
  
| Subsystem | Command Name   | Arguments (args)         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------|----------------|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| net       | 'getAllDevIds' | { [ncName] }             | Get identifiers of all devices on freebird Server. **ncName** is a string and is optional. If **ncName** is given, only identifiers of devices managed by that netcore will be returned from Server.                                                                                                                                                                                                                                                                                                                                                                                         |
| net       | 'getAllGadIds' | { [ncName] }             | Get identifiers of all gadgets on freebird Server. **ncName** is a string and is optional. If **ncName** is given, only identifiers of gadgets managed by that netcore will be returned from Server.                                                                                                                                                                                                                                                                                                                                                                                         |
| net       | 'getDevs'      | { ids }                  | Get information of devices by their ids. **ids** is an array of numbers and each number is a device id, i.e., given `{ ids: [ 5, 6, 77 ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| net       | 'getGads'      | { ids }                  | Get gadget information by gadget id. **ids** is an array of numbers and each number is a gadget id, i.e., given `{ ids: [ 23, 14, 132 ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| net       | 'getNetcores'  | { ncNames }              | Get netcore information by netcore names. **ncNames** is an array of strings and each string is a netcore name, i.e., given `{ ncNames: [ 'ble-core', 'zigbee-core' ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| net       | 'getBlacklist' | { ncName }               | Get blacklist of the banned devices. **ncName** is the netcore name of which you like to get the blacklist from. **ncName** should be a string. i.e., given `{ ncName: 'ble-core' }`                                                                                                                                                                                                                                                                                                                                                                                                         |
| net       | 'permitJoin'   | { ncName, duration }     | Allow or disallow devices to join the network. **ncName** is the name of which netcore you like to allow for device joining and **ncName** should be a string. **duration** is the time in seconds which should be a number. Set duration to 0 will immediately close the admission. For example, given `{ ncName: 'zigbee-core', duration: 60 }` will allow zigbee devices to join the zigbee network for 60 seconds.                                                                                                                                                                       |
| net       | 'maintain'     | { ncName }               | Maintain the network. **ncName** is the name of which netcore you like to maintain. **ncName** should be a string. When a netcore starts to maintain its own network, all devices managed by it will be refreshed. For example, given `{ ncName: 'ble-core' }` to let the BLE netcore do its maintenance.                                                                                                                                                                                                                                                                                    |
| net       | 'reset'        | { ncName }               | Reset the network. **ncName** is the name of which netcore you like to reset. **ncName** should be a string. Reset a network will remove all devices managed by that netcore. Once reset, the banned devices in the netcore blacklist will also be removed.                                                                                                                                                                                                                                                                                                                                  |
| net       | 'enable'       | { ncName }               | Enable the network. **ncName** is the name of which netcore you like to enable. (The netcore is enabled by default.)                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| net       | 'disable'      | { ncName }               | Disable the network. **ncName** is the name of which netcore you like to disable. If netcore is disabled, no messages can be send out and received from remote devices. That is, messages will be ignored and you will not get any message from the netcore on freebird Server.                                                                                                                                                                                                                                                                                                              |
| net       | 'ban'          | { ncName, permAddr }     | Ban a device from the network. Once a device has been banned, freebird will always reject its joining request. If a device is already in the network, freebird will first remove it from the network. **ncName** is the netcore that manages the device you'd like to ban. **permAddr** is the permanent address of  the banned device. For example, given `{ ncName: 'zigbee-core', permAddr: '0x00124b0001ce4b89' }` to ban a zigbee device with an IEEE address of 0x00124b0001ce4b89. The permanent address depends on protocol, such as IEEE address for zigbee devices, BD address for BLE devices, and MAC address for IP-based devices. |
| net       | 'unban'        | { ncName, permAddr }     | Unban a device. **ncName** is the netcore that manages the banned device. **permAddr** is the permanent address of the banned device. For example, given `{ ncName: 'zigbee-core', permAddr: '0x00124b0001ce4b89' }` to unban a zigbee device with an IEEE address of 0x00124b0001ce4b89.                                                                                                                                                                                                                                                                                                    |
| net       | 'remove'       | { id }                   | Remove a device from the network. **id** is the id of which device to remove.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| net       | 'ping'         | { id }                   | Ping a device in the network. **id** is the id of which device you like to ping.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| dev       | 'enable'       | { id }                   | Enable the device to activate its function and messages transportation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| dev       | 'disable'      | { id }                   | Disable the device.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| dev       | 'read'         | { id, attrName }         | Read an attribute on a device. **id** is the id of which device you like to read from. **attrName** is the attribute you like to read. For example, given `{ id: 20, attrName: 'location' }` to read the location attribute from the device with id = 20.                                                                                                                                                                                                                                                                                                                                    |
| dev       | 'write'        | { id, attrName, value }  | Write a value to an attribute on a device. **id** is the id of which device you like to write a value to. **attrName** is the attribute to be written. For example, given `{ id: 20, attrName: 'location', value: 'kitchen' }` to set the device location attribute to 'kitchen'.                                                                                                                                                                                                                                                                                                            |
| dev       | 'identify'     | { id }                   | Identify a device in the network. **id** is the id of which device to be identified.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| dev       | 'getProps'     | { id, propNames }        | Get device meta-properties. **propNames** is an array of strings and each string is a meta-property name, i.e., given `{ id: 3, propNames: [ 'location', 'description' ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                 |
| dev       | 'setProps'     | { id, props }            | Set device meta-properties. **props** is an object contains meta-properties to set, i.e., given `{ id: 60, props: { location: 'bedroom', description: 'for my baby' } }`.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| gad       | 'enable'       | { id }                   | Enable the gadget to activate its function and messages transportation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| gad       | 'disable'      | { id }                   | Disable the gadget.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| gad       | 'read'         | { id, attrName }         | Read an attribute on a gadget. **id** is the id of which gadget you like to read from. **attrName** is the attribute you like to read. For example, given `{ id: 2316, attrName: 'sensorValue' }` to read the sensed value attribute from a temperature sensor (the sensor is a gadget with id = 2316).                                                                                                                                                                                                                                                                                      |
| gad       | 'write'        | { id, attrName, value }  | Write a value to an attribute on a gadget. **id** is the id of which gadget you like to write a value to. **attrName** is the attribute to be written. For example, given `{ id: 1314, attrName: 'onOff', value: 1 }` to turn on a light bulb (the light bulb is a gadget with id = 1314).                                                                                                                                                                                                                                                                                                   |
| gad       | 'exec'         | { id, attrName[, params] } | Invoke a remote procedure on a gadget. **id** is the id of which gadget you like to perform its particular procedure. **attrName** is the attribute name of an executable procedure. **params** is an array of parameters given in order to meet the procedure signature. The signature depends on how a developer declare his(/her) own procedure. For example, given `{ id: 9, attrName: 'blink', params: [ 10, 500 ] }` to blink a LED on a gadget 10 times with 500ms interval.                                                                                                         |
| gad       | 'setReportCfg' | { id, attrName, [rptCfg](#reportCfg) }    | Set the condition for an attribute reporting from a gadget.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| gad       | 'getReportCfg' | { id, attrName }         | Get the report settings of an attribute on a gadget.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| gad       | 'getProps'     | { id, propNames }        | Get gadget meta-properties. **propNames** is an array of strings and each string is a meta-property name, i.e., given `{ id: 8, propNames: [ 'description' ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                             |
| gad       | 'setProps'     | { id, props }            | Set gadget meta-properties. **props** is an object contains meta-properties to set, i.e., given `{ id: 12, props: { description: 'temperature sensor 3', installed: true } }`.                                                                                                                                                                                                                                                                                                                                                                                                               |
  
********************************************
  
<a name="ResponseData"></a>
### Response  
  
The response message is an object with keys { __intf, subsys, seq, id, cmd, status, data }. `status` shows if the request is successful. The `data` field contains the respond data according to the request. `data` will always be an object, and it will be an empty object if the request is unsuccessful.  
  
| Subsystem | Command Name   | Data Key:Type      | Data Description                                     | Example                                               |
|-----------|----------------|--------------------|------------------------------------------------------|------------------------------------------------------ |
| net       | 'getAllDevIds' | ids:Number[]       | Array of device identifiers                          | { ids: [ 1, 2, 3, 8, 12 ] }                           |
| net       | 'getAllGadIds' | ids:Number[]       | Array of gadget identifiers                          | { ids: [ 2, 3, 5, 11, 12, 13, 14, 15 ] }              |
| net       | 'getDevs'      | devs:devInfo[]     | Array of device information objects                  | { devs: [ [devInfo](#devInfoObj), ...  ] }            |
| net       | 'getGads'      | gads:gadInfo[]     | Array of gadget information objects                  | { gads: [ [gadInfo](#gadInfoObj) , ... ] }            |
| net       | 'getNetcores'  | netcores:ncInfo[]  | Array of netcore information objects                 | { netcores: [ [ncInfo](#ncInfoObj), ... ] }           |
| net       | 'getBlacklist' | list:String[]      | Array of banned device permanent address             | { list: [ '0x00124b0001ce4b89', ... ] }               |
| net       | 'permitJoin'   | -                  | Response contains no data                            | {}                                                    |
| net       | 'maintain'     | -                  | Response contains no data                            | {}                                                    |
| net       | 'reset'        | -                  | Response contains no data                            | {}                                                    |
| net       | 'enable'       | -                  | Response contains no data                            | {}                                                    |
| net       | 'disable'      | -                  | Response contains no data                            | {}                                                    |
| net       | 'ban'          | -                  | Response contains no data                            | {}                                                    |
| net       | 'unban'        | -                  | Response contains no data                            | {}                                                    |
| net       | 'remove'       | permAddr:String    | Device permanent address                             | { permAddr: '0x00124b0001ce4b89' }                    |
| net       | 'ping'         | time:Number        | Round-trip time in ms                                | { time: 12 }                                          |
| dev       | 'enable'       | enabled:Boolean    | To show if the device is enabled                     | { enabled: true }                                     |
| dev       | 'disable'      | enabled:Boolean    | To show if the device is enabled                     | { enabled: false }                                    |
| dev       | 'read'         | value:Depends      | The read value. Can be anything                      | { value: 3 }                                          |
| dev       | 'write'        | value:Depends      | The written value. Can be anything                   | { value: 'kitchen' }                                  |
| dev       | 'identify'     | -                  | Response contains no data                            | {}                                                    |
| dev       | 'getProps'     | props:Depends      | Meta-properties got. The props got is an object.     | { props: { name: 'mywifi-dev', location: 'kitchen' } } |
| dev       | 'setProps'     | -                  | Response contains no data                            | {}                                                    |
| gad       | 'enable'       | enabled:Boolean    | To show if the gadget is enabled                     | { enabled: true }                                     |
| gad       | 'disable'      | enabled:Boolean    | To show if the gadget is enabled                     | { enabled: false }                                    |
| gad       | 'read'         | value:Depends      | The read value. Can be anything                      | { value: 371.42 }                                     |
| gad       | 'write'        | value:Depends      | The written value. Can be anything                   | { value: false }                                      |
| gad       | 'exec'         | result:Depends     | The data returned by the procedure. Can be anything  | { result: 'completed' }                               |
| gad       | 'setReportCfg' | -                  | Response contains no data                            | {}                                                    |
| gad       | 'getReportCfg' | cfg:Object         | Report settings object                               | { cfg: [rptCfg](#reportCfg) }                         |
| gad       | 'getProps'     | props:Depends      | Meta-properties got. The props got is an object.     | { props: { name: 'mysensor1' } }                      |
| gad       | 'setProps'     | -                  | Response contains no data                            | {}                                                    |
  

********************************************

<br />
  
<a name="IndicationData"></a>
### Indication  
  
The indication message is an object with keys { __intf, subsys, type, id, data }. `type` shows the type of indication. The `data` field contains indication data which is an object.  

| subsys  | Indication Type     | Data Object **Key:Type**                                       | Description                                                                                    |
|---------|---------------------|----------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| net     | 'error'             | `{ netcore:String, error: { code:Number, info:String } }`      | Netcore error occurs                                                                           |
| net     | 'started'           | `{ netcore:String }`                                           | A netcore is started                                                                           |
| net     | 'stopped'           | `{ netcore:String }`                                           | A netcore is stopped                                                                           |
| net     | 'enabled'           | `{ netcore:String }`                                           | A netcore is enabled                                                                           |
| net     | 'disabled'          | `{ netcore:String }`                                           | A netcore is disabled                                                                          |
| net     | 'permitJoining'     | `{ netcore:String, timeLeft:Number }`                          | A netcore is now allowing or disallowing devices to join the network                           |
| net     | 'bannedDevIncoming' | `{ netcore:String, permAddr:String }`                          | A banned device is trying to join the network                                                  |
| net     | 'bannedGadIncoming' | `{ netcore:String, permAddr:String, auxId:String \| Number }`  | A banned gadget is trying to join the network                                                  |
| net     | 'bannedDevReporting'| `{ netcore:String, permAddr:String }`                          | A banned device is trying to report its attributes                                             |
| net     | 'bannedGadReporting'| `{ netcore:String, permAddr:String, auxId:String \| Number }`  | A banned gadget is trying to report its attributes                                             |
| dev     | 'error'             | `{ code:Number, info:String }`                                 | Device error occurs                                                                            |
| dev     | 'devIncoming'       | [devInfo](#devInfoObj)                                         | A new device is incoming                                                                       |
| dev     | 'devLeaving'        | -                                                              | A device is leaving                                                                            |
| dev     | 'netChanged'        | [netInfo](#netInfoObj)                                         | Network information of a device has changed                                                    |
| dev     | 'statusChanged'     | `{ status:String }`                                            | Status of a device has changed. The status can be 'online', 'sleep', 'offline', and 'unknown'  |
| dev     | 'propsChanged'      | [devInfo.props](#devPropsObj)                                  | Meta-property(ies) of a device has changed                                                     |
| dev     | 'attrsChanged'      | [devInfo.attrs](#devAttrsObj)                                  | Attribue(s) on a device has changed                                                            |
| dev     | 'attrsReport'       | [devInfo.attrs](#devAttrsObj)                                  | A report message of certain attribute(s) on a device.                                          |
| gad     | 'error'             | `{ code:Number, info:String }`                                 | Gadget error occurs                                                                            |
| gad     | 'gadIncoming'       | [gadInfo](#gadInfoObj)                                         | A new gadget is incoming                                                                       |
| gad     | 'gadLeaving'        | -                                                              | A gadget is leaving                                                                            |
| gad     | 'panelChanged'      | [gadInfo.pannel](#gadPanelObj)                                 | Panel information of a gadget has changed                                                      |
| gad     | 'propsChanged'      | [gadInfo.props](#gadPropsObj)                                  | Meta-property(ies) of a gadget has changed                                                     |
| gad     | 'attrsChanged'      | [gadInfo.attrs](#gadAttrsObj)                                  | Attribue(s) on a gadget has changed                                                            |
| gad     | 'attrsReport'       | [gadInfo.attrs](#gadAttrsObj)                                  | A report message of certain attribute(s) on a gadget                                           |

* Indication Example: indMsg of 'permitJoining'  

    ```js
    {
        __intf: 'IND',
        subsys: 'net',
        type: 'permitJoining',
        id: 0,
        data: {
            netcore: 'zigbee-core',
            timeLeft: 60            // netcore does not allow for devices to join the network if timeLeft is 0
        }
    }
    ```
  
* Indication Example: indMsg of 'devIncoming'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'dev',
        type: 'devIncoming',
        id: 18,                 // device id is 18
        data: {                 // data is devInfo object
            netcore: 'mqtt-core',
            id: 18,
            gads: [ 116, 117, 120 ],
            net: {
                enabled: true,
                joinTime: 1458008311,
                timestamp: 1458008617,
                role: 'client',         // depends on protocol
                parent: '0',            // parent is the netcore
                status: 'online'
                address: {
                    permanent: '00:0c:29:ff:ed:7c',
                    dynamic: '192.168.1.24'
                },
                traffic: {              // accumulated data since device joined
                    in: {
                        hits: 2,
                        bytes: 24
                    },
                    out: {
                        hits: 4,
                        bytes: 32
                    }
                }
            },
            props: {    // props: name, description, and location are writable and can be modified by users
                name: 'sample_device',
                description: 'This is a device example',
                location: 'bedroom'
            },
            attrs: {
                manufacturer: 'freebird',
                model: 'lwmqn-7688-duo',
                serial: 'lwmqn-2016-04-29-03',
                version: {
                    hw: 'v1.2.0',
                    sw: 'v0.8.4',
                    fw: 'v2.0.0'
                },
                power: {
                    type: 'line',
                    voltage: '5V'
                }
            }
        }
    }
    ```
  
* Indication Example: indMsg of 'devLeaving'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'dev',
        type: 'devLeaving',
        id: 27,                   // device id is 27
        data: undefined
    }
    ```
  
* Indication Example: indMsg of device 'netChanged'  

    ```js
    {
        __intf: 'IND',
        subsys: 'dev',
        type: 'netChanged',
        id: 18,                 // device id is 18
        data: {                 // partial changes of netInfo object
            address: {
                dynamic: '192.168.1.32'
            },
            status: 'online'    // status changed, an 'statusChanged' indication will also be fired  
        }
    }
    ```
  
* Indication Example: indMsg of device 'statusChanged'  

    ```js
    {
        __intf: 'IND',
        subsys: 'dev',
        type: 'statusChanged',
        id: 18,                 // device id is 18
        data: {
            status: 'online'
        }
    }
    ```
  
* Indication Example: indMsg of device 'propsChanged'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'dev',
        type: 'propsChanged',
        id: 37,                   // device id is 37
        data: {
            location: 'kitchen'
        }
    }
    ```

* Indication Example: indMsg of device 'attrsChanged'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'dev',
        type: 'attrsChanged',
        id: 16,                   // device id is 16
        data: {
            version: {
                sw: 'v1.2.3'
            },
            serial: '2016-05-01-001'
        }
    }
    ```

* Indication Example: indMsg of device 'attrsReport'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'dev',
        type: 'attrsReport',
        id: 77,                   // device id is 77
        data: {
            power: {
                voltage: '12V'
            }
        }
    }
    ```

* Indication Example: indMsg of 'gadIncoming'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'gad',
        type: 'gadIncoming',
        id: 2763,                   // gadget id is 2763
        data: {
            id: 2763,       // gadget id
            dev: {
                id: 822,    // device id
                permAddr: '0x00124b0001ce4b89'
            },
            panel: {
                enabled: true,
                profile: 'home_automation',
                class: 'lightCtrl'
            },
            props: {
                name: 'sampleLight',
                description: 'This is a simple light controller'
            },
            attrs: {
                onOff: 1,
                dimmer: 80
            }
        }
    }
    ```
  
* Indication Example: indMsg of 'gadLeaving'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'gad',
        type: 'gadLeaving',
        id: 41,                   // gadget id is 41
        data: undefined
    }
    ```
  
* Indication Example: indMsg of gadget 'panelChanged'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'gad',
        type: 'panelChanged',
        id: 21,                   // gadget id is 21
        data: {
            enabled: true
        }
    }
    ```
  
* Indication Example: indMsg of gadget 'propsChanged'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'gad',
        type: 'propsChanged',
        id: 52,                   // gadget id is 52
        data: {
            description: 'A good smart radio in my bedroom'
        }
    }
    ```
  
* Indication Example: indMsg of gadget 'attrsChanged'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'gad',
        type: 'attrsChanged',
        id: 83,                   // gadget id is 83
        data: {
            onOff: 0
        }
    }
    ```
  
* Indication Example: indMsg of gadget 'attrsReport'  
  
    ```js
    {
        __intf: 'IND',
        subsys: 'gad',
        type: 'attrsReport',
        id: 64,                   // gadget id is 64
        data: {
            sensorValue: 18
        }
    }
    ```
  
********************************************

<br />
  
<a name="RspCodes"></a>
## 4. RSP Status Code  
  
[**CAUTION**] The response code is TBD and is subject to change.  
  
| Code Id | Code Name      | Description                                                                        |
|---------|----------------|------------------------------------------------------------------------------------|
| 0       | 'success'      | Response is ok                                                                     |
| 1       | 'fail'         | Operation fails                                                                    |
| 2       | 'busy'         | Server or remote device is busy. Try later                                         |
| 3       | 'unavail'      | Remote device is unreachable, it may be offline or sleeping                        |
| 4       | 'badRequest'   | Request parameter in arguments cannot be recognized or given with wrong type       |
| 5       | 'notFound'     | The allocated device, gadget is not found                                          |
| 6       | 'notAllowed'   | Method is not allowed. For example, write a value to a read-only attribute         |
| 7       | 'unauthorized' | The operation is unauthorized                                                      |
| 8       | 'timeout'      | Request timeout                                                                    |

********************************************

<br />
  
<a name="Appendix"></a>
## 5. Appendix  
  
<a name="devInfoObj"></a>
### Device Information (devInfo) Object
  
* Properties  

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | id           | Number          | Device id                                                                                             |
    | netcore      | String          | Name of the netcore that holds this device                                                            |
    | gads         | Number[]        | A list of gadget ids that this device owns                                                            |
    | net          | Object          | Network information of the device.                                                                    |
    | props        | Object          | Meta-properties of the device. This is for client users to set something to the device                |
    | attrs        | Object          | Attributes of the device. This is attributes of the remote device.                                    |

    - `net` object

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | enabled      | Boolean         | Is this device enabled?                                                                               |
    | joinTime     | Number          | Device join time. UNIX time in secs                                                                   |
    | timestamp    | Number          | Device last activity. UNIX time in secs                                                               |
    | role         | String          | Device role. Depends on protocol, i.e., 'peripheral' for BLE devices, 'router' for zigbee devices     |
    | parent       | String          | Parent device permanent address. It is string '0' if device parent is its netcore                     |
    | status       | String          | Device status, can be 'online', 'sleep', 'offline', or 'unknown'                                      |
    | address      | Object          | Device permanent and dynamic addresses. { permanent: '00:0c:29:ff:ed:7c', dynamic: '192.168.1.101' }  |
    | traffic      | Object          | Accumulated inbound and outbound data since device joined. { in: { hits: 6, bytes: 24 }, out: { hits: 3, bytes: 30 } } (unit: bytes)        |


    <a name="devPropsObj"></a>
    - `props` object

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | name         | String          | Device name. This is not on the remote device and can be set by end-users.                            |
    | description  | String          | Device description. This is not on the remote device and can be set by end-users.                     |
    | location     | String          | Device location. This is not on the remote device and can be set by end-users.                        |
    | _others_     | Any             | Anything end-users like to set to device `props` by setProps API.                                     |

    <a name="devAttrsObj"></a>
    - `attrs` object

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | manufacturer | String          | Manufacturer name                                                                                     |
    | model        | String          | Model name                                                                                            |
    | version      | Object          | Version tags. { hw: '', sw: 'v1.2.2', fw: 'v0.0.8' }                                                  |
    | power        | Object          | Power source. { type: 'battery', voltage: '5V' }. The type can be 'line', 'battery' or 'harvester'    |
  
* Example  
  
    ```js
    {
        id: 6,
        netcore: 'mqtt-core',
        gads: [ 116, 117 ],
        net: {
            enabled: true,
            joinTime: 1458008208,
            timestamp: 1458008617,
            role: 'client',         // depends on protocol
            parent: '0',            // parent is the netcore
            status: 'online'
            address: {
                permanent: '00:0c:29:ff:ed:7c',
                dynamic: '192.168.1.73'
            },
            traffic: {              // accumulated data since device joined
                in: {
                    hits: 6,
                    bytes: 72
                },
                out: {
                    hits: 12,
                    bytes: 96
                }
            }
        },
        props: {    // props: name, description, and location are writable and can be modified by users
            name: 'sample_device',
            description: 'This is a device example',
            location: 'balcony'
        },
        attrs: {
            manufacturer: 'freebird',
            model: 'lwmqn-7688-duo',
            serial: 'lwmqn-2016-03-15-01',
            version: {
                hw: 'v1.2.0',
                sw: 'v0.8.4',
                fw: 'v2.0.0'
            },
            power: {
                type: 'line',
                voltage: '5V'
            }
        }
    }
    ```

<a name="gadInfoObj"></a>
### Gadget Information (gadInfo) Object
  
* Properties  

    | Property     | Type             | Description                                                                                               |
    |--------------|------------------|-----------------------------------------------------------------------------------------------------------|
    | id           | Number           | Gadget id                                                                                                 |
    | netcore      | String           | Name of the netcore that holds this gadget                                                                |
    | dev          | Object           | Id and permanent address of which device owns this gadget. { id: 3, permAddr: '0x00124b0001ce4b89' }      |
    | auxId        | String           | The auxiliary id to identify the gadget on a device                                                       |
    | panel        | Object           | Basic information about the gadget. { enabled: true, profile: 'home', class: 'temperature' }              |
    | props        | Object           | Meta-properties of this gadget. This is for client users to set something to the gadget                   |
    | attrs        | Object           | Attributes of this gadget                                                                                 |

    <a name="gadPanelObj"></a>
    - `panel` object

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | enabled      | Boolean         | Is this gadget enabled?                                                                               |
    | profile      | String          | Optional. The profile defines the application environment, may mot be given.                          |
    | class        | String          | The class defines what application the gadget is. The [gadget class](#gadClasses) to denote its application, i.e. 'illuminance', 'temperature', 'lightCtrl'                          |

    <a name="gadPropsObj"></a>
    - `props` object

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | name         | String          | Gadget name. This is not on the remote device and can be set by end-users.                            |
    | description  | String          | Gadget description. This is not on the remote device and can be set by end-users.                     |
    | _others_     | Any             | Anything end-users like to set to gadget `props` by setProps API.                                     |

    <a name="gadAttrsObj"></a>
    - `attrs` object

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | _Depends_    | Any             | The gadget attributes that could be read, written, or excuted on the remote device.                   |

  

* Example  
  
    ```js
    {
        id: 308,
        netcore: 'zb-core',
        dev: {
            id: 26,
            permAddr: '0x00124b0001ce4b89'
        },
        panel: {
            enabled: true,
            profile: 'home_automation', // it will be an empty string '' if no profile given
            class: 'lightCtrl'
        },
        props: {    // props: name and description writable and can be modified by end-users
            name: 'sampleLight',
            description: 'This is a simple light controller'
        },
        attrs: {
            onOff: 1,
            dimmer: 80
        }
    }
    ```
  
<a name="ncInfoObj"></a>
### Netcore Information (ncInfo) Object
  
* Properties  
  
    | Property        | Type            | Description                                                                                      |
    |-----------------|-----------------|--------------------------------------------------------------------------------------------------|
    | name            | String          | Netcore name                                                                                     |
    | enabled         | Boolean         | Is this netcore enabled?                                                                         |
    | protocol        | Object          | Network protocol of this netcore.                                                          |
    | startTime       | Number          | Start time of this netcore. (UNIX time in secs)                                                  |
    | traffic         | Object          | Accumulated inbound and outbound data since netcore started. { in: { hits: 6, bytes: 24 }, out: { hits: 3, bytes: 30 } } (unit: bytes)  |
    | numDevs         | Number          | Number of devices managed by this netcore                                                        |
    | numGads         | Number          | Number of gadgets managed by this netcore                                                        |
  
* Example  
  
    ```js
    {
        name: 'zigbee-core',
        enabled: true,
        protocol: {              // required
            phy: 'ieee802.15.4', // required, physical layer
            dll: '',             // optional, data link layer
            nwk: 'zigbee',       // required, network layer
            tl: '',              // optional, transportation layer
            sl: '',              // optional, session layer
            pl: '',              // optional, presentation layer
            apl: 'zcl',          // optional, application layer
        },
        startTime: 1458008208,
        defaultJoinTime: 180,
        traffic: {
            in: {
                hits: 6,
                bytes: 24
            },
            out: {
                hits: 3,
                bytes: 30
            }
        },
        numDevs: 32,
        numGads: 46
    }
    ```

  
<a name="gadClasses"></a>
### Gadget Classes

Freebird framework uses the class property on a gadget to define its application. The classes are Smart Object Identfiers defined by [IPSO SmartObject Guideline(Smart Objects Starter Pack1.0)](http://www.ipso-alliance.org/smart-object-guidelines/). Here is the table of Object ids from [lwm2m-id](https://github.com/simenkid/lwm2m-id#5-table-of-identifiers) library.

| Class Name                 | Description            |
|----------------------------|------------------------|
| 'dIn'                      | Digital Input          |
| 'dOut'                     | Digital Output         |
| 'aIn'                      | Analogue Input         |
| 'aOut'                     | Analogue Output        |
| 'generic'                  | Generic Sensor         |
| 'illuminance'              | Illuminance Sensor     |
| 'presence'                 | Presence Sensor        |
| 'temperature'              | Temperature Sensor     |
| 'humidity'                 | Humidity Sensor        |
| 'pwrMea'                   | Power Measurement      |
| 'actuation'                | Actuation              |
| 'setPoint'                 | Set Point              |
| 'loadCtrl'                 | Load Control           |
| 'lightCtrl'                | Light Control          |
| 'pwrCtrl'                  | Power Control          |
| 'accelerometer'            | Accelerometer          |
| 'magnetometer'             | Magnetometer           |
| 'barometer'                | Barometer              |

<a name="reportCfg"></a>
### Attribute Report Configuration (rptCfg) Object
  
* Properties  
  
    | Property     | Type         | Mandatory | Description                                                                                      |
    |--------------|--------------|-----------|--------------------------------------------------------------------------------------------------|
    | pmin         | Number       |  optional | Minimum Period. Minimum time in seconds the gadget should wait from the time when sending the last notification to the time when sending a new notification.                                                                                     |
    | pmax         | Number       |  optional | Maximum Period. Maximum time in seconds the gadget should wait from the time when sending the last notification to the time sending the next notification (regardless if the value has changed).                                                                         |
    | gt           | Number       |  optional | Greater Than. The gadget should notify its attribute when the value is greater than this setting. Only valid for the attribute typed as a number.                                                          |
    | lt           | Number       |  optional | Less Than. The gadget should notify its attribute when the value is smaller than this setting. Only valid for the attribute typed as a number.                                                        |
    | step         | Number       |  optional | Step. The gadget should notify its value when the change of the attribute value, since the last report happened, is greater than this setting.                                                        |
    | enable       | Boolean      | required  | It is set to true for the gadget to start reporting an attribute. Set to false to stop reporting. |
  
* Example  
  
    ```js
    // start reporting with the following settings
    {
        pmin: 10,
        pmax: 60,
        lt: 120
        gt: 260
        enable: true
    }

    // the time chart of reporting: (O: report triggered, xxxxx: pmin, -----: pmax)
    //       O             O     O             O     O             O     O             O
    // |xxxxx|-------------|xxxxx|-------------|xxxxx|-------------|xxxxx|-------------|
    // 0s   10s           70s   80s           140s 150s           210s  220s          280s
    //  <10s> <    60s    >

    // note:
    //     1. only in |---------| duration, any change meets lt or gt condition will be reported
    //     2. In this example, the attribute will be reported when its value is greater than 260 or is less than 120
    ```

  
    ```js
    // start periodical reporting
    {
        pmin: 0,
        pmax: 30,
        enable: true
    }

    // the time chart of reporting: (O: report triggered, xxxxx: pmin, -----: pmax)
    // O             O             O             O             O             0
    // |-------------|-------------|-------------|-------------|-------------|
    // 0s           30s           60s           90s          120s          150s

    // note:
    //     1. In this example, the attribute will be reported every 30 seconds
    ```
  
    ```js
    // set but not start reporting
    {
        gt: 50
        enable: false
    }
    ```
  
    ```js
    // start reporting with the current settings
    {
        enable: true
    }
    ```
  
    ```js
    // stop reporting
    {
        enable: false
    }
    ```
  