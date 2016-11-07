Looking at the problem statement, we can see some characters are capitalized: APNG
>Somebody leAked TrumP's favorite colors, looks like they used a really esoteric format. Some chiNese hacker named "DanGer Mouse" provided us the leak, getting this crucial info could really sway voters at the polls! [trump_likes_colors.png](https://s3.amazonaws.com/hackthevote/trump_likes_colors.bcddf8152cf2848c058310655c280a7dbb4f22fcc3687f00a26b6e9a57657dc4.png)

This tells us that the file might be an animated PNG. We can use ffmpeg to extract all the frames from the file.

`ffmpeg -i trump_likes_colors.png "colors%d.png"`

ffmpeg gives us 16,384 frames from the png file. All the images look like the original image except with the color bars having various lengths. Interestingly enough, 16,384 is a perfect square (128x128).

Looking at the next part of the problem statement, we see that the format used to encode the flag is a `esoteric format` leaked by `Danger Mouse`. Searching up `colors esoteric danger mouse` leads us to this [link](http://www.dangermouse.net/esoteric/piet.html). The site links us to an [online interpreter](http://www.bertnase.de/npiet/npiet-execute.php) for the Piet programming language.

Uploading one of the frames to the online interpreter gives us an hex color string. From here, we can reasonably figure out that each frame is a Piet program to give an hex color string which makes up a 128x128. All that is left to do is to write a program to generate the image from the frames.

I downloded the npiet windows [executable](http://www.bertnase.de/npiet/npiet-1.3a-win32.zip) and wrote a quick python program to compile all the hex strings for each frame

```python
from subprocess import check_output

ff = open('out12.txt', 'w')

for x in range(1, 128**2+1):
    try:
        mess = check_output(".\\npiet.exe colors" + str(x) + ".png", shell=True).decode()
        ff.write(mess + "\n")
        print(str(x) + "|" + mess)
    except:
        print("#bad")
        ff.write("#bad\n")

ff.close()
'''
Then I took the hex color strings and compiled them into an image.
