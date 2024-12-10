The MecoTrans ASCII Protocol

The protocol has two forms. Both forms exist side by side and can be used in combination. The Mecotrans protocol is only available on AKS devices with a graphical user interface. It is a pure ASCII protocol, in which characters/chains ("commands") are sent and received in plain text using a terminal program or proprietary controls. Upper and lowercase letters are not relevant. `<CR>` (ASCII character 13dec) is the only non-readable control character used, while everything else is readable.

One form is based on the Parco protocol, while the second form implements plain text commands.

The Parco form is the same on all Mecotec devices with a graphical user interface, while the plain text commands can vary from device to device, which is also useful.

It involves a master-slave communication, where the commanding device represents the master, and the actor is the slave, who only responds when addressed by the master.

Each formally correctly received command generates a response. For read commands, the response consists of the value to be read upon successful execution, or an error message in case of failure. For set commands, the response is "ACK" in the case of successful execution; in case of error, the response is an error message.

A Mecotrans command to the actor begins with `@` as the start character (ASCII character 64dec) and is terminated with `<CR>` (ASCII character 13dec). The actor's response begins with a leading `@`, followed by `<CR>`.

To use the protocols, the MecoTrans ASCII protocol must be activated in the communication settings of the device to be controlled. Please refer to the corresponding user manuals for instructions.

### 1. The Parco Form

There are read and write commands. The individual contents are separated by colons.

#### Reading:

```
@200:R:70:F<CR>
@SlaveAddress:Command:ParameterIndex:Format<CR>
```

#### Slave Address:
The bus address of the device to be addressed. Default addresses are:

- 80 Current/Voltage Module Channel 1  
- 81 Current/Voltage Module Channel 2  
- 82 Current/Voltage Module Channel 3  
- 110 ChargeControl  
- 120...135 I/O Cards (135 ScndSensCtrl IO Board)  
- 150 Barometric Pressure Sensor (APC 5210; APP 3010/3070); Sensor 1 (APL 2100 without Barometric Sensor)  
- 151 Sensor 2 (APL 2100, Software Sensor with APC 4010, 5210 with APX 5160)  
- 152 Sensor 3 (APL 2100, Software Sensor with APC 4010, 5210 with APX 5160)  
- 153 Sensor 4 (APL 2100)  
- 158 Encoder  
- 159 Digital Channel of the DPPC Sensors  
- 160 Temperature Sensor  
- 190 Control Sensor (APR 1300)  
- 200 Control Sensor 1 (APC 4010/5210; APP 3010/3070/3160/3250)  
- 201 Control Sensor 2 (APC 4010/5210 with APX 5160 for models with separate control boards)  
- 202 Control Sensor 3 (APC 4010/5210 with APX 5160 for models with separate control boards)

### Note:

Sensor addresses always start with the largest measurement range.

Sensors with addresses in the 20x range are control sensors. Sensors with addresses in the 15x range are display sensors or software sensors used internally by the device for calculations.

---

### Examples:

#### APP 3070 with control range -1 to 1 bar and 0 to 2 bar absolute:
- **200:** Control sensor for -1 to 1 bar  
- **150:** Barometer  
The absolute pressure control range is formed by combining the relative pressure sensor and the barometer.

---

#### APL 2100 with barometer and 2 additional measurement ranges:
- **150:** Barometer  
- **151:** Measurement range 40 bar  
- **152:** Measurement range 4 bar  

---

#### APC 5210 with APX 5160 extension unit, special design with 3 control boards:
- **200:** Measurement range 20 bar  
- **201:** Measurement range 10 bar  
- **202:** Measurement range 2 bar  

### Command:
There are two commands:  
- **R** = Reading a value from the device  
- **W** = Writing a value to the device  

---

### Parameter Index:
The index of the parameter to be read or written.

---

### Format:
The format of the parameter value:

#### 32-Bit Formats:
- **F** - Floating-point number  
- **L, I, I32** - Positive or negative integer  
- **UL, UI, UI32** - (Unsigned) positive-only integer  

#### 16-Bit Formats:
- **S, I16** - Positive or negative integer  
- **US, UI16** - (Unsigned) positive-only integer  

#### 8-Bit Formats:
- **C, I8** - Positive or negative integer  
- **UC, B, UI8** - (Unsigned) positive-only integer  

### Writing:

```
@200:W:120:F:1234.5<CR>
@SlaveAddress:Command:ParameterIndex:Format:Value<CR>
```

Slave address, command, parameter index, and format -> see **Reading** section.

---

### Value:
The value to be written in the specified format. Pressure values are always in millibar.

---

### Slave Response:
- **ACK<CR>**  
- **Error information<CR>**  
Error information -> see **Reading** section.

### 2. The Plain Text Command Form

Examples:

```
@SetPress:1.2345<CR>
@Stop<CR>
@Command[:Value[:Option[:...]]]<CR>
```

---

### Response from the Pressure Controller:
- **123.456<CR>**  
- **ACK<CR>**  
- **Value (or error information)<CR>Value:**  
  The value is in the specified format.

---

### Error Information:
- **ACK** - Success  
  (Further error information can be found below.)

---

### Command:
Case sensitivity is not relevant. Pressure values are always in the unit configured on the controller unless a unit is optionally specified.

#### The pressure controllers of the APC and APP series support the following commands:

- **`?, Test, check, hello`**  
  Used to test communication. In case of successful communication, the DPC responds with "Hello!"

- **`ReadPress[:{Unit}]`**  
  Returns the current pressure in the unit configured or specified in the device. Supported units depend on `SetUnit`.

- **`SetPress:{Value}[:{Unit}]`**  
  Sets the pressure on the controller in the configured or specified unit. Supported units depend on `SetUnit`.
  ### Commands and Descriptions:

- **`WaitPress:{Value}[:{Unit}]`**  
  The controller adjusts the pressure in the configured or specified unit and only responds when it is within the tolerance band. Supported units depend on `SetUnit`.

- **`WaitPressRaw:{Value}[:{Unit}]`**  
  The controller adjusts the pressure in the configured or specified unit and responds only after the PI control loop is completed. Supported units depend on `SetUnit`.

- **`Stop`**  
  Stops the pressure regulation.

- **`Vent`**  
  Vents the pressure against the environment.

- **`WaitVent:{Value}[:{Unit}]`**  
  The controller releases pressure and responds only when the pressure falls below the given value or after the vent valve's opening time expires (response: "ErrFunction"). Supported units depend on `SetUnit`.

- **`TickPress`**  
  Opens the pressure valve briefly (pulse duration pressure).

- **`TickVac`**  
  Opens the vacuum/vent valve briefly (pulse duration vacuum).

- **`ToggleZero`**  
  Activates zero-offset adjustment or resets it if already active.

- **`ReadZeroOffset[:{Unit}]`**  
  Returns the zero offset of the active range in the configured or specified unit. Supported units depend on `SetUnit`.

- **`ReadStatus[:bin]`**  
  Returns the state of the regulator's state machine. The value (byte) is returned in decimal format.  
  If the `bin` option is set, the value is returned in binary format.  

#### Status Bit Descriptions:
- **Bit0:** Fine regulation - Pressure is within the tolerance band.  
- **Bit1:** PI control completed.  
- **Bit2:** Vent valve is open.  
- **Bit3:** Regulator is in overload state.  
- **Bit4:** Zero-offset compensation is active.  
- **Bit5:** Regulator is in timeout state.

### Commands and Descriptions:

- **`ReadMode`**  
  Returns the current operating mode (Control, Measure, Vent).

- **`SetMode:{Mode}`**  
  Switches the current operating mode to one of the following: Control, Measure, or Vent.

- **`ReadBaro[:{Unit}]`**  
  Returns the pressure value of the barometric sensor in the configured or specified unit. Returns "ErrFunction" if no barometric sensor is available. Supported units depend on `SetUnit`.

- **`ReadUiValue`**  
  Returns the current or voltage measurement value of the current/voltage option in milliamps or volts. Returns "ErrFunction" if the option is not available.

- **`ReadUiPress[:{Unit}]`**  
  Returns the pressure value scaled according to the current or voltage measurement in the configured or specified unit. Returns "ErrFunction" if no current/voltage module is available. Supported units depend on `SetUnit`.

- **`StartProgr`**  
  Starts the program sequence.

- **`StopProgr`**  
  Stops the program sequence.

- **`ContProgr`**  
  Continues the program sequence.

- **`LoadProgr:{Filename}`**  
  Loads a saved program sequence by its filename. The file must be in the "UserFiles" directory, and the filename should be given without the `.prg` extension.

- **`LoadParSet:{ParameterSetNumber}`**  
  Loads a saved set of regulation parameter settings into the controller, based on the specified parameter set number.

- **`ReadUnit`**  
  Returns the current unit. Units are defined in `SetUnit`. Absolute pressure units have an "a" suffix.

- **`ReadUnitFactor`**  
  Returns the unit factor of the current unit in millibar per unit.
  ### Commands and Descriptions:

- **`SetUnit:{Unit}`**  
  Changes the current unit. Some operations or events may override this setting if associated with a unit change. Display/output format remains unchanged. Units must match the range settings.  
  Supported units:  
  `mbar, bar, Pa, hPa, kPa, MPa, torr, atm, psi, lb_ft2, kg_m2, kg_cm2, mmH2O_4C, cmH2O_4C, mmH2O_20C, cmH2O_20C, mH2O_20C, mmHg_0C, cmHg_0C, inHg_0C, inH2O_4C, inH2O_20C, inH2O_60F, ftH2O_4C, ftH2O_20C, ftH2O_60F`

- **`SetRange:{Range-Index}`**  
  Sets the controller to the appropriate control range and enables auto-ranging if available. The range index corresponds to the row number in the dropdown list, starting from 0.

- **`SetAutoRange:{AutoRange-Index}`**  
  Enables the controller's auto-ranging procedure. An index less than 0 disables auto-ranging.

- **`TestLeak:{TestDuration}:{Stabilization}:{TestPressure}[:{Unit}]`**  
  Performs a leak test. Parameters include test pressure in the configured or specified unit, stabilization time, and test duration in seconds. After the test, the pressure drop rate in the configured or specified unit per minute is returned. Supported units depend on `SetUnit`.

- **`SetOutput:{Address}:OnByte:OffByte`**  
  Activates the binary-coded outputs specified by `OnByte` and deactivates those specified by `OffByte` for the board at the given address.

- **`ReadInput:{Address}[:bin]`**  
  Returns the binary-coded state of the digital inputs for the board at the specified address. If the `bin` option is set, the value is returned in binary format.

- **`GetMessage[:{Back}]`**  
  Returns the last controller message in the message box, if any. If `Back` is greater than 0, it returns a previous message. If no message exists, it returns `<none>`.

- **`ResetCtrl[:{Address}]`**  
  Resets the controller state machine, clearing Overload and Timeout errors. If `Address` is specified, it resets only the board at the specified bus address. If 0, all boards are reset and the option exits.
  
  ### Short Form Aliases:
- **`rp`** = ReadPress  
- **`sp`** = SetPress  
- **`s`** = Stop  
- **`v`** = Vent  
- **`z`** = ToggleZero  
- **`tp`** = TickPress  
- **`tv`** = TickVac  
- **`rs`** = ReadStatus  
- **`gm`** = GetMessage  
- **`reset`** = ResetCtrl  

---

### Error Information:
- **`CER`** - Communication error  
- **`PER`** - Parameter error  
- **`VER`** - Value error  
- **`TER`** - Timeout error  
- **`RER`** - Max retry error  
- **`FER`** - Format error  
- **`SER`** - Statement/command error  
- **`LER`** - Input locked error  

Additional errors:
- **`ErrUnkCmd`** - Unknown command  
- **`ErrFunction`** - Functional error  
- **`ErrParameter`** - Parameter error  
- **`ErrInputLocked`** - Input locked error  
