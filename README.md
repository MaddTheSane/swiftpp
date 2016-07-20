Note
====
Re-entrency and thread safety has been fixed in the C++ / swift bridge.
Update to that latest version.

swiftpp
=======

An attempt at a C++ to swift bridge.


Introduction
===========

swiftpp is a clang tool that automatically generate the glue code to let
a swift class inherit from a C++ class.  You can override C++ virtual
methods from the swift derivative as well.

Things like this become possible:

```C++
// source.cpp
class swift MyCXXClass
{
	public:
		MyCXXClass();
		virtual ~MyCXXClass();

		virtual void doSomething( const std::string &i_text );
};
```

```
// source.swift
class MySwiftClass : MyCXXClass
{
	override func doSomething( text: String! )
	{
		// ...
		super.doSomething( text )
	}
}
```

Building
========

You will need a clang/llvm install to be able to compile swiftpp itself.
Look in setup_llvm.txt for instructions. I worked with llvm trunk and try
to use the latest Xcode to track the latest swift version. When things
get more stable, I'll try to settle on an llvm version. As of this
writing I have llvm revision #260064 and Xcode 7.2.1.

If clang/llvm is installed at the same path as in setup_llvm.txt
(/opt/llvm), you can then open and compile swiftpp.xcodeproj. You will
have to edit the project's paths otherwise.

That's it, you should now have a tool named swiftpp at the repository
root level.

Note that if you do an non-optimised (debug) build of clang/llvm, the
swiftpp pre-compiler might run very slow.


Usage
=====

Just like the Objective-C-to-swift bridge rely on a special header (PROJECT_NAME-Bridging-Header.h) to find what Objective-C classes you are exporting to swift, swiftpp rely on a header (cxx-Bridging-Header.h in the example) to know what C++ classes should be made visible to the Objective-C/swift side.

cxx-Bridging-Header.h will be parsed by swiftpp and every class that are annotated with the "swift" keyword will be exported.

example:

    class __attribute__((annotate("swift"))) MyCXXClass { ... };

 * You can use a `#define swift __attribute__((annotate("swift")))` to
 simplify the syntax

Then, you will be able to instanciate or subclass MyCXXClass in swift.

swiftpp will also use every unary function defined in the namespace
swift_converter as data converter to/from Objective-C/swift compatible type.

example:

```C++
namespace swift_converter
{
NSBezierPath *to( const Path2D &s ); // Path2D is a C++ class that represent a Bezier path
Path2D from( NSBezierPath *s );
}
```

If your exported class (MyCXXClass) uses types that are not compatible
with the swift world, they will be auto converted using those converters.
Converter names are not important, type inference is used to select the
right one.  There are also some builtin conversions:
 - std::string - NSString
 - std::vector - NSArray
 - std::map/std::unordered_map - NSDictionary

If swiftpp is already built you can try an extremely basic example in
examples/SimpleShapeDetector


Project setup
=============

This is the complicated part! You can always refer to
examples/SimpleShapeDetector to see how it's done...

1. swiftpp need to be run as a "Run Script" build phase before real
   compilation happen.
2. The "Run Script" should be:
	path/to/swiftpp "$PROJECT_DIR/path-to-your-cxx-Bridging-Header.h"
4. swiftpp will output 4 source files (cxx-objc-protocols.h,
   cxx-objc-proxies.h, cxx-objc-proxies.mm and cxx-subclasses.mm). Those
   need to be added back to your project.
5. cxx-objc-proxies.h need to be #imported in the Objective-C bridge
   header (PROJECT_NAME-Bridging-Header.h)

Those are one-time project setup, should work from now on, everytime you
compile.

Refer to examples/SimpleShapeDetector !

Notes
=====

OK, pretty cool!

But, performance-wise, with all the data conversion that goes on between
the 2 sides, this is not exactly a "bridge" as in the Objective-C/swift
bridge. Still, this is a usefull, auto-generated communication channel
between the 2, and the communication protocol used is defined by a
straight C++ class, no special syntax or idl required.

In that perspective it is more similar to swig, the C/C++ interface
generator, so why not use swig? Well, the 2 major advantages of swiftpp
are 1) it does not require a special "interface file", the C++ class
interface itself is the definition, thanks to clang parsing and 2) swig
does NOT (officially) support Objective-C, making it unsuitable for the
task.

Another tool that was released while I was working on this is djinni
(see https://github.com/dropbox/djinni). It uses a custom idl (Interface
Definition Language) to automatically generate bridges and support C++,
Java and Objective-C.

License
=======

Provided under the MIT license.


Comments are welcome.
