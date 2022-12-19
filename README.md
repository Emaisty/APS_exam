# APS Exam

## 1. Single-cycle CPU

### Single-cycle CPU, data and control paths.


#### Instructions
|Instruction|Syntax|Operation|Type ( [From Type to Imm](#immidiate-for-inst-types) )|wires|
|---|---|---|---|---|
|<b>add</b>|add rd, rs1, rs2|rd = rs1 + rs2;|R|<img width="600" alt="image" src="Img/CPU/CPU_R.png">|
|<b>sub</b>|sub rd, rs1, rs2|rd = rs1 - rs2;|R|<img width="600" alt="image" src="Img/CPU/CPU_R.png">|
|<b>and</b>|and rd, rs1, rs2|rd = rs1 & rs2;|R|<img width="600" alt="image" src="Img/CPU/CPU_R.png">|
|<b>slt</b>|slt rd, rs1, rs2|if (rs1 < rs2)<br>&nbsp;&nbsp;&nbsp;&nbsp; rd=1; <br>else <br>&nbsp;&nbsp;&nbsp;&nbsp; rd=0;|R|<img width="600" alt="image" src="Img/CPU/CPU_R.png">|
|<b>addi</b>|addi rd, rs1, imm<sub>11:0</sub>|rd = rs1 + imm<sub>11:0</sub>;|I|<img width="600" alt="image" src="Img/CPU/CPU_I.png">|
|<b>lw</b>|lw rd,imm<sub>11:0</sub>(rs1)|rd = Memory[rs1 + imm<sub>11:0</sub>]|I|<img width="600" alt="image" src="Img/CPU/CPU_LW.png">|
|<b>sw</b>|sw rs2,imm<sub>11:0</sub>(rs1)|Memory[rs1 + imm<sub>11:0</sub>] = rs2;|I|<img width="600" alt="image" src="Img/CPU/CPU_SW.png">|
|<b>slli</b>|slli rd, rs1, imm<sub>4:0</sub>| rd = rs1 << imm<sub>4:0</sub>|I|<img width="600" alt="image" src="Img/CPU/CPU_I.png">|
|<b>srai</b> (Arithmetic)|srai rd, rs1, imm<sub>4:0</sub>| rd = (signed) rs1 >> imm<sub>4:0</sub>|I|<img width="600" alt="image" src="Img/CPU/CPU_I.png">|
|<b>srli</b> (Logic)|srli rd, rs1, imm<sub>4:0</sub>| rd = (unsigned) rs1 >> imm<sub>4:0</sub>|I|<img width="600" alt="image" src="Img/CPU/CPU_I.png">|
|<b>bne</b>|bne rs1, rs2, imm<sub>12:1</sub>|if (rs1 != rs2)<br>&nbsp;&nbsp;&nbsp;&nbsp; PC = PC + imm<sub>12:1</sub>; <br>else <br>&nbsp;&nbsp;&nbsp;&nbsp; PC = PC + 4;|B|<img width="600" alt="image" src="Img/CPU/CPU_B.png">|
|<b>beq</b>|bne rs1, rs2, imm<sub>12:1</sub>|if (rs1 == rs2)<br>&nbsp;&nbsp;&nbsp;&nbsp; PC = PC + imm<sub>12:1</sub>; <br>else <br>&nbsp;&nbsp;&nbsp;&nbsp; PC = PC + 4;|B|<img width="600" alt="image" src="Img/CPU/CPU_B.png">|
|<b>jal</b> (Jump and Link)|jal rd, imm<sub>20:1</sub>| rd=PC+4;<br>PC=PC+imm<sub>20:1</sub>;|J|<img width="600" alt="image" src="Img/CPU/CPU_JAL.png">|
|<b>jalr</b> (Jump Register)|jalr rd, rs1, imm<sub>11:0</sub>|rd=PC+4;<br>PC = rs1 + imm<sub>11:0</sub>;|I|<img width="600" alt="image" src="Img/CPU/CPU_JALR.png">|
|<b>lui</b>|lui rd, imm<sub>31:12</sub>|rd = {imm<sub>31:12</sub>,'0000 0000 0000'};|U||


### Memory-mapped I/O.

#### Idea

Implement something, through which we will communicate with I/O peripheral devices (keyboards, monitores, printers). We can use part of main memory, just let's reserve some part of it for this devices. This is called <b>Memory mapped I/O</b>.

<img alt="image" src="Img/CPU/Memory_mapped_IO.png">

We will extend part after Date Memory Bus. Let's add Addtess Decoder. If address coresponds to I/O device -- it will activate I/O device (not main memory) and read from/wrtie to it.
CPU checks, whether device is ready to recieve a data by periodic checking of its flag. This is called <b>polling</b> or <b>busy waiting</b>.

<img alt="image" src="Img/CPU/Address_decoder.png">

There is also another approache, to create CPU I/O communication -- <b>Port-mapped I/O</b>. Create a separated I/O address space, which accessable by special I/O instructions (in and out for x86).

[Example](#example-for-memory-mapped)

### Pipelinging

Let's decompose instruction execution into 5 stages : <b> Instruction Fetch, Instruction Decode, Execute, Memory access and Write Back</b>. The decomposition of an instr cycle into more stages is called <b>instruction pipelining</b> (<b>IP</b>). We will add <b>interstage registers</b> into several stages. <b>Data and control signals</b> of the IP are <b>transferred to the next stage</b> from pr at the rising edge. 

||Picture|
|---|---|
|Stages|<img alt="image" src="Img/CPU/Intr_for_Pipel.png">|
|The Current Microarchitecture.|<img alt="image" src="Img/CPU/Pipelined_CPU.png"><br>[HMU inputs and outputs](#hmu-inputs-and-outputs)|



#### Data and control hazards

Since multiple instructions are processed simultaneously, requirements for <b>shared recources</b> van result in conflicts(<b>hazards</b>).

<b>Example</b>: squential instructions. In first one we store result (`add x1,x2,x3`) and in the next -- work with it(`sub x5,x1,x4`). The result of the first instruction is going to be ready only in the last stage(write back), while second instruction already read wrong(old) data from register x1 on the 2<sup>nd</sup> stage and used it in the exec stage.

<b>Hazard types:</b>
<ol>
<li><b>Data</b> - caused by data dependencies.</li>

|Example|Explanation and Solution|
|---|---|
|`add x1,x2,x3`<br>`add x4,x1,x5`|&nbsp;&nbsp;&nbsp;&nbsp;The result of the 1<sup>st</sup> instr going to be written into x1 register only in the last<br> stage(write back), while the second intruction going to reach 4<sup>th</sup> stage and<br> had read wrong data on the 2<sup>nd</sup> and procced it in the 3<sup>rd</sup> stage.<br>&nbsp;&nbsp;&nbsp;&nbsp;We are going to resolve it by <b>forwarding</b>. The data from 1<sup>st</sup> instruction is already<br> available in the 4<sup>th</sup> stage (MEM) and therefore it can be taken from those data paths<br> (and stored into regWriteM)before writing it into the destination register.|
|`lw x8, 40(x0)`<br>`add x4,x8,x1`|*The problem the same, as higher.*<br>&nbsp;&nbsp;&nbsp;&nbsp; The result of lw is not going to be ready in the 4<sup>th</sup> stage, cause result of reading<br> from memory going to appear only in the last stage. So, simple there is <b>no way</b><br> how to get result for the 2<sup>nd</sup> inst, while it reached 3<sup>rd</sup> stage. We will resolve it<br> by <b>stallying</b>(= frozen) pipeline(activate Stall D). Pr instructions not going forward, until we will<br> finish current instruction.|
<li><b>Control</b> - caused by instructions that change PC.</li>

|Example|Explanation and Solution|
|---|---|
|`beq x1,x2, loop`<br>`and x5,x8,x9`<br>`or x6,x4,x8`<br>`loop:`<br>`add x2,x3,x4`|&nbsp;&nbsp;&nbsp;&nbsp;The result of beq will be known only on the 3<sup>rd</sup> stage. In the mean time,<br> IP <b>has started</b> precessing of 2 futher instructions. If we are not taking branch,<br> everything alright. Otherwise, instructions must be canceled and this translates<br> into insertion of <b>2 bubbles</b>.(Flush the 1<sup>st</sup> and the 2<sup>nd</sup> stages by Flush D and Flush E).<br> With instructions `jal` and `jalr` will be the same. Their result will be computed<br> on the execution stage and only after that forwarded to fetch stage.(Flush D, Flush E<br> and store destitaion PC into BranchOutcomeE).|
<li><b>Structural</b> - the number of simultaneous requirements for a given resource exceeds the number of its instances.</li>
<img alt="image" src="Img/CPU/Struct_Hazard.png">
</ol>

[Full example](#pipeling-example)

### Interrupt subsystem and security levels.

## 2. Cache Memory

## 3. Superpipelinging CPU


<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

# Extra materials

## Immidiate for Inst Types
<img alt="image" src="Img/CPU/Imm_for_instruct.png">

## Example for Memory Mapped

<img alt="image" src="Img/CPU/exml_MM_1.png">
<img alt="image" src="Img/CPU/exml_MM_2.png">
<img alt="image" src="Img/CPU/exml_MM_3.png">

## HMU inputs and outputs

<img alt="image" src="Img/CPU/HMU.png">

## Pipelining example

|Picture|Explanation|
|---|---|
|<img alt="image" src="Img/CPU/Pipeline_example/Example_1.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_2.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_3.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_4.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_5.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_6.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_7.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_8.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_9.png">||
|<img alt="image" src="Img/CPU/Pipeline_example/Example_10.png">||