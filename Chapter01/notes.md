## The philoshophy of C++

To leverage the type-safety features offered by the language:

```cpp
#include <chrono>

using namespace std::literalts::chrono_literals;

struct Duration {
    std::chrone::milliseconds millis_;
};

void example() {
    auto d = Duration();
    // d.millis_ = 100; //compilation error, as 100 could mean anything
    d.millis_ = 100ms;  //okay
    auto timeout = 1s;  // or std::chrono::seconds(1);
    d.millis_ = timeout;    // okay, converted automatically to milliseconds
}
```

[source coroutine](https://en.cppreference.com/w/cpp/language/coroutines)

A coroutine is a function that can suspend execution to be resumed later. Coroutines are stackless: they suspend execution by returning to the called and the data that is required to resume execution is stored seperately from the stack. This allows for sequential code that executes asynchronously (e.g. to handle non-blocking IO without explicit callbacks), and also supports algorithms on lazy-computed infinite sequences and other uses.

A function is a coroutine if its definition contains any of the following:

- the `co_await` operator — to suspend execution until resumed
    ```cpp
    task<> tcp_echo_server() {
        char data[1024];
        while (true) {
            size_t n = co_await socket.async_read_some(buffer(data));
            co_await async_write(socket, buffer(data, n));
        }
    }
    ```
- the `co_yield` — to suspend execution returning a value
- the `co_return` — to complete execution returning a value

Another great set of abstraction, offered by the standard library, are algorithms.

```cpp
int count_dots(std::string_view str) {
    return std::count(std::begin(str), std::end(str), '.');
}
```

## Following the SOLID and DRY principles

SOLID:

- **Single responsibility principle**: each code unit should have exactly one responsibility
- **Open-closed principle**: code should be open for extension but closed for modification
  - Open for extension means that we could extend the list of types the code supports easily
  - Closed for modification means existing code shouldn't change
- **Liskov substitution principle**: if a function works with a pointer or reference to a base object, it must also work with a pointer or reference to any of its derived objects.
- **Interface segregation**: no client should be forced to depend on methods that it does not use
- **Dependency inversion**: high-level modules should not depend on lower-level ones. Instead, both should depend on abstractions.

Each develepor is constructed by the `Project` class. This approach is not ideal, though, since now the higher-level concept, `Project`, depends on lower-level ones -- modules for individual developers. Instead we can define our developers to depend on an interface as follows:

```cpp
class Developer {
    public:
        virtual ~Developer() = default;
        virtual void develop() = 0;
};

class FrontEndDeveloper : public Developer {
    public:
        void develop() override {developFrontEnd();}
    private:
        void developFrontEnd();
};

class BackEndDeveloper : public Developer {
    public:
        void develop() override {developBackEnd();}
    private:
        void developBackEnd();
};
```

Now, the `Project` class no longer has to know the implementations of the developers.
Because of this, it has to accept them as constructor arguments:

```cpp
class Project {
    public:
        using Developers = std::vector<std::unique_ptr<Developer>>;
        explicit Project(Developers developers)
            : developers_{std::move(developers)} {}

        void deliver() {
            for (auto &developer : developers_) {
                developer->develop();
            }
        }
    private:
        Developers developers_;
};
```

In this approach, `Project` is decoupled from the concrete implementations and instead depends only on the polymorphic interface named `Developer`. The "lower-level" concrete classes also depend on this interface. This allows for much easier unit testing -- now you can easily pass mocks as arguments in your test code. 

There is another way of inverting dependencies that doesn't have those drawbacks. Let's see how this can be done using a variadic template, a generic lambda for C++14, and `variant`, either from C++17 or a third-party library such as Abseil or Boost. First the developer classes:

```cpp
class FrontEndDeveloper {
    public:
        void develop() {
            developFrontEnd();
        }
    private:
        void developFrontEnd();
};

class BackEndDeveloper {
    public:
        void develop() {
            developBackEnd();
        }
    private:
        void developBackEnd();
};
```

Now we don't rely on an interface anymore, so no virtual dispatch will be done. The `Project` class will still accept a vector of `Developers`:

```cpp
template <typename... Devs>
class Project {
    public:
        using Developers = std::vector<std::variant<Devs...>>;

        explicit Project(Developers developers)
            : developers_{std::move(developers)} {}

        void deliver() {
            for (auto &developer : developers_) {
                std::visit([](auto &dev) { dev.develop(); }, developer);
            }
        }

    private:
        Developers developers_;
};
```

If you're not familiar with `variant`, it's just a class that can hold any of the types passed as template parameters. Because we're using a variadic template, we can pass however many types we like. To call a function on the object stored in the variant, we can either extract it using `std::get` or use `std::visit` and a callable object -- in our case, the generic lambda. 

Because `Project` is now a template, we have to either specify the list of types each time we create it or provide a type alias. You can use the final class like so:

```cpp
using MyProject = Project<FrontEndDeveloper, BackEndDeveloper>;
auto alice = FrontEndDeveloper();
auto bob = BackEndDeveloper();
auto new_project = MyProject{{alice, bob}};
new_project.deliver();
```



