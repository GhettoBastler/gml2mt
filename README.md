# gml2mt

Draw GML files on a Minitel. Written in Python3

https://user-images.githubusercontent.com/100698182/163582119-9fff3aa1-d630-42d3-9983-df86574c8849.mp4

*[View tag on #000000book](https://000000book.com/data/67604)*

Details about this project can be found on [my website](https://ghettobastler.com/gml2mt.html)

## Requirements

1. A Minitel 1B or 2 (with a DIN-5 port on the back)
1. Python 3.10+ with the pyserial library
1. A logic level converter for adapting the voltages between the Minitel and the computer (this [tutorial by Pila (in French)](https://pila.fr/wordpress/?p=361) explains how to build your own USB adapter)

## Installation

Clone this repository :
```
git clone https://github.com/GhettoBastler/gml2mt.git
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

## License

The code for this project is licensed under the terms of the GNU GPLv3 license.
