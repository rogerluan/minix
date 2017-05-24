# Minix - An ideal system to learn the basics of OS concepts.

In this exercise, I'll explain very briefly how to add a new system call to your Minix. In the way, you'll learn how to compile the kerner of the minix OS, as well as get more familiarized with unix-like systems.

To get started, we'll start preparing the system to have a new system call. The system call we wanna add here is called MDC, a function that calculates the Greatest Common Divisor, given 2 numbers as parameters.

## Part 1: Creating Your Own System Call
### Step 1: Defining the name of the system call
```nano /usr/src/minix/include/minix/callnr.h```

[comment]:<http://jmp.sh/UHqxBwV>

![callnr.h](https://storage.jumpshare.com/preview/XbJw5tZUCvyxDB48n8-wUd1pXeHtJ2cIsuSKNFsBY4bx1bb4TMuaRXv32TC0Foq6yneWOJhKHz7rc0UyVcLNqd0Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)

### Step 2: Adding your system call to the system table of calls
```nano /usr/src/minix/servers/pm/table.c```

[comment]:<http://jmp.sh/Zr1ujw6>

![table.c](https://storage.jumpshare.com/preview/MzU0bLErkx-QcM6c3Yf_IzxcKTgw9rySIbK6m-gCSLOwv4udeDrkoueX4t54mGEcBhGB81DUpAGtxOlHU6N9z90Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)


### Step 3: Adding the prototype (or signature) of your method to the system prototype file
```nano /usr/src/minix/servers/pm/proto.h```

[comment]:<http://jmp.sh/8mfQVXA>

![proto.h](https://storage.jumpshare.com/preview/A08I3Dj8mFkDsU-IeTFScQWr9GC7c0q7LaMrGTGkguRQpx0sv0I0hSTUbKrMfvbPz92exxFIXTlVE_T6MJUsE90Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85

### Step 4: Implementing the method that will be performed upon the call of your function
```nano /usr/src/minix/servers/pm/mdc.c```

It will look something like this:

[comment]:<http://jmp.sh/hHPKbp3>

![mdc.c](https://storage.jumpshare.com/preview/dtcR0mRH858FcmixKZr1l8WgJ_H0cAv2uNcuyKfZtoBkWZSiakntw-8YOnsrkjuLYdXvQlq_1eCjLI0x5ZN_Y90Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)

But here's the full source code:

```
//
//  Created on May 23rd 2017
//  Source: https://github.com/rogerluan/minix
//

#include <stdio.h>
#include "pm.h"

int calculate_gcd_r(long a, long b);
int calculate_gcd(long a, long b);

int do_mdc() {
    /** m_in is a global variable set to PM's incoming message, 
     *  So m_in.m1_i1 is the integer parameter set in the gcd program.
     */
    int a = m_in.m1_i1;
    int b = m_in.m1_i2;
    int result = 0;
    if (a == 0 || b == 0) {
	printf("One or both of the given numbers are equal to zero. Quitting . . ."); 
        return 0;
    }
    result = calculate_gcd(a, b);
    printf("Greatest Common Divisor is: %ld\n", (long)result);
    return 1; 
}

int calculate_gcd(long a, long b) {
    int temp;
    while (b != 0) {
        temp = a % b;
        a = b;
        b = temp;
    }
    return a;
}

// This is not being used, but it's an alternative way to implement a Greatest Common Divisor recursively.
int calculate_gcd_r(long a, long b) {
    return b == 0 ? a : calculate_gcd_r(b, a % b);
}
```

### Step 5: Adding your method to the compilation process
The system needs to know what to compile. To let it know, we have to edit the following Makefile and add the filename of our recently implemented file.

```nano /usr/src/minix/servers/pm/Makefile```

[comment]:<http://jmp.sh/PqWb0fH>

![Makefile](https://storage.jumpshare.com/preview/rWtA_HsVFBgfQvkrVwUYSJr7RuwudntD1yfMWjz_bZwIxPMn73bc72aIt53GR7SbxK2sAoTuFbhPWxQf3ve8Yt0Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)

### Step 6: Compile

At this point we have almost everything pretty much set up to be able to use this new system call. We need to compile the edited files to apply the recent changes. To do so:

1. ```cd /usr/src/releasetools/```
2. ```make services```
3. ```sync```
4. ```reboot```

### Step 7: Testing
You can now run the following code to test your new system call.

```
cd /home
nano ex1a.c
```

```
#include <stdio.h> // printf()
#include <stdlib.h> // atoi()
#include <lib.h> // _syscall and message

int main(int argc, char **argv) {
    if (argc < 3) {
        printf("This method expects 2 integers as parameters.\nExiting... \n");
        exit(1); // it should expect 2 params
    }
    message m;
    m.m1_i1 = atoi(argv[1]);
    m.m1_i2 = atoi(argv[2]);
    return _syscall(PM_PROC_NR, PM_MDC, &m);
}
```
```
clang ex1a.c -o ex1a
./ex1a 10 16
./ex1a 100 5000
./ex1a 34 85
```

[comment]:<http://jmp.sh/eOPCuZ8>

![ex1a.c](https://storage.jumpshare.com/preview/8pZKmg33dhPxtLvhyKbi7vbYFekudzUrC7ApVzAwAPUPR60ebvyOGr6Pmupfu620DhYD75rca8lggbulcjJP5d0Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)


## Part 2: Creating Your Own Library


### Step 1: Create your library
```nano /usr/src/include/customlib.h```

[comment]:<http://jmp.sh/l287lFf>

![customlib.h](https://storage.jumpshare.com/preview/Bf3SuwgMxdU0TQINwZnzRM0F6e2oLY7Eh2_hykEpAJVtxFE-udcex6y0TVzBQRDNjx5eBy8YCrdFlkIKT5csP90Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)


### Step 2: Add it to the compiling sources

```nano /usr/src/include/Makefile```

[comment]:<http://jmp.sh/FaTXi3T>

![Makefile](https://storage.jumpshare.com/preview/9DxoX6e4Z39_uTy7TQwekgzxfwHslE8Anz-nNp6jqu4rNEbSpqL-HVxcBi1BpgMEz7HtdnTsCDcvvzdZRMUg_t0Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)

### Step 3: Compile
Again, we need to compile the system to apply the changes. To do so, follow the commands:

```
cd /usr/src/
make build > /root/build_dump.txt
reboot
```

The `make build` command will build your whole kernel from ground up, so it may take from 20 to 40 minutes. The `> /root/build_dump.txt` part is optional, but recommended. It will dump all the compiler logs to a text file  located in your `/root` folder, thus it shouldn't display useless comments in your terminal window, only informative warnings and eventual errors. This makes it easier to identify possible errors and how to fix them.

If you're confident that your recent changes to your system can be compiled incrementally, and you have already ran `make build` command before, you can build your system incrementally by calling:

```
make build MKUPDATE=yes > /root/build_dump.txt
```

### Step 4: Testing

As every good programmar, you need to test what you implement. Let's implement the same program in Part 1, but now using our own library and implementing our own system call.

```
cd /home
nano ex1b.c
```
```
#include <stdio.h> // printf()
#include <stdlib.h> // atoi()
#include "customlib.h" //mdc()

int main(int argc, char **argv) {
    if (argc < 3) {
        printf("This method expects 2 integers as parameters.\nExiting... \n");
        exit(1); // it should expect 2 params
    }
    long a = atoi(argv[1]);
    long b = atoi(argv[2]);
    return mdc(a, b);
}
```

```
clang ex1b.c -o ex1b
./ex1b 10 16
./ex1b 100 5000
./ex1b 34 85
```

[comment]:<http://jmp.sh/oAVzEIT>

![ex1b.c](https://storage.jumpshare.com/preview/31m-pr1tZKMeUx7n7YZH9DiQ11qRja8byBcAgba4u0JyFfo77XPWztbIBBkOb8EC8-oDosTn0FmNoTe-8liVv90Iq-_ZMIwlJNqsu6s4bO0F1kR3dMUjedqC16uBUu85)

[comment]:<http://jmp.sh/VBIAyhU>

## Bonus: Exporting Your Files

If you're running your minix in a virtual machine like VMWare or VirtualBox (it's likely that you are), you can export your files to a VM shared folder quickly.
To do so, first make sure your shared folder is mounted:
```mount -t vbfs -o share=sharedFolder none /mnt```

Then implement the following script:

```nano /home/export.sh```

```
#!/bin/sh
echo "We're now exporting all the files to our shared folder."
echo "Exporting table.c file..."
cp /usr/src/minix/servers/pm/table.c /mnt/table.c
echo "Exporting callnr.h file..."
cp /usr/src/minix/include/minix/callnr.h /mnt/callnr.h
echo "Exporting proto.h file..."
cp /usr/src/minix/servers/pm/proto.h /mnt/proto.h
echo "Exporting mdc.c file..."
cp /usr/src/minix/servers/pm/mdc.c /mnt/mdc.c
echo "Exporting pm/Makefile file..."
cp /usr/src/minix/servers/pm/Makefile /mnt/Makefile_pm
echo "Exporting customlib.h file..."
cp /usr/src/include/mylibrary.h /mnt/mylibrary.h
echo "Exporting include/Makefile file..."
cp /usr/src/include/Makefile /mnt/Makefile_include
echo "Exporting ex1a.c file..."
cp /root/ex1a.c /mnt/ex1a.c
echo "Exporting ex1b.c file..."
cp /root/ex1b.c /mnt/ex1b.c
echo "Finished exporting everything!"
```
Save your file, allow it to be executed, and then execute it:
```
chmod +x export.sh
./export.sh
```

## Support

If you have any further questions regarding the implementation of these files, or better explanations, please feel free to contact me.

This guide was written purely to give some light to the folks out there that are having issues implementing system calls in Minix v3.3.0. It still needs much improvement and a better explanation about the implementation of each file.

### To-do:

- Explain what each file does, and why we changed what we changed;
- Provide screenshots of the tests made;
- Explain how to mount a shared folder;
- Improve the formatting of this guide;
- Localize this guide into Portuguese