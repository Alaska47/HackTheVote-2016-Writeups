>Somebody leAked TrumP's favorite colors, looks like they used a really esoteric format. Some chiNese hacker named "DanGer Mouse" provided us the leak, getting this crucial info could really sway voters at the polls! [trump_likes_colors.png](https://s3.amazonaws.com/hackthevote/trump_likes_colors.bcddf8152cf2848c058310655c280a7dbb4f22fcc3687f00a26b6e9a57657dc4.png)

We are given this image:

![trump_likes_colors.png](https://raw.githubusercontent.com/Alaska47/HackTheVote-2016-Writeups/master/forensics/250-Trump-likes-colors/trump_likes_colors.png)

Looking at the problem statement, we can see some characters are capitalized: APNG

This tells us that the file might be an animated PNG. We can use ffmpeg to extract all the frames from the file.

`ffmpeg -i trump_likes_colors.png "colors%d.png"`

ffmpeg gives us 16,384 [frames](https://raw.githubusercontent.com/Alaska47/HackTheVote-2016-Writeups/master/forensics/250-Trump-likes-colors/trump_colors_split.zip) from the png file. All the images look like the original image except with the color bars having various lengths. Interestingly enough, 16,384 is a perfect square (128x128).

Looking at the next part of the problem statement, we see that the format used to encode the flag is a `esoteric format` leaked by `Danger Mouse`. Searching up `colors esoteric danger mouse` leads us to this [link](http://www.dangermouse.net/esoteric/piet.html). The site links us to an [online interpreter](http://www.bertnase.de/npiet/npiet-execute.php) for the Piet programming language.

Uploading one of the frames to the online interpreter gives us an hex color string. From here, we can reasonably figure out that each frame is a Piet program to give an hex color string which makes up a 128x128. All that is left to do is to write a program to generate the image from the frames.

I downloded the npiet windows [executable](http://www.bertnase.de/npiet/npiet-1.3a-win32.zip) and wrote a quick python program to compile all the hex strings for each frame

```python
from subprocess import check_output

ff = open('out.txt', 'w')

for x in range(1, 128**2+1):
    try:
        mess = check_output(".\\npiet.exe colors" + str(x) + ".png", shell=True).decode()
        ff.write(mess + "\n")
        print(str(x))
    except:
        print("!")
        ff.write("#bad\n")

ff.close()
```

Then I took the hex color strings and compiled them into an image using this program.

```python
from PIL import Image
import struct

t = open('out.txt').read().split()
tup = []
for each in t:
   hex1 = each[1:]
   tup.append(struct.unpack('BBB', hex1.decode('hex')))

im =  Image.new("RGB", (128,128))
im.putdata(tup)
im.save("temp1.png")
```
I ended up with this image: ![](https://raw.githubusercontent.com/Alaska47/HackTheVote-2016-Writeups/master/forensics/250-Trump-likes-colors/temp1.png)

Looking at the Piet program, it is clear that it does something, however executing it using the online interpreter reveals nothing. So somehow the program loads the flag and just stores it without printing anything out. After reading up on Piet and looking up some sample programs, I came accross this [link](http://progopedia.com/example/hello-world/323/). It looks like the number of pixels in each block of colors should be the ASCII-code of a character. Applying the same logic to our generated image, I wrote a program to interpret the image:

```python
from PIL import Image

def parse_length(im, col):
   num = 0
   while(pix[col,num] != (0, 0, 0)):
      num += 1
   return chr(num)

im = Image.open('temp1.png')
pix = im.load()

height = im.height
x = 0
while(x < height and pix[x,0] != (0, 0, 0)):
   print(parse_length(im, x)),
   x += 2
```

Running the program gives us our flag: `flag{7h15_w45n7_3v3n_4_ch4ll3n63._54d_.}`
