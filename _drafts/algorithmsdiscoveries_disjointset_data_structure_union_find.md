---
layout: post
title: Algorithms Discoveries - Disjoint-set Data Structure and Union-Find
tags: algorithm
---

If you have many sets of non-overlapping elements, how to quickly find if two elements are in the same set? This problem can be solved using the **Union-Find algorithms**

Below I'll place a few notes for a few algorithms I learned from Coursera Algorithms, Part I.

Quick-Find
==========

## Main ideas:

 * Array of "ids"
 * Connected components will have the same "id"
 * If two different components have the same "id", then they are connected (same set)
 * **Union is expensive**

## Code:

{% highlight cpp %}
#include <iostream>
#include <array>

using namespace std;

template <int N>
struct QuickFind {
    array<int, N> ids_;
    
    QuickFind() {
        for (int i = 0; i < N; ++i) {
            ids_[i] = i;
        }
    }
    
    bool Find(int p, int q) {
        cout << "Find(" + to_string(p) +
            "," + to_string(q) + ") = " +
            ((ids_[p] == ids_[q]) ? "Yeap" : "No")
            << endl;
        // constant time verification
        // Same ID == Same Set!
        return ids_[p] == ids_[q];
    }
    void Union(int p, int q) {
        cout << "Union(" + to_string(p) +
            "," + to_string(q) + ")" << endl;
        
        // Sets q's ID over the p's ID;
        // Do this for all elements with the p's ID.
        int old_p = ids_[p];
        ids_[p] = ids_[q];
        for (auto& i : ids_) {
            if (i == old_p) {
                i = ids_[q];
            }
        }
    }
};

int main(void) {
    QuickFind<5> qf{};
    qf.Find(1,2);
    qf.Union(1,2);
    qf.Find(1,2);
    qf.Union(3,4);
    qf.Find(3,4);
    qf.Find(1,4);
    qf.Union(3,0);
    qf.Find(0,4);
    qf.Find(1,4);
    for (auto& v : qf.ids_) {
        cout << to_string(v) << ", ";
    }
    cout << endl;
    return 0;
}
{% endhighlight %}


Quick-Union
===========

# Main Ideas:

 * Avoid doing work until have to
 * Creates a forest
 * Each tree will be a disjoint set
 * Each element will point to the tree's root
 * The find operation verify the root of each element. If the roots are the same, than they belong to the same set.
 * **Trees can get too tall!**
 * **Find is expensive**

# Code:

{% highlight cpp %}
#include <iostream>
#include <array>

using namespace std;

template <int N>
struct QuickUnion {
    array<int, N> ids_;
    
    QuickUnion() {
        // each element points to itself;
        // roots points to itself;
        // so, each element is a root.
        for (int i = 0; i < N; i++) {
            ids_[i] = i;
        }
    }
    
    bool Find(int p, int q) {
        cout << "Find(" + to_string(p) +
            "," + to_string(q) + ") = " +
            ((ids_[p] == ids_[q]) ? "Yeap" : "No")
            << endl;
            
        return Root(p) == Root(q);
    }
    // Union just changes the p's root!
    void Union(int p, int q) {
        cout << "Union(" + to_string(p) +
            "," + to_string(q) + ")" << endl;
            
        int i = Root(p);
        int j = Root(q);
        ids_[i] = j;
    }
private:
    int Root(int i) {
        // roots points to itself;
        while (ids_[i] != i) i = ids_[i];
        return i;
    }
};

int main(void) {
    QuickUnion<5> qu{};
    qu.Find(1,2);
    qu.Union(1,2);
    qu.Find(1,2);
    qu.Union(3,4);
    qu.Find(3,4);
    qu.Find(1,4);
    qu.Union(3,0);
    qu.Find(0,4);
    qu.Find(1,4);
    for (auto& v : qu.ids_) {
        cout << to_string(v) << ", ";
    }
    cout << endl;
    return 0;
}
{% endhighlight %}


Weighted Quick-Union
====================

# Main Ideas:
 * **Same idea from Quick-Union, BUT, trees will not get too tall!**
 * When creating an union, plug the smaller tree into the bigger tree
 * Find is proportional to the height of the element =P
 * Depth of any node is at most lg N

#Code:

{% highlight cpp %}
#include <iostream>
#include <array>

using namespace std;

template <int N>
struct WeightedQuickUnion {
    array<int, N> ids_;
    array<int, N> szs_;
    
    WeightedQuickUnion() {
        // each element points to itself;
        // roots points to itself;
        // so, each element is a root.
        for (int i = 0; i < N; i++) {
            ids_[i] = i;
            // each tree will start with size 1
            szs_[i] = 1;
        }
    }
    
    bool Find(int p, int q) {
        cout << "Find(" + to_string(p) +
            "," + to_string(q) + ") = " +
            ((ids_[p] == ids_[q]) ? "Yeap" : "No")
            << endl;
        // same as Quick-Union
        return Root(p) == Root(q);
    }
    // Union just changes the p's root!
    void Union(int p, int q) {
        cout << "Union(" + to_string(p) +
            "," + to_string(q) + ")" << endl;
            
        int i = Root(p);
        int j = Root(q);
        // Bigger tree will be the root!
        if (szs_[i] < szs_[j]) {
            ids_[i] = j;
            szs_[j] += szs_[i];
        } else {
            ids_[j] = i;
            szs_[i] += szs_[j];
        }
    }
private:
    // same as Quick-Union
    int Root(int i) {
        // roots points to itself;
        while (ids_[i] != i) i = ids_[i];
        return i;
    }
};

int main(void) {
    WeightedQuickUnion<5> qu{};
    qu.Find(1,2);
    qu.Union(1,2);
    qu.Find(1,2);
    qu.Union(3,4);
    qu.Find(3,4);
    qu.Find(1,4);
    qu.Union(3,0);
    qu.Find(0,4);
    qu.Find(1,4);
    for (auto& v : qu.ids_) {
        cout << to_string(v) << ", ";
    }
    cout << endl;
    for (auto& v : qu.szs_) {
        cout << to_string(v) << ", ";
    }
    cout << endl;
    return 0;
}
{% endhighlight %}

Union-Find algorithms time complexity
=====================================


| Algorithm | Initialization | Union | Find |
|:---------:|:--------------:|:-----:|:----:|
| Quick-Find | $$\mathcal{O}(n)$$ | $$\mathcal{O}(n)$$ | $$\mathcal{O}(1)$$ |
| Quick-Union | $$\mathcal{O}(n)$$ | $$\mathcal{O}(n)$$(because of find root) | $$\mathcal{O}(n)$$ |
| Weighted Quick-Union | $$\mathcal{O}(n)$$ | $$\mathcal{O}(\log n)$$ | $$\mathcal{O}(\log n)$$ |


