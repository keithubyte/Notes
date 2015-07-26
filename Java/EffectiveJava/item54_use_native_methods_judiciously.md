### Item54 : Use native methods judiciously

----------

The Java Native Interface (JNI) allows Java applications to call *native methods*, which are special methods written in *native programming languages* such as C or C++.

Historically, native methods have had three main uses:

- They provided access to platform-specific facilities such as registries and file locks.
- They provided access to libraries of legacy code, which could in turn provide access to legacy data.
- native methods were used to write performance-critical parts of applications in native languages for improved performance.

It is legitimate to use native methods to access platform-specific facilities, but as the Java platform matures, it provides more and more features previously found only in host platforms.

**It is rarely advisable to use native methods for improved performance**. In early releases (prior to 1.3), it was often necessary, but JVM implementations have gotten **much faster**. For most tasks, it is now possible to obtain comparable performance without resorting to native methods.

The use of native methods has serious disadvantages. 

- Because native languages are not safe, applications using native methods are no longer immune to memory corruption errors. 
- Because native languages are platform dependent, applications using native methods are far less portable. 
- Applications using native code are far more difficult to debug. 
- There is a fixed cost associated with going into and out of native code, so native methods can decrease performance if they do only a small amount of work. 
- Finally, native methods require "glue code" that is difficult to read and tedious to write.

#### Summary

In summary, think twice before using native methods. Rarely, if ever, use them for improved performance. If you must use native methods to access low-level resources or legacy libraries, use as little native code as possible and test it thoroughly. A single bug in the native code can corrupt your entire application.