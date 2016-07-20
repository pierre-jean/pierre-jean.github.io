---
layout: post
title: "TTY: under the hood"
date: 2016-07-08 12:00:00 +0100
comments: true
categories: Linux
---

It all started with simple instruction: `-t` _flag assigns a pseudo-tty or terminal inside the new container_... and a few seconds of puzzling thoughts... _What is exactly a pseudo TTY?_ _What does it mean to attach or detach a process from it?_

{% img center /images/linux/gnome-terminal.png %}

Beware my friend, 'cause this article will lead you into the depths of forgotten history and guide you through arcane kernel mechanisms, but light of knowledge should shine upon you at the end of this journey.

<!-- More -->

The origin of the myth: the Teletype
------------------------------------

When you act as Neo from Matrix by popping up a fancy green terminal to impress your friends around (what do you mean, _"nobody does that"_?), you are actually playing with the legacy of a very old device, that surely not many of you have ever seen in your life: the Teletype.

### A legendary object

{% img center /images/linux/teletype-model-28.jpg %}

Do you see this beauty in the picture? That's a Teletype. A _model 28_ by _Teletype Corporation_ to be precise. And precise should I not be, as there are countless of models, built by many different forgotten companies. The history of teletypes finds its roots in some first experimentations during the late 19th century, but truly began in the 20s and finds an end in the 70s, when fax technology began to be good enough to replace them.

A teletype is basically a machine that sends letters you typed on the keyboard through electric signals to another machine or network, and prints (literally prints, on a paper!) letters received through the reception cable. Obviously, the different models evolved with time to offer more features and improved performance:

 * Use of _Multiplex signal_, in order to allow the usage of one physical cable to send and receive messages
 * Support of _punched card_ to send prepared messages at full speed without the need of typing them
 * Use of video screen (you're welcome, trees!)
 * Increase of speed (from 50 baud to 150000 baud)

This type of machine was at the time the best way to transmit data in a reliable and fast way.

### Stop your boring oldish gibberish... Why the heck are you telling me about this?

Because instead of building new devices to interact with computers, pragmatic people at the time decided to reuse existing teletypes to connect to them.
Hear me well, I'm not talking here about the _Personal Computer_ (PC) you are familiar with, I'm talking about a big massive machine located in its own dedicated room where you had no access if you were not cool enough, and where you had to send commands from the teletype and read the output printed back on the device.

{% img center /images/linux/mainframe2.jpg %}

Actually, one terminal was directly connected to the machine within the same room : _the console_. Man, I can't tell you about the hype on being the lucky one behind the console. Well, I can't because I'm not that old, but I'm sure that it should have been a big thing back then.

Anyway, you must now realize a few things: it would be nice for instance when using a terminal to see what you are typing. So what about asking the computer to echo back to us what it received, so that it is printed on our teletype? And what about erasing with backspace what have been typed? Yep, the computer at the end of the cable should take care of that for us, we only use a dummy Teletype after all.

### Under the hood

Here is a diagram of how a teletype was interacting with a computer:

{% img center /images/linux/teletype-mainframe-workflow.png %}

 1. Each machine is (conceptually or physically) connected to two cables: one to send instructions to the computer and one to receive output from the computer.
 2. These cables are connected to the computer through a serial cable plugged into an Universal Asynchronous Receiver and Transmitter (_UART_) that transforms the asynchronous flow of data into bytes words.
 3. The computer has an UART driver to read for the hardware device.
 4. The sequence of characters is passed to the line discipline. The line discipline will be in charge of converting special characters (like _end of line_, _backspaces_), and echoing (reprinting) what has been received back to the teletype, so that the user can visualize what he/she types.
 5. The flow of instruction is passed to the TTY driver, that passes them to the *foreground* processes for the *session* associated with this TTY. Indeed, as a user, you can execute several processes in parallel, but only interact with one at a time, letting the others working (or waiting) in the background.

The whole stack as defined above is called a *TTY device*, and several ones can exist at the same time for a computer. So different line disciplines can be set for different devices, each TTY having its own foreground job, etc.

From the Teletype to the Terminal
---------------------------------

Besides unpredictable haircuts and memorable rythms, the 80s have also brought us what they called _intelligent terminals_. Slowly, terminals evolved to badass devices, with screen, memory, and even small processors to manage specific features on their side.

{% img center /images/linux/terminal_vt100.jpg %}

They started to *look like* your current PC desktop. Beware, though, they are in no way comparable! They are still dummy objects, despite their name. They do not compute things on their own: managing fancy colors and having a fast refresh frequency is far from being even close to a computer. It's the 80s after all, it's hard to call anything smart from that period of time...

These devices worked the same way as teletypes, but also introduced some new special features that had to be supported by the software to be managed correctly (colors, special movements, etc).

Wake up Neo, it is all virtual
------------------------------

The massive set of wardrobes that used to constitute a computer gradually turned down to a nice little box that you could fit under your desk. And there were no more dozens of terminals connected to it, but only one monitor and one keyboard. Nevertheless, your current Linux machine keeps emulating several (usually 7 by default) terminals connected to your hardware. But to protect you from the effort of getting up and physically going to another chair, the OS allows you to switch from one terminal to another by a simple press of keys (`Ctrl`+`Alt`+`F1` to `Ctrl`+`Alt`+`F7`). This feature is called _virtual terminals_, and is represented by the files `/dev/tty1` to `/dev/tty7`. You can see any of these files as a duplex cable connected to a terminal. If you write to it, you send the information to be printed to the terminal, if you read from it, you receive what is typed from the terminal (try it, it works).

{% img center /images/linux/virtual-terminal-workflow.png %}

When you switch from one virtual terminal to another, the OS detach your _seat_ (a set of input and output devices like monitor, keyboard, mouse, etc. representing the hardware interface with the user) from the first virtual terminal and attach it to the one requested by your shortcut. The processes from the first virtual terminals keep running, writing and reading from their virtual tty file (`dev/tty1` for instance), but this file won't receive any event from the seat and won't be able to send output to the seat. The information will be buffered instead (until your reattach your seat to this terminal), making the switch between sessions transparent for running jobs.

I know no master
----------------

{% img center /images/linux/matrix-operator.jpg %}

But I imagine only a few of you are actually using the virtual tty just mentioned. You are usually using a terminal console application launched from a graphic environment that is itself launched from a virtual terminal (yep, breathe and read again).

So when you launch your favorite terminal emulator like _xterm_ or _gnome-terminal_, how do the processes know where to write the output, and where to get the input from?

{% img center /images/linux/ptmx-pts-workflow.png %}

Basically, when you launch a terminal within a graphic environment like this, it will spawn its own equivalent of `/dev/ttyX`: the terminal emulator will open a special file located in `/dev/ptmx`, called the _master side_ of the _pts_, will do some magic with `ioctl` function, which will create a _slave side_ of the pts in `/dev/pts/X`. 

The processes running in the session will be attached to this file, that will behave like any file from the virtual terminal, except that there is no attachment to a seat: you can open several terminal emulator windows at the same time and display them side by side, having different sessions running in parallel.

Going further
-------------

We could dig the topic endlessly, speaking about the function `ioctl`, detailing how the kernel handle the session, expressing our endless admiration for the great 70s look of the [DEC VT05][vt05] terminal...
But we should keep a bit for further articles, and there are anyway plenty of great resources already available if you are interested. To share a few:

 * [the TTY demystified, by Linus Åkesson][linusakesson]: Simply _the_ reference on the topic, that will also explain signals, processes, etc.
 * [Ponyhof's session management][ponyhof1] and [vt-switching][ponyhof2] articles: Great to understand the session and seats concepts.
 * [Unix StackExchange][unix-stackexchange] Stéphane Chazelas' answer, that put a lot of effort to makes things clear when they were a lot confusing for me.

I realize I took a lot of shortcuts in this article and it would be natural that some part appear blurry in your mind. So if you have any question or need precisions, please leave a comment, I will try my best to provide a clear answer!

[vt05]: http://terminals.classiccmp.org/wiki/images/f/fb/DEC_VT05_121708587772-2.jpg
[linusakesson]: http://www.linusakesson.net/programming/tty/
[ponyhof1]: https://dvdhrm.wordpress.com/2013/08/24/session-management-on-linux/
[ponyhof2]: https://dvdhrm.wordpress.com/2013/08/24/how-vt-switching-works/
[unix-stackexchange]: http://unix.stackexchange.com/questions/117981/what-are-the-responsibilities-of-each-pseudo-terminal-pty-component-software
