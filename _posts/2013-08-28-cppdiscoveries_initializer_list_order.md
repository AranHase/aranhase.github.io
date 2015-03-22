---
layout: post
title: C++ Discoveries - Initializer List Order
tags: cpp
---

When a constructor with an initializer list is called over another constructor? Below is my experimentation:


{% highlight cpp %}
#include <iostream>
#include <vector>
#include <initializer_list>

using namespace std;

struct Vec {
    vector<double> v_;
    
    Vec(int dimensions = 2,double value = 0) : v_(dimensions,value) {cout << "Started without list" << endl;}
    Vec(initializer_list<double> list) : v_(list) {cout << "Started with list" << endl;}
    
    void print() {
        cout << "V = {";
        for (auto& v : v_) cout << v << ",";
        cout << "};" << endl;
    }
    
    
};

int main(void) {
    Vec v1; //Started without list
    v1.print();
    Vec v2{3,5}; //Started with list
    v2.print();
    Vec v3 = {5,5,4}; //Started with list
    v3.print();
    Vec v4(10); //Started without list
    v4. print();
    Vec v5(1,5); //Started without list
    v5.print();
    Vec v6{}; //Started without list
    v6.print();
    Vec v7{1}; //Started with list
    Vec v8{int(4),double(1)}; //Started with list
    return 0;
}
{% endhighlight %}
