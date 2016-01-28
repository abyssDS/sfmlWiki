## FPacket class : flagged packets.

Let you send and receive flagged (or you could say tagged) data across the network.

FPacket objects are usual SFML packets, in which you can put data and send them across SFML sockets as usual (they inherit from sf::Packet). The difference is that you can set a flag to the packet, in order to easily specify the use case of the packet. The flag will be sent automatically across the network when you send the packet, and automatically extracted on the other side when the packet is received, provided you use this same class to receive.

If you receive flagged data with another packet class (like sf::Packet), or the other way around, you will lose some data or your program will crash, so if you start using flagged packets, it is highly recommended that you use them everywhere in both server and client programs.

The flag is an enumerated type PFlag. You can construct a packet directly with a PFlag, or set the flag later. You can easily extend the PFlag enumeration (in FPacket.hpp, line 37) to match the needs of your application.
When you receive a flagged packet, you can then easily check the flag : it will tell you what's the purpose of the received packet, what type of data it contains, so you'll know what to do with it. See usage example.

FPacket is a really simple class. Feel free to correct the files if you see any mistakes.

### Usage example
```cpp
sf::TcpSocket socket;


// Common sending usage.
// We construct a FPacket that we will use to send some text.
FPacket textPacket;       

// Put a string into the packet. We could also add any other data, like the sender id for example.
sf::String str = "A string.";
textPacket << str;  

// Set the proper flag to the packet. 
textPacket.setFlag(FPacket::Text);  

// Send the packet. The flag will be automatically sent.
socket.send(textPacket);  



// Common receiving usage.
FPacket receivedPacket;

// We retrieve data into the packet. The flag is automatically set on reception.
socket.receive(receivedPacket);  

// Let's check what kind of packet we received.
if (receivedPacket.flag() == FPacket::Text)     
{
    sf::String str;
    receivedPacket >> str;
    // do something with the string
}

// You can also check the flag by directly comparing the packet itself
else if (receivedPacket == FPacket::Ping)
{
    // And you can construct a FPacket with a given flag directly
    FPacket packet(FPacket::Pong);   
    socket.send(packet);
}

```

### FPacket.hpp
```cpp
#ifndef FPACKET_HPP
#define FPACKET_HPP
#include <SFML/Network.hpp>

class FPacket : public sf::Packet
{
public:
    /** Enumerates the possible flags a packet can have.
        It will be automatically sent and received when using this class for sharing data across sockets.
        Feel free to extend this enumeration to match your need.  */
    enum PFlag {
                None,
                Ping,
                Pong,
                Connect,
                Disconnect,
                Connected,
                Disconnected,
                Text,
                Move
				// ... Add your flags here
    };

    /** Default constructor. */
    FPacket();
    /** Overloaded constructor. Constructs a Packet and sets the flag as specified. */
    FPacket(PFlag);

    /** Sets the flag to the packet */
    void setFlag(PFlag);

    /** Reads the flag. */
    inline PFlag        flag() const { return static_cast<PFlag>(m_flag); }
    /** Reads the flag as an 8 bytes unsigned int. */
    inline sf::Uint8 flagAsU() const { return m_flag; }
    /** Checks if a flag has been set on this Packet. Returns false if flag equals to FPacket::None. */
    inline bool    isFlagged() const { return m_flag == FPacket::None ? false : true; }

    /** Hiding base class clear() function.
        Clears the packet as usual and sets the flag to 'None', so isFlagged() will return false.  */
    virtual void clear();


private:
    sf::Uint8    m_flag;

    virtual void onReceive(const void*, std::size_t);
    virtual const void* onSend(std::size_t&);

};


/** Overloading of comparison operator. Checks a FPacket object against a PFlag enumeration type instance (FPacket::PFlag). */
bool operator== (const FPacket& p1, const FPacket::PFlag& p2);
/** Overloading of comparison operator. Checks a Packet object against a PFlag enumeration type instance (FPacket::PFlag). */
bool operator!= (const FPacket& p1, const FPacket::PFlag& p2);


#endif // FPACKET_HPP

```

### FPacket.cpp
```cpp
#include "FPacket.hpp"
#include <cstring>


FPacket::FPacket() :
    m_flag(None)
{

}


FPacket::FPacket(PFlag flag) :
    m_flag(flag)
{

}



void FPacket::setFlag(PFlag flag)
{
    m_flag = flag;
}



void FPacket::clear()
{
    static_cast<sf::Packet*>(this)->clear();
    m_flag = None;
}



void FPacket::onReceive(const void* data, std::size_t size)
{
    std::size_t so_flag = sizeof m_flag;
    std::memcpy(&m_flag, data + size - so_flag, so_flag);

    append(data, size - so_flag);
}



const void* FPacket::onSend(std::size_t& size)
{
    *this << m_flag;

    size = getDataSize();
    return getData();
}



bool operator== (const FPacket& p1, const FPacket::PFlag& p2)
{
    return p1.flag() == p2;
}

bool operator!= (const FPacket& p1, const FPacket::PFlag& p2)
{
    return p1.flag() != p2;
}

```

That's all folks !