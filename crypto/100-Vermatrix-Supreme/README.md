100 - Vermatrix Supreme

>Working in IT for a campaign is rough; especially when your candidate uses his password as the IV for your campaign's proprietary encryption scheme, then subsequently forgets it. See if you can get it back for him. The only hard part is, he changes it whenever he feels like it. [handout.py](https://s3.amazonaws.com/hackthevote/handout.4838bbdb8619b3a581352c628c6b0b86475b94c9519347a520c90cf1822351ae.py)

nc vermatrix.pwn.democrat 4201

After downloading the file, we open it up in IDLE and we see that the chall() function has to return true for it to print the flag.
We can see that it uses IV and seed to generate a encrypted list called res. The user then has to input 9 numbers separated by "," which matches IV in order for the function to return true.
I used dummy test data in order to see what the program is doing. I set my seed as "abcdefghiqwertyuio" and the IV as "123456789". I added a print statement after block is initialized so see what it is set to.

`[[['1', '2', '3'], ['4', '5', '6'], ['7', '8', '9']], [[97, 98, 99], [100, 101, 102], [103, 104, 105]], [[113, 119, 101], [114, 116, 121], [117, 105, 111]]]`

It seems like it creates a list of the 9 IV values with the ascii value of the seed.
Next we have to determine how res is being encrypted. We take a look at the fixmatrix function.

```python
out[cn][rn] = (int(matrixa[rn][cn])|int(matrixb[cn][rn]))&~(int(matrixa[rn][cn])&int(matrixb[cn][rn]))
```

This piece of code is what is encrypting the values. We can see that in simple terms, it is (a OR b) AND (NOT (A AND B))
After using boolean algebra to simplify it, you find that it is equivalent to a XOR b.

Then, I added print statements to see what is being XORed with what and what the result is.

![](https://github.com/Alaska47/HackTheVote-2016-Writeups/blob/master/crypto/100-Vermatrix-Supreme/xors1.PNG)

It seems that the nine IV values are each being XORed with two values in the seed. Next we just have to discover which index of IV is being XORed with which index of the seed.

![](https://github.com/Alaska47/HackTheVote-2016-Writeups/blob/master/crypto/100-Vermatrix-Supreme/xors2.PNG)

We can see the first value in IV is being XORed with the 1st and 9th value in seed. Then, the second value in IV is being XORed by the 4th and 13th. Then the third is with the 7th and 16th, and so on.
Then, the end result is what res is.
If we connect to nc vermatrix.pwn.democrat 4201, we find that it gives the Seed and res.
The inverse of XOR is XOR, so you would simply take the res value and XOR it with the two values the original character was XORed with.
For instance, 17 XOR 113 (ascii of r) XOR 97 (ascii of a) = 1, the first value.

Now we just have to write a program to netcat there, and calculate the correct IV!

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("vermatrix.pwn.democrat", 4201))

inp = repr(s.recv(1024))
print inp
seed = inp.split("\\n")[0][6:].strip()
res = inp.split("\\n")[1] + " " + inp.split("\\n")[2] + " " + inp.split("\\n")[3]
res = res.strip().split(" ")

print (seed)
print (res)

flag = ""
c = 0
for i in range(len(res)):
    if (len(seed) == 18):
        flag += str(int(res[i]) ^ ord(seed[c]) ^ ord(seed[i + 9])) + ","
    else:
        flag += str(int(res[c]) ^ ord(seed[c])) + ","
    c += 3
    if c > 8:
        c -= 8
flag = flag[:-1]
print (flag)
s.send(flag.encode())
print (repr(s.recv(1024)))
s.shutdown(socket.SHUT_WR)
s.close()
```

![](https://github.com/Alaska47/HackTheVote-2016-Writeups/blob/master/crypto/100-Vermatrix-Supreme/flag.PNG)

flag{IV_wh4t_y0u_DiD_Th3r3}
