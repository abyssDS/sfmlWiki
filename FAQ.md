### Most FAQ entries have been moved to [the official SFML FAQ](http://www.sfml-dev.org/faq.php).

# Frequently asked questions (FAQ)

**[Building and Using SFML](#build-use)**
- [I want to fuse the libraries into one. (Not recommended)](#build-fuse)

**[SFML Audio](#audio)**
- [What audio formats does SFML support?](#audio-formats)
- [Why can't I hear any sound?](#audio-sound-problem)

**[SFML Networking](#network)**
- [How do I create &lt;insert popular application type here&gt;?](#network-create-network-app)
- [Should I use TCP or UDP sockets?](#network-tcp-vs-udp)
- [Should I use blocking or non-blocking sockets? And how do selectors work?](#network-blocking-non-blocking-selectors)
- [I can't connect to the other computer over the internet!](#network-internet-network)

**[SFML Window](#window)**
- [How do I make my window transparent?](#window-transparent-window)
- [What happened to getFrameTime()?](#window-get-frame-time)
- [Why doesn't setFramerateLimit() always set the frame rate to the specified limit?](#window-set-framerate-limit)

**[SFML System](#system)**
- [Does SFML support Unicode?](#system-unicode)
- [How do I convert from sf::String to &lt;type&gt; and vice-versa?](#system-string-convert)
- [My program keeps crashing when I use threads!](#system-threads-crash)
- [How do I use sf::Mutex?](#system-mutex)
- [Why can't I store my sf::Thread in an STL container?](#system-thread-container)
- [Why doesn't sf::sleep sleep for the amount of time I want it to?](#system-sleep)

**[Programming in General](#programming)**
- [What is RAII and why does it rock?](#prog-raii)
- [Why shouldn't I manually manage my memory?](#prog-mmm)
- [What are smart pointers?](#prog-smart)
- [Why shouldn't I use global variables?](#prog-global)
- [What should I use then instead of global variables?](#prog-insteadof-global)
- [Why is the singleton pattern not a good one?](#prog-singleton)

**[Troubleshooting](#trouble)**
- [General](#tr-grl)
 - [I'm having trouble using SFML.](#tr-grl-trouble)
 - [I keep getting "undefined reference to &lt;some strange thing that looks like an SFML function&gt;"!](#tr-grl-undefined-ref)
 - [Why can't I use SFML as a 64-bit library on my 64-bit system?](#tr-grl-64bit)
 - [My computer crashes when I run my SFML program!](#tr-grl-crash)
 - [I found a bug!](#i-found-a-bug)
 - [What is minimal code?](#tr-grl-minimal)
 - [And how can I easily obtain this minimal code?](#tr-grl-obtain-minimal)
- [Code::Blocks](#tr-cb)
 - [I'm getting linker errors.](#tr-cb-linker)
- [Visual Studio](#tr-vs)
 - [My project crashes randomly, but I don't get any compiler or linker errors.](#tr-vs-crash)
 - [I keep getting ```fatal error LNK1112: module machine type 'XYZ' conflicts with target machine type 'ZYX'```. Help!](#tr-vs-arch)
- [Windows](#tr-win)
 - [Why does a console attach itself to my project?](#tr-win-console)
- [Linux](#tr-lnx)
 - [(Debian) I can't compile the source code.](#tr-lnx-compile)
 - [There is no titlebar visible and/or artifacts from windows are visible.](#tr-lnx-titlebar)

**[Licensing](#licensing)**
- [What license does SFML have?](#lic-license)
- [Can I use SFML in commercial applications?](#lic-commercial)
- [Can I link SFML statically?](#lic-static)
- [Can I use the code from the example directory?](#lic-examples)
- [Do I have to pay any license fees or royalties?](#lic-pay)

**[Libraries for SFML](#libraries)**
- [Does SFML have a GUI package?](#libraries-gui-package)
- [Can you interface SFML with a GUI library?](#libraries-gui-lib)
- [Can I read video files with SFML?](#libraries-video)
- [What exactly is Thor?](#libraries-thor)

**[Miscellaneous](#misc)**
- [Are there any famous projects with SFML?](#misc-projects)
- [How can I distribute my game?](#misc-distr)
- [Where can I upload my game to?](#misc-upload)
- [Are there any example projects I can learn from?](#misc-examples)

---

## <a name="build-use"/>Building and Using SFML

### <a name="build-fuse"/>I want to fuse the libraries into one. (Not recommended)

To fuse two libraries, you can use the ar.exe utility provided with MinGW. You'll also need a minimal Unix environment (like [CYGWIN](http://www.cygwin.com/)). The syntax is:

    ar xv lib1.a | cut -f3 -d ' ' | xargs ar rvs lib2.a

Here are the commands to together the external dependencies:

    ar xv libgdi32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s-d.a && rm *.o && echo 'done'
    ar xv libgdi32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s.a && rm *.o && echo 'done'
    ar xv libopengl32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s-d.a && rm *.o && echo 'done'
    ar xv libopengl32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s.a && rm *.o && echo 'done'
    ar xv libwinmm.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s-d.a && rm *.o && echo 'done'
    ar xv libwinmm.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s.a && rm *.o && echo 'done'
    ar xv libws2_32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-network-s-d.a && rm *.o && echo 'done'
    ar xv libws2_32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-network-s.a && rm *.o && echo 'done'
    ar xv libfreetype.a | cut -f3 -d ' ' | xargs ar rvs libsfml-graphics-s-d.a && rm *.o && echo 'done'
    ar xv libfreetype.a | cut -f3 -d ' ' | xargs ar rvs libsfml-graphics-s.a && rm *.o && echo 'done'
    ar xv libopenal32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s-d.a && rm *.o && echo 'done'
    ar xv libopenal32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s.a && rm *.o && echo 'done'
    ar xv libsndfile.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s-d.a && rm *.o && echo 'done'
    ar xv libsndfile.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s.a && rm *.o && echo 'done'

## <a name="audio"/>SFML Audio

### <a name="audio-formats"/>What audio formats does SFML support?

In addition to the formats supported by [libsndfile](http://www.mega-nerd.com/libsndfile/#Features) (wav, flac, aiff, au, raw, paf, svx, nist, voc, ircam, w64, mat4, mat5 pvf, htk, sds, avr, sd2, caf, wve, mpc2k, rf64) the Audio module is also capable of playing ogg files.
Unfortunatly MP3 is covered by a license from Thompson Multimedia and thus support for it is not included in SFML. For more information regarding the MP3 license, see http://www.mp3licensing.com.

### <a name="audio-sound-problem"/>Why can't I hear any sound?

If everything compiles and seems to work correctly, but yet no sound is coming out of your speakers you should check the obvious. Ensure your speakers and plugged in and working correctly before assuming something is wrong with SFML. You can do this by opening one of your audio assets in another audio player such as Windows Media Player or [VLC](http://www.videolan.org/). If audio fails to play correctly there, then check that your PC audio is not muted and that the volume control on your speakers is turned up. Once it plays correctly in an external player then the problem may be with SFML.

## <a name="network"/>SFML Networking

### <a name="network-create-network-app"/>How do I create &lt;insert popular application type here&gt;?

The first thing you need to do is understand how the underlying networking of said application works. Behind every complex looking application there is a simple system of how systems send data to each other and what kind of data they send.

There are 2 kinds of topologies:

* Client-Server
* Client-Client (Peer-to-Peer)

Client-Server is generally easier to set up and often achieves higher performance than Client-Client due to the fact that dedicated servers are already configured to handle a large amount of traffic from multiple connected systems. When running a server application you can also be sure that clients can not manipulate the game state (cheat) themselves unless they have direct access to the server and can execute things there (which requires logging in etc.).

When running in a Client-Client configuration, the first thing to overcome is the initial connection establishment. Home/Office gateways/routers are mostly configured by default to not accept any incoming connection requests. As such none of the sides can establish a connection to the other. There is a way of overcoming this called [NAT hole punching](http://en.wikipedia.org/wiki/NAT_hole_punching), however this is beyond the scope of this FAQ. Care has to be taken to ensure that nobody is able to cheat in this topology. This is usually done by mirroring the game state across all involved systems and performing checks and synchronization at every game step.

Once you have picked a suitable topology for your application, you can start to think about what kind of data you want to send between the systems. There is no general answer or recommendation for this as this is very application specific and you will have to rely on good judgement to make the right choices.

### <a name="network-tcp-vs-udp"/>Should I use TCP or UDP sockets?

One must first understand what the difference is between TCP and UDP.

TCP is connection based and provides **reliable and ordered delivery of data** from one endpoint to the other. Connection based means you need to establish a TCP connection before you can make use of it for data transfer. It takes care of everything you can possibly think of (and might not think of) except creating and using the data that it sends and receives, that is still your task. Simply put, when you have an established TCP connection, whatever you shove into one end will in almost all cases come out the other end, in the right order, even over very very bad internet connections. If it doesn't arrive at the other end, both sides will be notified of the error and can perform the necessary error recovery. This comes at a price however, TCP packet headers are **at least 20 bytes and at most 60 bytes** large. This means that if you send a packet containing 1 byte over a TCP connection, your network adapter will need to transmit at least 21 bytes to get the data to the other end (disregarding the overhead from the lower OSI layers which is always present both in TCP and UDP). Also note that TCP streams are bi-directional, meaning you can use them to send and receive data in both directions, there is no need to create 2 separate TCP connections to transfer data between 2 endpoints (unless the data streams have different purposes and you want to separate them at the transport layer).

UDP provides **unreliable delivery of data** from one endpoint to the other. It is not connection based and **does not guarantee anything**. This means that if you are unlucky, you might not receive what you send, what you send might be received in a different order than the order you sent in or you might even receive duplicates of what you sent. In case of a transfer failure UDP also won't notify you that anything happened, it is completely up to you to take care of detecting and handling all these cases. The advantage of UDP is that its **header size is always 8 bytes (40% of TCP)** so if you send very small packets very often, you can save a lot of bandwidth if you use UDP. At larger packet sizes the header overhead is amortized and both protocols are just as efficient bandwidth-wise as the other. In the case of UDP, because there is no concept of a connection, it is your responsibility to keep track of who is sending and receiving what data because in general a server will only bind a single UDP socket with which all clients will send data to (unless they open up a new socket per client). You can also send data to multiple clients over one UDP socket because you always have to specify the destination of a datagram.

You might be wondering: "Woah! Who would even use UDP instead of TCP? There are so many disadvantages."

Well... If you implement your own reliable transport protocol on top of UDP, you can consider it as a form of "fine-tuned TCP" that does exactly what you want and no more. You could reduce the communication overhead considerably and even have the advantage that UDP traffic generally gets processed faster when travelling over the internet, which means lower latency. This is the reason why many latency critical applications choose to use UDP instead of TCP.

If you are developing your first networked application, stick with TCP as long as you can. Bandwidth and latency issues will not be your biggest concern until you are sure you can make money off it. Once that time comes, you will have gained so much experience that the decision will be trivial.

### <a name="network-blocking-non-blocking-selectors"/>Should I use blocking or non-blocking sockets? And how do selectors work?

It depends on what kind of an application you are developing and what your preference is. If you are just beginning to learn network programming, you can use blocking sockets when writing your first echo server (a server that waits for data to be received and instantly sends back a response). In this case the server does not do anything else but reply to incoming data and as such can block as much as it wants.

If, however, the server has other duties as well, such as updating an internal state every frame, blocking the state update thread must be avoided at all costs. This means, either you block in a separate thread, you call blocking operations **when you know they should not block** using selectors or you just use non-blocking sockets.

Because creating a separate thread for each blocking socket can result in a large amount of threads created, this method is not recommended unless you are sure that the number of expected connections stays manageable.

Using non-blocking sockets might seem the easiest at first, but one must consider that every time you poll the socket for new data, you are making numerous operating system calls which means a lot of overhead just to determine if the socket could be read or not. Even at a moderate number of sockets, this can become quite expensive. This is why selectors exist.

To solve the previous problem, operating systems provide methods of polling a large number of resources for their readiness simultaneously or at least in a very efficient manner. The principle idea is that you tell the operating system which resources you want to monitor and it sets the respective field within the selector when that resource becomes ready. That way you only have to check if the field is set, and the operating system only sets the field when something really does happen thus resulting in a very efficient way of checking for readiness. The `sf::SocketSelector` in SFML wraps all this functionality. You can ask the selector if a socket is ready and it will perform all the low level operations for you.

The most universal choice is using selectors and blocking sockets, as they are suitable for any scenario with little to no drawbacks. Many high performance applications still use selectors nowadays although there newer ways are constantly being developed to do the same which are just a bit more efficient.

### <a name="network-internet-network"/>I can't connect to the other computer over the internet!

This is probably one of the hardest "problems" to solve, as the number of error sources is quite high (up to the point where you aren't even responsible for the error yourself).

Since there is no general solution to this problem, here is a list of things you can check:

1. Make sure that it isn't just a programming error. Read the documentation and assure that what you are trying to achieve is reflected in your code.

2. Make sure that the addresses you use are correct. There are 3 ways you can identify an endpoint.
Obviously if you want to address an endpoint that isn't part of your local network you can not use the first option.
    * By private address e.g. 192.168.1.1 (192.168.*.*, 10.*.*.* and 172.16.*.* to 172.31.*.* are all private networks)
    * By public address e.g. 123.123.123.123
    * By FQDN (Fully-Qualified Domain Name) e.g. www.sfml-dev.org (www is the _hostname_ and sfml-dev.org is the _domain name_)

3. Make sure that data transmission is not hindered by anything in the networking infrastructure (routers, firewalls etc.), if you are not sure about this, it most likely means that the port you are trying to use is either closed or not configured to be forwarded behind a NAT.

4. Make sure that data is really being sent and received by the hosts independent of your application. It might occur that you try to send data within your application, SFML doesn't report an error, but the operating system refuses to transmit it. To check if this problem exists, it is recommended that you install some form of packet capturing software such as [Wireshark](http://www.wireshark.org) on both systems.

5. Make sure that the data leaves the local network over the router. There is a possibility that the router blocks outgoing data as well.

6. Make sure that the data arrives at the destination network router and is properly forwarded. If you are sure that the data leaves the origin network but never arrives at the destination network, try using a different port. Some ISPs have policies that block traffic from certain software and because they are not interested in using a better filtering mechanism, they decide to block the whole port instead of only traffic that really stems from the specific software. If you happen to use one of those ports, you are unlucky and should just try another.

7. If you are sure that the port you use isn't blocked, in very very rare cases it might be an error somewhere on the way from the source the the destination within some ISPs network. If this is really the case, you are as good as out of luck and should just try again at another time or be prepared to make a lot of phone calls with a lot of uninterested people.

8. If the ISP assures you that there is nothing wrong with their network, you must always take this with a pinch of salt because unless they contractually bind themselves to what they say, it might have been just to make you end the call.

9. When all else fails, you can always come to the SFML forum and ask for help there.

## <a name="window"/>SFML Window

### <a name="window-transparent-window"/>How do I make my window transparent?

Unfortunately SFML can't help you with this. The style and representation of the application window within your window manager/desktop environment is specific to the environment you are currently in. Because of this SFML cannot provide a uniform interface for controlling the representation of your application window.

SFML does however provide sf::Window::getSystemHandle(). Using the handle you can do a bit of research and find out how to manipulate the window representation yourself using the functions of your window manager.

### <a name="window-get-frame-time"/>What happened to getFrameTime()?

getFrameTime() was removed from SFML at the beginning of 2012. The reasoning for it can be found here: http://en.sfml-dev.org/forums/index.php?topic=6831.0

Users have to create an sf::Clock object now and keep time themselves. This has more advantages than disadvantages including:

* Correct time reporting (getFrameTime() reported the time spent **during the last frame**)
* More control over between which points in your code the time is to be measured
* More control over the precision required

### <a name="window-set-framerate-limit"/>Why doesn't setFramerateLimit() always set the frame rate to the specified limit?

setFramerateLimit() is implemented using a call to sf::sleep every frame. The intricacies of sf::sleep are explained [here](#system-sleep).

## <a name="system"/>SFML System

### <a name="system-unicode"/>Does SFML support Unicode?

SFML supports the input and display of international characters, via the UTF-16 encoding. Input is provided via sf::Event::TextEntered as sf::String and can be display with sf::Text.

### <a name="system-string-convert"/>How do I convert from sf::String to &lt;type&gt; and vice-versa?

For conversions to `sf::String`, you can just construct the `sf::String` directly from whatever object you already have. 
```cpp
sf::String sfml_string( cpp_string );
```
There are enough constructors that take care of implicit conversion from all standard C++ string types. If you want to see what these look like, take a look in the [`sf::String` documentation](http://sfml-dev.org/documentation/2.0/classsf_1_1String.php). If you want to convert from a non-C++ string to `sf::String`, it is recommended to first convert to a C++ string and then to an `sf::String`. Since any library using custom string types should provide support for this, this shouldn't be problematic.

For conversions from `sf::String` to any other custom string type, it is also recommended to first convert to a C++ string then from C++ string to that type.

`sf::String` supports implicit conversion to `std::string` and `std::wstring`, so things like
```cpp
std::cout << sfml_string << std::endl;
cpp_string += sfml_string;
std::size_t pos = cpp_string.find( sfml_string );
```
are all valid. Additionally, if implicit conversion isn't something that comforts you, you can explicitly call `.toAnsiString()` or `.toWideString()` to convert to the corresponding types.

Because internally `sf::String` stores its data in a `std::basic_string<sf::Uint32>`, conversions to this type (which is a C++ string type) are lossless. Ironically however, conversion to this type is not supported directly by `sf::String` and one has to perform it via iterators as such:
```cpp
std::basic_string<sf::Uint32> basic_uint32_string( sfml_string.begin(), sfml_string.end() );
```
Once you have a `std::basic_string<sf::Uint32>` you can use all of the same functionality as `std::string` since `std::string` is just a typedef for `std::basic_string<char>`.

### <a name="system-threads-crash"/>My program keeps crashing when I use threads!

Threading is a very advanced concept, and not something you should try merely because you heard it _can_ increase performance. In fact, if used improperly, _it even decreases performance_! You should always be able to argue in favour of using threads before even writing your first line of threaded code. __If you don't fully understand why your application is going to run faster with threads, then just don't use them.__

When you are sure you will benefit from using threads, you will have to be more careful with how you access memory. Almost all crashes when using threads are attributed to wrong memory access patterns. What you want to avoid are:

* Multiple concurrent reads and at least one write to the same memory location
* Multiple concurrent writes to the same memory location

Concurrent reads to the same memory location are generally unproblematic.

In order to protect against the above scenarios, you will need to make use of mutex objects. SFML provides a cross-platform mutex implementation. See [below](#system-mutex) for how to use them.

Even after you have protected against concurrent access, you still need to be wary of the order in which statements are executed. Once you venture into threading, there is no longer an execution ordering guarantee, and it is ultimately up to you to make sure things are done in the right order across different threads.

Good system designs often make threading optional and provide an option to disable them, whether for debugging or other purposes.

### <a name="system-mutex"/>How do I use sf::Mutex?

sf::Mutex is used to lock (acquire) a resource for exclusive access and unlock (release) a resource when exclusive access is no longer necessary. If you try to lock a mutex that has already been locked by another thread, you will have no choice but to wait for the locking thread to release the lock in order for execution to proceed.

It is good practice to not lock/unlock an sf::Mutex directly, but to rely on RAII sf::Lock objects to automatically unlock their owned mutex on destruction.

For more information on sf::Mutex and sf::Lock, refer to the [official documentation](http://sfml-dev.org/tutorials/2.1/system-thread.php#protecting-shared-data).

### <a name="system-thread-container"/>Why can't I store my sf::Thread in an STL container?

sf::Thread inherits from sf::NonCopyable meaning you cannot copy or assign an sf::Thread. This is however a requirement to use a data type with an STL container.

You might be wondering how it would still be possible to store multiple threads in an STL container if you are trying to implement some sort of thread pool. The answer is simple: instead of storing instances, store pointers to the sf::Threads.

The sf::NonCopyable restrictions can only apply to instances of an sf::Thread. When copying pointers, the copy constructor or assignment operator of a class are not invoked. As such it is perfectly legal to copy and pass pointers around.

Since working with raw pointers is something you want to avoid in modern C++, you can use [smart pointers](#prog-smart) to great extent in combination with this technique.

### <a name="system-sleep"/>Why doesn't sf::sleep sleep for the amount of time I want it to?

When calling sf::sleep you are requesting that the operating system halts execution of the current thread and yields the time slot allocated to it by the task scheduler to another task, a task being anything requiring execution time, from processes to threads and fibers if they are supported.

Because the CPU doesn't actually sleep in a multi-tasking environment, this has to be realized by an entry into an operating system specific lookup table. This lookup table informs the scheduler to skip the sleeping thread when selecting the next task to execute for a given time slice. Depending on the implementation of the operating system's sleep facility, keeping track of how long the thread has slept for and when to wake it up again is performed differently.

There are multiple possible reasons sf::sleep might return prematurely, and even overdue. It is entirely dependant on what the operating system chooses to do and a bit of luck.

As described above, the operating system has to keep track of the amount of time slept and when to wake the thread up. Because high resolution timers might not be available on all systems, the default resolution may be set very low. If as an example, the system timers have a resolution of 10ms and you request to sleep for 11ms, the operating system will be forced to round up in most cases and make your thread sleep for 20ms instead because it is a multiple of the base resolution. In this case sf::sleep sleeps longer than it should.

If however, you requested to sleep for 5ms and that request was made right before the internal operating system timer ticks, it will count as "10ms has passed" to the operating system and it will wake your thread up again although the 5ms have not fully passed. In this case, sf::sleep would sleep for shorter than it should.

In recent revisions of SFML, the timer resolution is temporarily increased during a call to sf::sleep to increase the likelihood of your sf::sleep call sleeping for the correct amount of time.

One thing to remember is that although the operating system marks your thread as "awake" after it is done sleeping, even for exactly the correct duration, it doesn't mean it resumes execution immediately. It could have just missed the moment at which the scheduler selects which task to execute next and thus must wait for the next transition. In this case, although your thread slept for the correct amount, it will appear to you as if it slept for more. SFML doesn't allow you to sleep for 0 duration, however if you could, you would notice that it in fact takes a slight bit of time as well. This is because it is common for specifying 0 to the operating system to translate to simply yielding your execution time slice to another thread.

In the end, what this means is that sf::sleep is merely a guideline, and not a contract. The longer you sleep for, the more accurate it will be. The shorter you sleep for, the less accurate it will be and to a certain extent more dependant on luck it will become.

## <a name="programming"/>Programming in General

### <a name="prog-raii"/>What is RAII and why does it rock?

RAII is an acronym for **R**esource **A**cquisition **I**s **I**nitialization. It is a programming idiom/technique that is used extensively in C++ and can be used in other programming languages as well.

The origin of this idiom lies in the way of how exceptions work in C++. When an exception is thrown in a code block, execution is diverted to the relevant handlers and the program flow continues on from there. If the programmer has to rely on ALL his code in a block being executed to perform the necessary clean up of resources, this can be a problem in the case an exception is thrown because it is not guaranteed that the clean up code will be executed and resources will not be destroyed properly.

To address this issue, RAII takes care of the fact that although not all code is executed when an exception is thrown, the destructors of an object are guaranteed to be called because all objects have to be destroyed when the corresponding scope is left.

This can be used in any resource owning object or in places where certain functions have to be called to make sure the program stays in a clean running state. A good example of this is an sf::Texture. No matter what happens, if the sf::Texture object gets destroyed because it goes out of scope, the underlying OpenGL resources also get freed.

From the example just mentioned, it should be apparent that the same can be said of dynamically allocated memory. It is also a resource that we need to free when we are done with it. Because of this, to promote RAII in modern C++, smart pointer facilities have made their way into the language, either as an extension or in C++11 as a part of the STL. Any memory governed by them is guaranteed to be released when they go out of scope (in the case of non-shared ownership e.g. scoped_ptr) due to RAII. This makes it much easier to use dynamically allocated objects, as you do not have to worry too much about memory management anymore.

### <a name="prog-mmm"/>Why shouldn't I manually manage my memory?

Nobody forces you to use automatic memory management, however in practice, the larger and more complex a project gets, the harder it is to keep track of such things as well. Generally it is a good idea to use automatic memory management because it frees you to dedicate more time to more important parts of your project. Not only that, it will be easier to debug and leaks will be nearly impossible.

C++ has come a long way and in its latest incarnation, C++11, many new memory management facilities made it into the C++ STL. This lets you use these constructs without the need for external or self-written libraries.

### <a name="prog-smart"/>What are smart pointers?

Smart pointers, as their name implies are pointers that are... well... smart :). Regular "raw pointers" are just values, merely an address in memory where an object is located. Unless you do something with those values, nothing is going to happen with the object, and it will sit there until you choose to free the corresponding memory block or close the program and destroy the virtual memory pages associated with it.

Smart pointers on the other hand, do things with the values that they contain. They are no longer just values but objects that govern the memory they point to. This can help you to track how many pointers exist that point to a certain block of memory (shared_ptr) or to prevent you from having multiple pointers point at the same block of memory at the same time (unique_ptr). The best part is, if you let these smart pointers manage your memory for you, they will also take care of freeing it when they think it isn't used any more.

### <a name="prog-global"/>Why shouldn't I use global variables?

Usage of global variables is considered as bad programming practice. They might seem like an easy solution to your initial problem but they will become a headache later on when the project gets bigger or you are unaware of the implications of declaring something in global scope.

One of the most dangerous things of declaring non-POD ([plain old data](http://en.wikipedia.org/wiki/Plain_old_data_structure)) objects in global scope is that you can never be sure when they are actually constructed and when they will be destroyed. This means that if they need to own resources you need to make sure they are available before the object is created, which can be tricky to do if that takes place before your main() code gets executed. Analogous to that, the object might get destroyed after your main() returns thus leaving resource destruction up to some self-clean-up mechanism or in the worst case to a resource manager that already got destroyed before main() returned. This leads to leaks and is very bad practice. Furthermore the initialization order and destruction order is not well-defined. It is only defined _within one translation unit_ as being dependant on the order of declaration, however you can't count on global variables from different translation units being constructed or destroyed in a specific order, it is pure luck here.

Another problem with global variables is that sooner or later you are going to have so many of them that they will clog up your namespace. Unless you declare them in a separate namespace they will all be in the same giant one: the global one. If you happen to declare a local variable in one of your functions that happens to have the same name as the global one you are actually referring to, you will not notice the global variable get shadowed unless you have certain warnings switched on. Some people suggest using Hungarian notation to solve this problem but the modern demeanour tends to avoid Hungarian notation as well.

Furthermore, global variables work against code modularity. Global variables can be accessed from anywhere and thus bypass well-defined interfaces between modules. This introduces hidden dependencies in the code, which is not only an additional maintenance burden, but can lead to very difficult-to-track bugs. Simply because you are not able to control the access to global variables, as they can be changed anywhere, at any time.

As if this were not enough, global variables also play very badly in multi-threaded environments. Access to global variables from different threads must be protected by mutexes. This requires additional care by the developer accessing the variable and often leads much more synchronization overhead than necessary, because variables are protected prematurely. On the other hand, unprotected global variables can silently introduce bugs if an application starts to use multiple threads.

There really isn't any reason to use global variables. Code using global variables can always be written without them and will never perform worse, most of the time performing better as a result of better memory usage. Keep in mind that the same reasoning applies to static variables, the only difference is their visibility (class/function scope) and in the case of function-static variables, their construction time (at first call instead of program start). Don't use `static` to "optimize" the construction of function-local variables.

### <a name="prog-insteadof-global"/>What should I use then instead of global variables?

You should try to confine variables to the scope(s) where they are used. Construct and hold them in the object which is supposed to own them as well as manage their lifetime and pass them to other objects/functions using references/pointers. Avoid passing by value. Often you cannot copy objects and are thus not allowed to pass by value, other times copying the object can take much more time because temporary memory has to be allocated just for the call and freed after the function returns.

If you happen to use a C++11 compliant compiler then you can be sure that many Standard Library objects you pass around will have move constructors defined which makes it less painful to "pass by value" since if certain conditions are met, the compiler will use the Rvalue reference version of certain functions to speed up execution by a substantial amount.

### <a name="prog-singleton"/>Why is the singleton pattern not a good one?

In short, singleton classes are global variables, they just hide it better. As a result, they share almost all of the related problems (construction/destruction time, implicit dependencies, multithreading). The fact that singletons are referred to as an OOP design pattern makes people think "it's OOP, so it must be good", which is not only a generally questionable conclusion, but particularly in this case. Having a class around it doesn't make code object-oriented; on the contrary, core OOP principles such as modularity or encapsulation are broken in the case of singletons.

A frequent misconception is the idea that things that are only instantiated once should become singletons. The purpose is to technically enforce that no two instances of a class can coexist. While this on its own sounds reasonable, the singleton technique mixes the solution to this problem with an unrelated aspect: providing a global access point for that one and only instance. And this often introduces far more problems than it promises to solve.

There are indeed use cases for classes of which only one object should exist, e.g. management-related tasks like rendering, resource handling, configuration, etc. As simple as it may sound, the most straightforward way to have only one instance at runtime is to create only one. The problematic of accidentally creating more is largely overstated, there is usually a clear place where instantiation should happen. And even if that problem should pose a serious threat, it can be trivially mitigated through assertions. That alone is rarely a good reason to pay the high price of having global variables.

## <a name="trouble"/>Troubleshooting

### <a name="tr-grl"/>General

### <a name="tr-grl-trouble"/>I'm having trouble using SFML.

First, make sure that you have followed the installation instructions in the [official tutorials](http://www.sfml-dev.org/tutorials/).

Have you:

* Provided the path to the SFML headers to your compiler?
* Provided your text editor with the path to the SFML library?
* Included the headers for the packages you're using? (“SFML/[capitalized name of module].hpp”)
* Linked with the packages you're using? (See the dependencies section of this document)
* On Windows, have you copied the libsndfile-1.dll and openal32.dll files (you can find them in the complete SDK) into the folder for executable, along with the DLLs for the packages you're using (and all of their dependencies)?
* On Linux, have you installed the libraries (sudo make install in the SFML folder)?

If you've checked all of those, and SFML still refuses to work, see [I found a bug!](#grl-bug).

### <a name="tr-grl-undefined-ref"/>I keep getting "undefined reference to &lt;some strange thing that looks like an SFML function&gt;" errors!

See [What and how do I link to use SFML?](#build-link)

### <a name="tr-grl-64bit"/>Why can't I use SFML as a 64-bit library on my 64-bit system?

First of all, you have to ask yourself: Do I really need to use SFML as a 64-bit library? There are some benefits to building 64-bit applications, but it is recommended that beginners do not try this until they are confident with the compile and linking process.

Most of the confusion stems from the fact that with Windows, although modern versions make use of 64-bit processor instruction sets, they are able to run 32-bit applications as well. By default, most "standard" installations build 32-bit executables. The fact that you are running a 64-bit operating system doesn't mean that you automatically build 64-bit executables. When in doubt, download/build 32-bit executables.

Let's start with the basics. The processors that you use everyday execute instructions which is what programs are essentially made of. The set of instructions that a processor understands and is able to execute properly is known as its _instruction set_. Because Intel released a family of very popular processors quite a while ago that supported at first 16-bit registers and eventually 32-bit registers whose model numbers ended with 86, the instruction set that they provided came to be known as the x86 instruction set. Nowadays, x86 is synonymous with 32-bit, and any time you see an architecture marked as x86 you should immediately tell that it is a 32-bit architecture. Because of how addresses have to be stored in registers, eventually 32-bit registers were no longer sufficient (could only address up to 4GiB of RAM) and processors had to start providing 64-bit registers. This marked the start of the x64 era. This time around, AMD was the first to come up with the instruction set for 64-bit architectures, and thus you will hear of the term AMD64 a lot. x64, x86-64 and AMD64 all tend to refer to 64-bit architectures and are often used interchangeably.

When you compile your code, the compiler ends up translating your syntactically legal code into machine instructions to be executed on the _target CPU_. The compiler will always produce code that can run on a _specific architecture_. Because x64 architectures are fully backwards-compatible with x86 instructions (it is merely a superset), they can run executables compiled for x86 systems as well, however, the operating system kernel has to support this and allow it.

Some systems such as Linux and OS X choose to trade executable portability for performance. You will not be able to transfer binary files between systems of different architectures, but for that they will be optimized more for their target architecture. Saying that you want to "compile for 64-bit" on these systems doesn't make much sense since you will have to do so automatically anyway. There are ways of forcing compilation for 32-bit systems in order to build _somewhat_ portable binaries, but that is beyond the scope of this FAQ. Just understand that on these systems, you will almost never have to worry about choosing an architecture type.

Windows, as mentioned earlier, chose to go down a different route. They favoured portability over potential performance gains. Because Microsoft couldn't easily stop supporting legacy applications (most of them being proprietary and barely maintained) on their newer operating systems, they had to make it possible to run 32-bit applications side by side with native 64-bit applications. This technology is called _Windows on Windows_ or _Windows 32-bit on Windows 64-bit_, WoW64 in short. Every 64-bit Windows installation has a second copy of the operating system files installed as well, the 32-bit version. When running programs, the executable loader checks for the executable's architecture and loads the corresponding dynamic libraries that it needs to function. Because this information is embedded into the executable itself, it is hard to tell what architecture an executable was built for without inspecting it closer. It is perfectly normal to build for 32-bit Windows systems even though you know that all your users will be on 64-bit systems, it will simply work.

This is one of the main reasons why confusion arises when building software using libraries as well. Unless the target architecture is explicitly noted in the file name, you will not know whether the library file is compatible with the architecture type your compiler is targeting. SFML does not explicitly append an architecture type to its library files, so you will simply have to remember this after building SFML or downloading a pre-built library from somewhere.

When building with Microsoft Visual Studio, the linker checks the machine types (architectures) of all modules that are to be linked with each other. If it finds a mismatch, it will abort with an error.

The error commonly looks like this:
```
fatal error LNK1112: module machine type 'X86' conflicts with target machine type 'x64'
```
It is possible that X86 and x64 are exchanged.

To resolve this error, simply make sure that no conflicts arise. If you are targeting x64 with your executable, make sure you are only using x64 libraries as well. The same applies for x86.

If you are compiling using GCC or clang, it is less obvious when architecture conflicts arise. Instead of having explicit checks as with Visual Studio, the linkers in these toolchains merely ignore all symbols with mismatching architectures. It will look like the linker accepted the library files, but in fact it didn't process any of them at all, leading to a large number of undefined references during linking. This is especially annoying because these errors can stem from many other causes as well. It is recommended to not build for x64 on Windows if using any of these toolchains. If you really must build for x64 on Windows, use Visual Studio instead.

### <a name="tr-grl-crash"/>My computer crashes when I run my SFML program!

SFML was designed in a way that should not cause your computer to crash/freeze/hang/BSOD in any way. If it does exhibit such behaviour specifically when running your SFML program, it might be only indirectly because of SFML.

If you are using unstable drivers (still in beta or off a development branch in your package manager) chances are that they are the root cause of the problem. The reason why they cause more problems with SFML than with other programs/games is mainly because OpenGL related bugs in drivers are given low priority over DirectX related bugs simply because the latter affect more games. You'll just have to be patient and wait for the relevant bug to get fixed or revert to an older, more stable driver.

If you are not using unstable drivers, crashing might still be caused by overclocking (either self-performed or automatic) of your hardware. Try running everything at standard performance settings and see if that fixes the problem. If your hardware comes overclocked by default, search around the internet for other people who might be experiencing the same problems. If this is the case you should contact your retailer/card manufacturer as this is a hardware related issue and you won't be able to do much on your own.

### <a name="tr-grl-bug"/>I found a bug!

Most of the time any unexpected behaviour is a result of misunderstanding how to use SFML. Out of many bug reports only few of them turn out to be real bugs **which are caused by SFML itself and nothing else**.

If you think you have found a bug and are still using SFML 1.6, note that support for 1.6 had ceased long ago. It is highly recommended to upgrade to 2.0. Any bug reports made for SFML 1.6 will be ignored unless they were carried over to 2.0 as well, however this is very unlikely. If you are using 2.0, try using the latest [nightly build](http://en.sfml-dev.org/forums/index.php?topic=9513.0). There are many things that have already been fixed between the RC which is available on the site and the latest development version.

If the bug is still present in the latest SFML version, try to produce a [minimal compilable code example](#tr-grl-minimal) that displays the bug and nothing else. That way the developers and others can focus on why it is occurring.

If you can reproduce what you think is a bug, if you have another computer at your disposal, try to run it there as well. If the bug does not occur there, try to reconfigure the corresponding hardware/software settings on the first PC. A lot of strange behaviour is a result of misconfigured/faulty software/drivers. **WARNING: Trying to report a bug that is a result of the usage of beta drivers is not a good idea. The source of the problem does not lie within the responsibility of the SFML developers and as such they can't do much to fix it themselves.**

When you are sure that the bug is a result of SFML internals and is platform independent, you can go ahead and post in the forum of the package in question, and don't forget to provide a precise description of your problem, the version of SFML you're using, your system configuration, and the compilable code, and if the situation requires, the logs of your compiler and/or linker.
Also make sure that the bug hasn't already been reported (use the [search function](http://en.sfml-dev.org/forums/index.php?action=search)), confirmed (look on the [issue tracker](https://github.com/LaurentGomila/SFML/issues?page=1&state=open)) or even resolved in the latest source (check also the [closed issues](https://github.com/LaurentGomila/SFML/issues?page=1&state=closed)).

### <a name="tr-grl-minimal"/>What is a minimal code?

A minimal code example is a snippet of source code that is compilable with very little effort.

This implies that:
* it consists of a single file (no extra .h, .hpp or .cpp files)
* there are no special compiler/linker options that need to be set
* the provided code is specific to the matter at hand and does not do more than it needs to

Such an example should never have to be longer than 40 lines of code (including the header include lines which of course have to be provided as well) unless it only happens in very specific cases.

An example of minimal code:
```cpp
#include <SFML/Graphics.hpp>

int main() {
	sf::RenderWindow window( sf::VideoMode(640, 480, 32), "Bug" );
	
	// Some bug specific code here...
	
	while( window.isOpen() ) {
		sf::Event event;
		while( window.pollEvent( event ) ) {
			if( event.type == sf::Event::Closed ) {
				window.close();
			}
		}
		
		// ... and here.
		
		window.display();
	}

	return EXIT_SUCCESS;
}
```

See also [the rules](http://en.sfml-dev.org/forums/index.php?topic=5559.msg36368#msg36368) for further details.

### <a name="tr-grl-obtain-minimal"/>And how can I easily obtain this minimal code?

Easy :

* Create a separate sandbox project with a single file consisting of a `main()` function
* Copy and paste your code into the body of the `main()` in such a way that it can compile
* One by one delete all lines which are not relevant to the actual problem and try to compile after every deletion to see if the problem still persists

Side note: This technique will often help you troubleshoot the problem on your own.

---

### <a name="tr-cb"/>Code::Blocks

### <a name="tr-cb-linker"/>I'm getting linker errors.

With MinGW, you must link the libraries in a precise order: if libX depends on libY, libX MUST be linked before libY. For example, if you use the Graphics and Audio modules, the correct linking order would be the following: sfml-audio, sfml-graphics, sfml-window, sfml-system.

If you use the dynamic versions of the SFML 1.6 libraries, you must also define the SFML_DYNAMIC symbol in the options for your project.
If you use the static version of the SFML 2.0 libraries, you must also define the SFML_STATIC symbol in the options for your project.
For more details, see the installation tutorial for Code::Blocks.

---

### <a name="tr-vs"/>Visual Studio

### <a name="tr-vs-crash"/>My project crashes randomly, but I don't get any compiler or linker errors.

Make sure that you're linking against the correct version of the libraries for your project. If you're compiling in Debug mode, you must link with the Debug versions of SFML, and vice-versa for Release mode. To recall, the naming conventions for SFML are:

* sfml-[module].lib for the Release DLL
* sfml-[module]-d.lib for the Debug DLL
* sfml-[module]-s.lib for the static Release DLL
* sfml-[module]-s-d.lib for the static Debug DLL.

If you link with the DLL versions, you must copy the required DLLs beside your executable:

* sfml-[module].dll for the Release DLL
* sfml-[module]-d.dll for the Debug DLL

### <a name="tr-vs-arch"/>I keep getting ```fatal error LNK1112: module machine type 'XYZ' conflicts with target machine type 'ZYX'```. Help!

See [here](#tr-grl-64bit).

---

### <a name="tr-win"/>Windows

### <a name="tr-win-console"/>Why does a console attach itself to my project?

In Windows, if you compile your project, you will have a console that attaches itself behind your window. To avoid this, you must create the correct type of project or change its type after creation. Many IDEs have options allowing you to choose whether you want to create a console or a GUI application. It is however recommended to create an empty project and manually set its type afterwards. This ensures that nothing else is set automatically that might cause strange behaviour later on.

If you have already created the console project or created an empty one:

* In Code::Blocks, open the project options (Project Menu -> Properties). In the Build targets tab, select the build target you wish to change on the left (most of the time only Debug and Release exist) and change its type option in the drop-down list on the right side from "Console application" to "GUI application".
* In Visual Studio, go to the project options (Project Menu -> Properties). In the tree on the left, expand the "Configuration properties" tree and expand the "Linker" sub-tree. Select "System" from the sub-tree, and in the SubSystem field on the right side change "Console (/SUBSYSTEM:CONSOLE)" to "Windows (/SUBSYSTEM:WINDOWS)" by clicking on the field and using the drop-down list.

To maintain a portable entry point (`int main()` function), you can link your program against the small sfml-main.lib library in the case of Visual Studio or libsfml-main.a in the case of Code::Blocks/MinGW.

Alternatively to hide the console, you can also define your own Windows entry point for graphical applications.

```cpp
int WINAPI WinMain(HINSTANCE hThisInstance, HINSTANCE hPrevInstance, LPSTR lpszArgument, int nCmdShow)
```

Replace your `int main()` or `int main(int argc, char** argv)` with this function and it will be called by the operating system when your program is executed just like the classical `int main()` function.

---

### <a name="tr-lnx"/>Linux

### <a name="tr-lnx-compile"/>(Debian) I can't compile the source code.

Before anything else, make sure that you've followed the [official tutorial](http://www.sfml-dev.org/tutorials/) and then check if the following packages have been installed:

* libgl1-mesa-dev
* libglu1-mesa-dev
* libopenal-dev
* libopenal1-dbg
* libsndfile1-dev
* libx11-dev
* libx11-6-dbg
* libfreetype6-dev
* libxrandr-dev
* libxrandr2-dbg

Though the library names might vary, especially regarding the version number.

### <a name="tr-lnx-titlebar"/>There is no titlebar visible and/or artifacts from windows are visible.

If you're running compiz, then turn it off, because compiz interfere with other OpenGL applications.

You can use this simple script to toggle compiz on and off, if you're using metacity as your window manager:

```bash
    #!/bin/bash
    pid=`ps --no-heading -C compiz.real | cut -d "?" -f1`;
    if [ -n "$pid" ]; then
      metacity --replace &
    else
      compiz --replace &
    fi
```

## <a name="licensing"/>Licensing

### <a name="lic-license"/>What license does SFML have?

SFML is under the [zlib/png license](http://www.opensource.org/licenses/zlib-license.php). You can use SFML for both open-source and proprietary projects, including paid or commercial ones. If you use SFML in your projects, a credit or mention is appreciated, but is not required.

### <a name="lic-commercial"/>Can I use SFML in commercial applications?

Yes, you may use SFML in commercial applications. You don't even have to mention that you used SFML in your application, but the zlib license states that if you do mention it, you are not allowed to state that you yourself are the author of SFML. You are also not allowed to modify SFML and represent it as being the original.

### <a name="lic-static"/>Can I link SFML statically?

Yes, you can link SFML statically. This can be done in any operating system, although in Linux and Mac OS X it is recommended to only link dynamically unless you have special requirements.

When linking statically, do not forget to specify the SFML_STATIC define on your command line.

### <a name="lic-examples"/>Can I use the code from the example directory?

The code from the example directory is not marked as being provided under a separate license. As such it is also governed by the zlib/png license and you are free to do anything you want with the code as long as it complies to the license.

### <a name="lic-pay"/>Do I have to pay any license fees or royalties?

The zlib/png license is a permissive free software license which means it has no provisions to monetize the software it covers. As such SFML can be used for free with no requirement to pay any fees or royalties to its author.

## <a name="libraries"/>Libraries for SFML

### <a name="libraries-gui-package"/>Does SFML have a GUI package?

No, SFML does not have a GUI package, but you can essentially use any OpenGL-based GUI library. Here's a obviously incomplete list:

* [SFGUI](http://sfgui.sfml-dev.de/)
* [GWEN](https://github.com/garrynewman/GWEN)
* [TGUI](https://tgui.eu)
* [CEGUI](http://www.cegui.org.uk/wiki/index.php/Main_Page)
* [Guichan](https://code.google.com/p/guichan/)
* [libRocket](http://librocket.com/)

### <a name="libraries-gui-lib"/>Can you interface SFML with a GUI library?

Yes, you can! See examples for Qt, wxWidgets, and the native Win32 and X11 APIs in the [official tutorials](http://www.sfml-dev.org/tutorials/).

### <a name="libraries-video"/>Can I read video files with SFML?

SFML does not have a video playback module, but one can easily connect FFMPEG or similar libraries with SFML. There even exists a couple of maintained projects from some SFML users called [sfeMovie](http://lucas.soltic.etu.p.luminy.univ-amu.fr/sfeMovie/) and [Motion](http://en.sfml-dev.org/forums/index.php?topic=16221.0).

### <a name="libraries-thor"/>What exactly is Thor?

Thor is an open-source and cross-platform library written in the programming language C++. It is an extension to SFML and comes with high-level features that base on SFML and that are supposed to help in daily C++ routine, especially with respect to graphics and game programming.

The git repository is also hosted on [GitHub](https://github.com/Bromeon/Thor) and the official website can be found at: http://www.bromeon.ch/libraries/thor/

## <a name="misc"/>Miscellaneous

### <a name="misc-projects"/>Are there any famous projects with SFML?

As SFML is open source and has a permissive free software license, game creators are not forced to specify that they've used SFML in their games, thus it might well be that SFML has been used in bigger commercial projects, we don't know of. As for the known projects, there's a [dedicated page](Projects) on the SFML's wiki.

### <a name="misc-distr"/>How can I distribute my game?

### <a name="misc-upload"/>Where can I upload my game to?

There are many possible places to share your game and game related data with the world wide web and their users. [HashCookie](http://hashcookie.net/) is a simple and easy to use site. Another provider to consider is [MediaFire](http://www.mediafire.com/). Both are simple and fast file sharing hosts.

Also [SFML Projects](http://sfmlprojects.org/) is currently being developed as a place to put many of the projects made with SFML.

### <a name="misc-examples"/>Are there any example projects I can learn from?

Examples of the usage of each module are provided with SFML itself if you download the full SDK version. If you want full projects that make use of many features of SFML at the same time your best bet is to check out the [projects forum](http://en.sfml-dev.org/forums/index.php?board=10.0). People frequently showcase what they are working on there for the community to see. Often they might provide the source code as well, so if you feel brave enough you can look in there if you find a certain project interesting.