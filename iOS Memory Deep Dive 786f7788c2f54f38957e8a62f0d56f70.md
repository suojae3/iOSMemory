# iOS Memory Deep Dive

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled.png)

- So why reduce memory? The easy answer is users have a better experience. Not only will your app launch faster. The system will perform better.
- a memory page is given to you by the system, and it can hold multiple objects on the heap.
- And some objects can actually span multiple pages. They're typically 16K in size, and they can come in clean or dirty.

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%201.png)

- The memory use of your app is actually the number of pages multiplied by the page size.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%202.png)

- clean memory is data that can be paged out. Now, these are the memory-mapped files we just talked about. Could be images, data Blobs, training models. They can also be frameworks.
- So every framework has a DATA CONST section.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%203.png)

- Dirty memory is any memory that has been written to by your app.
- these can be objects, anything that has been malloced -- strings, arrays, et cetera.
- Frameworks have a data section and a data dirty section as well.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%204.png)

- Now, compressed memory is pretty cool. iOS doesn't have a traditional disk swap system.
- We get a memory warning, and we decide to remove all objects from our cache.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%205.png)

- Now, every app has a footprint limit.

<br/>

## Tools for profiling Footprint

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%206.png)

- it's a great way for quickly seeing the memory footprint of your app.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%207.png)

- Allocations profiles the heap allocations made by your app, and Leaks will check for memory leaks in a process over time.
- he talked about dirty and compressed memory. Well, the VM Tracker provides a great way to profile this.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%208.png)

- VM memory trace provides a deep view into the virtual memory system's performance with regards to your app.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%209.png)

- If you approach the memory limit of the device, you'll receive an EXC resource exception.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2010.png)

- The memory debugger for Xcode was shipped in Xcode 8, and it helps you track down object dependencies, cycles, and leaks.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2011.png)

- It prints details of the size in memory of the region, the amount of the region that's dirty, and the amount of memory that's swapped or compressed in iOS. And remember, it's this dirty and swap that's really important here.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2012.png)

- Another command-line utility that macOS developers might be familiar with already is leaks.
- It tracks objects in the heap that aren't rooted anywhere at runtime.
- So remember, if you see an object in leaks, it's dirty memory that you can never free.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2013.png)

- So when faced with a memory problem, which tool do you pick? Well, there are 3 ways to think about this.
- Do you want to see object creation? Do you want to see what references an object or address in memory? Or do you just want to see how large an instance is?
- If you just want to see what references an object in memory, you can use leaks and a bunch of options that it has in the page to help you there.
- finally, if you just want to see how large a region or an instance is, vmmap and heap are the go-to tools

<br/>

## Images

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2014.png)

- The most important thing about images to remember is that the memory use is related to the dimensions of the image, not its file size.
- As an example, I have this really beautiful picture that I want to use as a wallpaper for an iPad app.
- It measures 2048 by 1536, and the file on disk is 590 kilobytes. But how much memory does it use really? 10 megabytes.
- And the reason why is because multiplying the number of pixels wide by high, 2048 by 1536, by 4 bytes per pixel gets you about 10 megabytes.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2015.png)

- So why is it so much larger? Well, we have to talk about how images work on iOS. There's a load, a decode, and a render phase.
- We can go down to what we call the alpha 8 format. Now, alpha 8 just has 1 channel, 1 byte per pixel. Very small. It's 75% smaller than SRGB.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2016.png)

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2017.png)

- So what we really need to do is know how to pick the right format.
- The short answer is don't pick the format. Let the format pick you.
- If you migrate away from using the UIGraphics BeginImageContext WithOptions API, which has been in iOS since it began, and instead switch to the UIGraphics ImageRenderer format, you can save a lot of memory because the UIGraphics BeginImageContext WithOptions is always a 4-byte-per-pixel format. It's always sRGB.
- if you use the UIGraphics ImageRenderer API, which came in iOS 10, as of iOS 12, it will automatically pick the best graphics format for you.
- Just using the new API, I'm now getting a 1-byte-per-pixel image. This means that it's 75% less memory use. That's a great savings and the same fidelity.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2018.png)

- I can use this not just as a black circle, but as a blue circle, red circle, green circle with no additional memory cost. It's really cool.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2019.png)

- ImageIO can actually downsample the image, and it will use a streaming API such that you only pay the dirty memory cost of the resulting image. So this will save you a memory spike.

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2020.png)

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2021.png)

<br/>

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2022.png)

![Untitled](iOS%20Memory%20Deep%20Dive%20786f7788c2f54f38957e8a62f0d56f70/Untitled%2023.png)

- I have an image in an app, full screen. It's beautiful. I'm loving it. But then, I need to go to my Home screen to take care of a notification or go to a different app. But That image is still in memory.
- There are 2 ways to do this. The first is the app life cycle. So if you background your app or foreground it, the app life cycle events are great to, are a great way to know.

<br/>