# CTAD for qualified-ids help avoid breaking usercode when classes are templetized
==================================================================================

> We would like to suggest that CTAD propagates through to qualified-ids when-ever the template argument of the nested name specifier can be derived from its default constructor. This broadens CTAD's utility when converting existing classes into template classes and further avoids breaking existing user code.   

CTAD for qualified-ids help avoid breaking usercode when classes are templetized

<table>
<tr>
<th>
Current Situation
</th>
<th>
Extended Library
</th>
</tr>

<tr>

<td>
<pre class="language-cpp">
```cpp
struct A{
    using container = std::vector<int>;
    static void func() {};
    enum class Color {blue, red };
    const static int member(20); 
};
A a{};
A::container container_of_a{};
A::func();
A::Color color{};
A::member 
```
</pre>
</td>

<td>
<pre class="language-cpp">
```cpp
template <typename T = int>
struct A{
    using container = std::vector<T>;
    static void func() {};
    enum class Color {blue, red };
    const static T member{20}; 
};
A a{}; !\textcolor{teal}{// Works fine, thanks to CTAD}!
A::container container_of_a{}; !\textcolor{red}{// breaks.}!
A::func(); !\textcolor{red}{// breaks.}!
A::Color color{}; !\textcolor{red}{// breaks.}!
A::member  !\textcolor{red}{// breaks.}!
```

</pre>
</td>

</tr>
</table>


#Motivation
CTAD permits library authors to turn an existing class into class templates with default arguments without breaking existing user code. Unfortunately, CTAD does not infer the types of qualified-ids (nested typedefs, nested enums or classes, const static member variables, or static functions). Extended the reach of template argument deduction to qualified ids broadens the scope of template argument deduction. 





Relevant godbolt link: \url{https://godbolt.org/z/sM8eGP9ja}

#Possible Implementation
We would like to suggest that CTAD propagates through to qualified-ids whenever the template argument of the nested name specifier can be derived from its default constructor. More precisely, one could implement CTAD for nested memebers' as:
```cpp
decltype(A{})::container container_of_a{};
decltype({A{})::func();
decltype({A{})::Color color{};
decltype({A{})::member;
```

#Comparison with Current Standard
Needs modification of
- Nested name specifier \url{https://timsong-cpp.github.io/cppwp/n4861/expr.prim.id.qual#nt:nested-name-specifier}
- Template name \url{https://timsong-cpp.github.io/cppwp/n4861/temp.names#nt:template-name}
