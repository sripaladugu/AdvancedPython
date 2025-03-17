In Python, both Abstract Base Classes (ABC) and Protocols are used to define interfaces or contracts that can be implemented by other classes. However, they serve slightly different purposes and have different implications for the design of your code.

** Abstract Base Classes (ABC) **
Abstract Base Classes are a way to define interfaces in Python that can enforce the implementation of certain methods in subclasses. They are part of the abc module and are used when you want to ensure that a derived class implements specific methods.

** Key Characteristics of ABC: **

- Method Enforcement: Using ABCs allows you to define abstract methods that must be implemented by any subclass.
- Instantiation: You cannot instantiate an ABC directly; you must subclass it and implement its abstract methods.
- Decorator: Abstract methods are defined using the @abstractmethod decorator.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

class Circle(Shape):
    def __init__(self, radius: float) -> None:
        self.radius = radius

    def area(self) -> float:
        return 3.14 * self.radius ** 2

circle = Circle(5)
print(circle.area())  # Output: 78.5
```

** Protocols **
Protocols, introduced in Python 3.8 with PEP 544, provide a way to define "structural" subtyping (also known as "duck typing"). Unlike ABCs, protocols do not enforce method implementation; instead, they specify a set of methods or properties that a class must have to be considered as implementing the protocol.

** Key Characteristics of Protocols: **

- Structural Typing: Protocols define a set of method signatures that can be implemented by any class without needing explicit subclassing.
- Flexibility: Classes that have the required methods are considered as implementing the protocol, even if they do not inherit from the protocol.

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...

class Circle:
    def draw(self) -> None:
        print("Drawing a circle")

def render(shape: Drawable):
    shape.draw()

circle = Circle()
render(circle)  # This works because Circle has a draw() method
```

Where Abstract Base Classes tend to break down is when you have to introduce abstraction in a place where part of the code is not under the control of you.
For example, let's say you are using a third party library and you don't have access to the code of that library but you still want to use it in your own code, but you also don't want to have too much coupling.
In that case, you can't define Abstract Base Classes because you don't have access to the code of the library, so you can't create subclasses. This is a scenario where Protocols come in handy because they allow you to introduce abstractions without having to have a direct inheritance relationship.

For example, let's say you have 3 different logger libraries (FileLogger, ConsoleLogger, NetworkLogger) that you want to use in your code, but you don't want to have a direct dependency on any of them.
You then create some adapters i.e, wrappers around these loggers. But when you do want to do the logging then you use a LoggerProtocol class that has just a log method and we use that in our code.

```python
from typing import Protocol

# Third-party logger library 1
class FileLogger:
    def log_to_file(self, message: str) -> None:
        print(f"Logging to file: {message}")

# Third-party logger library 2
class ConsoleLogger:
    def console_log(self, message: str) -> None:
        print(f"Logging to console: {message}")

# Third-party logger library 3
class NetworkLogger:
    def send_log(self, message: str) -> None:
        print(f"sending log over network: {message}")

# Adapter for FileLogger
class FileLoggerAdapter:
    def __init__(self, logger: FileLogger) -> None:
        self.logger = logger

    def log(self, message: str) -> None:
        self.logger.log_to_file(message)

# Adapter for ConsoleLogger
class ConsoleLoggerAdapter:
    def __init__(self, logger: ConsoleLogger) -> None:
        self.logger = logger

    def log(self, message: str) -> None:
        self.logger.console_log(message)

# Adapter for NetworkLogger
class NetworkLoggerAdapter:
    def __init__(self, logger: NetworkLogger) -> None:
        self.logger = logger

    def log(self, message: str) -> None:
        self.logger.send_log(message)

# Logger Protocol
class LoggerProtocol(Protocol):
    def log(self, message: str) -> None:
        ...

def log_message(logger: LoggerProtocol, message: str) -> None:
    logger.log(message)

# Usage
def main() -> None:
    file_logger = FileLoggerAdapter(FileLogger())
    console_logger = ConsoleLoggerAdapter(ConsoleLogger())
    network_logger = NetworkLoggerAdapter(NetworkLogger())

    log_message(file_logger, "file logging example")
    log_message(console_logger, "console logging example")
    log_message(network_logger, "network logging example")

main()
```

** Excercise 1: **

** Social Media Channels **
You just landed a job at "SocialOverlord", a company developing a SaaS product allowing you to write and schedule posts to a variety of social networks.

The whole backend is written in Python (yay!), but unfortunately, the person you're replacing didn't know classes exist (ehrm...) and so they used tuples to represent all the data in the system (not so yay...). Here's a code example:

```python
# each social channel has a type
# and the current number of followers
SocialChannel = tuple[str, int]

# each post has a message
# and the timestamp when it should be posted
Post = tuple[str, int]

def post_a_message(channel: SocialChannel, message: str) -> None:
  type, _ = channel
  if type == "youtube":
    post_to_youtube(channel, message)
  elif type == "facebook":
    post_to_facebook(channel, message)
  elif type == "twitter":
    post_to_twitter(channel, message)

def process_schedule(posts: list[Post],
 channels: list[SocialChannel]) -> None:
  for post in posts:
    message, timestamp = post
    for channel in channels:
      if timestamp <= time():
        post_a_message(channel, message)
```

Solution:

```python
from dataclasses import dataclass, field
from time import time
from typing import Protocol

class SocialChannel(Protocol):
    def post_message(self, message: str) -> None:
        ...

@dataclass
class YouTube:
    followers: int
    type: str = field(init=False, default="youtube")
    def post_message(self, message:str) -> None:
        print("type: ", self.type, "message:", message, "followers:", self.followers)

@dataclass
class Facebook:
    followers: int
    type: str = field(init=False, default="facebook")
    def post_message(self, message:str) -> None:
        print("type: ", self.type, "message:", message, "followers:", self.followers)

@dataclass
class Twitter:
    followers: int
    type: str = field(init=False, default="twitter")
    def post_message(self, message:str) -> None:
        print("type: ", self.type, "message:", message, "followers:", self.followers)


@dataclass
class Post:
    message: str
    timestamp: int

def process_posts(posts:list[Post], channels:list[SocialChannel]) -> None:
    for post in posts:
        for channel in channels:
            if post.timestamp <= time():
                channel.post_message( message=post.message )

channels = [YouTube(1000), Twitter(1000), Facebook(5)]
posts = [Post("Yenkamma Wow!!!",0), Post("Pullamma Hehe!!!", 0)]

def main() -> None:
    process_posts(posts, channels)

main()
```

** Exercise: HTML Trees **
Consider the following classes:

```python
from dataclasses import dataclass
from typing import Self

@dataclass
class Div:
   parent: Self | None = None
   x: int = 0
   y: int = 0

   def compute_screen_position(self) -> tuple[int, int]:
     if not self.parent:
       return (self.x, self.y)
     parent_x, parent_y = self.parent.compute_screen_position()
     return (parent_x + self.x, parent_y + self.y)

@dataclass
class Button:
   parent: Div
   x: int
   y: int

   def click(self) -> None:
     print("Click!")

@dataclass
class Span:
   parent: Div
   x: int
   y: int
   text: str
```

Of course, HTML has many more different elements. In the elements above, there is some duplication (in particular, the parent, x and y instance variables). Also, we'd like to be able to compute the position of any type of element, not just divs and be able to combine different elements in a tree.

a) Abstraction
Refactor the class hierarchy by introducing an abstract base class HTMLElement that divs, buttons, and spans are subclasses of. Minimize code duplication and make sure that each element is able to compute its position on the screen.

b) Protocols
Can you use a protocol class just as easily instead of an abstract base class? What would need to be changed in the code for this to work?

Solution:

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Protocol

class HTMLElement
(ABC):
    @abstractmethod
    def compute_screen_position(self) -> tuple[int, int]:
        ...


@dataclass
class
```
