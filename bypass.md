# Bypass Walkthrough
* * *

## TL;DR
Bypass is an easy reversing challenge on hackthebox.eu

This challenge was done on a windows machine and used the following tools
* strings
* dnSpy 

Modifying values on runtime is a good skill to have.

Knowing how to use breakpoints is an even better skill to have.

## Basic Enumeration
We downloaded a zipped up file from HTB and unzipped it, this gave us a single executable file called Bypass.

When we ran the executable we seemed to get a prompt asking for a username and password in a loop.

![Bypass Executed](/resources/Bypass/execute.PNG)
*Figure 1: Running Bypass.exe*
<br>

Running strings on the executable usually lets us see if the executalbe is compressed/packed in any way, so we ran it.
Strings also lets us see what alphanumeric combinations exist in the executable, this tends to give us some information about the content of the precompiled code.

![Bypass Strings](/resources/Bypass/strings.PNG)
*Figure 2: Strings output on Bypass.exe*
<br>

In the last couple of lines of strings we see an interesting output. 
One of the final lines seems to expose information about what language the precompiled codebase was written in. 

![Bypass Net](/resources/Bypass/net.PNG)
*Figure 3: Strings output on Bypass.exe*
<br>

As you might have guessed from the output of strings shown in figure 3, the codebase was written in .NET and so C#.
This became relevant once we tried to exploit the executable.

## Exploitation

After attempting to reverse the executable in Radare2 for a while we realised that we werent getting anywhere.
Some research on the reversing of .NET binaries led us to a tool called [dnSpy](https://github.com/dnSpy/dnSpy).
dnSpy lets you decompile .NET code and set breakpoints during runtime in the same way as Radare2.

After playing around with dnSpy and making sure that we were using the right version (32 bit rather than 64 bit), we managed to start making some
coherent sense out of the binary.

![dnSpy](/resources/Bypass/dnSpy1.PNG)
*Figure 4: Decompiled binary*
<br>

The programme seemed to have a bunch of functions which checked the input by the user against a secondary set of values.

Looking at the boolean values flag and flag2 we saw that they were equal to eachother and since flag was equal to the output of function 1() we knew that flag2 would be too.
After looking at function 1() we realised that here is where we were getting prompted to enter a username or password, but there was a trick, the function always returned false.
Since the function 1() always returned false we could never get to line 13 in function 0() and so could never move to the secondary check (which we discuss later).

Our solution for this problem was to change the value of flag2 on the fly, and some we set a breakpoint before the if check executed and changed the value of flag2. The breakpoint is shown by a red dot on figure 4.


![beakpoint 1](/resources/Bypass/break1.PNG)
*Figure 5: Hitting the if statement*
<br>

With our breakpoint set we ran the programme and entered the username and password values.

As you can see in figure 5, once we hit our breakpoint we saw what we expected, flag and flag2 were both equal to false.
After modifying the value of flag2 to equal true we managed to pass the if statement and run the function 2() (as seen in line 13 of figure 4)

![beakpoint 2](/resources/Bypass/break2.PNG)
*Figure 6: Strings output on Bypass.exe*
<br>

In function 2() we got prompted for a secret key (lines 36-37 in figure 6), and that secret key got compared to a specific value. If the secret key was equal to the given value then true was stored in flag, if not, false was stored in flag.

Thanks to our trusty breakpoint in line 39 we saw that the value which we were comparing our input against was "ThisIsAReallyReallySecureKeyButYouCanReadItFromSourceSoItSucks".

![Final](/resources/Bypass/w.PNG)
*Figure 7: Strings output on Bypass.exe*
<br>

Once we ran the executable again and inputted the correct key we got the password for HTB! Success!

<br><br>
[back](./ctfs.md)
