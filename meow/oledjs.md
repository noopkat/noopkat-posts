post about oledjs

http://www.garrettbartley.com/graphpaper.html

It all started with one of those projects you NEVER actually start on, let alone complete. You know the type I mean.

My nemesis in this scenario is to make my own watch. Not to sell, just to make one for myself. I've bought a lot of hardware for it, but every time I'm satisfied with the new approach, something else comes along that seems cooler or smaller, or easier. Check out this photo below of only part of the 'collection' I've acquired just trying to start on this thing:

![collection of bits](http://f.cl.ly/items/0K0j1s0m3k3X062y3C0g/watch-collection.png)

I am sure I'll start on it one day (hahahaha) BUT this nemesis project did lead me down a particularly enjoyable rabbit hole recently. 

There are three OLED screens in the picture above. These screens are pretty sweet. They run on minimal power, they're compact, and very bright despite the low power draw. Most of them use a common driver/controller, and I think they look really cool when used in wearables. The current end goal is to run one as a watch display, powered by an Arduino and a rechargable LiPo battery.

Normally when I want to play with a sweet new piece of Arduino compatible technology, I look up the class to use within the really cool hardware library [Johnny-Five](https://github.com/rwaldron/johnny-five). Johnny-Five is a NodeJS library which lets you write javascript to interface with Arduino and any hardware attached to it, such as sensors, screens, motors etc. It's really useful to create both prototypes and end products alike. So I jump into the johnny-five docs and look around only to discover that there's no class for playing with OLED screens. Bummer! A quick look through the issues confirms this, and so I'm thinking I'll have to just write Arduino code and do the sorta clumsy save -> compile -> upload workflow when developing for Arduino projects.

However I've been wanting to contribute to a cool project like Johnny-Five for a while now. It's been so cool to use this framework with a number of personal projects (not to mention Las Vegas Nodebots Day each year), that I wanted to give back as a way of saying thank you to [Rick Waldron](https://github.com/rwaldron) and the [other amazing authors](https://github.com/rwaldron/johnny-five/graphs/contributors) of Johnny-Five.

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

Above shows the bit values within our byte, when 3 pixels have been filled in. So what would our byte value be now? The answer is **00011101**. Notice anything? "It's written in reverse to what I was expecting!" you say. Well spotted! Why? We'll cover this in more depth later, but essentially the positions of our bits need to be counted from right to left.

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


