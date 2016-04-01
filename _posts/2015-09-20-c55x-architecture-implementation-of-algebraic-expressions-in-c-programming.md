---
author: clementcole
layout: post
title: "C55x Architecture Implementation Of Algebraic Expressions in C programming"
date: 2015-09-20 02:00
comments: true
category: computer architecture design embedded programming
tags:
- C55x
- Algebraic expressions
- Program counters
- code address
- data address
- accumulators
---


## C55x Architecture:
Features:
  1. Its byte/word addressable.
  2. Codes are stored as bytes ==> 8-bit
  3. Data is stored in words ==> 16bits.
  4. Has Parallel instruction rules
  5. Variable-length instructions in memory
  6. Has several memory modes.
  7. MMR addresses.
..* NB: Offsets of fields defined in .struct or .union constructs are always counted in words.

## Definition of Code Sections:
The assembler identifies a code section if the section starts with a (.text directive) or (has any C55x assembly syntax) example:

```
.text ; PC is counted in bytes
MOV AR1, AR0
ADD #1, AC0
               
.data  ;Pc is counted in words
.word  4, 5, 6, 7
foo    .word 1
```

+       In the example above, the machine code will look something of this sort:

```
000000         ;----> .text  PC is counted in bytes
000000 2298    ;----> MOV AR1, AR0
000002 4010    ;----> ADD #1, AC0
            
000000         ;.data =  Pc is counted in words = 0
000000 0004    ;.word[0] = 4
000001 0005    ;.word[1] = 5,
000002 0006    ;.word[2] = 6,
000003 0007    ;.word[3] = 7
000004 0001    ; load .word[0] to the stack foo()
```
     So from 1 - 3, the PC counts in bytes AR1 and AR0 are addresses to accumulator registers. AR1 and AR0 are accumulator registers that hold the address of auxiliary devices such as buses
      or links to other peripherals.
     For the C code such as " x = (a*b) + (a - b)"
      - Lets assume that the memory address for a is 0x100 and that of b is 0x101 and that of x is 0x102
      - a. First we need to move a to an auxiliary accumulator register
      - b. Second we have to move b to another accumulator register
      - c. Then we perform the necessary operations from the c code
      - So expressing the steps a, b and c in C55x assembly syntax we have:

```
MOV 0x100, AR0  ; Move address of a in AR0
MOV 0x101, AR1  ; Move address of b in AR1
MOV 0x102, AR2  ; Move address of x in AR2
```
+         Then the next step is to move the actual contents in a and b in order to perform the operation.

```
MOV *ARO, AC0  ; Move contents of 0x100 to accumulator AC0
MOV *AR1, AC1  ; Move contents of 0x101 to accumulator AC1
SUB AC1, AC0  ; Subtract AC1 from AC0 => AC0 = a - b
MOV *AR0, HI(AC1) ; Move AR0 to (31 - 16) bits of AC1
MOV *AR1, HI(AC2) ; Move AR1 to (31 - 16) bits of AC2
MPY AC2, AC1      ; Multiply AC1 and AC2 ==> AC1 = a*b
ADD AC1, AC0      ; Add AC0 and AC1 ==> AC0 = 
MOV AC0, *AR2     ; Store AC0 in 0x102
```
  * Remember the process is simply Assign address to AR0 to AR7 address {auxiliary addresses for input and output storage}
  * Then move the values from the addresses into AC0 - AC7 {accumulators registers for data manupulation}
  * The next step is to perform the reverse of step number 1. by storing the resultant value stored in AC0 back into the designation register assigned in step 1.

### Another example: 
                    <?xml version='1.0'?>
                    <math xmlns='http://www.w3.org/1998/Math/MathML'>
                     <semantics>
                      <apply>
                       <times />
                       <apply>
                        <times />
                        <ci>a</ci>
                        <ci>b</ci>
                       </apply>
                       <apply>
                        <times />
                        <ci>a</ci>
                        <ci>b</ci>
                       </apply>
                      </apply>
                      <annotation-xml encoding='MathML-Presentation'>
                       <mrow>
                        <mrow>
                         <mo>(</mo>
                         <mrow>
                          <mi>a</mi>
                          <mo>&#8290;</mo>
                          <mi>b</mi>
                         </mrow>
                         <mo>)</mo>
                        </mrow>
                        <mo>&#8290;</mo>
                        <mrow>
                         <mo>(</mo>
                         <mrow>
                          <mi>a</mi>
                          <mo>&#8290;</mo>
                          <mi>b</mi>
                         </mrow>
                         <mo>)</mo>
                        </mrow>
                       </mrow>
                      </annotation-xml>
                     </semantics>
                    </math>
  1. **x -> __0x104__**
  2. **a -> __0x100__**
  3. **b -> __0x101__**
  4. **c -> __0x102__**
  5. **d -> __0x103__**


  * Step 1. Assign address locations 0x100, 0x101,  0x102, 0x103 and 0x104  to AR0, AR1, AR2, AR3 and AR4 respectively.

```javascript
MOV 0x100, AR0
MOV 0x101, AR1
MOV 0x102, AR2
MOV 0x103, AR3
MOV 0x104, AR4  
```

  * Step 2. Dereference AR0, AR1, AR2, AR3, AR4 to get the values of a, b, c, d and x respectively.

```javascript
MOV *AR0, AC0  ; Contents of a
MOV *AR1, AC1  ; Contents of b
MOV *AR2, AC2  ; Contents of c
MOV *AR3, AC3  ; Contents of d
MOV *AR4, AC4  ; Contents of x
```
  * Step 3. Now once we have the actual contents of a, b, c, d and x we are ready to actually perform the arithmetic operation assigned by the c statement.

   NB: We can use the MPY operator from the C55x package but that will require us to move the contents in AR0 to the 31st to 16th bits of AC registers.

```
MOV *AR0, HI(AC0)
MOV *AR1, HI(AC1)
MPY AC0, AC1  ; This will enable us to store the value of a*b into AC0's register
              ; Since it is squared, we need to repeat this operation again so we have
MPY AC1, AC1  ; Here we multiply AC1 by itself and then store it back into AC1, the result is AC1 = AC1 * AC1 or AC1 = (*(AR0)) * (*(AR1))
```

  * Step 4. Now we need to take care of the data in c and d which is also stored in AC3 and AC4 using the format ( SUB src, dest  ==> dest = dest - src )

```javascript
SUB AC3, AC2  ; In this particular case we have AC2 = AC2 - AC3, resulting in the implementation of d = c-d.
```
                               

  * Step 5. Now we combine both <?xml version='1.0'?>
<!DOCTYPE math PUBLIC '-//W3C//DTD MathML 2.0//EN' 'http://www.w3.org/TR/MathML2/dtd/mathml2.dtd'>
<math xmlns='http://www.w3.org/1998/Math/MathML'>
 <semantics>
  <apply>
   <times />
   <apply>
    <times />
    <ci>a</ci>
    <ci>b</ci>
   </apply>
   <apply>
    <times />
    <ci>a</ci>
    <ci>b</ci>
   </apply>
  </apply>
  <annotation-xml encoding='MathML-Presentation'>
   <mrow>
    <mrow>
     <mo>(</mo>
     <mrow>
      <mi>a</mi>
      <mo>&#8290;</mo>
      <mi>b</mi>
     </mrow>
     <mo>)</mo>
    </mrow>
    <mo>&#8290;</mo>
    <mrow>
     <mo>(</mo>
     <mrow>
      <mi>a</mi>
      <mo>&#8290;</mo>
      <mi>b</mi>
     </mrow>
     <mo>)</mo>
    </mrow>
   </mrow>
  </annotation-xml>
 </semantics>
</math>
and then store into x.
Remember (a*b)*(a*b) is stored in accumulator register AC1. Also (c - d) is stored in register AC2

```javascr
ADD AC1, AC2  ; This will implement the above algebraic expression the store the value into AC2.
```

  * Step 6. Finally, we store the value of AC2 into our auxiliary register of choice which in this case we have as AR4 to transmit the value of x.

```
MOV AC2, *AR0  ;storing contents of AC2 accumulator into ARO auxiliary register which mapps to x
```

  * Step 7. Putting it all together
  
```javascript
MOV 0x100, AR0  ;
MOV 0x101, AR1  ;
MOV 0x102, AR2  ;
MOV 0x103, AR3  ;
MOV 0x104, AR4  ;
MOV *AR0, HI(AC0) ;
MOV *AR1, HI(AC1) ;
MPY AC0, AC1            
MPY AC1, AC1  ;
SUB AC3, AC2  ;
ADD AC1, AC2  ;
MOV AC2, *AR0 ;
```

