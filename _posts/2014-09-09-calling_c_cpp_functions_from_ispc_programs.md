---
layout: post
title: Calling C/C++ functions from a ispc program
tags: algorithm
---

Recently I started working with [ispc][2] and I needed to call C++ functions
from inside a ispc kernel. The ispc site at the time didn't have a good
solution for this problem, so heres how I solved it.


**Update:** Made a way simpler `Makefile` based on the [Embree][1] build system. Check the last section.


If you attempt to blindly call a C++ function inside an ispc program you will
get a lot of `undefined reference`. The problem is that C++ doesn't have an
standard ABI. The main trick to solve this issue is to wrap your C++ function
into a C function. By doing this we remove the "mangled" function names and
both programs should be able to understand each other.


The process is quite simple, so it's better to show a minimal working code.
This sample will make an ispc program access a `std::map` instance.  First,
lets create a small program with this file structure:

    ./main.cpp        # main program
    ./foo.cpp         # the "foo()" function implementation
    ./Makefile        # our build system
    ./ispc/bar.ispc   # our ispc program (or "bar()" function)
    

We will start working with the `foo()` function below. The `extern "C"` let
your compiler know that it should create a C function interface. Inside we have
a very simple procedure that works with C++ types, in this case a `std::map`.
Note how I'm passing the `std::map` as a `void pointer`. Since this is a C
interface it must follow all the rules, so no references or types with
namespace and any other C++ stuff.

{% highlight cpp %}
    // file: foo.cpp
    #include <map>
    
    extern "C" {
    int foo(void* input_ptr, int i)
    {
       std::map<int,int>* m = static_cast<std::map<int,int>*>(input_ptr);
       return (*m)[i];
    }
    }
{% endhighlight %}
    

Now lets verify how `foo()` is used in our main program below. `foo()` gets an
`extern "C"` in front of its declaration. It's your usual way of using C
functions without including its header. A `map` is populated, and we can
access it by passing its memory address to `foo()`. In here `foo()` works just
like any C function. Next step is to see how `ispc::bar()` uses the `foo()`
function.

{% highlight cpp %}
    // file: main.cpp
    #include <iostream>
    #include <map>
    
    #include "ispc/bar_ispc.h"
    
    using namespace std;
    
    extern "C" int foo(void* input_ptr, int i);
    
    int main()
    {
      map<int, int> input;
      int output[16];
      for (int i = 0; i < 16; ++i) {
        input[i] = i*i;
        output[i] = 0;
      }
      cout << input[5] << endl;
      cout << foo(&input, 5) << endl;
      ispc::bar(&input, 5, 16, output);
    
      for (int i = 0; i < 16; ++i) {
        cout << output[i] << endl;
      }
    }
{% endhighlight %}

The `ispc::bar()` function defined below uses `foo()` almost the same as the
`main()` does, the main difference is in its declaration (note the `uniform`
addition). The `uniform` is required for exchanging data with the ouside world
(everything outside the ispc kernel). The way we use the function is just like
any other C function, but with `uniform` parameters and return types.

{% highlight cpp %}
    // file: ispc/bar.ispc
    extern "C" uniform int foo(void* uniform input_ptr, uniform int i);
    
    export void bar(void* uniform input_ptr, uniform float x,
        uniform int depth, uniform int output[])
    {
      uniform int v = foo(input_ptr, x);
      foreach (i = 0 ... depth) {
        print("%d %f %d\n", i, v, (int)x);
        output[i] = v + i;
      }
    }
{% endhighlight %}

The next step is our build system. We need to compile `foo()`, `main()` and
`bar()` separately, and then link them all together. Following partially the
instructions from the [ispc site][3], I got the Makefile below working. Note
that I'm using `clang`. Others compilers shouldn't work as this solutions
requires the usage of some LLVM intermediate files. The `${LLC_OPTS}` also
should be set properly for your target machine, otherwise you will get a
compilation error.

    // file: Makefile
    ISPC=ispc
    CXX=clang
    LINK=llvm-link-3.4
    OPT=opt-3.4
    LLC=llc-3.4
    LLC_OPTS=-mattr=+avx
    EXE=ispc_cpp
    
    LIBS=-lm -lstdc++
    
    
    all: ispc_cpp
    
    ispc/bar_ispc.bc:
        ${ISPC} --emit-llvm --arch=x86-64 --addressing=64 -o ispc/bar_ispc.bc -h ispc/bar_ispc.h ispc/bar.ispc
    
    foo.bc:
        ${CXX} -O2 -emit-llvm -o foo.bc -c foo.cpp
    
    foo.o: foo.bc ispc/bar_ispc.bc
        ${LINK} foo.bc ispc/bar_ispc.bc -o - | ${OPT} -O3 -o foo_opt.bc
        ${LLC} ${LLC_OPTS} -filetype=obj foo_opt.bc -o foo.o
    
    ispc_cpp: foo.o
        ${CXX} ${LIBS} foo.o main.cpp -o ${EXE}
    
    clean:
        rm -rf *.o
        rm -rf *.bc
        rm -rf ispc/*.bc
        rm -rf ispc/bar_ispc.h
    

Let me know if there is any problem with this code, thank you!

Update
======

It turns out I don't have to do each compilation step manually (kind obvious
now). Below is the way easier `Makefile`: (just like any normal multi-compiler
way of doing things)

    ISPC=ispc
    CXX=clang
    EXE=ispc_cpp
    
    LIBS=-lm -lstdc++
    
    
    all: ispc_cpp
    
    ispc/bar_ispc.o:
        ${ISPC} -o ispc/bar_ispc.o -h ispc/bar_ispc.h ispc/bar.ispc
    
    ispc_cpp: ispc/bar_ispc.o
        ${CXX} ${LIBS} -I./ ispc/bar_ispc.o foo.cpp main.cpp -o ${EXE}
    
    clean:
        rm -rf ispc_cpp
        rm -rf *.o
        rm -rf *.bc
        rm -rf ispc/*.bc
        rm -rf ispc/*.o
        rm -rf ispc/bar_ispc.h

 [1]: http://embree.github.io/
 [2]: https://ispc.github.io/
 [3]: https://ispc.github.io/faq.html#is-it-possible-to-inline-ispc-functions-in-c-c-code
