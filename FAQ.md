

# About the project #

## What is BridJ ? ##

BridJ is a library that lets you call [C](CLanguage.md), [C++](CPlusPlus.md) and [Objective-C](ObjectiveC.md) libraries from Java as if you were writing native code. It is a recent alternative to [JNA](http://jna.dev.java.net).

With the help of [JNAerator](http://jnaerator.googlecode.com/), it provides [clean and unambiguous mappings](TypeMapping.md) to C/C++ code and a very straightforward and highly typed API to manipulate pointers, classes, structs, unions...

It bridges Java's gap when it comes to native-world interoperability.

<table><tr valign='top'><td>
For instance, you can allocate a <code>new float[10]</code> almost as in C++ :<br>
<pre><code>void someFunc(float* values);<br>
<br>
float* array = new float[10];<br>
float value = array[4];<br>
someFunc(array);<br>
delete[] array;<br>
</code></pre>
</td></tr><tr valign='top'><td>
Scala code :<br>
<pre><code>import org.bridj.Pointer._<br>
...<br>
@native def someFunc(values: Pointer[Float]): Unit<br>
...<br>
val array = allocateFloats(10)<br>
val value = array(4)<br>
someFunc(values)<br>
<br>
// optional : will be eventually called by the Garbage Collector<br>
array.release <br>
</code></pre>
</td><td>
Java code :<br>
<pre><code>import org.bridj.Pointer;<br>
import static org.bridj.Pointer.*;<br>
...<br>
public static native void someFunc(Pointer&lt;Float&gt; values);<br>
...<br>
Pointer&lt;Float&gt; array = allocateFloats(10);<br>
float value = array.get(4);<br>
someFunc(values);<br>
array.release();<br>
</code></pre>
</td></tr></table>

Read more about :
  * [Pointers](Pointers.md)
  * [TypeMapping](TypeMapping.md)
  * [CLanguage](CLanguage.md)
  * [CPlusPlus](CPlusPlus.md)
  * [COM](COM.md)
  * [Design](Design.md)
  * [Using BridJ with Scala](BridJAndScala.md)

## I found a bug in BridJ ! Where do I report it ? ##

Getting lots of bug reports is crucial to making BridJ better, so we'd like to encourage you to file issues as you find them without worrying too much about the contents : it's often better for us to have an incomplete report than no report at all, and anyway we'll let you know very politely if your issue comes from a misuse rather than a bug in BridJ.

Here's the issue tracker : [NativeLibs4Java's Issues on GitHub](https://github.com/ochafik/nativelibs4java/issues).

And here's a guide on [How to Submit Bugs](SubmittingBugs.md).

## Why another interop library ? ##

In three words : [C++, Performance and Usability](Design.md)

## What is BridJ's license ? ##

BridJ is [dual-licensed under a very liberal BSD license and the very liberal Apache 2.0 license](CreditsAndLicense.md).

## What are BridJ's runtime dependencies ? ##

BridJ is released as a self-contained JAR.

However, it includes a copy of the [ASM](http://asm.ow2.org/) library and its native library are statically linked with a modified version of [Dyncall](http://dyncall.org).

More details here : [CreditsAndLicense](CreditsAndLicense.md).

## Where can I get support for BridJ ? ##

On [NativeLibs4Java's mailing-list](http://groups.google.com/group/nativelibs4java).

## What is BridJ's development status ? ##

Please read the [Change Log](https://github.com/ochafik/nativelibs4java/blob/master/libraries/Runtime/BridJ/CHANGELOG) and have a look at the [CurrentState](CurrentState.md) page.

## What platforms does BridJ support ? ##

Binaries are currently available and routinely tested for the following platforms :
  * Windows (x86, x64)
  * Linux (x86, x64, armhf, armel)
  * Mac OS X (x86, x64, ppc)
  * Solaris 10 (x86... soon x64 and sparc)
  * Android 3.0 (armv5)

([help and / or test hardward welcome](CreditsAndLicense.md) to help with other platforms)

## Can I build BridJ from sources ? ##

Sure, please read [Build](Build.md).

## How can I help ? ##

Please read [HowToHelp](HowToHelp.md).

## How is BridJ different from... ##

### JNA ###

Things BridJ aims at doing better than JNA :
  * BridJ is BSD-licensed (while JNA is LGPL)
  * [Supports C++](CPlusPlus.md) (really : virtual methods, inheritance, templates...)
  * Vastly better structs : lightweight, fast (one or two orders of magnitude faster to create), without any data duplication
  * Better performance for most function calls :
    * Direct mode is the default
    * Some calls are optimized with assembly code to reach near-JNI performance
  * [Supports COM](COM.md) (special case of C++ support) : lets you use "real" Windows APIs (IShellFolder, ITaskBarList3...)
  * [Cleaner API](TypeMapping.md) with Java 1.5 generics :
    * [Iterable and typed `Pointer&lt;T&gt;`](Pointers.md)
```
import static org.bridj.Pointer.*;
...
Pointer<Integer> p = pointerToInts(1, 2, 3, 4);
p.set(0, 10);
for (int v : p)
  System.out.println("Next value : " + v);
```
    * Typed enums (that can still be combined as flags)
    * No such thing as IntByReference, LongByReference... Simply use `Pointer<Integer>`, `Pointer<Long>`...
    * No such thing as Structure.ByReference vs. Structure.ByValue : if a function accepts a pointer to MyStruct, it's gonna be `f(Pointer<MyStruct> s)`, if it accepts a MyStruct by value it will be `f(MyStruct s)`. Simple.
    * Wanna cast a pointer to a function pointer ? `ptr.as(MyFunctionPointer.class)`.
  * Pervasive multiple endianness support : if you retrieved a pointer to some GPU-filled memory and you know the GPU is little-endian while your CPU is big-endian, you can just call `myPtr.order(ByteOrder.LITTLE_ENDIAN)`. You can then retrieve data from it as usual, and even cast to a struct pointer : it will just work as intended.
  * BridJ has better [JNAerator](http://jnaerator.googlecode.com/) support (in particular, the experimental `-convertBodies` option will convert function bodies from C/C++ into Java/BridJ code !), so you won't need to write any interface by hand.

JNA's advantages over BridJ :
  * BridJ doesn't handle structs by value in function calls yet (neither in arguments nor in return value). This is being worked on, though.
  * JNA is Java 1.4 compatible (while BridJ requires Java 1.5+)
  * JNA is mature and stable, used by **many** projects
  * BridJ supports less platforms (Windows 32/64, MacOS X Universal, Linux x86/amd64/armhf/armel for now)
  * JNA has tons of working examples

### SWIG ###

BridJ's advantages :
  * it only requires you to build your Java program, whereas SWIG requires a painful native build on every target platform
  * the resulting API is generally nicer looking, without any manual configuration involved
  * you work directly with the original C/C++ headers, no need to write special header as with SWIG

SWIG's advantages :
  * it's C++ support is much more solid : once it compiles, you're pretty sure it's gonna work (with BridJ, errors will show up at runtime, typically at startup)
  * it has support for STL types (std::string, std::vector...)
  * it's more mature and stable

## Who is using BridJ ? ##

Please read [Who's using BridJ](WhoUsesBridJ.md).

# Usage #

## How do I bind C / C++ function / struct / class / type XXX with BridJ ? ##

Just paste your C header [into JNAerator](http://jnaerator.sourceforge.net/webstart/JNAerator/JNAeratorStudio.jnlp), choose BridJ as runtime and click on JNAerate.

## Can I ship my library's binaries in a cross-platform JAR ? ##

You sure can ! BridJ provides a seamless cross-platform binaries JAR extraction mechanism, which it uses for its own needs.

For detailed information about embedded libraries, please read [Libraries Lookup](LibrariesLookup.md).

## How can I trace/log the native calls ? ##

You can set the `-Dbridj.logCalls=true` property or the BRIDJ\_LOG\_CALLS=1 environment variable.

In this log mode :
  * only the Java method will be logged (no arguments nor return value)
  * direct calls will be disabled (those wired with optimized assembler glue), so performance might be degraded a bit

## How to specify different calling conventions for different platforms ? ##

Short answer is : you don't need to.

Only Windows x86 has different calling conventions, so `@Convention` annotations will typically be ignored silently on other platforms.

## How to I check for errno / `GetLastError()` ? ##

Simply declare that your function throws [LastError](http://nativelibs4java.sourceforge.net/bridj/api/development/org/bridj/LastError.html), that's it !

You'll get exceptions soon enough ;-)

# Troubleshooting #

## BridJ hangs with 100% CPU usage ! ##

If you're on Windows x86 (or on 64 bits windows in a 32 bits JVM), this is typically a case of bad calling convention being used for a native function or callback binding.
Please triple-check the calling convention (could it be stdcall ?) and add a proper `@Convention(Convention.Style.xxx)` annotation.

## Segmentation faults when using callbacks? ##

You need to keep strong references to callbacks, otherwise they'll be garbage-collected and the native code will crash when trying to call them.

You can use [BridJ.protectFromGC](http://nativelibs4java.sourceforge.net/bridj/api/development/org/bridj/BridJ.html#protectFromGC(T)) to do that, or use your own strong references with finer-grained lifespan.

## How do I debug an EXCEPTION\_ACCESS\_VIOLATION crash ? ##

This crash indicates that the program tried to access memory it wasn't supposed to access (for instance someone read at index 100 in an array of 10 elements).

It may also be caused by an invalid calling convention being used, see previous FAQ item)

The tricky part here is that this kind of errors usually does not surface immediately : the crash will typically occur in later calls, giving false leads on where the error actually occurred.

The only reliable way to debug such errors is to make use of a native debugger. On Windows, Visual C++ comes with a very nice debugger and is available for free in its Express edition (under some conditions).

To debug your crash with Visual C++, you should compile your native library in debug mode if possible, then run the java process with the debugger (or attach a debugger to it before any native call is performed). You'll want to turn as many debug checks on as possible in the [debugger settings](http://msdn.microsoft.com/en-us/library/x85tt0dd.aspx).

Also, it will be easier to set the full path to the debug version of your DLL using a manual path override : if your library is named `MyLib` in BridJ (the value inside the `@Library("MyLib")` annotation), you can :
  * Set the path in the run properties dialog with the following environment variable :
```
    BRIDJ_MYLIB_LIBRARY=c:\...\MyProject\Debug\MyLib.dll
```
  * Or add the following property argument to your java process command-line :
```
    "-Dbridj.MyLib.library=c:\...\MyLib.dll"
```
You can then check which exact version of your DLL was picked by BridJ by scanning through Visual C++'s "Modules" window.

As far as BridJ is concerned, you may try turning the direct mode off with BRIDJ\_DIRECT=0 or -Dbridj.direct=false and see if the issue still happens.

A last option would be to turn BridJ's "protected" mode on, but right now you'd have to [recompile it from sources](Build.md) and uncomment the `#define ENABLE_PROTECTED_MODE` directive in `Exceptions.h`. Note that protected mode is currently slow and unstable which is why it's not enabled by default ;-).

## How do I debug a `LoadLibrary : The specified module could not be found` error ? ##

If BridJ fails to load a library, it can be that :
  * you're trying to load a 64 bits on a 32 bits platform, or vice-versa (please check which version `java -version` yields : if it does not say it's 64 bits, it means it's 32 bits)
  * it didn't find the exact path you specified
  * it didn't find a DLL dependency of the library you're trying to load (please open your DLL with [Dependency Walker](http://www.dependencywalker.com/) and make sure all its dependencies are resolved and in the path). Dependent libraries must be declared as:
```
@Library("MainLib", dependencies = { "Dependency1", "Dependency2"... })
```

## How do I debug an `UnsatisfiedLinkError`? ##

This means your native method was not successfully registered by BridJ. You should:
  * Make sure you've added a `static { BridJ.register(); }` block in your class
  * Inspect BridJ's verbose logs (set the `BRIDJ_VERBOSE=1` or `BRIDJ_DEBUG=1` environment variables, or the `-Dbridj.verbose=true` or `-Dbridj.debug=true` System properties through command-line flags)

If the logs show that BridJ did try and register your method(s) but failed to find the relevant symbols in your shared library (.dll, .so, .dylib), then:
  * Make sure you've got the right DLL name
  * Inspect the library's symbols with Dependency Walker (for Windows DLLs) or `nm` / `otool` (for Unix binaries)

If the (mangled) symbol is indeed present in the library, BridJ's demangler might have a bug (or two) that prevented it from matching the symbol with your signature. You can try and put the raw (mangled) symbol in a `org.bridj.ann.Symbol` annotation on the method.

If none of this works, please [file a bug](https://github.com/ochafik/nativelibs4java/issues) or [ask the mailing-list](http://groups.google.com/group/nativelibs4java).