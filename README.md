# Nyuzi Processor

<img src="https://github.com/jbush001/NyuziProcessor/wiki/teapot-icon.png">

Nyuzi is a GPGPU processor core implemented in SystemVerilog. This project 
also includes a C++ toolchain based on LLVM, an instruction set simulator, 
software libraries, and verification tests. It is useful as a platform for 
microarchitecture experimentation, performance modeling, and parallel 
software development.   

License: GPLv2/LGPLv2.  
Documentation: https://github.com/jbush001/NyuziProcessor/wiki  
Mailing list: https://groups.google.com/forum/#!forum/nyuzi-processor-dev  

# Running in Verilog simulation

This environment allows cycle-accurate simulation of the hardware without the
need for a FPGA. 

## Prerequisites

The following software packages are required. 

1. GCC 4.8+ or Apple Clang 4.2+
2. Python 2.7
3. [Verilator 3.864+](http://www.veripool.org/projects/verilator/wiki/Installing).  
4. C/C++ cross compiler toolchain targeting this architecture. Download and 
   build from https://github.com/jbush001/NyuziToolchain using instructions  
   in the README file in that repository.
5. Optional: Emacs v23.2+, for 
   [AUTOWIRE/AUTOINST](http://www.veripool.org/projects/verilog-mode/wiki/Verilog-mode_veritedium). (This can be used in batch mode by typing 'make autos' in the rtl/ directory). 
6. Optional: Java (J2SE 6+) for visualizer app 
7. Optional: [GTKWave](http://gtkwave.sourceforge.net/) for analyzing waveform files 

On Linux, these can be installed using the built-in package manager (apt-get, yum, etc). 
Here is the command line for Ubuntu:

    sudo apt-get install gcc g++ python emacs openjdk-7-jdk gtkwave

Some package managers do have verilator, but the version is pretty old. Bug 
fixes in the most recent version are necessary for this to run correctly, so 
you will probably need to build it manually. 

MacOS should have python by default. You will need to install XCode from the App Store 
to get the host compiler.

I have not tested this under Windows.

## Building and running

1. Build verilog models, libraries, and tools. From the top directory of this 
project, type:

        make

2. To run verification tests (in Verilog simulation). From the top directory: 

        make test

3. To run 3D Engine (output image stored in fb.bmp)

        cd software/render-object
        make verirun

# Running on FPGA

This currently only works under Linux.  It uses Terasic's DE2-115 evaluation 
board http://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&No=502

## Prerequisites
The following packages must be installed:

1. libusb-1.0
2. [USB Blaster JTAG tools](https://github.com/swetland/jtag)
3. [Quartus II FPGA design software] 
   (http://www.altera.com/products/software/quartus-ii/web-edition/qts-we-index.html)
4. C/C++ cross compiler toolchain described above https://github.com/jbush001/NyuziToolchain.

## Building and running
1. Build USB blaster command line tools
 * Update your PATH environment variable to point the directory where you 
   built the tools.
 * Create a file /etc/udev/rules.d/99-custom.rules and add the line (this 
   allows using USB blaster tools without having to be root)

            ATTRS{idVendor}=="09fb" , MODE="0660" , GROUP="plugdev" 

2. Build the bitstream (ensure quartus binary directory is in your PATH, by
   default installed in ~/altera/[version]/quartus/bin/)

        cd rtl/fpga/de2-115
        make

3. Make sure the FPGA board is in JTAG mode by setting SW19 to 'RUN'
4. Load the bitstream onto the FPGA (note that this will be lost if the FPGA 
   is powered off).

        make program 

5.  Load program into memory and execute it using the runit script as below.
    The script assembles the source and uses the jload command to transfer
    the program over the USB blaster cable that was used to load the bitstream.
    jload will automatically reset the processor as a side effect, so the
    bitstream does not need to be reloaded each time. This test will blink the
    red LEDs on the dev board in sequence.

        cd ../../../tests/fpga/blinky
        ./runit.sh

