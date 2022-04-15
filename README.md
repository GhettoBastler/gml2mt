# gml2mt

Draw GML files on a Minitel. Written in Python3

https://user-images.githubusercontent.com/100698182/163582119-9fff3aa1-d630-42d3-9983-df86574c8849.mp4

*[View tag on #000000book](https://000000book.com/data/67604)*

## Requirements

1. A Minitel 1B or 2 (with a DIN-5 port on the back)
1. Python 3.10+ with the pyserial library
1. A logic level converter for adapting the voltages between the Minitel and the computer (this [tutorial by Pila (in French)](https://pila.fr/wordpress/?p=361) explains how to build your own USB adapter)

## Installation

Download gml2mt from this repository :
```
wget https://raw.githubusercontent.com/GhettoBastler/gml2mt/main/gml2mt
```

You will also need a GML file to display. You can download one from [#000000book.com](https://000000book.com/data)

## Usage

Connect to the Minitel using the DIN-5 connector on the back and power it on. I personally use a Raspberry Pi instead of a computer, but this is not required. Any computer with a USB port and the correct adapter should do the trick.

<img src="https://user-images.githubusercontent.com/100698182/163582250-018aaead-036e-4ae7-a0ed-9a3b6f445443.jpg" width="500">

### On the Minitel
1. Switch to graphics mode by pressing **Fnct + T** followed by **V**
1. Change the baudrate to 4800 (the maximum) by pressing the **Fnct + P** followed by **4**.

### On the computer/Pi

Run gml2mt, passing the serial port to use and the GML file as arguments (if you are using a Raspberry Pi, the serial port should be /dev/ttyAMA0)

```
gml2mt.py SERIAL_PORT GML_FILE
```

The Minitel should start the animation after a few seconds

https://user-images.githubusercontent.com/100698182/163586065-db7536a1-450e-47ed-a80d-eba596f499ec.mp4

## About this project

### What is GML ?
[GML](https://en.wikipedia.org/wiki/Graffiti_Markup_Language) (for Graffiti Markup Language) is a file format that allows graffiti artists to store tags as computer files. Because these files record the motion of a tag being drawned, they can then be reused in a variety of ways, like [tagging robots](https://www.youtube.com/watch?v=_e7Hh2SS_44), [large scale projections](https://www.youtube.com/watch?v=EFWcAkxzkv4) and a [a bunch of other neat stuff](https://000000book.com/apps). The open repository [#000000book](https://000000book.com) allows users to share their work and even though the whole project have been inactive for some time, people still upload new tags every day.

### The Minitel
Because of its relative availability (at least in France) and very low-resolution screen, I figured using a [Minitel terminal](https://en.wikipedia.org/wiki/Minitel) to display street art would be an interesting project. There already are programs that display images on these devices (like [this one by phooky](https://github.com/phooky/Minitel)), but they usually draw rasterized images. I wanted the Minitel to animate the tag being drawn stroke by stroke. Which meant that I had to write my own code to convert a GML file into drawing instructions that the Minitel could recognize.

### How it works
#### The basic approach

I will use [this GML file](https://000000book.com/data/67744) as an example.

<img src="https://user-images.githubusercontent.com/100698182/163583321-e2f8f97d-d32a-4fc8-8f26-cf0dfb2618a0.jpg">

A GML file stores a graffiti as a list of strokes. Each stroke is a collection of points with X and Y coordinates ranging from 0 to 1 (there can also be a timestamp and a Z-coordinate, but these are optional and not used here).

<img src="https://user-images.githubusercontent.com/100698182/163583077-fa3062bf-a17c-49a6-bd12-91091cce9e32.gif" alt="a tag being drawned and an extract of the corresponding GML file" width="500">

The Minitel screen can display 24 rows of 40 characters each. By sending data through a serial connection we can move a cursor and draw a character anywhere on this 40x24 grid.
So drawing a graffiti on the Minitel screen would essentially consists of the following steps :

1. Extract the coordinates from the GML file
1. Scale each coordinate to match the size of the Minitel screen
1. Send commands through the serial port to move the cursor at the right place
1. Send a command to draw a point

Doing just that, this is what we get :

https://user-images.githubusercontent.com/100698182/163583419-3c04ec5b-00e2-4285-a443-6e207f16948b.mp4

This is not quite right. We can guess the general shape of the tag, but a lot of it is missing. This is because a GML file only stores samples of the points that make up a line, and it is our job to fill in the gaps. A simple way to do that is to use *linear interpolation* : drawing multiple evenly spaced points along a line that goes from one set of coordinate to the next. I arbitrily chose to add 10 points between each sample.

Implementing this gives us the following result :

https://user-images.githubusercontent.com/100698182/163583522-96039e39-f11e-49da-9c10-1cfca6dbb604.mp4

There it is ! Now we can see the tag being drawn line by line. We could stop here, but this 40x24 resolution really is not that great. Fortunately there is a way we can go beyond this limitation.

#### Increasing the resolution

The Minitel can display graphics at a higher resolution using special *block characters*. These are essentially blocks of 2x3 pixels that can either be colored or not. Since each space on the original 40x24 can display one of these 2x3 pixel block, we get a total resolution of 80x72 pixels. Here is a diagram that show every possible block character :

<img src="https://user-images.githubusercontent.com/100698182/163584757-5f029bf1-6c5b-4e39-8efd-a44795f75910.gif" width=600>

So our program needs to do the following:

1. Extract the coordinates from the GML file
1. Scale each coordinate to match a *80x72* screen
1. Interpolate between each successive point
1. **"Paint" the graffiti on a 80x72 pixel grid**
1. **Convert the painted pixels into 2x3 blocks**
1. Send commands to draw the corresponding block character at the correct position on the Minitel

The animation bellow shows how the grid can be converted into 2x3 blocks
<img src="https://user-images.githubusercontent.com/100698182/163583554-2d4e7b7e-fa67-4490-9dea-ffe5d55636fc.gif" width=600>

This is what we get after doing all of these steps :

https://user-images.githubusercontent.com/100698182/163584198-94aa98af-8186-4a04-811c-758f9f18b4f5.mp4

Success! For comparison, here are the results of the three versions side-by-side :

![side_by_side](https://user-images.githubusercontent.com/100698182/163584846-8ccd8d57-94c5-4803-8eec-ec4a35a877cb.jpg)

These are the basic workings of gml2mt. There is also some additional stuff (checking which way is up, manage overlapping strokes, etc), but for the most part, this is it.

### Possible improvements

- Use timestamps instead to order the strokes (right now we take each point in the order they appear in the file, but this might not be reliable)
- Add an option to keep the original aspect ratio (in this version the drawing is stretched to fill the screen)
- Use the timestamps to emulate brush dynamics (maybe varying the width and/or speed of the stroke)
- Add some effects (changing colors, dripping, transitions between tags...)

## License

The code for this project is licensed under the terms of the GNU GPLv3 license.
