---
layout: post
title: Variadic Event Signals
tags:
    - cpp
---
![png]({{ site.url }}/assets/20161113_variadic_signals/smoke_signal.jpg)

Probably should've posted this before the [previous]({{ site.url }}/2016/10/29/Meta-Rotate-Cpp/) Variadic TMP write-up. Anyhow, a variadic template is a new feature in C++11 that made the old va_args completely obsolete (woot!). The new variadic inherits the "..." (ellipses) operator, which expands a pack of zero or more types or value arguments.

<!--more-->

To see it in action, this is a bit of an old code; I was writing up my own event-driven engine, which is a container of function pointers that are called when arguments are passed to them. The function pointer would come in different flavors depending on the paramenters needed of the receiving functions, whch sounded like a perfect job for variadic templates. I was able to get it working, until I realized that [boost::signals2](http://www.boost.org/doc/libs/1_62_0/doc/html/signals2.html){:target="blank"} does exactly the same thing. Nonetheless, it was a great programming excercise, and let's step through the [source code](https://github.com/Estinox/coding-practices/blob/master/random_code/subscriber_observer.cpp){:target="blank"}.

{% highlight cpp %}

template <typename... Args> class Signal {
  using fp = void (*)(Args...);
  std::vector<fp> subscribers_;

  public:
    Signal() : subscribers_(){};

    void subscribe(fp f) {
      subscribers_.push_back(f);
    }

    void notify(Args... args) {
      std::for_each(subscribers_.begin(),
              subscribers_.end(),
              [&](auto& func){func(args...);});
    }
};

{% endhighlight %}

You would start off the header like any template, except the ... follows typename, which implies that Args is a list of arbitrary length type names. 

{% highlight cpp %}

  using fp = void (*)(Args...);
  std::vector<fp> subscribers_;

{% endhighlight %}

There's a lot of magic in the fp typedef. Before it conceals all the ugliness of the function pointer syntax, it's taking in the user defined parameter pac of the template and expanding it out into the parameter definition of the function pointer. So for example, if we instantiated Signal<int, double>, it'll mean that any function pointer taking in an int then a double, returning void, can be added to the subscribers_ vector.


{% highlight cpp %}
    void subscribe(fp f) {
      subscribers_.push_back(f);
    }

    void notify(Args... args) {
      std::for_each(subscribers_.begin(),
              subscribers_.end(),
              [&](auto& func){func(args...);});
    }
{% endhighlight %}

Nothing too fancy here, we have an inserter then a notify function that takes in the exact same args to what the function pointer expects. Then using the std::for_each loop, we create a generic lambda that invoke all the function pointers within our vector.

And that's it! Here's a main that drives all of this. 

{% highlight cpp %}

void multiply(int a, int b)
{
  std::cout << a << " times "
      << b  << " is " << a*b << std::endl;
}

void subtract(int a, int b)
{
  std::cout << a << " minus " << b << " is " << a-b << std::endl;
}


int main() {
  auto signal = Signal<int, int>();

  signal.subscribe(&multiply);
  signal.subscribe(&subtract);
  signal.notify(3,1);
  signal.notify(8,2);
}

{% endhighlight %}

The main should output:

3 times 1 is 3

3 minus 1 is 2

8 times 2 is 16

8 minus 2 is 6

Alright, that's a wrap! If anyone has any questions, feel free to email me.
