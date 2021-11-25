# JbdBms library for Arduino v0.21

This is a library for working with battery manager system JBD.
If you can configure your BMS trough the JBDTool, then this code will suit you.
See [my project in hack a day](https://hackaday.io/project/162806-jbd-bms-protocol).

## Hardware

Connect UART pins to an Arduino board DIGITAL pins:
* RX
* TX
* GND

Attention! UART GND circuit is electrically isolated from the power GND BMS.

## Usage

### Initialization

```c++
#include "JbdBms.hpp"

JbdBms myBms(6,7); // RX, TX

void setup()
{
  myBms.begin();
}
```
### Reading basic data from BMS

```c++
if (myBms.readBmsData()== true)
{
  // use the get-functions
}
```
After calling `readBmsData ()` request is sent to the BMS.
In case of a successful read response, the method returns `true`.

You can work with the data obtained using the get-functions: getChargePercentage(), getCurrent(), getProtectionState().

### Get-functions
#### Charge percentage
```c++
myBms.getChargePercentage();
```
This method return the float value.

#### Total Voltage
```c++
myBms.getVoltage();
```
This method return the float value.  
unit is 1.0V  

#### Consumption current
```c++
myBms.getCurrent();
```
This method return the float value.
If it exceeds 32768, xor it and return it as a negative value.  
unit is 0.01A.  

#### Residual Capacity
```c++
myBms.getResidualcap();
```
This method return the float value.  
unit is 1.0Ah.  

#### Protection state
```c++
myBms.getProtectionState();
```
This method return the uint16_t value.

If this value is not 0, then BMS detected some errors. You can check this value on the mask by using the following error values:
```c++
#define BMS_STATUS_OK				0
#define BMS_STATUS_CELL_OVP			1
#define BMS_STATUS_CELL_UVP			2		///< Power off
#define BMS_STATUS_PACK_OVP			4
#define BMS_STATUS_PACK_UVP			8		///< Power off
#define BMS_STATUS_CHG_OTP			16
#define BMS_STATUS_CHG_UTP			32
#define BMS_STATUS_DSG_OTP			64		///< Power off
#define BMS_STATUS_DSG_UTP			128		///< Power off
#define BMS_STATUS_CHG_OCP			256
#define BMS_STATUS_DSG_OCP			512		///< Power off
#define BMS_STATUS_SHORT_CIRCOUT	1024	///< Power off
#define BMS_STATUS_AFE_ERROR		2048
#define BMS_STATUS_SOFT_LOCK		4096
#define BMS_STATUS_CHGOVERTIME		8192
#define BMS_STATUS_DSGOVERTIME		16384	///< Power off
#define BMS_POWER_OFF_ERRORS		0x46CA
```
Some errors lead to power down (labeled "Power off"). If you want to read them a device that is powered from the battery, it will not work.

### MosFET Status
```c++
int mosfet_state = myBms.getMosfet();
```
This method return the integer value.

return 0 : Charge MosFET OFF   DisCagrge MosFET OFF  
return 1 : Charge MosFET ON    DisCagrge MosFET OFF  
return 2 : Charge MosFET OFF   DisCagrge MosFET ON  
return 3 : Charge MosFET ON    DisCagrge MosFET ON  

### Cycle counter
```c++
int cycleCount = myBms.getCycle();
```
This method return the integer value.

### Data from temperature sensors
Depending on your BMS, the sensor can be located either on the board or be external.
Find it out empirically.
```c++
float temp1 = myBms.getTemp1();
float temp2 = myBms.getTemp2();
```

### Reading voltage on the cells

```c++
if (myBms.readPackData()== true)
{
  // use the get-functions
}
```
After calling `readPackData ()` request is sent to the BMS.
In case of a successful read response, the method returns `true`.

You can work with the data obtained using the get-function getPackCellInfo().

#### Voltage on the cells
```c++
packCellInfoStruct cellInfo;
cellInfo = myBms.getCurrent();
```
This method return the packCellInfoStruct.
Structure structure is presented below:
```c++
typedef struct packCellInfoStruct{
  uint8_t NumOfCells;
  uint16_t CellVoltage[15]; // Max 16 Cell BMS
  uint16_t CellLow; // Minimal voltage in cells
  uint16_t CellHigh; // Maximal voltage in cells
  uint16_t CellDiff; // difference between highest and lowest
  uint16_t CellAvg; // Average voltage in cells
} packCellInfoStruct;
```

