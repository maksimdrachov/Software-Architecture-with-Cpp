## Chapter 2: Architectural styles

### Deciding between stateful and stateless approaches

Stateful and stateless are two opposite ways to write software, each with their own pros and cons.

Stateful software's behaviour depends on its internal state. Let's take a web service, for instance. If it remembers its state, the consumer of the service can send less data in each request, because the service remembers the context of those requests. 

One approach would be to expose the properties required for calculation purposes:

This approach has two cons. The first is that it's not thread-safe; a single instance of such a `PaymentCalculator` class cannot be used in multiple threads without locks. The second is that once our calculations become more complicated, the class will probably start duplicating more fields fron out `Consultant` class. 

To eliminate the duplication, we could rework our class to store a `Consultant` instance like this:

```
class PaymentCalculator {
    public:
        ...
    private:
        gsl::not_null<const Consultant *> consultant_;
        double taxPercentage_;
};
```

We're using a helper class from the Guideline Support Library (GSL) to store a rebindable pointer in a wrapper that automatically ensures we're not storing a null value.

This approach still has the disadvantage of not being thread-safe. Can we do any better? It turns out that can make the class thread-safe by making it stateless:

```cpp
class PaymentCalculator {
    public:
        static double calculate(const Consultant &c, double taxPercentage);
};
```

### Stateless and statefull services

The same principles that we discussed for classes can be mapped to higher-level concepts, for instance, microservices.

What does a stateful service look like? Let's take FTP as an example. It it's not anonymous, it requires the user to pass a username and password to create a session. The server stores this data to identify the user as still connected, so it's constantly storing state. Each time the user changes the working directory. the state gets updated. Each change done by the user is reflected as a change of state, even when they disconnect. Having a stateful service means that depending on the state. you can be returned different result for two identically looking GET requests. 

Stateless services, as the REST ones described later in the book, take the opposite approach. Each request must contain all the data required in order for it to be successfully processed. 

**Representational State Transfer** (**REST**), carries the notion that all the state required for processing the request must be transferred within it. If this is not the case, you can't say you have a RESTful service. 

Keeping sessions on the server side is a bad approach for services for several reasons: they add a lot of complexity that could be avoided, they make bugs harder to replicate, and most importantly, they don't scale. If you'd want to distribute the load to another server, chances are you'd have trouble replicating the sessions with the load and synchronizing them between servers. All session information should be kept on the client's side.

### Understanding monoliths -- why they should be avoided, and recognizing exceptions

### Common event-based topologies

The two main topologies of event-driven architectures are broker-based and mediator-based. Those topologies differ in how the events flow through the system.

The mediator topology is best used when processing an event that requires multiple tasks or steps that can be performed independently. All events produced initially land in the mediator's event queue. The mediator knows what need to be done in order to handle the event, but instead of performing the logic itself, dispatches the even to appropriate event processors through each processor's event channel.

A broker, on the other hand, is a lightweight component that contains all the queues and doesn't orchestrate the processing of an event. It can require that the recipients subscribe to specific kinds of events and then simply forward all the ones that are interesting to them. Many message queues rely on brokers, for example, ZeroMQ.

### Event sourcing

You can think of events as notifications that contain additional data for the notified services to process. There is, however, another way to think of them: a change of state. Think how easy it would be to debug issues with your application logic if you'd be able to know the state in which it was when the bug occurred and what change was requested of it. 

### Understanding layered architecture

One interesting concept adopted by the C++ Micro Services framework is a new way to deal with singletons. The `GetInstance()` static function will, instead of just passing a static instance object, return a service reference obtained from the bundled context. So effectively, singleton objects will get replaced by services that you can configure. It can also save you from the static deinitialization fiasco, where multiple singletons that depend on each other have to unload in a specific order. 

