# About com4j
com4j is a java library developed by [Kohsuke Kawaguchi](https://github.com/kohsuke).  
It's pretty old by today's standards, but the Microsoft technology it binds to is old as well. The library is pretty versatile - it can generate java code from type-library definitions (via _tlbimp.jar_, which you will be able to build from source) and use those definitions.

## Things to remember
- `tlbimp.jar` will likely create one class named `ClassFactory`, which should be your main entry point to creation of instances of implementations of TypeLibrary-defined interfaces.
- don't cast the interfaces/classes to your expected type, use `.queryInterface()` method.
  - for example:
  ```java
  SomeClass someClass = ClassFactory.createSomeClass();
  SomeInterface someIface = someClass.queryInterface(SomeInterface.class);
  ```
  instead of:
  ```java
  SomeClass someIface = ClassFactory.createSomeClass();
  SomeInterface someIface = (SomeInterface)someIface;
  ```
- whenever a generated interface method expects an `Object` as an input, it is likely that it actually wants a `Variant` and probably of type `VT_EMPTY`, which you can get by `Variant vt = new Variant();`
  - at least that is the case for `MSFileReader` library from Thermo Scientific that I'm using it for. In my case the expected input types were kind of documented, but otherwise you have no better choice than to provide a `Variant` which is `VT_EMPTY`.
  - Upon return from the method call you should examine the `Varinat`'s type (if it has changed from `VT_EMPTY`) and to get the contents (e.g. some array) call `vt.get()` and then cast to an appropriate type. I devise the type by running the code in a debugger and examining the return type manually.



## This fork of com4j
This fork was only created to address some issues with arrays that are not part of the binary build which you can clone from the original repo. This fork was recompiled using MSVS 2013 and includes minor custom tweaks.  
The major thing is inclusion of _system dependent_ __include directories__ to compiler directives which provide the path to _jni.h_ and friends (so you'll need to change that in visual studio by right clicking the project, selecting `Properties -> C/C++ -> General -> Include Directories`).  
Same applies to the _jnitl_ submodule.


## Building this clone
Building com4j is divided into two parts. Native and Java.

To build:

- clone
- `git submodule init`
- `git submodule update`
- Visual Studio 2013
    - in the `jnitl` subdirectory's MSVS project, replace C++ include directories to match your JDK's `<jdk-path>/include` and `<jdk-path>/include/win32` direcotries
    - first build `jnitl` project
    - then build `libffi` project
    - int the main com4j project (`native` directory), replace C++ include directories to match your JDK's `<jdk-path>/include` and `<jdk-path>/include/win32` direcotries
    - do the builds for all combos of Debug/Release and x32/x64 build configurations (you should have done the same for `jnitl` and `libffi`)


## javah
If you change the Java classes that define native methods, be sure to execute `native/run_javah.bat` to keep header files in sync
