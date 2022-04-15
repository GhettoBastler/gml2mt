# gml2mt

Draw GML files on a Minitel. Written in Python3

[title gif/video]

*[View tag on #000000book](https://000000book.com/data/67604)*

## Requirements

1. A Minitel 1B or 2 (with a DIN-5 port on the back)
1. Python 3.10+ with the pyserial library
1. A logic level converter for adapting the voltages between the Minitel and the computer (this [tutorial by Pila (in French)](https://pila.fr/wordpress/?p=361) explains how to build your own USB adapter)

*I personally use a Raspberry Pi as the computer. If you want to do the same, I have included the schematics for a level converter I made, which connects to the GPIO pins of the Pi.*

## Installation

Download gml2mt from this repository :
```
wget https://raw.githubusercontent.com/GhettoBastler/gml2mt/main/gml2mt
```

You will also need a GML file to display. You can download one from [#000000book.com](https://000000book.com/data)

## Usage

Connect the Minitel to your computer/Raspberry Pi using the DIN-5 connector on the back and power it on.

![A Raspberry Pi and a Minitel connected together with a level converter](https://github.com/GhettoBastler/gml2mt/raw/main/media/converter.jpg)

### On the Minitel
1. Switch to graphics mode by pressing **Fnct + T** followed by **V**
1. Change the baudrate to 4800 (the maximum) by pressing the **Fnct + P** followed by **4**.

### On the computer/Pi

Run gml2mt, passing the serial port to use and the GML file as arguments (if you are using a Raspberry Pi, the serial port should be /dev/ttyAMA0)

```
gml2mt.py SERIAL_PORT GML_FILE
```

## About this project

### What is GML ?
[GML](https://en.wikipedia.org/wiki/Graffiti_Markup_Language) (for Graffiti Markup Language) is a file format that allows graffiti artists to store tags as computer files. Because these files record the motion of a tag being drawned, they can then be reused in a variety of ways, like [tagging robots](https://www.youtube.com/watch?v=_e7Hh2SS_44), [large scale projections](https://www.youtube.com/watch?v=EFWcAkxzkv4) and a [a bunch of other neat stuff](https://000000book.com/apps). The open repository [#000000book](https://000000book.com) allows users to share their work and, even though the whole project have been inactive for some time, people still upload new tags every day.

### Why the Minitel ?
Because of its relative availability (at least in France) and very low-resolution screen, I figured using a [Minitel terminal](https://en.wikipedia.org/wiki/Minitel) to display street art would be an interesting project. There already are programs that display images on a Minitel (like [this one by phooky](https://github.com/phooky/Minitel)), but these usually draw rasterized images on the Minitel. I wanted something different : the Minitel should show the tag being drawn stroke by stroke. Which meant that I had to write my own code to convert a GML file into drawing instructions that the Minitel could recognize.

### How it works
#### The basic approach

A GML file stores a graffiti as a list of strokes. Each stroke is a collection of points with X and Y coordinates ranging from 0 to 1 (there can also be a timestamp and a Z-coordinate, but these are optional and not used here).

![a tag being drawned and an extract of the corresponding GML file](https://github.com/GhettoBastler/gml2mt/raw/main/media/boombap_with_code.gif)

The Minitel screen can display 24 rows of 40 characters each. By sending data through a serial connection we can move a cursor and draw a character anywhere on this 40x24 grid.
So drawing a graffiti on the Minitel screen would essentially consists of the following steps :

1. Extract the coordinates from the GML file
1. Scale each coordinate to match the size of the Minitel screen
1. Send commands through the serial port to move the cursor at the right place
1. Send a command to draw a point

We will test this with the tag from the title:
![tag that says boombap](https://github.com/GhettoBastler/gml2mt/raw/main/media/67744.jpg)

Doing just that, this is what we get :


This is not quite right. We can guess the general shape of the tag, but a lot of it is missing. This is because a GML file only stores samples of the points that make up a line, and it is our job to fill in the gaps. A simple way to do that is to use *linear interpolation* : drawing multiple evenly spaced points along a line that goes from one set of coordinate to the next. I arbitrily chose to add 10 points between each sample.

![illustration of linear interpolation](https://github.com/GhettoBastler/gml2mt/raw/main/media/interpolation.jpg)

Implementing this gives us the following result :

[Image test2]

And there it is ! Now we can see the tag being drawn line by line. We could stop here, but this 40x24 resolution really is not great. Fortunately there is a way we can go beyond this limitation.

#### Increasing the resolution

The Minitel can display graphics at a higher resolution using special *block characters*. These are essentially blocks of 2x3 pixels that can either be colored or not. Since each space on the original 40x24 can display one of these 2x3 pixel block, we get a total resolution of 80x72 pixels. Great!

![animation showing all the possible block characters](https://github.com/GhettoBastler/gml2mt/raw/main/media/blocks.gif)

What our program have to do now is follow our interpolated points, painting pixels along the way using a predefined "brush" (I used a 2x2 square). The corresponding pixels are then converted into 2x3 blocs to draw on the Minitel.

![animation showing all the steps from interpolated points to block characters](https://github.com/GhettoBastler/gml2mt/raw/main/media/pixels_to_blocks.gif)

Putting all these steps together, we get this :

1. Extract the coordinates from the GML file
1. Scale each coordinate to match a 80x72 screen
1. Interpolate between each successive point
1. "Paint" the graffiti on a 80x72 pixel grid
1. Convert the painted pixels into 2x3 blocks
1. Send commands to draw the corresponding block character at the correct position on the Minitel

Which gives us :

[Image test3]

Success! There is also some additional stuff that the program does (checking which way is up, manage overlapping strokes, etc), but this is basically it. There are however some ways this project could be improved.

### Possible improvements

- Use timestamps instead to order the strokes (right now we take each point in the order they appear in the file, but this might not be reliable)
- Add an option to keep the original aspect ratio (in this version the drawing is stretched to occupy all of the screen)
- Use the timestamps to emulate brush dynamics (maybe varying the width and/or speed of the stroke)
- Add some effects (changing colors, dripping, transitions between tags...)

## License

The code for this project is licensed under the terms of the GNU GPLv3 license.
