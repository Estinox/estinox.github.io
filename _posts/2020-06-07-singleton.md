---
layout: post
title: Staying Perfectly Singleton
tags:
    - cpp
---

![png]({{ site.url }}/assets/one_and_only.jpeg)

### When do we need a singleton?
This can be a contentious topic.

There are always ways around a singleton, but it does make life easier when used correctly. However, it's true that global variables are frawned upon, so we must think carefully when embarking upon this single path.
<!--more-->

An alternative for singleton could be passing an instantiated obj around, parameterizing *all* the classes that needs it. Sure, this could work if you own the entire code base (ie: you are both the library developer and the client), and do not mind the time-consuming work of adding doing so. Nice thing is, this will also make the dependencies explicit, making it easier to test and debug. 

I like this example, std::cout, a single global resource, do you really want to parameterize this variable for all client programs? Imagine what you have to do if it wasn't global.

Regardless of the implementation, use singleton when:
    - Global access is very useful (ie: logger, thread pool, configuration manager)
    - One and only one will ever be needed - multiple instances creates an invalid state

### How to properly implement one?

Long story short, there are several ways of doing this. The best way I've found is to define a static variable within a namespaced non-member function:

{% highlight cpp %}

namespace Singleton {
  auto& instance() {
    static std::unordered_map<int, std::string> _instance;
      return _instance;
  }
};

{% endhighlight %}

This guarantees destruction at the end of program life cycle compared to new-ing, and ensures the singleton is always created on use.


#### Additional Implementation Considerations
  - This is not thread-safe as noted here
    - Order of destruction
      - Performance : magic static - compiler runs a small check every time to make sure static variables are always instantiated

### References:
      - https://stackoverflow.com/questions/1434937/namespace-functions-versus-static-methods-on-a-class
      - http://www.gotw.ca/gotw/084.htm
      - Magic Statics: https://herbsutter.com/2013/09/09/visual-studio-2013-rc-is-now-available/

