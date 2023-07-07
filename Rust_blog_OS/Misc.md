- In order to debug using VSCode debugger, make sure to:
1) add the -s S to the end, like 
`qemu-system-x86_64 -drive format=raw,file=target/x86_64-blog_os/debug/bootimage-blog_os.bin -s -S` 

The `-s` and `-S` options are command-line options used when running QEMU. Here's what they mean:

1. `-s`: The `-s` option enables the use of a gdbserver for debugging purposes. It starts QEMU with a gdbserver listening on port 1234 by default, allowing a debugger (e.g., GDB) to connect to it and debug the running virtual machine. This option is commonly used in combination with remote debugging tools to enable debugging of the virtual machine.
    
2. `-S`: The `-S` option freezes the CPU at startup, allowing you to connect a debugger before execution begins. When QEMU starts with this option, the CPU is paused, and the virtual machine does not start executing the guest code immediately. This gives you time to connect your debugger and set breakpoints or perform other debugging setup before allowing the virtual machine to start running.
    

By using both `-s` and `-S` together, you can establish a connection to the QEMU gdbserver on port 1234 and have the virtual machine halted at startup, providing an opportunity to set up breakpoints and perform debugging actions before the execution begins.

It's important to note that the behavior and availability of these options may depend on the specific version of QEMU you are using. Always consult the QEMU documentation or the output of `qemu-system-x86_64 -h` to confirm the behavior of these options in your particular version of QEMU.

2) Run the launch script