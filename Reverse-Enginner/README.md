# Intoduction:

During the Kaizen Arabia CTF, I found interesting challenge (Little F0rt) under Reverse Engineering category, which request for a flag starting with the format “SAFCSP{}” in a given executable called “Bombs_Landed”

## Step \#1: Strings
First thing of course starting with the strings of the executable file, I found a string with thet given flag format:

![alt text](https://i.imgur.com/BHJvM24.png)

But unfortunately it was not the correct flag. 


## Step \#2: Type of the file
To reverse executable file, first we need to know its type, using Kali command I found that it is ELF 64-bit executable file for Linux Operating System: 
 
 ![alt text](https://i.imgur.com/ifL41RB.png)

 
And for that I used IDA Pro Linux server to start debugging it remotely.
 
 ![alt text](https://i.imgur.com/0B1Wuuq.png)

 
And then from Windows machine, I used IDA Pro remotely to execute Bombs_Landed executable and start debugging it.


## Step \#3: Debugging:
I Did some tracing with the execution until if found a strange behavior, there was four functions with name end with XOR, and after each call for those functions it overwrites one of the parameters passed to that function, I realized that it could be manipulating the content of that parameter and then overwrite the new result (which is what every function of those functions did).
The following graph for each one of these functions:

![alt text](https://i.imgur.com/SUvMPzl.png)
![alt text](https://i.imgur.com/YiNq49B.png)
![alt text](https://i.imgur.com/qx0rybU.png)
![alt text](https://i.imgur.com/DfqdxIA.png)
     
What is important is not what the functions do, but what are the results of them, so I set a breakpoint after each call of these functions:
 
 ![alt text](https://i.imgur.com/zQlhHBX.png)
 
And since the executable overwrite every parameter it passes to the function, using the Hex view on the address 0x0000000000600248 (which is the address of _GLOBAL_OFFSET_TABLE_):
 
  ![alt text](https://i.imgur.com/buOjTx9.png)
 
From this the only thing that is interesting the fake flag and word “RE4LLY!!” but when we run the executable and we stop on the first breakpoint after FirstXOR, we can see that the content of the Hex view changed:
 
  ![alt text](https://i.imgur.com/WingRJx.png)
 
Here we can see the changed highlighted “53414643”, and then the next instruction “mov     _GLOBAL_OFFSET_TABLE_, 0” will overwrite it with zeros.
All next functions do the same, XOR the content of the given parameter and then after the call it overwrite it. I collected the result of every function before overwriting, the results combined as following:
*
53414643

53507b30

6f485f53

65454d73

5f53306d

45304e65

5f50774e

5f6d457d
*
From what it looks like it seems an ASCII test in HEX format, and using tools or online web sites (like.rapidtables.com) to convert HEX to ASCII, the result will be the flag which is:

SAFCSP{0oH_SeEMs_S0mE0Ne_PwN_mE}

Note: some results of the functions will be more than 4 bytes; you need take only the last four bytes of the edited bytes.

Conclusion:
This idea of this challenge, it writes the Hex value of the flag on the memory and then overwrite it, and it use XOR to not be clear on the strings result and confuse with the fake flag. Also, we can see that it uses some system function’s names (like heapalloc, writememory) so that you may not notice that it contains the important data. Another way to solve this without debugging is by extracting the content of the functions parameter and perform manual XOR instead of making the debugger execute it for you.

I hope this explain how I solved this challenge.

