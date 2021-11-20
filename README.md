Tested on DELONGHI Ecam 370.95.S

# Protocol

Based on:
https://github.com/manekinekko/cafy
https://github.com/otto-dev/delonghi-coffee-link-python


## Request/Response Packet Format

### Request Packet

```text
  00              01                      N               n-1                n
+----+----------------------------+---+---+---+---+------------------+----------------+
| 0d |   request packet size (n)  |     data      |  checksum byte   | checksum byte  |
+----+----------------------------+---+---+---+---+------------------+----------------+
```

### Response Packet

```text
  00              01                      N               n-1                n
+----+----------------------------+---+---+---+---+------------------+----------------+
| d0 |   request packet size (n)  |     data      |  checksum byte   | checksum byte  |
+----+----------------------------+---+---+---+---+------------------+----------------+
```

### Checksum Algorithm

```javascript
let deviser = 0x1d0f;
for (let byteIndex = 0; byteIndex < bytes.length - 2; byteIndex++) {
  let i3 =
    (((deviser << 8) | (deviser >>> 8)) & 0x0000ffff) ^
    (bytes[byteIndex] & 0xffff);
  let i4 = i3 ^ ((i3 & 0xff) >> 4);
  let i5 = i4 ^ ((i4 << 12) & 0x0000ffff);
  deviser = i5 ^ (((i5 & 0xff) << 5) & 0x0000ffff);
}
checksum = deviser & 0x0000ffff;
```

Example: `bytes=[0d 05 75 0f da 25]`

1. for all bytes except the last 2 bytes: `0d 05 75 0f`
   1. let i3 = (((i << 8) | (i >>> 8)) & 0x0000ffff) ^ (bytes[0] & 0xffff);
      1. 0x1d0f << 8 = 0x1d0f00 (1904384)
      1. 0x1d0f >>> 8 = 0x1d (29)
      1. (0x1d0f00 | 0x1d) = 0x1d0f1d (1904413)
      1. (0x1d0f1d & 0x0000ffff) = 0xf1d (3869)
      1. (0xd0 & 0xffff) = 0xd0 (-48)
      1. 0xf1d ^ 0xd0 = 0xfcd (4045)
   1. let i4 = i3 ^ ((i3 & 0xff) >> 4);
      1. 0xfcd & 0xff = 0xcd
      1. 0xcd >> 4 = 0xc
      1. 0xfcd ^ 0xc = 0xfc1
   1. let i5 = i4 ^ ((i4 << 12) & 0x0000ffff);
      1. 0xfc1 << 12 = 0xfc1000
      1. 0xfc1000 & 0x0000ffff = 0x1000
      1. 0xfc1 ^ 0x1000 = 0x1fc1
   1. i = i5 ^ (((i5 & 0xff) << 5) & 0x0000ffff);
      1. 0x1fc1 & 0xff = 0xc1
      1. 0xc1 << 5 = 0x1820
      1. 0x1820 & 0x0000ffff = 0x1820
      1. 0x1fc1 ^ 0x1820 = 0x7e1

## Monitoring Data

### Request Packet

```text
  00   01   02   03   04   05
+----+----+----+----+----+----+
| 0d | 05 | 75 | 0f | da | 25 |
+----+----+----+----+----+----+

00= Request magic byte
01= Request packet size
02= Monitoring data type (T0=0x70, T1=0x75, T2=0x75)
03= 0x0f (??)
04= Checksum byte
05= Checksum byte
```

### Response Packet

```text
  00   01   02   03   04   05   06   07   08   09   10   11   12   13   14   15   16   17   18
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+
| d0 | 12 | 75 | 0f | 01 | 01 | 00 | 08 | 00 | 00 | 02 | 00 | 00 | 00 | 00 | 00 | 00 | 7d | 05 |
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+

00= Response magic byte
01= Response packet size
02= Operation ID
03= ??
04= Accessory Present (00: no accessory, 01: water spout, 02: milk spout, 03: chocolate spout, 04: milk clean dial)
05= ??
06= ??
07= ??
08= ??
09= ??
10= Machine Model Id
11= Dispensing Percentage
12= ??
13= ??
14= ??
15= Main Board Software Release
16= ??
17= Checksum byte
18= Checksum byte
```

### Rinsing process

```
Status:
  Decoder       - InStandBy             = false
  Decoder       - TurningOn             = false
  Decoder       - ReadyToWork           = false
  Decoder       - ShuttingDown          = false
  Decoder       - ShutDown              = false
  Decoder       -------------------------
  Decoder       - AccessoryPresent      = 1
  Decoder       - ActiveAlarms          = 2,11
  Decoder       - BeverageType          = undefined
  Decoder       - CoffeeInfuserPos      = 0
  Decoder       - CoffeePowderQty       = -1
  Decoder       - CoffeeWasteCounter    = undefined
  Decoder       - DispensingPercentage  = 0
  Decoder       - FunctionOngoing       = 4
  Decoder       - HeaterTemp            = undefined
  Decoder       - MachineModelId        = -1
  Decoder       - MainBoardSwRelease    = 0
  Decoder       - OnLoads               =
  Decoder       - OnSwitches            = 0,2
  Decoder       - OnSwitchesToShowUser  =
  Decoder       - PressedKeys           =
  Decoder       - RequestedWaterQty     = -1
  Decoder       - SteamerTemp           = undefined
  Decoder       - Timestamp             = undefined
  Decoder       - Type                  = 2
  Decoder       - Value                 = [d0 12 75 0f 01 05 00 04 08 04 09 00 00 00 00 00 00 fa 12] (19)
  Decoder       - WaterFlowQty          = 4608
```