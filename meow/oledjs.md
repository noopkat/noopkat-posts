post about oledjs

http://www.garrettbartley.com/graphpaper.html

It all started with one of those projects you NEVER actually start on, let alone complete. You know the type I mean.

My nemesis in this scenario is to make my own watch. Not to sell, just to make one for myself. I've bought a lot of hardware for it, but every time I'm satisfied with the new approach, something else comes along that seems cooler or smaller, or easier. Check out this photo below of only part of the 'collection' I've acquired just trying to start on this thing:

![collection of bits](http://f.cl.ly/items/0K0j1s0m3k3X062y3C0g/watch-collection.png)

I am sure I'll start on it one day (hahahaha) BUT this nemesis project did lead me down a particularly enjoyable rabbit hole recently. 

There are three OLED screens in the picture above. These screens are pretty sweet. They run on minimal power, they're compact, and very bright despite the low power draw. Most of them use a common driver/controller, and I think they look really cool when used in wearables. The current end goal is to run one as a watch display, powered by an Arduino and a rechargable LiPo battery.

Normally when I want to play with a new piece of Arduino compatible technology, I look up the class to use within the really cool hardware library [Johnny-Five](https://github.com/rwaldron/johnny-five). Johnny-Five is a NodeJS library which lets you write javascript to interface with an Arduino and any hardware attached to it, such as sensors, screens, motors etc. It's really useful to create both prototypes and end products alike. So I jump into the johnny-five docs and look around only to discover that there's no class for playing with OLED screens. Bummer! A quick look through the issues confirms this, and so I'm thinking I'll have to just write Arduino code and do the sorta clumsy save -> compile -> upload workflow when developing for Arduino projects.

However I've been wanting to contribute to a large project like Johnny-Five for a while now. It's been so cool to use this framework with a number of personal projects (not to mention [Las Vegas Nodebots Day](http://nodebotsdaylv.com/) each year), that I wanted to see if I could help extend its use.

And so on and off for the past six weeks I've been attempting to code a drop in library for controlling OLED screens with JavaScript. What came out the other end was [oled-js](https://github.com/noopkat/oled-js).

This is my favourite project I've worked on so far. There were a few reasons for this:

+ I learned a lot of things from it
+ it was the first library I've ever written
+ I had to consider how others would use it, and what features they might want
+ it was the largest project to date that I've worked on
+ I had no idea what the heck I was doing
+ I felt super motivated and engrossed in it the whole time

Here is a photo of using oled-js to display a cat:

![cat oled](http://f.cl.ly/items/2G041X2C1o2A1n2D3S18/cat-oled.png)

Displaying text:

![text oled](http://f.cl.ly/items/0P050b1B3P463c2b0C2e/oled-text.png)

You can also:

+ draw lines
+ draw individual pixels
+ draw filled rectangles
+ smooth scroll the display
+ dim the display
+ invert the pixels on the display
+ clear the display
+ turn the display on and off

Oled-js supports both I2C and SPI protocols. I'd recommend I2C, as oled-js uses the Arduino Firmata library (via Johnny-Five) and there is no current support for fast SPI within Firmata. My library just flips pins over the USB cable connection to get SPI happening, which is a huge bottleneck to getting those pixels updating on the screen as fast as possible. See the [oled-js readme](https://github.com/noopkat/oled-js/blob/master/README.md) for more in context information on how to use each protocol.

If you have a need for it, have a play! If you break it, open an issue for me :)

### What did I learn?

Oh man, a lot of things. I've never really taken the opportunity to explore or teach myself some of the basic things that go on closer to the metal of how computers work in general. This project really got me started down the path, which while mysterious at times is completely fascinating. I've never had a reason to stumble into this stuff before (I did not study computer science, where it is often dropped on you) but now that I have I'd like to explore it even more.

But it's time I passed on this stuff to YOU! So below, is some of the things I now know after publishing oled-js.

### How do the pixels work on an OLED screen?

If you've ever worked with Photoshop or any other form of raster/pixel based image software before, this will be super familar to you. For starters, each screen has a set pixel dimension. For example, one screen I own is 128px x 32px in size, but you can buy other sizes too.

So let's look at a 48px x 24px screen. This means that there are 48 pixels stretching across horizontally, and 24 pixels spanning down. That's 1152 pixels available on the screen total. Each pixel can be referenced by an x and y coordinate, like a 2d vector. See the grid below for our 48 x 24 px screen:

![screen 1](http://cl.ly/image/0g3W2y3r1y2V/oled-screen01.png)

Pretty straightforward. Now, if you place some pixels on the screen, you can reference their locations pretty easily:

![screen 2](http://cl.ly/image/1M272I3w471K/oled-screen02.png)

The first pixel is located at x0, y0, or 0,0 for short. The second pixel is located at x7, y9, or 7,9 for short. Remember that we start counting across and down from 0, not 1.

In order to tell the screen what to display, we manipulate an object called a 'framebuffer'. This framebuffer is simply an array containing all of the pixel data. Each pixel is either a 0 or a 1. A 0 dictates that the pixel is 'off', and a 1 indicates that pixel should be 'on'. We send this framebuffer (either all of it at once or only part of it) as data to the OLED screen, which can then interpret it and display our pixels correctly on the screen. We prime the screen for the data with some other commands first, but that's not important to dive into right now.

Considering we already know that a 48 x 24 pixel screen contains 1152 pixels, we can expect that our framebuffer array would also have a length of 1152, right? Wrong! It's actually 144 in length instead. Wait, what? That makes no sense at all! Let me explain.

Our OLED screen accepts our framebuffer in the form of individual bytes. So, each item in our framebuffer array is a byte to be sent to the screen. A byte is simply a unit of digital information that computers understand. It is the smallest addressable unit of memory in most computers. That's pretty small! Working with bytes means that devices don't have to have massive amounts of memory installed in them, and it's more efficient to send information across a wire.

One byte contains 8 bits. Each bit is either a 0, or a 1. So a byte is binary! A visual example of a byte is 0010110.

Hang on just one tick, you say. If a byte contains 8 bits, and there are 144 bytes in our framebuffer, that would be **144 * 8**, which equals... **1152**! Shut the front door! Isn't that the *same* number as the total amount of pixels on our screen? Does this mean that one pixel is represented in our framebuffer as one bit? Yep, you got it. 

Therefore, there are 8 pixels of data in each byte of our framebuffer. And remember how we said that a pixel is either a 0 or a 1? So is a bit! This is a really effective way of storing our information.

But how does this work? How do you figure out which byte in our framebuffer contains our pixel? This is where 'pages' come in, but we'll cover that in a second.

A byte as we just discovered, has 8 pixels in the form of bits. A byte of pixels is painted on the screen in a downwards direction, taking up only one column. The following pixels highlighted below are a representation of the first byte in our framebuffer array:

![one byte](http://cl.ly/image/1g1e3M020W3F/oled-screen03.png)

The byte above has no pixels turned on in it. Therefore, the byte would read **00000000** in our framebuffer array. A shorthand way of writing this would be hex format **0x00**. 

But what if we put some pixels in there?

![screen 4](http://cl.ly/image/2Y0E420p0N0t/oled-screen04.png)

Above shows the bit values within our byte, when 4 pixels have been filled in. So what would our byte value be now? The answer is **00011101**. Notice anything? "It's written in reverse to what I was expecting!" you say. Well spotted! Why? We'll cover this in more depth later, but essentially the positions of our bits need to be counted from right to left.

To sum up: pixels are painted on the screen in sets of 8 bits, within a byte. Each byte is painted in the next column over from the last. 

Because a byte is reponsible for writing muliple pixels in a vertical fashion, a concept known a row or 'page' enters. Each page is 8 pixels high. Our first byte is in page 0. See the picture below to understand:

![screen 5](http://cl.ly/image/2t1D2l2S000v/oled-screen05.png)

In our example screen we're using, to write an entire row or 'page' of pixels will use 48 bytes. This is the same number as the width of our screen. The next row or 'page' will need the next 48 bytes in our framebuffer array to complete it, and finally the last set of 48 bytes will take care of the last row or 'page'.

I add another byte in the picture below, which is located in page 1:

![screen 6](http://cl.ly/image/2Q2A0m373W2V/oled-screen06.png)

What number byte in our framebuffer is this new byte I added? You can count from left to right in the diagram starting from byte 0. It is byte 55. 

To arrive at the same answer you can also assume 48 bytes in page 0, then start counting across from page 1. Add the result to 48 from there. Always subtract 1 of course, due to our index starting at 0.

Bonus point: what is the value of our new byte?  
.  
.  
.  

(the answer is **10111110**)

Now you understand the basics of an OLED display and how it interprets incoming data.

### A closer look at the framebuffer

Here is a 48 x 24 image we might want to display on the screen:

<img src="http://cl.ly/image/371h0G04192U/ok.png" style="border: 1px solid #bbb" alt="image with noop text written within it"/>

And here is how it is represented as bytes in the framebuffer:

```
[0xFF, 0xFF, 0xFF, 0x7F, 0x7F, 0xFF, 0xFF, 0x7F, 0x7F, 0x7F, 0xFF, 0xFF, 
0xFF, 0xFF, 0xFF, 0xFF, 0x7F, 0x7F, 0x7F, 0x7F, 0x7F, 0xFF, 0xFF, 0xFF, 
0xFF, 0xFF, 0xFF, 0x7F, 0x7F, 0x7F, 0x7F, 0x7F, 0xFF, 0xFF, 0xFF, 0xFF, 
0x7F, 0x7F, 0xFF, 0x7F, 0x7F, 0x7F, 0x7F, 0x7F, 0xFF, 0xFF, 0xFF, 0xFF, 
0xFF, 0xFF, 0xFF, 0x0, 0x0, 0x0, 0x1, 0xFE, 0xF8, 0x0, 0x1, 0x0, 0x10, 
0xEF, 0x2D, 0x7F, 0x1A, 0xF8, 0xFA, 0xF8, 0x88, 0x88, 0xB9, 0xFF, 0xE7, 
0x5, 0x27, 0x1A, 0xF8, 0xFA, 0xF8, 0x88, 0x88, 0x99, 0xAD, 0x5A, 0x5A, 
0xF0, 0xB0, 0xF2, 0x70, 0x3C, 0x20, 0x4, 0xC2, 0xC7, 0xFF, 0xFF, 0xFF, 
0xFF, 0xFF, 0xFC, 0xFC, 0xFC, 0xFC, 0xFF, 0xFF, 0xFC, 0xFC, 0xFC, 0xFE, 
0xFC, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 
0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xF5, 0xE5, 
0xED, 0xFC, 0xFC, 0xFE, 0xFE, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF]
```

We know a couple of things looking at this. One is that any byte with a value of **0xFF** indicates that it contains only white or 'on' pixels. This is because when you convert 0xFF hex to binary you get **11111111**. That's 8 'on' pixels, as we covered earlier.

We also know that any byte in there with a value of **0x0** or **0x00** contains only black or 'off' pixels. This is because when you convert **0x0** hex to binary you get, yep, you guessed it - **00000000**.

Let's convert another. Take **0x7F** for example. Converting that to binary equates to **01111111**. This byte contains 1 black or 'off' pixel, and the remainder are white or 'on'.

Easy peasy!

### Manipulating bits and bytes

How do we find out the bit values within a byte using JavaScript? 

Let's say we're trying to read just one bit within a byte. I do this in oled-js to look for a certain bit value in a response byte I can read from the screen itself. This specific bit acts as a flag to let me know if the chip is busy or ready for more data. 0 is ready, 1 is busy.

Firstly, we need to know the position of that bit (0-7). Let's say, we want to read the bit in position 5. Position 5 is the third spot from the far left bit in our byte (we covered this briefly earlier).

Now that we know the position, we need to apply a bitwise operation to find the value. Bitwise is a very broad and complicated topic, so we'll just stick to what's happening in this specific task only. I highly recommend reading the basics on [Wikipedia](http://en.wikipedia.org/wiki/Bitwise_operation), then following up with the [MDN explanation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) for the JavaScript implentation.

Bit shifting against a mask is the most common way to pull out a bit's value. Shifting a byte's bits right once looks like this:

![shift right](http://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Rotate_right_logically.svg/150px-Rotate_right_logically.svg.png)

As you can see above, a 0 is pegged onto position 7, every bit moves right 1 spot, and the bit in position 0 actually drops off and is discarded.

In this example, we're going to shift the bits in the byte right until bit 5 is sitting at position 0. You'll see why in the next step.

Let's work with this byte - **01100101**, or **0x65**. We'll shift the bits over 5 places so that the bit at position 5 is now sitting at position 0:

```javascript
var byte = 0x65;
var newByte = (byte >> 5); // 00000011
```

A total of 5 zeros will now be shifted in from the left, forcing the bits to the right. Bit 5's position is now position 0.

Now, we're going to use the '&' operator to compare this bit against the mask of 1. The '&' operator in bitwise will return a 1 if the bit in line with it is equal to 1. It will return a 0 if not equal to a 1. This may not make sense if you have not read the suggested resources mentioned earlier.

```javascript
var byte = 0x65;
var newByte = (byte >> 5); // 00000011
var busy = newByte & 1; // 1
```

The comparison can be represented visually. The top number is the mask (1 in binary), and the bottom value is our shifted byte.

```
  00000001
& 00000011
  --------
  00000001
```

As you can see, the result is equal to the binary representation of 1. So our busy byte is equal to 1 (the chip is busy!).



----
Lastly: thank you to [Rick Waldron](https://github.com/rwaldron) and the [other amazing authors](https://github.com/rwaldron/johnny-five/graphs/contributors) of Johnny-Five.