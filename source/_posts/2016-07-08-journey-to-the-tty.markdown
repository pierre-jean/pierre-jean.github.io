---
layout: post
title: "TTY: under the hood"
date: 2016-07-08 12:00:00 +0100
comments: true
categories: Linux
---

It all started with simple instruction: `-t flag assigns a pseudo-tty or terminal inside the new container`... And a few seconds of puzzling thoughts. _What is exactly a pseudo TTY?_ _What does it mean to attach or detach a process from it?_

Beware my friend, cause this article will lead you into the depths of forgotten history and guide you through the arcane of the kernel mechanism, but light of knowledge should shine upon you at the end of this journey.

<!-- More -->

The origin of the myth: the Teletype
------------------------------------

When you act as Neo from Matrix by popping up a fancy green terminal to impress your friends around (what do you mean, "nobody does that"?), you are actually playing with the legacy of a very old device, that surely not much of you have even seen in your life: the Teletype.

## The lengendary object

{% img/linux/teletype.jpg %}

Do you see this beauty in the picture? That's a Teletype. A Creed & company model 7. And speaking about Teletype models, I can tell you that there are countless, built by different companies. The history of Teletype finds its roots in first experimentations late 19th century, but truly began in the 20s and find an end in the 70s, when Fax where good enough to replace them.

A teletype is basically a machine that sends the letters you typed to the keyboard through electric signal to another machine or network, and prints (literally prints, on a paper!) letters received through the reception cable. Obviously, the different models evolved with time to offer more features and performances:

 * Usage of _Multiplex signal_, in order to allow the usage of one physical cable to send and receive messages
 * Support of _punched card_ to send prepared messages at full speed without the need of typing them
 * Usage of video screen (you're welcome, trees!)
 * Increase of speed (from 50 baud to 150000 baud)

This machine was at the time the best way to transmit data on a reliable and fast way.

## Stop your boring oldish gibberish... Why the heck are you telling me about this?

Because instead of building new devices to interact with computers, pragmatic people at the time decided to reuse the Teletype to connect to them.
Hear me well, I'm not talking here about the _Personal Computer_ (PC) you are familiar with, I'm talking about a big massive machine in its own dedicated room where you had no access if you were not cool enough, and where you had to send command from the teletype and read the output through the teletype remotely.

{% img/linux/mainframe.jpg }

Actually, one terminal was directly connected connected to the machine within the same room : _the console_. Man, I can't tell you about the hype on being the lucky one behind the console. Well, I can't because I'm not that old, but I'm sure that should have been a big thing back then.

Anyway, you must now realize a few things: it would be nice for instance when using a terminal to see what you are typing. So what about asking the computer to echo back to the terminal what it received, so that it is printed? And what about erasing with backspace what have been typed? Yep, the OS should take care of that for us, we only use a dummy Teletype after all.

## Under the hood

Here is a diagram of how how a Teletype was interacting with a computer:

//TODO

 1- Each machine is (conceptually or physically) connected to two cables: one to send instruction to the computer and one to receive output from the computer.
 2- These cable are connected to the computer through an Universal Asynchronous Receiver and Transmitter (_UART_) that transform the asynchronous flow of data into bytes words.
 3- The computer has a UART driver to read for the hardware device
 4- The sequence of characters are passed to the line discipline.The line discipline will be in charge of converting special characters (end of line, backspace), and echoing (reprinting) what has been received back to the teletype, so that the user can visual what he/she types.
 5- The flow of instruction is passed to the TTY driver, that pass them to the *foreground* processes for the *session* associated with this TTY. Indeed, as a user, you can execute several process in parallel, but only interact with one at a time, letting the others working (or waiting) in the background.

The whole stack define below can be called a TTY device, and several can exist at the same time for a computer. So different line discipline can be set for different devices, each TTY has its own foreground job, etc.

From the Teletype to the Terminal
---------------------------------

80s has brought many things, among them: not so intelligent movies and intelligent terminals. The first is debatable but the second is fact. Slowly, terminals evolved to badass devices, with screen, memory, and even small processors to handle some little stuffs on their side.

{% img/linux/terminal.png %}

They started to *look like* your current PC desktop. Beware, though, they are in no way comparable! They are still dummy stuff, despite their name. They do not compute things on their own: managing fancy colors and having a good screen refresh is far from being even close to a computer.

Nevertheless, as you can imagine, these newly sophisticated devices will inherit part of the Teletype mechanism.

Here is a graph of how it works:

//TODO

Master and slave: how the world works today
-------------------------------------------

## The virtual TTY







I'm getting more and more interested in how my system works "under the hood" lately. These fundations are essential to understand some behaviors / technical choices in technologies oriented for end users (such as [Docker][docker-site]).

That's why I decided to write, mostly as a reminder for myself, a serie of articles explaining some fundamental Linux mechanisms. As I have to start somewhere, I'll start with the boot process.

<!-- More -->

**Disclaimer: I will assume the machine is running with BIOS and Grub as bootloader. If you would like to have explanation for other type of configuration, with UEFI for instance, please let a comment to request such article.**

Power on : BIOS POST execution
------------------------------

When you switch on your computer, the processor doesn't know anything about its environment. So it will executes the instruction located in a predefined memory location (`FFFF:0000h` in X86 based computers).

The instructions will redirect to another memory location, which is mapped to the **[BIOS][bios-article] memory**.

Indeed, when booting, the processor runs in a mode called **real mode**. In this mode, the processor access directly the memory via addresses starting from 0 to 1MB (`0xFFFFF`). But this memory is not mapped entirely to the *[RAM][ram-article]* memory. Some addresses are mapped to the *BIOS memory*, and other to the *legacy video card memory*.

The BIOS instructions will then be executed.

{% img /images/linux/boot_00.png %}

The first BIOS action is the **POST (Power On Self Test)**. The POST will check the presence and integrity of a set of important hardware devices, and inform you if it detect defects (with a series of *bip* sound).

Reading the CMOS and choose the bootable device
-----------------------------------------------

You cannot modify the BIOS instructions by yourself (without flashing it), but in order to save some information (time, user preferences), the motherboard is shipped with a memory called *CMOS*.

The CMOS will contains for instance an ordered list for the bootable devices.
The BIOS will check the presence of each device from this list, and also if it contains a bootable media.

{% img /images/linux/boot_01.png %}

Let's say that the bootable media is a hard drive for the rest of the explanation.

Grub stage 1
------------

A hard drive contains a limited space in their first sector called **Master Boot Record (MBR)**. This MBR contains only 512 Bytes. And in theses 512 Bytes must enter:

 * The instructions to boot the system (446 Bytes)
 * The information about the partitions of the hard drive (64 Bytes)
 * A checksum to verify the integrity of the MBR, called *Magic Number* (2 Bytes)

The fact that the MBR is so small will lead to many difficulties: will the code to load the Linux kernel fit in this 446 Bytes? This limited size is also the reason why you can only have 4 primary partitions (4 descriptions of 16 Bytes), limited to 8GB or 2TB depending on the method they are described (*CHS* vs *LBA*).

{% img /images/linux/boot_02.png %}

To solve that, the boot process is done in several steps. The first step will load the instructions from the 446 Bytes length **primary boot loader code** and execute it. This is called the Grub Stage 1. **[GRUB][grub-article]** is what we called a *boot loader*.

Grub stage 1.5
--------------

The partitions start at sector 63. The MBR is written in sector 0. This area between the MBR and the beginning of partitions is called **MBR GAP**. GRUB uses this space to put some extra code in it.

The code in *GRUB stage 1*, doesn't know how to read a Linux partition, and as a consequence load the kernel image that is inside. That's why the *GRUB stage 1* load the content of the *MBR GAP*, which contains instructions for opening a Linux partition. 

{% img /images/linux/boot_03.png %}

The execution of the code of the *MBR GAP* is called **Grub stage 1.5**.

Grub stage 2
------------

But a part of the Grub configuration is modified by the user in configuration files located inside the Linux partition. That's why *Grub stage 1.5* needs to open the partition (the one that is called *active*) and read the file `/boot/grub/grub.cnf`.

{% img /images/linux/boot_04.png %}

When this is done, the Grub is in *stage 2*, and can prompt an interface to let the user make choices (such as choosing which kernel to load or add boot options parameters).

Kernel Boot execution
---------------------

The Linux kernels are located in the `/boot` folder, and have name such as `vmlinuz-x.xx.x-xx-[...]`. These kernels are compressed files, that contains a *generic* core Linux kernel (if you didn't build it yourself). As this kernels have been generated by other people, who don't know the specificity of your configuration, some modules (as drivers) can be missing.

That's why you can also find in the `/boot` folder image called `initrd.img-x.xx.x-xx-[...]`. An *initrd* file is a temporary file-system that will be mounted during the boot process, and that will load the missing modules in the kernel. It allows to keep the kernel images small and to load dynamically only what is needed.

{% img /images/linux/boot_05.png %}

The vmlinuz files are executables files, that are actually composed of different chunks (`bootsect.o` + `setup.o` + `misc.o` + `piggy.o`). The `piggy.o` contains the gzipped vmlinux file. The processor is still in *real mode* which mean it can only address a memory area of 1 MB. The problem is that the compressed image is bigger than 1 MB. That's why the property of *bzImage* that allows to split the image in discontinuous area is convenient here. The `piggy.o` chunk is loaded outside the 1MB area zone (by switching the processor to protected mode), and the `bootsect.o`, `setup.o`, and `misc.o` will be load in the *real mode* map area.

The `bootsect.o` contains a legacy code that is now ignored (used to be the bootloader code for the *MBR*), and `setup.o` and `misc.o` initialize some variables and configuration (memory, video). After that the CPU pass into **protected mode**.

In *protected mode*, the CPU can address up to 4GB of memory, and can finally execute the `decompress_kernel()` instruction, that will uncompress the kernel image and mount the system layout, for finally launch the **init process**.

The **init** process is the first process launched in user space, and has the responsibility to launch all other needed services.

Historically, it was the **[SysVinit][sysvinit-article]** that did this job, but some distribution use **[Upstart][upstart-article]**, or the new **[Systemd][systemd-article]**. All these solutions work really differently (and would deserve a dedicated article), but they all get the same job done: launching all other needed user processes to start the system.


Sources
-------

I based my understanding of the boot process from these great articles:

 * [The Master Boot Record by Dewassoc][mbr-knowledge-center]
 * [Linux Booting Process by Slashroot][linux-boot-process-slashroot]
 * [Kernel Boot Process by Gustavo Duarte][kernel-boot-process-duarte]

And also thanks to many articles of [Wikipedia][wikipedia] and answers of the [Unix and Linux StackExchange][unix-stackexchange] community.

For any opinion, correction or question, feel free to comment!

[mbr-knowledge-center]: http://www.dewassoc.com/kbase/hard_drives/master_boot_record.htm
[linux-boot-process-slashroot]: http://www.slashroot.in/linux-booting-process-step-step-tutorial-understanding-linux-boot-sequence
[kernel-boot-process-duarte]: http://duartes.org/gustavo/blog/post/kernel-boot-process/

[wikipedia]: https://en.wikipedia.org
[unix-stackexchange]: https://unix.stackexchange.com/


[docker-site]: http://docker.io
[bios-article]: https://en.wikipedia.org/wiki/BIOS
[ram-article]: https://en.wikipedia.org/wiki/RAM
[grub-article]: https://www.gnu.org/software/grub/

[sysvinit-article]: https://en.wikipedia.org/wiki/Sysvinit
[upstart-article]: https://en.wikipedia.org/wiki/Upstart
[systemd-article]: https://en.wikipedia.org/wiki/Systemd
