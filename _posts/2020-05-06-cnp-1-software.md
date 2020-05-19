---
layout: post
title: CNP-1 - Software & BIOS
tags: [Homebrew, CNP-1]
fullview: true
---

### Resources



### Workflow

General workflow goes like this:

1. Modify some code in the CNP repository rom_image directory
2. Run make to assemble & link a new rom image
3. Test it in the emulator
4. Burn a new image to eprom
5. Test the new image on the real hardware

### Tools

* Make
* Assembler: I chose to use ca65, which is part of [cc65](https://github.com/cc65/cc65)
* EPROM burner: [Minipro](https://gitlab.com/DavidGriffith/minipro/) to work with the tl866
* Emulator: My own forked version of [Symon](https://github.com/twistdroach/symon).  I will probably write an entire entry about this at some point.

### BIOS

In order to prevent tightly coupling the monitor (and higher level software) running on the system to the specific hardware of the CNP systems, I wanted to try to isolate the hardware interface behind a somewhat standardized BIOS.  This should (and did) allow me to do things like swap from serial output to using a video adapter without modifying too much code.

#### Initialization

The BIOS handles system initialization at boot.  The first thing it needs to do is some general 6502 Initialization stuff.  Clearing the zero and stack pages isn't really necessary, but it helps when looking at memory dumps if the SRAM isn't full of random data.

```assembly
main:
				 sei					;disable interrupts
				 cld					;clear decimal mode
				 
				 lda #0					;clear zero and stack pages
				 ldy #0
				 sta 00
				 sta 01
st_zp:									; clear zero page
				 sta (00),y
				 iny
				 bne st_zp
				 
				 tay					; clear stack page
				 inc 01
st_sp:
				 sta (00),y
				 iny
				 bne st_sp				 
				 
				 ldx #$FF				;set stack pointer
				 txs

				 jsr initialize
```

Now that the processor & ZP/Stack is initialized, we can start bringing up the rest of the hardware.  First thing is to get some ouptut to the user so we can see what is going on.  ```USE_I2C_INPUT``` and ```USE_DISPLAY``` are assembler macros that define what hardware is available on the system.

```assembly
				;;;; ACIA
				lda #%00001011				;No parity, no echo, no interrupt
				sta ACIA_COMMAND
				lda #%00011111				;1 stop bit, 8 data bits, 19200 baud
				sta ACIA_CONTROL

				;;;; I2C
				.ifdef USE_I2C_INPUT
				jsr INIT_I2C
				
				;;;; I2C Keyboard Interface
				.endif

				.ifdef USE_DISPLAY
				;;;; Display Init
				jsr display_initialize
				.endif
```

We then print a message to tell the user a little about the system.  The build timestamp is generated when the code is assembled.  Nothing is worse than banging your head against a wall testing a fix that actually isn't in the ROM you are running.

```
CNP-1 (build 1588964055)
16K RAM $0000 to $3FFF
32k ROM $8000 to $FFFF
6551 ACIA at $4400
6522 VIA at $6000
```

Next we run a quick RAM test, running the following procedure for each page of available RAM.  This is a pretty basic RAM test where we write a pattern to a full page, then read it back and verify it.  If something fails, then we just sit & loop.

```assembly
.proc test_page					;test a page of memory
	sta POSTADR+1				;A should hold page to test
	ldy #$00
	sty POSTADR

	lda #$FF					;A holds test pattern
	jsr test_page_w_pattern     ;test page
	lda #$AA
	jsr test_page_w_pattern
	lda #$55
	jsr test_page_w_pattern
	lda #%00010001
	jsr test_page_w_pattern
	lda #%00100010
	jsr test_page_w_pattern
	lda #%01000100
	jsr test_page_w_pattern
	lda #%10001000
	jsr test_page_w_pattern
	lda #%11101110
	jsr test_page_w_pattern
	lda #%11011101
	jsr test_page_w_pattern
	lda #%10111011
	jsr test_page_w_pattern
	lda #%01110111
	jsr test_page_w_pattern
	lda #$00
	jsr test_page_w_pattern
	rts
				
				
test_page_w_pattern:	ldy #$FF			; Start at top of page
fill_loop:	        sta (POSTADR),y		        ; fill page with pattern
			dey
			cpy #$FF
			bne fill_loop
			ldy #$FF
compare_loop:		cmp (POSTADR),y		        ; compare page with pattern
			bne error
			dey
			cpy #$FF
			beq success
			jmp compare_loop
						
success:		;puts "Ok"
			rts

error:			putsln ""
			putsln "Error occurred"
error_hang:		jmp error_hang
			rts

.endproc
```

Now we can jump into the monitor...but I think I will document that in another post.

#### BIOS Calls

BIOS calls are indexed into a "jump table" allowing higher level code to call them without knowing the underlying addresses.  This allows the ROM routine implementations to move around without needing to re-assemble/compile code that accesses them.  These are only used for code that is assembled outside of the rom_image folder.  This is very much in flux, as I haven't used it for anything but some testing yet.

```assembly
BIOSJMPTABLE=$FF00
 write_bufferln=$00
 write_buffer=$02
 write_char=$04
 read_char=$06
 get_line=$08
 char_avail=$0A
 read_char_nonblocking=$0C
```

#### Interrupt Vectors

* Reset: Points to the BIOS initialization routine
* IRQ: Right now is a dummy routine
* NMI: I use the NMI on these systems as a last ditch effort troubleshooting measure.  Triggering an NMI will cause the system to dump the current processor registers and return from the interrupt.  This is remniscent of the message that is printed when an Apple IIe "crashes" to its monitor.  Disclaimer: This wasn't my idea, it likely came from 6502.org or Garth Wilson's page.

```assembly
.export nmi_handler
.proc nmi_handler
				cld						;
				
				pha						; Preserve registers
				phx
				phy
				tsx						;    Preserve stack pointer

				putsln ""
				putsln "NMI received"
								
				puts "SP:"
				txa						; Print stack pointer
				adc #5
				jsr print_byte_as_hex_chars
				
				puts " Y:"
				inx
				lda $0100,x
				jsr print_byte_as_hex_chars
				
				puts " X:"
				inx
				lda $0100,x
				jsr print_byte_as_hex_chars
				
				puts " A:"
				inx
				lda $0100,x
				jsr print_byte_as_hex_chars
				
				inx
				inx
				puts " PC:"
				inx
				lda $0100,x
				jsr print_byte_as_hex_chars
				dex
				lda $0100,x
				jsr print_byte_as_hex_chars
				
				putsln ""
				
				ply						; Restore registers
				plx
				pla
				
				rti
.endproc
```
