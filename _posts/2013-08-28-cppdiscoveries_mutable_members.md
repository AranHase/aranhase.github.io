---
layout: post
title: C++ Discoveries - Mutable Members
tags: cpp
---


Sometimes we need to change values of a *member variable* inside of a *const
method*. This is common when we may want to buffer the return of the method,
think like a memoization technique. But how to do it?


In C++ we can change *member variables* of a class through a *const method*
with the **mutable** type modifier. What this modifier does is tell that the
*member* do not change the way the object looks to the outside world, but can
still be modified.

If we try to modify a *member variable* through a *const method*, we will get
an error like this:

`error: assignment of member ‘MyVecSum::cached_sum_’
in read-only object`

But, if we add the **mutable** modifier, the *member variable* can be modified
even within a *const method*. Since this is commonly used in caching, I made a
really simple example:


{% highlight cpp %}
#include <iostream>
#include <memory>
#include <vector>

using namespace std;

struct SumInterface {
    virtual ~SumInterface() {};
    virtual void addInt(int) = 0;
    virtual int getSum() const = 0;
};

struct MyVecSum : public SumInterface {
    vector<int> vector_;
    mutable bool cached_; // mutable!
    mutable int cached_sum_; // can be changed even with const methods
    
    void addInt(int i) {
        vector_.push_back(i);
        cached_ = false;
    }
    
    // const method
    int getSum() const {
        if (!cached_) {
            cached_sum_ = 0; // changing object state :)
            for (const auto v : vector_) {
                cached_sum_ += v;
            }
            cached_ = true;
        }
        
        return cached_sum_;
    }
};

int main(void) {
    unique_ptr<SumInterface> sumInterface(new MyVecSum());
    sumInterface->addInt(10);
    sumInterface->addInt(20);
    cout << sumInterface->getSum() << endl;
    cout << sumInterface->getSum() << endl;
    cout << sumInterface->getSum() << endl;
    return 0;
}
{% endhighlight %}
