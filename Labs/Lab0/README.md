# **MemLabs Lab 0 - Never Too Late Mister**

## **Challenge Description**

My friend John is an "environmental" activist and a humanitarian. He hated the ideology of Thanos from the Avengers: Infinity War. He sucks at programming. He used too many variables while writing any program. One day, John gave me a memory dump and asked me to find out what he was doing while he took the dump. Can you figure it out for me?

Challenge file: [Google drive](https://drive.google.com/file/d/1MjMGRiPzweCOdikO3DTaVfbdBK5kyynT/view)

### Initial Thoughts

In most beginner level CTFs whenever we are provided with a memory forensics challenge, we also have a description which gives out certain clues. Identifying these clues can be quite tricky at first but becomes easier if you go on playing more & more CTFs.

Now the clues that we can pick from this description are as follows:
+ Environmental Activist (Since the word is quoted)
+ John hates Thanos (Maybe useless but let us see)
+ John sucks at programming and used too many variables.

Now let us move into analyzing the memory dump.

### Memory dump analysis

We'll be analyzing the memory dump file (challenge.raw) using **Volatility 2.6** as it is best suited to our needs. All the labs in the repository can be solved using Volatility 2.

The very first thing that anyone needs to know before proceeding to forensic analysis of a memory dump is to determine the **profile** we are going to use.

The profile tells us the OS of the system or computer from which the dump was extracted. Volatility has a built-in plugin to help us determine the profile of the dump

Now, we'll be using the **imageinfo** plugin

`$ vol2 -f Challenge.raw imageinfo`

Now as you can see, volatility provides a lot of suggestions as to which profile you should use. In some cases, all of the suggested profiles may not be correct. To help get over this barrier, you may use another plugin called **kdbgscan**. As far as this challenge is concerned, using kdbgscan isn't required.

Now as a forensic analyst, one of the most important things we would like to know from a system during analysis would be:
+ Active processes
+ Commands executed in the shell/terminal/Command prompt
+ Hidden processes (if any) or Exited processes
+ Browser History (This is very much subjective to the scenario involved)

And many more...

Now, to list the active or running processes, we use the help of the plugin **pslist**.

`vol2 -f Challenge.raw --profile=Win7SP1x86 pslist`

Executing this command gives us a list of processes which were running when the memory dump was taken. The output of the command gives a fully formatted view which includes the name, PID, PPID, Threads, Handles, start time etc..

Observing closely, we notice some processes which require some attention.

+ cmd.exe
+ DumpIt.exe
+ explorer.exe

+ cmd.exe
    + This is the process responsible for the command prompt. Extracting the content from this process might give us the details as to what commands were executed in the system
+ DumpIt.exe
    + This process was used by me to acquire the memory dump of the system.
+ Explorer.exe
    + This process is the one which handles the File Explorer.

Now since we have seen that **cmd.exe** was running, let us try to see if there were any commands executed in the shell/terminal.

For this, we use the **cmdscan** plugin.

`vol2 -f Challenge.raw --profile=Win7SP1x86 cmdscan`


If you can see from the above image, a python file was executed. The executed command was `C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt`

So our next step would be check if this python script sent any output to **stdout**. For this, we use the **consoles** plugin.

`vol2 -f Challenge.raw --profile=Win7SP1x86 consoles`

We see that a certain string `335d366f5d6031767631707f` has been written out to **stdout**. Now as one might observe, this is a **hex-encoded** string. Once we try to revert out the hex encoding, we get a gibberish text.

Now if you remember, we tried to deduce some clues from the challenge description. The first one was something with the word "environment". Now there are certain system determined variables called **Environment variables**

To view the environment variables in a system, use the **envars** plugin. Going down the output, we see a strange variable by the name **Thanos** (Ah! so maybe that's why it was provided in the description.), the value of the variable is `xor and password`.

`vol2 -f Challenge.raw --profile=Win7SP1x86 envars`

Now, we have 3 things in total:

+ The gibberish text resulted from reverting the hex-encoded string
+ Xor
+ Password

Thinking for a while, makes us realise why the clue `xor` was provided. Let us try to attempt xor decoding on the gibberish text.

[CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')XOR_Brute_Force(1,100,0,'Standard',false,true,false,'')&input=MzM1ZDM2NmY1ZDYwMzE3Njc2MzE3MDdm)


There are only 255 possibilities and if you see, the 3rd output is a suspicious text **1_4m_b3tt3r}**. That looks like part of the flag.

Well, the next part is `password`. Using volatility, we can extract the NTLM password hashes using the **hashdump** plugin.

`vol2 -f Challenge.raw --profile=Win7SP1x86 hashdump`

Now the password hash we have to decrypt is `101da33f44e92c27835e64322d72e8b7`. We can use online NTLM hash cracking websites.

Well there you go, we have the other half of the flag --> **flag{you_are_good_but**. Concatenating the 2 parts gives us the whole flag.

FLAG: **flag{you_are_good_but1_4m_b3tt3r}**

I hope with the walkthrough provided for this simple CTF challenge, gives confidence and encourages you to try out the rest of the labs.

If you have any doubts/queries, feel free to reach out to me --> [Contact me](https://github.com/stuxnet999/MemLabs#author)

## Resources and Cheatsheets

+ https://github.com/volatilityfoundation/volatility/wiki/Command-Reference
+ https://downloads.volatilityfoundation.org/releases/2.4/CheatSheet_v2.4.pdf
+ https://blog.onfvp.com/post/volatility-cheatsheet/
