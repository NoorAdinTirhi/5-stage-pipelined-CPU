# 5-stage-pipelined-CPU

the CPU designed in this repositery is 24-bit pipelined MIPS-esque CPU

# DataPath
![circuitimage](https://user-images.githubusercontent.com/115925101/225586547-00724b57-085a-4822-92d4-293adb156dc3.png)

# Fetch Stage
![image](https://user-images.githubusercontent.com/115925101/225586727-f021a646-eae7-4a83-9b78-2c3f839b3593.png)

Program Counter: Register with its clock enable connected to not stall signal (when stall = 0, program counter take input on clock rising edge), PC input is from multiplexer which has the output:
PC + 1 when 	S = 0
PC + immediateJ 	S = 1
Reg [Rs]		S = 2
ImmediateI +PC	S = 3	Where S is the PCsrc, from control unit
NPC Register: used for Branch address calculation
Instruction Memory: gets address from PC and produces output to multiplexer leading into Instruction register.
Multiplexer has the following output:
  Instruction		S = 0
  0			S = 1, Where S is the kill1 bit, from control unit
  
# Decode Stage
![image](https://user-images.githubusercontent.com/115925101/225587010-d2d02ff5-896a-486d-9b04-b86b188ada71.png)

Instruction is separated into its bits, the condition, opcode and SF are sent into the control unit, the 9 bits of the registers are sent into the register file, the last 10 bits are the immediate value in I-type instructions, as for the immediate of J-type instructions, it is taken from the last 17 bits from the instruction.
The destination register is sent into a sequence of register, each in a separate stage, each register holds the WB register of the instruction in that stage.
Rd2 in the EX-stage → Rd3 in the MEM-stage → Rd4 in the WB-stage
The control unit takes the first 8 bits of the instruction, Cond + Opcode + SF, it also takes the BEQ bit from the EX-register, and the zero flag bits, it outputs control signals and send it into a MUX connected to the EX-register, the MUX has the following outputs:
Control Unit Signals	S = 0
0				S = 1, Where S is (kill2 OR stall)
Stall bit is calculated as EX.MemRd AND (ForwardA or ForwardB).
All control signals produced are sent to a sequence of registers that feed into each other, each register will hold the control signals of the instruction in its stage, for example if instA is in the Memory stage then register MEM will hold its control signals.
In this stage also the Branch and Jump addresses are calculated according as specified in the project, after that they are stored in the Jump and BTA registers.
The Immediate register takes either the J immediate or the I immediate, depending on the type of instruction, the DataA and DataB register can take input from register file, or from the Data Bus of later stages, this decided by the forwarding unit.

# Execution Stage
![image](https://user-images.githubusercontent.com/115925101/225587179-23d63901-2b9e-4c9c-a4e7-effbc98713b0.png)

The immediate and DataB registers enter a MUX that produces the ALU second operand, depending on wither the instruction R-type or not, first operand passes from the Data A register.
Rd2 is passed into Rd3, The ALU result is passed into ALU Res register, RB takes the values of DataA.

# Memory Stage
![image](https://user-images.githubusercontent.com/115925101/225587246-cf1a40b7-7eba-401e-b94c-39c8df1fc785.png)

ALU Res Register contents is passed into the memory address pin of memory unit, the RB   value is passed into the data pin of the memory unit.
Memory unit into a MUX, the MUX output is decided by the WB Data bit.

# WriteBack Stage
Returns data of the last register shown above to register file.

# Forwarding Unit
![image](https://user-images.githubusercontent.com/115925101/225587459-2d47ca69-55d2-4345-a9c6-f7c50ffcc990.png)

Takes in Rt and Rs, and the contents of Rd2, Rd3, Rd4 and the RegWr of the EX,Mem and WB stages, it produces the selection bits for the DataA and DataB Registers’ Multiplexers the bits is produced according to these rules:
![image](https://user-images.githubusercontent.com/115925101/225587531-7096a8b8-fa9d-481a-bc64-df43d17453c1.png)

Forwards B has the same rules.

# Register File
![image](https://user-images.githubusercontent.com/115925101/225587631-bfcebda9-3017-4771-991c-edf0d80f9826.png)

Typical design, same as Slides, but with output pins for each register for probing.
Decoder takes in RW 3-bit bus, produces a ‘1’ on only 1 lines and enables it for registers, and RegRW enables write in general.
Multiplexers take in RA and RB Select Busses, and sends to the output the contents of one register.






