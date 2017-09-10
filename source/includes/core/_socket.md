
## Socket

```cpp
// Server Side
int main()
{
    ServerSocket    server(8080);
    while(true)
    {
        DataSocket      connection = server.accept();
        ISocketStream   input(connection);

        std::string     request;
        std::getline(input, request);

        std::string     header;
        while(std::getline(input, header) && header != "\r")
        {}

        std::string     message;
        std::string     line;
        while(std:getline(input, line))
        {
            message += line;
            message += "<br>";
        }

        OSocketStream   output(connection);
        output << "HTTP/1.1 200 OK\r\n"
                  "Content-Length: " << (11 + message.size()) << "\r\n"
                  "\r\n"
                  "It Worked: " << message;
    }
}
```
```cpp--DeepDive
// Socket.h
class BaseSocket
{
    public:
        virtual ~BaseSocket();

        // Moveable but not Copyable
        BaseSocket(BaseSocket&& move)               noexcept;
        BaseSocket& operator=(BaseSocket&& move)    noexcept;
        void swap(BaseSocket& other)                noexcept;

        void close();
}

/*
 * DataSocket:  Defines a socket that you can read/write too.
 *              This class should not be directry created. You can get an object
 *              of this data by creating a `ConnectSocket` or calling accept on
 *              ServerScoket (See below).
 */
class DataSocket: public BaseSocket
{
    public:
        std::pair<bool, std::size_t> getMessageData(char* buffer, std::size_t size, std::size_t alreadyGot = 0);
        std::pair<bool, std::size_t> putMessageData(char const* buffer, std::size_t size, std::size_t alreadyPut = 0);
        void        putMessageClose();
};

/*
 * ConnectSocket:   Creates a connection to server.
 */
class ConnectSocket: public DataSocket
{
    public:
        ConnectSocket(std::string const& host, int port);
};

/*
 * ServerSocket:    Creates a ServerSocket that will accept incomming requests.
 *
 *                  A blocking call to accept will return a DataSocket for
 *                  an external connection that has been established.
 *
 *                  Non blocking calls should onlt be made if you know there
 *                  is a connection waiting to be made.
 */
class ServerSocket: public BaseSocket
{
    public:
        static constexpr int maxConnectionBacklog = 5;
        ServerSocket(int port, bool blocking = false, int maxWaitingConnections = maxConnectionBacklog);

        DataSocket accept(bool blocking = false);
};

/*
 * SocketStreamBuffer:  A stream buffer that is used by ISocketStream and OSocketStream
 */
class SocketStreamBuffer: public std::streambuf
{
    public:
        virtual ~SocketStreamBuffer() override;
        SocketStreamBuffer(DataSocket& stream,
                           Notifier noAvailableData, Notifier flushing,
                           std::vector<char>&& bufData = std::vector<char>(4000),
                           char const* currentStart = nullptr, char const* currentEnd = nullptr);
        SocketStreamBuffer(SocketStreamBuffer&& move) noexcept;
};

/*
 * ISocketStream:       An implementation of std::istream that uses a DataSocket
 */
class ISocketStream: public std::istream
{
    SocketStreamBuffer buffer;

    public:
        ISocketStream(DataSocket& stream,
                      Notifier noAvailableData, Notifier flushing,
                      std::vector<char>&& bufData, char const* currentStart, char const* currentEnd);
        ISocketStream(DataSocket& stream,
                      Notifier noAvailableData, Notifier flushing);
        ISocketStream(DataSocket& stream);
        ISocketStream(ISocketStream&& move) noexcept;
};

/*
 * ISocketStream:       An implementation of std::ostream that uses a DataSocket
 */
class OSocketStream: public std::ostream
{
    SocketStreamBuffer buffer;

    public:
        OSocketStream(DataSocket& stream,
                      Notifier noAvailableData, Notifier flushing);
        OSocketStream(DataSocket& stream);
        OSocketStream(OSocketStream&& move) noexcept;
};
```

A simple wrapper around BSD sockets so they are easy to use in C++.
<dl>
<dt>NameSpace:</dt><dd>ThorsAnvil::Nisse::Core::Socket</dd>
<dt>Headers:</dt><dd>ThorsNisseCoreSocket/</dd>
<dt>Socket.h</dt><dd>

* class BaseSocket
* class DataSocket: public BaseSocket
* class ConnectionSocket: public DataSocket
* class ServerSocket: public BaseSocket

</dd>
<dt>SocketStream.h</dt><dd>

* class SocketStreamBuffer: public std::streambuf
* class ISocketStream: public std::istream
* class OSocketStream: public std::ostream

</dd>
</dl>

