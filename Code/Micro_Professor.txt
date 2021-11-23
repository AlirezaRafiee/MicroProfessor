;; here we have 10-17h and 25h


jmp Start
org 03h
	clr EA
	clr 14h
	reti





Start:
	 setb EX0
	 setb IT0
	mov SP,#30h
	MOV DPTR,#0E003H ;PROGRAMMING CWR [CONTROL WORD REGISTER]. PORT_C[4..7]: OUTPUT, PORT_C[0..3]: INPUT. PORT_A: OUTPUT, PORT_B: OUTPUT.
	MOV A,#10000001B
	MOVX @DPTR,A
	MOV DPTR,#0C003H ;PROGRAMMING CWR [OF THE UPPER PPI]. PORT_A: OUTPUT, PORT_B: OUTPUT, PORT_C: OUTPUT
	MOV A,#10000000B
	MOVX @DPTR,A
	acall getByte
	mov 23h,A
	acall getByte
	mov 24h,A
	acall getByte
	mov 10h,A			;END_H
	acall getByte
	mov 11h,A			;END_L
	mov 12h,23h
	mov 13h,24h
Loop:
	mov A,12h
	cjne A,10h,ST_1
	mov A,13h
	cjne A,11h,ST_1
	jmp ST_2
ST_1:
	acall getByte
	mov DPH,12h
	mov DPL,13h
	movx @DPTR,A
	mov A,13h
	add A,#1
	mov 13h,A
	mov A,12h
	addc A,#0
	mov 12h,A
	jmp Loop

ST_2:
	startEx:
	;;mov 23h,#1
	;;mov 24h,#0
	;setting the PC
	Mov 1Ah,23h
	Mov 1Bh,24h
	;Mov 1Ch,#01h
	;Mov 1Dh,#00h
	mov 1Dh,24h
	mov A,#2
	add A,24h
	mov 1Ch,A

Main:
	clr EA
	mov A,10h
	cjne A,1Ah,MyC
	mov A,11h
	cjne A,1Bh,MyC
	jmp EE
MyC:
	
	jnb 11h,Main_C1
	jmp isINT
Main_C1:
	Mov DPH,1Ah
	Mov DPL,1Bh
	Movx A,@DPTR
	;;;mov A,#1   this was for debugging
	clr c
	rlc A
	mov 18h,A
	mov DPTR,#TABLE
	Movc A,@A+DPTR
	mov B,A
	mov A,18h
	inc A
	movc A,@A+DPTR
	mov DPL,A
	mov DPH,B
	mov A,#0
	jmp @A+DPTR
	
	
	
	
	
	
	
	
	
CNT:
	jnb 13h,CNT_C3
	jmp Main
	
CNT_C3:	
	 mov 26h,#0
	 mov 27h,#0
	 mov 28h,1ah
	 mov 29h,1bh
	 acall show
	 acall LDe
	 acall rSHOW
	acall getFour
	jb ACC.3,CNT_C1
	jb ACC.2,CNT_C2
	jmp Main
CNT_C2:
	setb 13h
	jmp Main
CNT_C1:
	jb ACC.2,CNT_C4
	mov 26h,#0Ah
	 mov 27h,#0Ah
	 mov 28h,#0Ah
	 mov 29h,#0Ah
	 acall SHOW
	 acall LDe
	 acall rSHOW
	acall getByte
	mov 26h,#0
	mov 27h,#0
	mov 28h,#0
	setb RS1
	setb RS0
	mov R0,A
	mov A,@R0
	mov 29h,A
	acall SHOW
	acall LDe
	acall rSHOW
	jmp CNT
CNT_C4:

	 mov 26h,#0Bh
	 mov 27h,#0Bh
	 mov 28h,#0Bh
	 mov 29h,#0bh
	 acall SHOW
	 acall LDe
	 acall rSHOW

	acall getByte
	setb RS1
	setb RS0
	mov R0,A
	
	mov 26h,#0Ch
	 mov 27h,#0Ch
	 mov 28h,#0Ch
	 mov 29h,#0Ch
	 acall SHOW
	 acall LDe
	 acall rSHOW
	
	
	acall getByte
	mov R1,A
	mov DPH,R0
	mov DPL,R1
	movx A,@DPTR
	mov 26h,#0
	mov 27h,#0
	mov 28h,#0
	mov 29h,A
	acall SHOW
	acall LDe
	acall rSHOW
	jmp CNT


	jmp Main
	
	
	
	
	;The system has 16 registers which will be the first two banks plus addressable bits 00h to 0Fh
	; The memory is yet to be desided
	;In this system A= our 00h
	;00h -> carry_bit
	
isAND:			;This is going to be a two byte code(The second byte is betwenn 0-15)
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	ANL A,@R0
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT
	
	
isANDI:			;This is going to be a two byte code(The second byte is betwenn 0-15)
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	ANL A,R0
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT
	
isOR:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	ORL A,@R0
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT
	
isORI:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	ORL A,R0
	mov 00h,A
	acall addPC
	acall addPC	
	jmp CNT

	
isNOT:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,@R0
	CPL A
	mov @R0,A
	acall addPC
	acall addPC
	jmp CNT

isRL:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,@R0
	RL A
	mov @R0,A
	acall addPC
	acall addPC
	jmp CNT


isRLC:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,@R0
	mov c,00h
	RLC A
	mov 00h,c
	mov @R0,A
	acall addPC
	acall addPC
	jmp CNT
	
isRR:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,@R0
	RR A
	mov @R0,A
	acall addPC
	acall addPC
	jmp CNT


isRRC:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,@R0
	mov c,00h
	RRC A
	mov 00h,c
	mov @R0,A
	acall addPC
	acall addPC
	jmp CNT
isADD:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	clr c
	ADD A,@R0
	mov 00h,c
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT
	
isADDI:	
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	clr c
	ADD A,R0
	mov 00h,c
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT

isADDC:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	mov c,00h
	ADDC A,@R0
	mov 00h,c
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT


isADDCI:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	mov c,00h
	ADDC A,R0
	mov 00h,c
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT



isSUBB:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	mov c,00h
	subb A,@R0
	mov 00h,c
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT
	
	
isSUBBI:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx A, @DPTR
	setb RS1
	setb RS0
	mov R0,A
	mov A,00h
	mov c,00h
	subb A,R0
	mov 00h,c
	mov 00h,A
	acall addPC
	acall addPC
	jmp CNT


isJZ:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A, 00h
	jz isJZ_C1
	acall addPC
	acall addPC
	acall addPC
	jmp CNT
isJZ_C1:
	mov A,R0
	add A,23h
	mov 1Ah,A
	mov A,R1
	addc A,24h
	Mov 1Bh,A
	jmp CNT
	
	
isJC:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov c,00h
	jc isJC_C1
	acall addPC
	acall addPC
	acall addPC
	jmp CNT
isJC_C1:
	mov A,R0
	add A,23h
	mov 1Ah,A
	mov A,R1
	addc A,24h
	Mov 1Bh,A
	jmp CNT



	
isLJUMP:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A,R0
	add A,23h
	mov 1Ah,A
	mov A,R1
	addc A,24h
	Mov 1Bh,A
	
	jmp CNT



isLCALL: ;;; 3 byte
	acall SavePC
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A,R0
	add A,23h
	mov 1Ah,A
	mov A,R1
	addc A,24h
	Mov 1Bh,A
	jmp CNT
	

isINT:
	acall SavePC
	setb 10h
	clr 11h
	
	mov R0,#18H
	mov R1,#03H
	mov A,R0
	add A,23h
	mov 1Ah,A
	mov A,R1
	addc A,24h
	Mov 1Bh,A
	jmp CNT
	
isRET:
	acall loadPC
	jmp CNT
	
isRETI:
	acall loadPC
	clr 10h
	jnb 12H,isRETI_C1
	setb EX1
	jmp CNT
isRETI_C1:
	clr EX1
	jmp CNT
	
isMASKINT:
	clr 12h
	jmp CNT
isUNMASKINT:
	setb 12h
	jmp CNT
isMOVRM: ;;; 4 byte
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	inc DPTR
	movx  A,@DPTR
	mov R6,A
	mov DPH,R1
	mov DPL,R6
	movx A,@DPTR
	mov @R0,A
	acall addPC
	acall addPC
	acall addPC
	acall addPC
	jmp CNT



isMOVMR:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R1, A
	inc DPTR
	movx  A,@DPTR
	mov R6,A
	inc DPTR
	movx  A,@DPTR
	mov R0,A
	mov DPH,R1
	mov DPL,R6
	mov A,@R0
	movx @DPTR,A
	acall addPC
	acall addPC
	acall addPC
	acall addPC
	jmp CNT






isINPUT:
	mov 26h,#0Ah
	mov 27h,#0Bh
	mov 28h,#0Ch
	mov 29h,#0Dh
	call SHOW
	acall getByte
	mov 00h,A
	acall addPC
	jmp CNT
isOUTPUT:
	 mov 26h,12
	 mov 27h,13
	 mov 28h,14
	 mov 29h,15
	acall SHOW
	acall LDe
	acall addPC
	jmp CNT
	;jmp EE
isMOVRR:		;; This is going to be a three byte memory
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A,@R1
	mov @R0,A
	acall addPC
	acall addPC
	acall addPC
	jmp CNT

isNOP:
	acall addPC
	jmp CNT

isMovI:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A,R1
	mov @R0,A
	acall addPC
	acall addPC
	acall addPC
	jmp CNT
isSBIT:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A,20h
	orl A,R0
	mov 20h,A
	mov A,21h
	orl A,R1
	mov 21h,A
	acall addPC
	acall addPC
	acall addPC
	jmp CNT
	
isCLR:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A,20h
	anl A,R0
	mov 20h,A
	mov A,21h
	anl A,R1
	mov 21h,A
	acall addPC
	acall addPC
	acall addPC
	jmp CNT
	
	
isCPR:
	Mov DPH,1Ah
	Mov DPL,1Bh
	inc DPTR
	movx  A,@DPTR
	setb RS1
	setb RS0
	mov R0, A
	inc DPTR
	movx  A,@DPTR
	mov R1,A
	mov A,20h
	xrl A,R0
	mov 20h,A
	mov A,21h
	xrl A,R1
	mov 21h,A
	acall addPC
	acall addPC
	acall addPC
	jmp CNT
	






savePC:
	mov A,1Ah
	mov DPH,1Ch
	mov DPL,1Dh
	movx @DPTR,A
	inc DPTR
	mov A,1Bh
	movx @DPTR,A
	mov A,1Dh
	add A,#2
	mov 1Dh,A
	mov A,1Ch
	addc A,#0
	mov 1Ch,A
	RET
	
	
	
loadPC:
	clr c
	mov A,1Dh
	subb A,#2
	mov 1Dh,A
	mov A,1Ch
	subb A,#0
	mov 1Ch,A
	mov DPH,1Ch
	mov DPL,1Dh
	movx A,@DPTR
	mov 1Ah,A
	inc DPTR
	movx A,@DPTR
	mov 1Bh,A
	ret
	
	
addPC:
	mov A,1Bh
	add A,#1
	mov 1Bh,A
	mov A,1Ah
	addc A,#0
	mov 1Ah,A
	ret 
	
	
getByte:
	 NOP
again:
	acall getFour
	mov 14h,A
	acall getFour
	mov B,A
	mov A,14h
	clr c
	rlc A
	clr c
	rlc A
	clr c
	rlc A
	clr c
	rlc A
	add A,B
	mov 14h,A
	mov 26h,#0FFh
	mov 27h,#0FFh
	mov 28h,#0FFh
	mov 29h,14h
	acall SHOW
	acall LDe
	acall rSHOW
	acall getFour
	cjne A,#0Fh,again
	mov A,14h
	ret




getFour:
	MOV DPTR,#0E003H ;PROGRAMMING CWR [CONTROL WORD REGISTER]. PORT_C[4..7]: OUTPUT, PORT_C[0..3]: INPUT. PORT_A: OUTPUT, PORT_B: OUTPUT.
	MOV A,#10000001B
	MOVX @DPTR,A
	MOV DPTR,#0E002H
	MOV A,#0fh
	Movx @DPTR,A

	setb 14h
	setb EA
	jb 14h,$
	clr EA
	

	acall SmDe
	movx A,@DPTR			;4lower are good  row
	CPL A
	mov B,A
	MOV DPTR,#0E003H ;PROGRAMMING CWR [CONTROL WORD REGISTER]. PORT_C[4..7]: OUTPUT, PORT_C[0..3]: INPUT. PORT_A: OUTPUT, PORT_B: OUTPUT.
	MOV A,#10001000B
	MOVX @DPTR,A
	MOV DPTR,#0E002H
	MOV A,#0
	Movx @DPTR,A			;4upper bits are good  colomn
	acall SmDe
	Movx A,@DPTR
	CPL A
	mov 15h,A
	mov A,B
	cjne A,#0F1h,CC1
	mov B,#0
	jmp K1
CC1:
	cjne A,#0F2h,CC2
	mov B,#1
	jmp K1
CC2:
	cjne A,#0F4h,CC3
	mov B,#2
	jmp K1
CC3:
	cjne A,#0F8h,K1
	mov B,#3
	jmp K1
K1:

	mov A,15h
	cjne A,#1Fh,CD1
	mov A,#0
	jmp K2
CD1:
	cjne A,#2Fh,CD2
	mov A,#1
	jmp K2
CD2:
	cjne A,#4Fh,CD3
	mov A,#2
	jmp K2
CD3:
	cjne A,#8Fh,K2
	mov A,#3
	jmp K2

K2:
	mov 15h,A
	mov A,B
	clr c
	rlc A
	clr c
	rlc A
	add A,15h
	mov 15h,A
	;acall LDe
	acall SmDe
	mov A,15h
	ret
	
	
	
SmDe:
   mov A,#0FFh
Sm_1:
   NOP
   NOP
   NOP
   NOP
   NOP
    dec A
    jnz Sm_1
    ret
	
	
	
LDe:
   mov B,#0Bfh
 LD_1:
   acall SmDe
   acall SmDe
   acall SmDe
   dec B
   mov A,B
   jnz LD_1
   ret
	
SHOW:
	MOV DPTR,#0C000H ;PORT A IS ACCESSED. OF THE UPPER PPI.
	MOV A,26H ;
	MOVX @DPTR,A ;
	MOV DPTR,#0E000H ;PORT A IS ACCESSED. OF THE PPI WHICH IS CONNECTED TO KEYPAD.
	MOV A,29H
	MOVX @DPTR,A ;
	MOV DPTR,#0C002H ;PORT C IS ACCESSED. OF THE UPPER PPI.
	MOV A,28H
	MOVX @DPTR,A ;
	MOV DPTR,#0C001H ;PORT B IS ACCESSED. OF THE UPPER PPI.
	MOV A,27H
	MOVX @DPTR,A ;
	RET
       
       
 rSHow:
   mov 26h,#0
   mov 27h,#0
   mov 28h,#0
   mov 29h,#0
    acall SHOW
    ret
	
TABLE:      
DW	isAND
DW	isANDI
DW	isOR
DW	isORI
DW	isNOT
DW	isRL
DW	isRLC
DW	isRR
DW	isRRC
DW	isADD
DW	isADDI	
DW	isADDC
DW	isADDCI
DW	isSUBB
DW	isSUBBI
DW	isJZ
DW	isJC
DW	isLJUMP
DW	isLCALL
DW	isINT
DW	isRET
DW	isRETI
DW	isMASKINT
DW	isUNMASKINT
DW	isMOVRM
DW	isMOVMR
DW	isINPUT
DW	isOUTPUT
DW	isMOVRR
DW	isNOP
DW	isMovI
DW	isSBIT
DW	isCLR
DW	isCPR



EE:
	jmp $



   
END