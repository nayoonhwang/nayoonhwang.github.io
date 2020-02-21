Debugging is an essential part of any modern application lifecycle. It’s not only useful for finding bugs as programmers often use debuggers to see and understand what happens in a new codebase they have to work with or when learning a new language.

There are two styles of debugging which people prefer:

print statements: which is logging as your code executes various steps that might run
using a debugger such as Delve, either directly or via an IDE: this gives more control over the execution flow, more capabilities to see what the code does which may not have been included in the original print statement, or even changing values during the application runtime or going back and forward with the execution of the application.
In this series we’ll focus on the second option, using an IDE to debug an application.

As you noticed from the description above, doing so provides a lot more control and capabilities to find bugs and as such this article is broken down in several sections:

debugging an application
debugging tests
debugging a running application on the local machine
debugging a running application on a remote machine

Debugging은 현대 Application Lifecycle의 중요한 부분이다. 