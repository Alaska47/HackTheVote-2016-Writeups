>Our Trump advertising campaign is incredible, it's skyrocketing! It's astronomical! Wait stop!! SLOW DOWN!!!
>[warp_speed](https://s3.amazonaws.com/hackthevote/warp_speed.5978d1405660e365872cf72dddc7515603f657f12526bd61e56feacf332cccad.jpg)

We are given this image:

![](https://raw.githubusercontent.com/Alaska47/HackTheVote-2016-Writeups/master/forensics/150-Warp-Speed/warp_speed.jpg)

It looks like the image has been split up and pieced back together weirdly. 

Looking closely at the image, we can see that the left half is indeed different compared to the right half of the image. Also, the horizontal bars are split up into sections of 1000x8. 

Knowing this, we can split the image up into 500x8 pieces using Imagemagick with the command: 

`convert warp_speed.jpg -crop 500x8 warp_left.jpg`. 

The command creates 64 new [images](https://raw.githubusercontent.com/Alaska47/HackTheVote-2016-Writeups/master/forensics/150-Warp-Speed/warp_left.zip) from the original one.

If we play around with the new sections using an image editor (I used [Pixlr](pixlr.com/editor/)), we see that each new section fits with the previous when offset by four pixels. 

With this new information, we can write a Python program to piece together the images in the right way to recreate the original.

```python
from PIL import Image

image1 = []
for x in range(64):
   image1.append("warp_left-" + str(x) + ".jpg")
images = map(Image.open, image1)
print(image1)

total_width = 63*4+500
total_height = 64*8

nim = Image.new("RGB", (total_width, total_height))

x_offset = 63*4
y_offset = 0
for im in images:
   nim.paste(im, (x_offset, y_offset))
   x_offset -= 4
   y_offset += 8

nim.save("no_warp.jpg")
```

Running the program outputs this image:

![](https://raw.githubusercontent.com/Alaska47/HackTheVote-2016-Writeups/master/forensics/150-Warp-Speed/no_warp.jpg)

As you can see, our flag is: `flag{1337_ph0t0_5k115}`
