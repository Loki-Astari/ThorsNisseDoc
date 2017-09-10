
## Service

```cpp
class MyHandler: public HandlerSuspendable<DataSocket>
{
    virtual bool eventActivateNonBlocking()
    {
        ISocketStream   input(stream);

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

        OSocketStream   output(stream);
        output << "HTTP/1.1 200 OK\r\n"
                  "Content-Length: " << (11 + message.size()) << "\r\n"
                  "\r\n"
                  "It Worked: " << message;
    }
};
int main()
{
    Server      server;
    server.listenOn<MyHandler>(8080);
    server.listenOn<MyHAndler>(ServerConnection(8081, 10));
}
```
```cpp--DeepDive
/*
 * If you just want to specify a port to Server::listenOn() this class will default constructor
 * with an integer. If you need specify the number of waiting connections allowed then use this
 * object as the parameter to `listenOn().
 */
struct ServerConnection
{
    public:
        ServerConnection(int port, int maxConnections = ThorsAnvil::Nisse::Core::Socket::ServerSocket::maxConnectionBacklog);
};

/*
 * A very simplistic server object.
 * Multiple listeners on different ports can be installed with `listenOn()`.
 * Multiple timers can be installed with `addTimer()`.
 * When listernes and timers have been installed start the event lop with `start()`.
 */
class Server
{
    public:
        Server();

        Server(Server&&);
        Server& operator=(Server&&);

        void start(double check = 10.0);
        void flagShutDown();

        template<typename Handler, typename Param>
        void listenOn(ServerConnection const& info, Param& param);

        void addTimer(double timeOut, std::function<void()>&& action);

        bool isRunning() const {return running;}
};

/*
 * HandlerBase:     The base of all handler classes.
 */
class HandlerBase
{
    public:
        HandlerBase(Server& parent, LibSocketId socketId, short eventType, double timeout = 0);
        virtual ~HandlerBase();
        void activateEventHandlers(LibSocketId sockId, short eventType);
        virtual short eventActivate(LibSocketId sockId, short eventType) = 0;
        virtual bool  suspendable() = 0;
        virtual void  close() = 0;
};

/*
 * A handler based that has the concept of a stream.
 * It can thus close the connection but not much else.
 */
template<typename Stream>
class HandlerStream: public HandlerBase
{
    public:
        HandlerStream(Server& parent, Stream&& stream, short eventType, double timeout = 0);
        virtual void  close() override;
};

/*
 * For handlers that can't be suspended (note not the same as blocked).
 * This is a special case that is more for internal use.
 */
template<typename Stream>
class HandlerNonSuspendable: public HandlerStream<Stream>
{
    public:
        using HandlerStream<Stream>::HandlerStream;
        virtual void suspend(short) final;
        virtual bool suspendable()  final;
};

/*
 * For handlers that can be suspended.
 * Most handlers will derive from this class.
 */
template<typename Stream>
class HandlerSuspendable: public HandlerStream<Stream>
{
    public:
        HandlerSuspendable(Server& parent, Stream&& stream, short eventType);
        HandlerSuspendable(Server& parent, Stream&& stream, short eventType, short firstEvent);
        virtual void suspend(short type);
        virtual bool suspendable();
        virtual short eventActivate(LibSocketId sockId, short eventType) final;
        virtual bool eventActivateNonBlocking() = 0;
};


/*
 * An internal handler used by the server.
 * This is used to wait on accepted connections and start the user specified handler.
 */
template<typename ActHand, typename Param>
class ServerHandler: public HandlerNonSuspendable<Socket::ServerSocket>
{
    public:
        ServerHandler(Server& parent, Socket::ServerSocket&& so, Param& param);
        ~ServerHandler();
        virtual short eventActivate(LibSocketId sockId, short eventType) override;
};

/*
 * An internal handler used by the server.
 * This handler deals with timer events and calls the appropriate user defined code.
 */
class TimerHandler: public HandlerNonSuspendable<int>
{
    public:
        TimerHandler(Server& parent, double timeOut, std::function<void()>&& action);
        virtual short eventActivate(LibSocketId sockId, short eventType) override;
};

```

A simple wrapper around libEvent.
<dl>
<dt>NameSpace:</dt><dd>ThorsAnvil::Nisse::Core::Service</dd>
<dt>Headers:</dt><dd>ThorsNisseCoreService/</dd>
<dt>Server.h</dt><dd>

* struct ServerConnection
* class Server

</dd>
<dt>Handler.h</dt><dd>

* class HandlerBase
* class HandlerStream: public HandlerBase
* class HandlerNonSuspendable: public HandlerStream
* class HandlerSuspendable: public HandlerStream

</dd>
<dt>ServerHandlers.h</dt><dd>

* class ServerHandler: public HandlerNonSuspendable
* class TimerHandler: public HandlerNonSuspendable

</dd>
</dl>


