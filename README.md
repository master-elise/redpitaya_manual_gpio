## Manual connecting the GPIO IP to the AXI bus of the Red Pitaya Zynq

Based on the information found in <a href="https://github.com/oscimp/oscimpDigital/tree/master/doc/tutorials/redpitaya/2-PL">2-PL</a> and <a href="https://github.com/oscimp/oscimpDigital/tree/master/doc/tutorials/redpitaya/3-PLPS">3-PLPS</a> of the OscimpDigital project

Launch Vivado, create a RTL project, select the target as
```
Zynq7010 -> clg400 -> speed grade: -1
```

Once the project is open, 
```
Create Block Design -> "+" -> ZynqPS
```

The Zynq7010 PS must be configured: add the <a href="https://github.com/oscimp/fpga_ip/blob/3e6c20893d4c5f10eecc241fec171aaf9c92fb06/preset/redpitaya.tcl">following</a> preset configuration file. Whenever Vivado offers to automate connection steps, acknowledge and let Vivado complete its job.

Once the PS has been configured, add (ctrl-i or "+") ``AXI GPIO``. Rename the
output pins as ``led_o`` instead of the default name ``gpio_rtl_0``. Resize the GPIO
to 8-bit width.

In Block Design -> Sources -> Constraints: add the pin assignment of HDL
signals to hardware pins found in the <a href="https://github.com/master-elise/redpitaya_axi_gpio/blob/main/ports.xdc">constraint</a> file. Notice in this
file the use, at the end, of ``led_o`` as the output LED pins, hence the output
signal name change above.

Right click: Design Souces -> Add HDL Wrapper

Launch the synthesis, and if all goes well a ``.bit`` file is generated in ``project.runs/impl_1/system_wrapper.bit``

Go in the directory holding the ``.bit`` file and create (text editor) a file named ``something.bif``
including the command ``all: {system_wrapper.bit}`` and execute the command 
```
bootgen -image something.bif -arch zynq -process_bitstream bin
```
to generate ``system_wrapper.bit.bin``

Copy this ``system_wrapper.bit.bin`` file on the Red Pitaya, in the ``/lib/firmware`` subdirectory.

Login the Red Pitaya and load the gateware by executing ``echo system_wrapper.bit.bin > /sys/class/fpga_manager/fpga0/firmware``. The blue LED of the Red Pitaya should light, indicating the gateware has 
configured the PL.

Based on the address found in the Address Editor of Vivado, use ``devmem`` to write to the
GPIO direction register (based address +4) 0s to set the pins as outputs, and GPIO value at
base address to switch on or off the yellow LEDs connected to the EMIO of the PL.

These steps are automated as one single TCL script found at https://github.com/master-elise/redpitaya_axi_gpio
