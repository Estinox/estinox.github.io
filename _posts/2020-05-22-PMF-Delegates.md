---
layout: post
title: Pointer to Member Function as Delegates
tags:
    - cpp
---

![png]({{ site.url }}/assets/delegates.png)

Over the past few days, I've been trying to find a way to store different PMF (pointer to member functions) in a container as delegates. And I wanted to type erase the PMF so I can store it in a container along with other typed PMF delegates. The only thing common tying the delegates together would be its call signature.

<!--more-->

Straight to the use case:

{% highlight cpp %}

int main() {
    auto delegate_container = DelegateContainer<void()>{};
    auto p1 = PlusOne();
    auto p2 = PlusTwo();

    delegate_container.insert(&PlusOne::plus_one, &p1);
    delegate_container.insert(&PlusTwo::plus_two, &p2);

    std::cout << "p1.val=" << p1.val << std::endl;
    std::cout << "p2.val=" << p2.val << std::endl;

    delegate_container.call();

    std::cout << "p1.val=" << p1.val << std::endl;
    std::cout << "p2.val=" << p2.val << std::endl;
}

{% endhighlight %}

You can see here, I'm able to store two different PMF's in the same container. The idea here is to capture the function pointer of each callable type as void * and invoke it when asked. Implementation below:

{% highlight cpp %}

#include <iostream>
#include <vector>

struct PlusOne {
    void plus_one() { val += 1; }
    int val = 0;
};

struct PlusTwo {
    void plus_two() { val += 2; }
    int val = 0;
};

struct Delegate {
    void *fp;
    void *obj;
};

template <typename CallSignature> struct DelegateContainer;

template <typename... Args> struct DelegateContainer<void(Args...)> {
    using call_signature = void (*)(void * /*obj*/, Args...);

    template <typename T> void insert(void (T::*fp)(Args...), T *obj) {
        _delegates.push_back(Delegate{reinterpret_cast<void *>(fp),
                                      reinterpret_cast<void *>(obj)});
    }

    void call(Args &&... args) {
        for (auto &d : _delegates) {
            (reinterpret_cast<call_signature>(d.fp))(d.obj, 
                                                     std::forward(args)...);
        }   
    }   

    std::vector<Delegate> _delegates;
};
{% endhighlight %}

So the magic here is how we reinterpreted cast both the PMF and object pointer to void *, and then stored the call structure in a typedef.

The idea came from the [Type Erasure CppCon talk](https://www.youtube.com/watch?v=tbUCHifyT24){:target="blank"} by Arthur O'Dwyer, full source code can be found [here](https://github.com/Estinox/coding-practices/blob/master/random_code/pmf_delegates.cpp){:target="blank"}, useful blog referenced [here](https://www.codeproject.com/Articles/1170503/The-Impossibly-Fast-Cplusplus-Delegates-Fixed?fid=1917788&df=90&mpp=25&sort=Position&spc=Relaxed&select=5700741&prof=True&view=Normal&fr=1#xx0xx){:target="blank"}.

