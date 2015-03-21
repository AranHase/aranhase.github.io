---
layout: post
title: C++ Discoveries - Order of initialization and destruction based on scope
tags: cpp
---
In what order objects are constructed/destroyed inside a block/scope? How
about globals variables? How about static variables?


Was asking myself these questions while debugging a program, so I decided to make a little program to test it and here it is:


{% highlight cpp %}
#include <iostream>
using namespace std;

struct A {
    int i_;
    A(int i) : i_{i} { cout << "A" << i_ << endl; }
    ~A() { cout << "~A" << i_ << endl; }
    A& operator=(const A& a) {/*do nothing on copy...*/}
};

struct B {
    static A a0;
    static A a8;
};

A B::a0{0}; // First to be constructed, last to be destroyed

A a1{1}; // Second construction!

int main() {
    cout << "N1" << endl;
    A a2{2}; // Destroyed at the end of scope
    cout << "N2" << endl;
    const A& a3{3};  // Destroyed at the end of scope, in reverse order
    cout << "N3" << endl;
    A a4{4};
    cout << "N4" << endl;
    a4 = {5}; // A{5} is destroyed at the end of the line!
              // Original a4 will be destroyed at the end;
    cout << "N5" << " ; " << a4.i_ << endl;
    A a6{6};
    A a7{7};
    a6 = a7; // A6 and A7 are destroyed at the end of the scope
    cout << "N6" << endl;
    A{8}; // Destroyed at the end of the line
    cout << "N7" << endl;
    return 0;
}

A a7{7}; // Third construction
A B::a8{8}; // Fourth to be constructed
{% endhighlight %}

Running this program gives me:

{% highlight text %}
A0
A1
A7
A8
N1
A2
N2
A3
N3
A4
N4
A5
~A5
N5 ; 4
A6
A7
N6
A8
~A8
N7
~A7
~A6
~A4
~A3
~A2
~A8
~A7
~A1
~A0
{% endhighlight %}

# Important notes:

 * Objects not being held by variables are destroyed at the end of the line;
 * `static` and `global` variables are the first to be constructed, in order. They are also the last to be destroyed, in reverse order.

