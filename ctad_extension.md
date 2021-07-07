<center> 
<h1> Propagting CTAD to qualified-ids when CTAD can deduce the type of the nested name-specifier from its default constructor</h1>


Kunal  Tyagi <br>
Fabian Schuetze
</center>

<blockquote>We like to suggest type deduction of qualified ids if CTAD can derive the type of the nested name specifier from its default constructor. In such situations, CTAD already deduces the nested-name-specifier's type. Extending this facility to qualified ids broads the utility of CTAD. </blockquote>

<h4> Code Example </h4>

<table border=1>
<tr>
<th>
Works very well
</th>
<th>
If the nested-name-specfier is a template, <br> types of qualified-ids aren't deducded, <br> although CTAD deduces the type of the specifier
</th>
</tr>

<tr>

<td>
<pre class="language-cpp">

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
</pre>
</td>

<td>
<pre class="language-cpp">
template &lt;typename T = int&gt;
struct A{
    using container = std::vector<T>;
    static void func() {};
    enum class Color {blue, red };
    const static T member{20}; 
};
A a{}; <span style="color:blue">Works fine, thanks to CTAD</span>. 
A::container container_of_a{}; <span style="color:red">Breaks</span>. 
A::func();  <span style="color:red">Breaks</span>. 
A::Color color{};<span style="color:red">Breaks</span>. 
A::member <span style="color:red">Breaks</span>. 
</pre>
</td>

</tr>
</table>


<h4>Motivation</h4>
One of the implications of CTAD is that library authors can turn an existing class into a class template with default arguments without breaking existing user code. Unfortunately, CTAD does not enable inference of qualified-ids (nested typedefs, nested enums or classes, const static member variables, or static functions), as documented above. Such failure is surprising if the nested-name-specifier's type is inferred correctly and if the context allows deducing the qualified-id's type.

<h4>Possible Implementation</h4>
Suppose the type of the nested-name-specifier ```c++ A<T>::``` can be derived automatically from the default constructor of ```A``` with CTAD. Then, one could attempt to deduce the qualified-id's type  ```A<T>::qualified_type``` with ```decltype(A{})::qualified_id```. In terms of the example specified above:  
<pre class="language-cpp">
decltype(A{})::container container_of_a{};
decltype({A{})::func();
decltype({A{})::Color color{};
decltype({A{})::member;
</pre>
