idapro_m6502
=======

Extension of the existing IDA processor module for the m6502 processor family, the aim was to debug NES roms 

Python was chosen because it doesnt need the SDK, compiler, or other overhead and is simpler to work with initially for beginners.
However performance maybe worse than C, some functionality maybe impossible or difficult, and the API varies from that of the standard C api a little and can lead to confusion.

This is a sample plugin for extending IDA Pro so you can connect via gdb,
and also to extend gdb support for step-over for the M6502,
and to enable type information support so you can press "y" on functions and 
have the parameters propagate inside and back out of the funciton.

When you install a hook in the HT_IDP category, it will be called
before the processor module's notify() function, so if you return
non-zero, your results will be used.

Steps used to implement these features are:
1) Add GDB style XML files for the CPU register descriptions to $IDA_HOME/cfg/
    <there is a an already integrated m6502 in the list>
	
2) Add to the section in $IDA_HOME/cfg/dbg_gdb.cfg that integrates that XML for GDB
    CONFIGURATIONS = { "m6502" : ...
    IDA_FEATURES = { "m6502" : ... 
    NOT NEEDED> ARCH_MAP = { "m6502" : ... 
    NOT NEEDED> FEATURE_MAP = ...
3) Add function to handle step-over understanding around branches
    The ev_calc_step_over interprets the current instruction.
    If it is a JSR routine call then it will advance the IP to the next instruction.
    In all other cases it will advance to the next insturction by passing back BADADDR
4) Add function to handle type info (funciton name, and paramater propagation)
    To enable type support you must set PR_TYPEINFO in ph.flag and 
    implement most of the related callbacks (see "START OF TYPEINFO 
    CALLBACKS" in idp.hpp from the IDA SDK).

Many thanks to Ilfak Guilfanov @ Hey-Rays for all his help.

Usage
-------
Copy the various files into the IDA installation.
Create a m6502 based deubg db, and set the gdb remote connection.     

Known Issues
-------

* XML doesnt match the m6502 currently in use in MAME, I requested a PC/SP swap

* MAME currently doesnt support NES gdbstub debugging via the following options:  
     mame64.exe -nes ...    -debug -debugger gdbstub  
  I have extended and the patch has been merged for the n2a03 CPU (m6502 alike) so you may need a rebuild of MAME until its in the public binaries.
  https://github.com/mamedev/mame/pull/7440#event-3969060591
  
* GDB handling in 7.1 had some issues, recommend 7.3 or later
  * crash if GDB port disconnects (fixed) and you step/run
  * thinks its connected to a GDB server if the port is blocked/not open (fixed by error message)
  * keeps jumping back to PC if you make edits elsewhere in database (fixed)
  * mishandling of types in 64bit version resulting in crash (fixed)

* Bugs in IDA 7.5
  * Instruction tracing doesnt work for m6502, reported to Hex Rays
  * Disconnecting / Reconnecting GDB looses the GDB option UNLESS you have a segment from 0x0000 - 0xFFFF

    


        