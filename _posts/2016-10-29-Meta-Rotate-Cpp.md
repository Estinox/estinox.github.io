---
layout: post
title: Rotating Variadic Template Metaprogramming
tags:
    - cpp
---
![png]({{ site.url }}/assets/20161029_tmp_rotate/musical_chairs.jpg)

Happy Halloween! If anyone is looking for a well-presented C++ core concept video, at msdn, Stephen T. Lavavej, has a series of [videos](https://channel9.msdn.com/Series/C9-Lectures-Stephan-T-Lavavej-Core-C-){:target="blank"} which are highly recommended.

I’m sure many of us have heard how the compiler is Turing complete. In Stephen’s [8th video](https://channel9.msdn.com/Series/C9-Lectures-Stephan-T-Lavavej-Core-C-/Stephan-T-Lavavej-Core-Cpp-8-of-n){:target="blank"}, he showed us how to manipulate template parameters to sort an int array of arbitrary length during compile time using variadic template metaprogramming. Mind = Blown.

Okay, the TMP isn't rotating, but I thought I get my hands dirty by implementing the [rotate](http://en.cppreference.com/w/cpp/algorithm/rotate){:target="blank"} algorithm from STL’s algorithm library. Rotate takes a list and shuffles all the elements forward by one, with the element at the front of the list being pushed to the back.

<!--more-->

I like to have my conclusion at the top for those with shorter attention spans, such as myself. TLDR, variadic template meteprgramming is essentially a special syntax for writing recurision with the compiler. Each parameter pack is expanded until the recursion's base case, which is typically a class template specialization with no entries. Something that took a bit for me to get used to is that partial specialization can have an arbitrary number of deduced template types that does not need to match the primary template, and these deduced types can be nested in other data structures, cool!


Alright, let's see some code. First we need to define a non-type variadic template that can hold the arbitrary length of ints we pass into.

{% highlight cpp %}
template <int... Vals> struct Ints {};
{% endhighlight %}

Our primary template, Rotate, takes in a single type that's specialized into taking an Ints struct holding an arbitrary number of ints.

{% highlight cpp %}
template <typename T> struct Rotate;

template <int One, int... Two>
struct Rotate<Ints<One, Two...>> {
    using type = Ints<Two..., One>;
};
{% endhighlight %}

To print out the rotated ints, we can declare another class template, MakeArray, that'll turn our types into a std::array.

{% highlight cpp %}
template <typename Ints> struct MakeArray;

template <int... Vals> struct MakeArray<Ints<Vals...>> {
    static std::array<int, sizeof...(Vals)> arr;
};

template <int... Vals>
std::array<int, sizeof...(Vals)>
MakeArray<Ints<Vals...>>::arr = {{Vals...}};

{% endhighlight %}

And to put all this together, we will need to pass in the rotated type from Rotate into MakeArray.

{% highlight cpp %}
template <int... Vals>
struct RotateArray 
  : public MakeArray<typename Rotate<Ints<Vals...>>::type> {};
{% endhighlight %}

A sample call from the main would be:

{% highlight cpp %}
int main() {
  using type = RotateArray<1, 2, 3, 4, 5>;
  for (auto const &v : type::arr)
    std::cout << v << std::endl;
  std::cout << std::endl;
}
{% endhighlight %}


Full sauce can be found [here](https://github.com/Estinox/coding-practices/blob/master/random_code/mtp_count.cpp){:target="blank"}, and there's something extra for how to sum a list of array TMP style as well.

