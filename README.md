# Nod
Dependency free, header only signals and slot library implemented with C++11.

## Usage

### Simple usage
The following example creates a signal and then connects a lambda as a slot.
```cpp
// Create a signal which accepts slots with no arguments and void return value.
nod::signal<void()> signal;
// Connect a lambda slot that writes "Hello, World!" to stdout
signal.connect([](){
		std::cout << "Hello, World!" << std::endl;
	});
// Call the slots
signal();
```

### Connecting multiple slots
If multiple slots are connected to the same signal, all of the slots will be
called when the signal is invoked. The slots will be called in the same order
as they where connected.
```cpp
void endline() {
	std::cout << std::endl;
}

// Create a signal
nod::signal<void()> signal;
// Connect a lambda that prints a message
signal.connect([](){
		std::cout << "Message without endline!";
	});
// Connect a function that prints a endline
signal.connect(endline);

// Call the slots
signal();

```

### Slot arguments
When a signal calls it's connected slots, any arguments passed to the signal
are propagated to the slots. To make this work, we do need to specify the 
signature of the signal to accept the arguments.
```cpp
void print_sum( int x, int y ) {
	std::cout << x << "+" << y << "=" << (x+y) << std::endl;
}
void print_product( int x, int y ) {
	std::cout << x << "*" << y << "=" << (x*y) << std::endl;
}


// We create a signal with two integer arguments.
nod::signal<void(int,int)> signal;
// Let's connect our slot
signal.connect( print_sum );
signal.connect( print_product );

// Call the slots
signal( 10, 15 );
signal(-5, 7);	

```

### Disconnecting slots
There are many circumstances where the programmer needs to diconnect a slot that
no longer want to recieve events from the signal. This can be really important
if the lifetime of the slots are shorter than the lifetime of the signal. That
could cause the signal to call slots that have been destroyed but not
disconnected, leading to undefined behaviour and probably segmentation faults.

When a slot is connected, the return value from the  `connect` method returns
an instance of the class `nod::connection`, that can be used to disconnect
that slot.
```cpp
// Let's create a signal
nod::signal<void()> signal;
// Connect a slot, and save the connection
nod::connection connection = signal.connect([](){
								 std::cout << "I'm connected!" << std::endl;
							 });
// Triggering the signal will call the slot
signal();
// Now we disconnect the slot
connection.disconnect();
// Triggering the signal will no longer call the slot
signal();
```	

### Scoped connections
To assist in disconnecting slots, one can use the class `nod::scoped_connection`
to capture a slot connection. A scoped connection will automatically disconnect
the slot when the connection object goes out of scope.
```cpp
// We create a signal
nod::signal<void()> signal;
// Let's use a scope to control lifetime
{ 
	// Let's save the connection in a scoped_connection
	nod::scoped_connection connection =
		signal.connect([](){
			std::cout << "This message should only be emitted once!" << std::endl; 
		});
	// If we trigger the signal, the slot will be called
	signal();
} // Our scoped connection is destructed, and disconnects the slot
// Triggering the signal now will not call the slot
signal();	
```

### More usage
```cpp
//todo: write some more example usage
```

## Thread safety
There are two types of signals in the library. The first is `nod::signal<T>` which 
is safe to use in a multi threaded environment. Multiple threads can read, write,
connect slots and disconnect slots simultaneously, and the signal will provide the 
nessesary synchronization. When triggering a slignal, all the registered slots will
be called and executed by the thread that triggered the signal.

The second type of signal is `nod::unsafe_signal<T>` which is **not** safe to use
in a multi threaded environment. No syncronization will be performed on the internal
state of the signal. Instances of the signal should theoretically be safe to read
from multiple thread simultaneously, as long as no thread is writing to the same 
object at the same time. There can be a performance gain involved in using the unsafe
version of a signal, since no syncronization primitives will be used.

`nod::connection` and `nod::scoped_connection` are thread safe for reading from
multiple threads, as long as no thread is writing to the same object. Writing in this
context means calling any non const member function, including destructing the object.
If an object is being written by one thread, then all reads and writes to that object
from the same or other threads needs to be prevented. This basically means that a 
connection is only allowed to be disconnected from one thread, and you should not
check connection status or reassign the connection while it is being disconnected.

## Building the tests
The test project uses [premake5](https://premake.github.io/download.html) to 
generate make files or similiar.

### Linux
To build and run the tests, execute the following from the test directory:
```bash
premake5 gmake
make -C build/gmake
bin/gmake/debug/nod_tests
```

### Visual Studio 2013
To build and run the tests, execute the following from the test directory:

```batchfile
REM Adjust paths to suite your environment
c:\path\to\premake\premake5.exe vs2013
"c:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\vsvars32.bat"
msbuild /m build\vs2013\nod_tests.sln
bin\vs2013\debug\nod_tests.exe
```

## The MIT License (MIT)

Copyright (c) 2015 Fredrik Berggren

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
