#Problem Statement

>A CNN reporter had only one question that she couldn't get off her mind

>Do we even know, who is this 4 CHAN???
>So she set out to find who this 400lb hacker is. During her investigation, she came across this cryptic message on some politically incorrect forum online, can you figure out what it means?

>[kek](https://s3.amazonaws.com/hackthevote/kek.43319559636b94db1c945834340b65d68f90b6ecbb70925f7b24f6efc5c2524e.txt)

#Solution

Upon opening the txt file give, we immediately noticed that it alternated between 2 things: `TOP` and `KEK`. Considering that this is a cryptography problem, we immediately thought binary. Then we noticed the `!`. From this, we can deduce that `TOP` and `KEK` indicate either `0` or `1` and `!` indicate the number of `0` or `1`. Now we just have to write a program to test this theory!

```python
for i in open("topkek.txt", "r").read().split(" "):
    if ("KEK" in i):
        print ("0" * (len(i) - 3), end = "")
    else:
        print ("1" * (len(i) - 3), end = "")
```

From this program we get
>0110011001101100011000010110011101111011010101000011000001101111001100000110111100110000011011110011000001101111001100000101000001011111010111110101111101011111010111110101111100110001011011010101111101101000001101000101011000110001011011100100011101011111010001100111010101001110010111110111001000110001011001110100100001110100010111110110111000110000010101110101111100110100010100100011001101011111011110010011000001110101010111110110100000110100011101100011000101101110011001110101111101100110011101010110111001011111010111110101111101011111010111110101111101001011001100110100101100100001001000010010000101111101

Then we can use this [website](http://www.binaryhexconverter.com/binary-to-ascii-text-converter) to convert the binary to ascii and we get the flag!

#Flag

>flag{T0o0o0o0o0P______1m_h4V1nG_FuN_r1gHt_n0W_4R3_y0u_h4v1ng_fun______K3K!!!}
