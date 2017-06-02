# Kaleidoscope C++ Coding Style

This document is currently a work in progress. While you certainly won't be penalized for following the style described herein, it's still a moving target as of June 2, 2017.

Our style guide is based on the Google C++ style guide which was current as of June 2, 2017, but has been modified to better reflect the constraints of embedded development and the peculiarities of an Arduino-compatible environment.

<div id="content">

# Google C++ Style Guide

<div class="main_body">

## Background

C++ is one of the main development languages used by many of Google's open-source projects. As every C++ programmer knows, the language has many powerful features, but this power brings with it complexity, which in turn can make code more bug-prone and harder to read and maintain.

The goal of this guide is to manage this complexity by describing in detail the dos and don'ts of writing C++ code. These rules exist to keep the code base manageable while still allowing coders to use C++ language features productively.

_Style_, also known as readability, is what we call the conventions that govern our C++ code. The term Style is a bit of a misnomer, since these conventions cover far more than just source file formatting.

Most open-source projects developed by Google conform to the requirements in this guide.

Note that this guide is not a C++ tutorial: we assume that the reader is familiar with the language.

### Goals of the Style Guide

<div class="stylebody">

Why do we have this document?

There are a few core goals that we believe this guide should serve. These are the fundamental **why**s that underlie all of the individual rules. By bringing these ideas to the fore, we hope to ground discussions and make it clearer to our broader community why the rules are in place and why particular decisions have been made. If you understand what goals each rule is serving, it should be clearer to everyone when a rule may be waived (some can be), and what sort of argument or alternative would be necessary to change a rule in the guide.

The goals of the style guide as we currently see them are as follows:

<dl>

<dt>Style rules should pull their weight</dt>

<dd>The benefit of a style rule must be large enough to justify asking all of our engineers to remember it. The benefit is measured relative to the codebase we would get without the rule, so a rule against a very harmful practice may still have a small benefit if people are unlikely to do it anyway. This principle mostly explains the rules we don’t have, rather than the rules we do: for example, `goto` contravenes many of the following principles, but is already vanishingly rare, so the Style Guide doesn’t discuss it.</dd>

<dt>Optimize for the reader, not the writer</dt>

<dd>Our codebase (and most individual components submitted to it) is expected to continue for quite some time. As a result, more time will be spent reading most of our code than writing it. We explicitly choose to optimize for the experience of our average software engineer reading, maintaining, and debugging code in our codebase rather than ease when writing said code. "Leave a trace for the reader" is a particularly common sub-point of this principle: When something surprising or unusual is happening in a snippet of code (for example, transfer of pointer ownership), leaving textual hints for the reader at the point of use is valuable (`std::unique_ptr` demonstrates the ownership transfer unambiguously at the call site).</dd>

<dt>Be consistent with existing code</dt>

<dd>Using one style consistently through our codebase lets us focus on other (more important) issues. Consistency also allows for automation: tools that format your code or adjust your `#include`s only work properly when your code is consistent with the expectations of the tooling. In many cases, rules that are attributed to "Be Consistent" boil down to "Just pick one and stop worrying about it"; the potential value of allowing flexibility on these points is outweighed by the cost of having people argue over them.</dd>

<dt>Be consistent with the broader C++ community when appropriate</dt>

<dd>Consistency with the way other organizations use C++ has value for the same reasons as consistency within our code base. If a feature in the C++ standard solves a problem, or if some idiom is widely known and accepted, that's an argument for using it. However, sometimes standard features and idioms are flawed, or were just designed without our codebase's needs in mind. In those cases (as described below) it's appropriate to constrain or ban standard features. In some cases we prefer a homegrown or third-party library over a library defined in the C++ Standard, either out of perceived superiority or insufficient value to transition the codebase to the standard interface.</dd>

<dt>Avoid surprising or dangerous constructs</dt>

<dd>C++ has features that are more surprising or dangerous than one might think at a glance. Some style guide restrictions are in place to prevent falling into these pitfalls. There is a high bar for style guide waivers on such restrictions, because waiving such rules often directly risks compromising program correctness.</dd>

<dt>Avoid constructs that our average C++ programmer would find tricky or hard to maintain</dt>

<dd>C++ has features that may not be generally appropriate because of the complexity they introduce to the code. In widely used code, it may be more acceptable to use trickier language constructs, because any benefits of more complex implementation are multiplied widely by usage, and the cost in understanding the complexity does not need to be paid again when working with new portions of the codebase. When in doubt, waivers to rules of this type can be sought by asking your project leads. This is specifically important for our codebase because code ownership and team membership changes over time: even if everyone that works with some piece of code currently understands it, such understanding is not guaranteed to hold a few years from now.</dd>

<dt>Be mindful of our scale</dt>

<dd>With a codebase of 100+ million lines and thousands of engineers, some mistakes and simplifications for one engineer can become costly for many. For instance it's particularly important to avoid polluting the global namespace: name collisions across a codebase of hundreds of millions of lines are difficult to work with and hard to avoid if everyone puts things into the global namespace.</dd>

<dt>Concede to optimization when necessary</dt>

<dd>Performance optimizations can sometimes be necessary and appropriate, even when they conflict with the other principles of this document.</dd>

</dl>

The intent of this document is to provide maximal guidance with reasonable restriction. As always, common sense and good taste should prevail. By this we specifically refer to the established conventions of the entire Google C++ community, not just your personal preferences or those of your team. Be skeptical about and reluctant to use clever or unusual constructs: the absence of a prohibition is not the same as a license to proceed. Use your judgment, and if you are unsure, please don't hesitate to ask your project leads to get additional input.

</div>

## Header Files

In general, every `.cc` file should have an associated `.h` file. There are some common exceptions, such as unittests and small `.cc` files containing just a `main()` function.

Correct use of header files can make a huge difference to the readability, size and performance of your code.

The following rules will guide you through the various pitfalls of using header files.

<a id="The_-inl.h_Files"></a>

### Self-contained Headers

<div class="summary">

Header files should be self-contained (compile on their own) and end in `.h`. Non-header files that are meant for inclusion should end in `.inc` and be used sparingly.

</div>

<div class="stylebody">

All header files should be self-contained. Users and refactoring tools should not have to adhere to special conditions to include the header. Specifically, a header should have [header guards](#The__define_Guard) and include all other headers it needs.

Prefer placing the definitions for template and inline functions in the same file as their declarations. The definitions of these constructs must be included into every `.cc` file that uses them, or the program may fail to link in some build configurations. If declarations and definitions are in different files, including the former should transitively include the latter. Do not move these definitions to separately included header files (`-inl.h`); this practice was common in the past, but is no longer allowed.

As an exception, a template that is explicitly instantiated for all relevant sets of template arguments, or that is a private implementation detail of a class, is allowed to be defined in the one and only `.cc` file that instantiates the template.

There are rare cases where a file designed to be included is not self-contained. These are typically intended to be included at unusual locations, such as the middle of another file. They might not use [header guards](#The__define_Guard), and might not include their prerequisites. Name such files with the `.inc` extension. Use sparingly, and prefer self-contained headers when possible.

</div>

### The #define Guard

<div class="summary">

All header files should have `#define` guards to prevent multiple inclusion. The format of the symbol name should be `_<PROJECT>___<PATH>___<FILE>__H_`.

</div>

<div class="stylebody">

To guarantee uniqueness, they should be based on the full path in a project's source tree. For example, the file `foo/src/bar/baz.h` in project `foo` should have the following guard:

<pre>#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_
</pre>

</div>

### Forward Declarations

<div class="summary">

Avoid using forward declarations where possible. Just `#include` the headers you need.

</div>

<div class="stylebody">

<div class="definition">

A "forward declaration" is a declaration of a class, function, or template without an associated definition.

</div>

<div class="pros">

*   Forward declarations can save compile time, as `#include`s force the compiler to open more files and process more input.
*   Forward declarations can save on unnecessary recompilation. `#include`s can force your code to be recompiled more often, due to unrelated changes in the header.

</div>

<div class="cons">

*   Forward declarations can hide a dependency, allowing user code to skip necessary recompilation when headers change.
*   A forward declaration may be broken by subsequent changes to the library. Forward declarations of functions and templates can prevent the header owners from making otherwise-compatible changes to their APIs, such as widening a parameter type, adding a template parameter with a default value, or migrating to a new namespace.
*   Forward declaring symbols from namespace `std::` yields undefined behavior.
*   It can be difficult to determine whether a forward declaration or a full `#include` is needed. Replacing an `#include` with a forward declaration can silently change the meaning of code:

    <pre>      // b.h:
          struct B {};
          struct D : B {};

          // good_user.cc:
          #include "b.h"
          void f(B*);
          void f(void*);
          void test(D* x) { f(x); }  // calls f(B*)
          </pre>

    If the `#include` was replaced with forward decls for `B` and `D`, `test()` would call `f(void*)`.
*   Forward declaring multiple symbols from a header can be more verbose than simply `#include`ing the header.
*   Structuring code to enable forward declarations (e.g. using pointer members instead of object members) can make the code slower and more complex.

</div>

<div class="decision">

*   Try to avoid forward declarations of entities defined in another project.
*   When using a function declared in a header file, always `#include` that header.
*   When using a class template, prefer to `#include` its header file.

Please see [Names and Order of Includes](#Names_and_Order_of_Includes) for rules about when to #include a header.

</div>

</div>

### Inline Functions

<div class="summary">

Define functions inline only when they are small, say, 10 lines or fewer.

</div>

<div class="stylebody">

<div class="definition">

You can declare functions in a way that allows the compiler to expand them inline rather than calling them through the usual function call mechanism.

</div>

<div class="pros">

Inlining a function can generate more efficient object code, as long as the inlined function is small. Feel free to inline accessors and mutators, and other short, performance-critical functions.

</div>

<div class="cons">

Overuse of inlining can actually make programs slower. Depending on a function's size, inlining it can cause the code size to increase or decrease. Inlining a very small accessor function will usually decrease code size while inlining a very large function can dramatically increase code size. On modern processors smaller code usually runs faster due to better use of the instruction cache.

</div>

<div class="decision">

A decent rule of thumb is to not inline a function if it is more than 10 lines long. Beware of destructors, which are often longer than they appear because of implicit member- and base-destructor calls!

Another useful rule of thumb: it's typically not cost effective to inline functions with loops or switch statements (unless, in the common case, the loop or switch statement is never executed).

It is important to know that functions are not always inlined even if they are declared as such; for example, virtual and recursive functions are not normally inlined. Usually recursive functions should not be inline. The main reason for making a virtual function inline is to place its definition in the class, either for convenience or to document its behavior, e.g., for accessors and mutators.

</div>

</div>

### Names and Order of Includes

<div class="summary">

Use standard order for readability and to avoid hidden dependencies: Related header, C library, C++ library, other libraries' `.h`, your project's `.h`.

</div>

<div class="stylebody">

All of a project's header files should be listed as descendants of the project's source directory without use of UNIX directory shortcuts `.` (the current directory) or `..` (the parent directory). For example, `google-awesome-project/src/base/logging.h` should be included as:

<pre>#include "base/logging.h"
</pre>

In `<var>dir/foo</var>.cc` or `<var>dir/foo_test</var>.cc`, whose main purpose is to implement or test the stuff in `<var>dir2/foo2</var>.h`, order your includes as follows:

1.  `<var>dir2/foo2</var>.h`.
2.  C system files.
3.  C++ system files.
4.  Other libraries' `.h` files.
5.  Your project's `.h` files.

With the preferred ordering, if `<var>dir2/foo2</var>.h` omits any necessary includes, the build of `<var>dir/foo</var>.cc` or `<var>dir/foo</var>_test.cc` will break. Thus, this rule ensures that build breaks show up first for the people working on these files, not for innocent people in other packages.

`<var>dir/foo</var>.cc` and `<var>dir2/foo2</var>.h` are usually in the same directory (e.g. `base/basictypes_test.cc` and `base/basictypes.h`), but may sometimes be in different directories too.

Within each section the includes should be ordered alphabetically. Note that older code might not conform to this rule and should be fixed when convenient.

You should include all the headers that define the symbols you rely upon, except in the unusual case of [forward declaration](#Forward_Declarations). If you rely on symbols from `bar.h`, don't count on the fact that you included `foo.h` which (currently) includes `bar.h`: include `bar.h` yourself, unless `foo.h` explicitly demonstrates its intent to provide you the symbols of `bar.h`. However, any includes present in the related header do not need to be included again in the related `cc` (i.e., `foo.cc` can rely on `foo.h`'s includes).

For example, the includes in `google-awesome-project/src/foo/internal/fooserver.cc` might look like this:

<pre>#include "foo/server/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <hash_map>
#include <vector>

#include "base/basictypes.h"
#include "base/commandlineflags.h"
#include "foo/server/bar.h"
</pre>

Sometimes, system-specific code needs conditional includes. Such code can put conditional includes after other includes. Of course, keep your system-specific code small and localized. Example:

<pre>#include "foo/public/fooserver.h"

#include "base/port.h"  // For LANG_CXX11.

#ifdef LANG_CXX11
#include <initializer_list>
#endif  // LANG_CXX11
</pre>

</div>

## Scoping

### Namespaces

<div class="summary">

With few exceptions, place code in a namespace. Namespaces should have unique names based on the project name, and possibly its path. Do not use _using-directives_ (e.g. `using namespace foo`). Do not use inline namespaces. For unnamed namespaces, see [Unnamed Namespaces and Static Variables](#Unnamed_Namespaces_and_Static_Variables).

</div>

<div class="stylebody">

<div class="definition">

Namespaces subdivide the global scope into distinct, named scopes, and so are useful for preventing name collisions in the global scope.

</div>

<div class="pros">

Namespaces provide a method for preventing name conflicts in large programs while allowing most code to use reasonably short names.

For example, if two different projects have a class `Foo` in the global scope, these symbols may collide at compile time or at runtime. If each project places their code in a namespace, `project1::Foo` and `project2::Foo` are now distinct symbols that do not collide, and code within each project's namespace can continue to refer to `Foo` without the prefix.

Inline namespaces automatically place their names in the enclosing scope. Consider the following snippet, for example:

<pre>namespace X {
inline namespace Y {
  void foo();
}  // namespace Y
}  // namespace X
</pre>

The expressions `X::Y::foo()` and `X::foo()` are interchangeable. Inline namespaces are primarily intended for ABI compatibility across versions.

</div>

<div class="cons">

Namespaces can be confusing, because they complicate the mechanics of figuring out what definition a name refers to.

Inline namespaces, in particular, can be confusing because names aren't actually restricted to the namespace where they are declared. They are only useful as part of some larger versioning policy.

In some contexts, it's necessary to repeatedly refer to symbols by their fully-qualified names. For deeply-nested namespaces, this can add a lot of clutter.

</div>

<div class="decision">

Namespaces should be used as follows:

*   Follow the rules on [Namespace Names](#Namespace_Names).
*   Terminate namespaces with comments as shown in the given examples.
*   Namespaces wrap the entire source file after includes, [gflags](https://gflags.github.io/gflags/) definitions/declarations and forward declarations of classes from other namespaces.

    <pre>// In the .h file
    namespace mynamespace {

    // All declarations are within the namespace scope.
    // Notice the lack of indentation.
    class MyClass {
     public:
      ...
      void Foo();
    };

    }  // namespace mynamespace
    </pre>

    <pre>// In the .cc file
    namespace mynamespace {

    // Definition of functions is within scope of the namespace.
    void MyClass::Foo() {
      ...
    }

    }  // namespace mynamespace
    </pre>

    More complex `.cc` files might have additional details, like flags or using-declarations.

    <pre>#include "a.h"

    DEFINE_FLAG(bool, someflag, false, "dummy flag");

    namespace a {

    using ::foo::bar;

    ...code for a...         // Code goes against the left margin.

    }  // namespace a
    </pre>

*   Do not declare anything in namespace `std`, including forward declarations of standard library classes. Declaring entities in namespace `std` is undefined behavior, i.e., not portable. To declare entities from the standard library, include the appropriate header file.
*   You may not use a _using-directive_ to make all names from a namespace available.

    <pre class="badcode">// Forbidden -- This pollutes the namespace.
    using namespace foo;
    </pre>

*   Do not use _Namespace aliases_ at namespace scope in header files except in explicitly marked internal-only namespaces, because anything imported into a namespace in a header file becomes part of the public API exported by that file.

    <pre>// Shorten access to some commonly used names in .cc files.
    namespace baz = ::foo::bar::baz;
    </pre>

    <pre>// Shorten access to some commonly used names (in a .h file).
    namespace librarian {
    namespace impl {  // Internal, not part of the API.
    namespace sidetable = ::pipeline_diagnostics::sidetable;
    }  // namespace impl

    inline void my_inline_function() {
      // namespace alias local to a function (or method).
      namespace baz = ::foo::bar::baz;
      ...
    }
    }  // namespace librarian
    </pre>

*   Do not use inline namespaces.

</div>

</div>

### Unnamed Namespaces and Static Variables

<div class="summary">

When definitions in a `.cc` file do not need to be referenced outside that file, place them in an unnamed namespace or declare them `static`. Do not use either of these constructs in `.h` files.

</div>

<div class="stylebody">

<div class="definition">

All declarations can be given internal linkage by placing them in unnamed namespaces, and functions and variables can be given internal linkage by declaring them `static`. This means that anything you're declaring can't be accessed from another file. If a different file declares something with the same name, then the two entities are completely independent.

</div>

<div class="decision">

Use of internal linkage in `.cc` files is encouraged for all code that does not need to be referenced elsewhere. Do not use internal linkage in `.h` files.

Format unnamed namespaces like named namespaces. In the terminating comment, leave the namespace name empty:

<pre>namespace {
...
}  // namespace
</pre>

</div>

</div>

### Nonmember, Static Member, and Global Functions

<div class="summary">

Prefer placing nonmember functions in a namespace; use completely global functions rarely. Prefer grouping functions with a namespace instead of using a class as if it were a namespace. Static methods of a class should generally be closely related to instances of the class or the class's static data.

</div>

<div class="stylebody">

<div class="pros">

Nonmember and static member functions can be useful in some situations. Putting nonmember functions in a namespace avoids polluting the global namespace.

</div>

<div class="cons">

Nonmember and static member functions may make more sense as members of a new class, especially if they access external resources or have significant dependencies.

</div>

<div class="decision">

Sometimes it is useful to define a function not bound to a class instance. Such a function can be either a static member or a nonmember function. Nonmember functions should not depend on external variables, and should nearly always exist in a namespace. Rather than creating classes only to group static member functions which do not share static data, use [namespaces](#Namespaces) instead. For a header `myproject/foo_bar.h`, for example, write

<pre>namespace myproject {
namespace foo_bar {
void Function1();
void Function2();
}  // namespace foo_bar
}  // namespace myproject
</pre>

instead of

<pre class="badcode">namespace myproject {
class FooBar {
 public:
  static void Function1();
  static void Function2();
};
}  // namespace myproject
</pre>

If you define a nonmember function and it is only needed in its `.cc` file, use [internal linkage](#Unnamed_Namespaces_and_Static_Variables) to limit its scope.

</div>

</div>

### Local Variables

<div class="summary">

Place a function's variables in the narrowest scope possible, and initialize variables in the declaration.

</div>

<div class="stylebody">

C++ allows you to declare variables anywhere in a function. We encourage you to declare them in as local a scope as possible, and as close to the first use as possible. This makes it easier for the reader to find the declaration and see what type the variable is and what it was initialized to. In particular, initialization should be used instead of declaration and assignment, e.g.:

<pre class="badcode">int i;
i = f();      // Bad -- initialization separate from declaration.
</pre>

<pre>int j = g();  // Good -- declaration has initialization.
</pre>

<pre class="badcode">std::vector<int> v;
v.push_back(1);  // Prefer initializing using brace initialization.
v.push_back(2);
</pre>

<pre>std::vector<int> v = {1, 2};  // Good -- v starts initialized.
</pre>

Variables needed for `if`, `while` and `for` statements should normally be declared within those statements, so that such variables are confined to those scopes. E.g.:

<pre>while (const char* p = strchr(str, '/')) str = p + 1;
</pre>

There is one caveat: if the variable is an object, its constructor is invoked every time it enters scope and is created, and its destructor is invoked every time it goes out of scope.

<pre class="badcode">// Inefficient implementation:
for (int i = 0; i < 1000000; ++i) {
  Foo f;  // My ctor and dtor get called 1000000 times each.
  f.DoSomething(i);
}
</pre>

It may be more efficient to declare such a variable used in a loop outside that loop:

<pre>Foo f;  // My ctor and dtor get called once each.
for (int i = 0; i < 1000000; ++i) {
  f.DoSomething(i);
}
</pre>

</div>

### Static and Global Variables

<div class="summary">

Variables of class type with [static storage duration](http://en.cppreference.com/w/cpp/language/storage_duration#Storage_duration) are forbidden: they cause hard-to-find bugs due to indeterminate order of construction and destruction. However, such variables are allowed if they are `constexpr`: they have no dynamic initialization or destruction.

</div>

<div class="stylebody">

Objects with static storage duration, including global variables, static variables, static class member variables, and function static variables, must be Plain Old Data (POD): only ints, chars, floats, or pointers, or arrays/structs of POD.

The order in which class constructors and initializers for static variables are called is only partially specified in C++ and can even change from build to build, which can cause bugs that are difficult to find. Therefore in addition to banning globals of class type, we do not allow non-local static variables to be initialized with the result of a function, unless that function (such as getenv(), or getpid()) does not itself depend on any other globals. However, a static POD variable within function scope may be initialized with the result of a function, since its initialization order is well-defined and does not occur until control passes through its declaration.

Likewise, global and static variables are destroyed when the program terminates, regardless of whether the termination is by returning from `main()` or by calling `exit()`. The order in which destructors are called is defined to be the reverse of the order in which the constructors were called. Since constructor order is indeterminate, so is destructor order. For example, at program-end time a static variable might have been destroyed, but code still running — perhaps in another thread — tries to access it and fails. Or the destructor for a static `string` variable might be run prior to the destructor for another variable that contains a reference to that string.

One way to alleviate the destructor problem is to terminate the program by calling `quick_exit()` instead of `exit()`. The difference is that `quick_exit()` does not invoke destructors and does not invoke any handlers that were registered by calling `atexit()`. If you have a handler that needs to run when a program terminates via `quick_exit()` (flushing logs, for example), you can register it using `at_quick_exit()`. (If you have a handler that needs to run at both `exit()` and `quick_exit()`, you need to register it in both places.)

As a result we only allow static variables to contain POD data. This rule completely disallows `std::vector` (use C arrays instead), or `string` (use `const char []`).

If you need a static or global variable of a class type, consider initializing a pointer (which will never be freed), from either your main() function or from pthread_once(). Note that this must be a raw pointer, not a "smart" pointer, since the smart pointer's destructor will have the order-of-destructor issue that we are trying to avoid.

</div>

## Classes

Classes are the fundamental unit of code in C++. Naturally, we use them extensively. This section lists the main dos and don'ts you should follow when writing a class.

### Doing Work in Constructors

<div class="summary">

Avoid virtual method calls in constructors, and avoid initialization that can fail if you can't signal an error.

</div>

<div class="stylebody">

<div class="definition">

It is possible to perform arbitrary initialization in the body of the constructor.

</div>

<div class="pros">

*   No need to worry about whether the class has been initialized or not.
*   Objects that are fully initialized by constructor call can be `const` and may also be easier to use with standard containers or algorithms.

</div>

<div class="cons">

*   If the work calls virtual functions, these calls will not get dispatched to the subclass implementations. Future modification to your class can quietly introduce this problem even if your class is not currently subclassed, causing much confusion.
*   There is no easy way for constructors to signal errors, short of crashing the program (not always appropriate) or using exceptions (which are [forbidden](#Exceptions)).
*   If the work fails, we now have an object whose initialization code failed, so it may be an unusual state requiring a `bool IsValid()` state checking mechanism (or similar) which is easy to forget to call.
*   You cannot take the address of a constructor, so whatever work is done in the constructor cannot easily be handed off to, for example, another thread.

</div>

<div class="decision">

Constructors should never call virtual functions. If appropriate for your code , terminating the program may be an appropriate error handling response. Otherwise, consider a factory function or `Init()` method. Avoid `Init()` methods on objects with no other states that affect which public methods may be called (semi-constructed objects of this form are particularly hard to work with correctly).

</div>

</div>

<a id="Explicit_Constructors"></a>

### Implicit Conversions

<div class="summary">

Do not define implicit conversions. Use the `explicit` keyword for conversion operators and single-argument constructors.

</div>

<div class="stylebody">

<div class="definition">

Implicit conversions allow an object of one type (called the <dfn>source type</dfn>) to be used where a different type (called the <dfn>destination type</dfn>) is expected, such as when passing an `int` argument to a function that takes a `double` parameter.

In addition to the implicit conversions defined by the language, users can define their own, by adding appropriate members to the class definition of the source or destination type. An implicit conversion in the source type is defined by a type conversion operator named after the destination type (e.g. `operator bool()`). An implicit conversion in the destination type is defined by a constructor that can take the source type as its only argument (or only argument with no default value).

The `explicit` keyword can be applied to a constructor or (since C++11) a conversion operator, to ensure that it can only be used when the destination type is explicit at the point of use, e.g. with a cast. This applies not only to implicit conversions, but to C++11's list initialization syntax:

<pre>class Foo {
  explicit Foo(int x, double y);
  ...
};

void Func(Foo f);
</pre>

<pre class="badcode">Func({42, 3.14});  // Error
</pre>

This kind of code isn't technically an implicit conversion, but the language treats it as one as far as `explicit` is concerned.</div>

<div class="pros">

*   Implicit conversions can make a type more usable and expressive by eliminating the need to explicitly name a type when it's obvious.
*   Implicit conversions can be a simpler alternative to overloading.
*   List initialization syntax is a concise and expressive way of initializing objects.

</div>

<div class="cons">

*   Implicit conversions can hide type-mismatch bugs, where the destination type does not match the user's expectation, or the user is unaware that any conversion will take place.
*   Implicit conversions can make code harder to read, particularly in the presence of overloading, by making it less obvious what code is actually getting called.
*   Constructors that take a single argument may accidentally be usable as implicit type conversions, even if they are not intended to do so.
*   When a single-argument constructor is not marked `explicit`, there's no reliable way to tell whether it's intended to define an implicit conversion, or the author simply forgot to mark it.
*   It's not always clear which type should provide the conversion, and if they both do, the code becomes ambiguous.
*   List initialization can suffer from the same problems if the destination type is implicit, particularly if the list has only a single element.

</div>

<div class="decision">

Type conversion operators, and constructors that are callable with a single argument, must be marked `explicit` in the class definition. As an exception, copy and move constructors should not be `explicit`, since they do not perform type conversion. Implicit conversions can sometimes be necessary and appropriate for types that are designed to transparently wrap other types. In that case, contact your project leads to request a waiver of this rule.

Constructors that cannot be called with a single argument should usually omit `explicit`. Constructors that take a single `std::initializer_list` parameter should also omit `explicit`, in order to support copy-initialization (e.g. `MyType m = {1, 2};`).

</div>

</div>

### Copyable and Movable Types

<a id="Copy_Constructors"></a>

<div class="summary">

Support copying and/or moving if these operations are clear and meaningful for your type. Otherwise, disable the implicitly generated special functions that perform copies and moves.

</div>

<div class="stylebody">

<div class="definition">

A copyable type allows its objects to be initialized or assigned from any other object of the same type, without changing the value of the source. For user-defined types, the copy behavior is defined by the copy constructor and the copy-assignment operator. `string` is an example of a copyable type.

A movable type is one that can be initialized and assigned from temporaries (all copyable types are therefore movable). `std::unique_ptr<int>` is an example of a movable but not copyable type. For user-defined types, the move behavior is defined by the move constructor and the move-assignment operator.

The copy/move constructors can be implicitly invoked by the compiler in some situations, e.g. when passing objects by value.

</div>

<div class="pros">

Objects of copyable and movable types can be passed and returned by value, which makes APIs simpler, safer, and more general. Unlike when passing objects by pointer or reference, there's no risk of confusion over ownership, lifetime, mutability, and similar issues, and no need to specify them in the contract. It also prevents non-local interactions between the client and the implementation, which makes them easier to understand, maintain, and optimize by the compiler. Further, such objects can be used with generic APIs that require pass-by-value, such as most containers, and they allow for additional flexibility in e.g., type composition.

Copy/move constructors and assignment operators are usually easier to define correctly than alternatives like `Clone()`, `CopyFrom()` or `Swap()`, because they can be generated by the compiler, either implicitly or with `= default`. They are concise, and ensure that all data members are copied. Copy and move constructors are also generally more efficient, because they don't require heap allocation or separate initialization and assignment steps, and they're eligible for optimizations such as [copy elision](http://en.cppreference.com/w/cpp/language/copy_elision).

Move operations allow the implicit and efficient transfer of resources out of rvalue objects. This allows a plainer coding style in some cases.

</div>

<div class="cons">

Some types do not need to be copyable, and providing copy operations for such types can be confusing, nonsensical, or outright incorrect. Types representing singleton objects (`Registerer`), objects tied to a specific scope (`Cleanup`), or closely coupled to object identity (`Mutex`) cannot be copied meaningfully. Copy operations for base class types that are to be used polymorphically are hazardous, because use of them can lead to [object slicing](https://en.wikipedia.org/wiki/Object_slicing). Defaulted or carelessly-implemented copy operations can be incorrect, and the resulting bugs can be confusing and difficult to diagnose.

Copy constructors are invoked implicitly, which makes the invocation easy to miss. This may cause confusion for programmers used to languages where pass-by-reference is conventional or mandatory. It may also encourage excessive copying, which can cause performance problems.

</div>

<div class="decision">

Provide the copy and move operations if their meaning is clear to a casual user and the copying/moving does not incur unexpected costs. If you define a copy or move constructor, define the corresponding assignment operator, and vice-versa. If your type is copyable, do not define move operations unless they are significantly more efficient than the corresponding copy operations. If your type is not copyable, but the correctness of a move is obvious to users of the type, you may make the type move-only by defining both of the move operations.

If your type provides copy operations, it is recommended that you design your class so that the default implementation of those operations is correct. Remember to review the correctness of any defaulted operations as you would any other code, and to document that your class is copyable and/or cheaply movable if that's an API guarantee.

<pre class="badcode">class Foo {
 public:
  Foo(Foo&& other) : field_(other.field) {}
  // Bad, defines only move constructor, but not operator=.

 private:
  Field field_;
};
</pre>

Due to the risk of slicing, avoid providing an assignment operator or public copy/move constructor for a class that's intended to be derived from (and avoid deriving from a class with such members). If your base class needs to be copyable, provide a public virtual `Clone()` method, and a protected copy constructor that derived classes can use to implement it.

If you do not want to support copy/move operations on your type, explicitly disable them using `= delete` in the `public:` section:

<pre class="code">// MyClass is neither copyable nor movable.
MyClass(const MyClass&) = delete;
MyClass& operator=(const MyClass&) = delete;
</pre>

</div>

</div>

### Structs vs. Classes

<div class="summary">

Use a `struct` only for passive objects that carry data; everything else is a `class`.

</div>

<div class="stylebody">

The `struct` and `class` keywords behave almost identically in C++. We add our own semantic meanings to each keyword, so you should use the appropriate keyword for the data-type you're defining.

`structs` should be used for passive objects that carry data, and may have associated constants, but lack any functionality other than access/setting the data members. The accessing/setting of fields is done by directly accessing the fields rather than through method invocations. Methods should not provide behavior but should only be used to set up the data members, e.g., constructor, destructor, `Initialize()`, `Reset()`, `Validate()`.

If more functionality is required, a `class` is more appropriate. If in doubt, make it a `class`.

For consistency with STL, you can use `struct` instead of `class` for functors and traits.

Note that member variables in structs and classes have [different naming rules](#Variable_Names).

</div>

### Inheritance

<div class="summary">

Composition is often more appropriate than inheritance. When using inheritance, make it `public`.

</div>

<div class="stylebody">

<div class="definition">

When a sub-class inherits from a base class, it includes the definitions of all the data and operations that the parent base class defines. In practice, inheritance is used in two major ways in C++: implementation inheritance, in which actual code is inherited by the child, and [interface inheritance](#Interfaces), in which only method names are inherited.

</div>

<div class="pros">

Implementation inheritance reduces code size by re-using the base class code as it specializes an existing type. Because inheritance is a compile-time declaration, you and the compiler can understand the operation and detect errors. Interface inheritance can be used to programmatically enforce that a class expose a particular API. Again, the compiler can detect errors, in this case, when a class does not define a necessary method of the API.

</div>

<div class="cons">

For implementation inheritance, because the code implementing a sub-class is spread between the base and the sub-class, it can be more difficult to understand an implementation. The sub-class cannot override functions that are not virtual, so the sub-class cannot change implementation. The base class may also define some data members, so that specifies physical layout of the base class.

</div>

<div class="decision">

All inheritance should be `public`. If you want to do private inheritance, you should be including an instance of the base class as a member instead.

Do not overuse implementation inheritance. Composition is often more appropriate. Try to restrict use of inheritance to the "is-a" case: `Bar` subclasses `Foo` if it can reasonably be said that `Bar` "is a kind of" `Foo`.

Make your destructor `virtual` if necessary. If your class has virtual methods, its destructor should be virtual.

Limit the use of `protected` to those member functions that might need to be accessed from subclasses. Note that [data members should be private](#Access_Control).

Explicitly annotate overrides of virtual functions or virtual destructors with an `override` or (less frequently) `final` specifier. Older (pre-C++11) code will use the `virtual` keyword as an inferior alternative annotation. For clarity, use exactly one of `override`, `final`, or `virtual` when declaring an override. Rationale: A function or destructor marked `override` or `final` that is not an override of a base class virtual function will not compile, and this helps catch common errors. The specifiers serve as documentation; if no specifier is present, the reader has to check all ancestors of the class in question to determine if the function or destructor is virtual or not.

</div>

</div>

### Multiple Inheritance

<div class="summary">

Only very rarely is multiple implementation inheritance actually useful. We allow multiple inheritance only when at most one of the base classes has an implementation; all other base classes must be [pure interface](#Interfaces) classes tagged with the `Interface` suffix.

</div>

<div class="stylebody">

<div class="definition">

Multiple inheritance allows a sub-class to have more than one base class. We distinguish between base classes that are _pure interfaces_ and those that have an _implementation_.

</div>

<div class="pros">

Multiple implementation inheritance may let you re-use even more code than single inheritance (see [Inheritance](#Inheritance)).

</div>

<div class="cons">

Only very rarely is multiple _implementation_ inheritance actually useful. When multiple implementation inheritance seems like the solution, you can usually find a different, more explicit, and cleaner solution.

</div>

<div class="decision">

Multiple inheritance is allowed only when all superclasses, with the possible exception of the first one, are [pure interfaces](#Interfaces). In order to ensure that they remain pure interfaces, they must end with the `Interface` suffix.

</div>

<div class="note">

There is an [exception](#Windows_Code) to this rule on Windows.

</div>

</div>

### Interfaces

<div class="summary">

Classes that satisfy certain conditions are allowed, but not required, to end with an `Interface` suffix.

</div>

<div class="stylebody">

<div class="definition">

A class is a pure interface if it meets the following requirements:

*   It has only public pure virtual ("`= 0`") methods and static methods (but see below for destructor).
*   It may not have non-static data members.
*   It need not have any constructors defined. If a constructor is provided, it must take no arguments and it must be protected.
*   If it is a subclass, it may only be derived from classes that satisfy these conditions and are tagged with the `Interface` suffix.

An interface class can never be directly instantiated because of the pure virtual method(s) it declares. To make sure all implementations of the interface can be destroyed correctly, the interface must also declare a virtual destructor (in an exception to the first rule, this should not be pure). See Stroustrup, <cite>The C++ Programming Language</cite>, 3rd edition, section 12.4 for details.

</div>

<div class="pros">

Tagging a class with the `Interface` suffix lets others know that they must not add implemented methods or non static data members. This is particularly important in the case of [multiple inheritance](#Multiple_Inheritance). Additionally, the interface concept is already well-understood by Java programmers.

</div>

<div class="cons">

The `Interface` suffix lengthens the class name, which can make it harder to read and understand. Also, the interface property may be considered an implementation detail that shouldn't be exposed to clients.

</div>

<div class="decision">

A class may end with `Interface` only if it meets the above requirements. We do not require the converse, however: classes that meet the above requirements are not required to end with `Interface`.

</div>

</div>

### Operator Overloading

<div class="summary">

Overload operators judiciously. Do not create user-defined literals.

</div>

<div class="stylebody">

<div class="definition">

C++ permits user code to [declare overloaded versions of the built-in operators](http://en.cppreference.com/w/cpp/language/operators) using the `operator` keyword, so long as one of the parameters is a user-defined type. The `operator` keyword also permits user code to define new kinds of literals using `operator""`, and to define type-conversion functions such as `operator bool()`.

</div>

<div class="pros">

Operator overloading can make code more concise and intuitive by enabling user-defined types to behave the same as built-in types. Overloaded operators are the idiomatic names for certain operations (e.g. `==`, `<`, `=`, and `<<`), and adhering to those conventions can make user-defined types more readable and enable them to interoperate with libraries that expect those names.

User-defined literals are a very concise notation for creating objects of user-defined types.

</div>

<div class="cons">

*   Providing a correct, consistent, and unsurprising set of operator overloads requires some care, and failure to do so can lead to confusion and bugs.
*   Overuse of operators can lead to obfuscated code, particularly if the overloaded operator's semantics don't follow convention.
*   The hazards of function overloading apply just as much to operator overloading, if not more so.
*   Operator overloads can fool our intuition into thinking that expensive operations are cheap, built-in operations.
*   Finding the call sites for overloaded operators may require a search tool that's aware of C++ syntax, rather than e.g. grep.
*   If you get the argument type of an overloaded operator wrong, you may get a different overload rather than a compiler error. For example, `foo < bar` may do one thing, while `&foo < &bar` does something totally different.
*   Certain operator overloads are inherently hazardous. Overloading unary `&` can cause the same code to have different meanings depending on whether the overload declaration is visible. Overloads of `&&`, `||`, and `,` (comma) cannot match the evaluation-order semantics of the built-in operators.
*   Operators are often defined outside the class, so there's a risk of different files introducing different definitions of the same operator. If both definitions are linked into the same binary, this results in undefined behavior, which can manifest as subtle run-time bugs.
*   User-defined literals allow the creation of new syntactic forms that are unfamiliar even to experienced C++ programmers.

</div>

<div class="decision">

Define overloaded operators only if their meaning is obvious, unsurprising, and consistent with the corresponding built-in operators. For example, use `|` as a bitwise- or logical-or, not as a shell-style pipe.

Define operators only on your own types. More precisely, define them in the same headers, .cc files, and namespaces as the types they operate on. That way, the operators are available wherever the type is, minimizing the risk of multiple definitions. If possible, avoid defining operators as templates, because they must satisfy this rule for any possible template arguments. If you define an operator, also define any related operators that make sense, and make sure they are defined consistently. For example, if you overload `<`, overload all the comparison operators, and make sure `<` and `>` never return true for the same arguments.

Prefer to define non-modifying binary operators as non-member functions. If a binary operator is defined as a class member, implicit conversions will apply to the right-hand argument, but not the left-hand one. It will confuse your users if `a < b` compiles but `b < a` doesn't.

Don't go out of your way to avoid defining operator overloads. For example, prefer to define `==`, `=`, and `<<`, rather than `Equals()`, `CopyFrom()`, and `PrintTo()`. Conversely, don't define operator overloads just because other libraries expect them. For example, if your type doesn't have a natural ordering, but you want to store it in a `std::set`, use a custom comparator rather than overloading `<`.

Do not overload `&&`, `||`, `,` (comma), or unary `&`. Do not overload `operator""`, i.e. do not introduce user-defined literals.

Type conversion operators are covered in the section on [implicit conversions](#Implicit_Conversions). The `=` operator is covered in the section on [copy constructors](#Copy_Constructors). Overloading `<<` for use with streams is covered in the section on [streams](#Streams). See also the rules on [function overloading](#Function_Overloading), which apply to operator overloading as well.

</div>

</div>

### Access Control

<div class="summary">

Make data members `private`, unless they are `static const` (and follow the [naming convention for constants](#Constant_Names)). For technical reasons, we allow data members of a test fixture class to be `protected` when using [Google Test](https://github.com/google/googletest)).

</div>

### Declaration Order

<div class="summary">

Group similar declarations together, placing public parts earlier.

</div>

<div class="stylebody">

A class definition should usually start with a `public:` section, followed by `protected:`, then `private:`. Omit sections that would be empty.

Within each section, generally prefer grouping similar kinds of declarations together, and generally prefer the following order: types (including `typedef`, `using`, and nested structs and classes), constants, factory functions, constructors, assignment operators, destructor, all other methods, data members.

Do not put large method definitions inline in the class definition. Usually, only trivial or performance-critical, and very short, methods may be defined inline. See [Inline Functions](#Inline_Functions) for more details.

</div>

## Functions

### Parameter Ordering

<div class="summary">

When defining a function, parameter order is: inputs, then outputs.

</div>

<div class="stylebody">

Parameters to C/C++ functions are either input to the function, output from the function, or both. Input parameters are usually values or `const` references, while output and input/output parameters will be pointers to non-`const`. When ordering function parameters, put all input-only parameters before any output parameters. In particular, do not add new parameters to the end of the function just because they are new; place new input-only parameters before the output parameters.

This is not a hard-and-fast rule. Parameters that are both input and output (often classes/structs) muddy the waters, and, as always, consistency with related functions may require you to bend the rule.

</div>

### Write Short Functions

<div class="summary">

Prefer small and focused functions.

</div>

<div class="stylebody">

We recognize that long functions are sometimes appropriate, so no hard limit is placed on functions length. If a function exceeds about 40 lines, think about whether it can be broken up without harming the structure of the program.

Even if your long function works perfectly now, someone modifying it in a few months may add new behavior. This could result in bugs that are hard to find. Keeping your functions short and simple makes it easier for other people to read and modify your code.

You could find long and complicated functions when working with some code. Do not be intimidated by modifying existing code: if working with such a function proves to be difficult, you find that errors are hard to debug, or you want to use a piece of it in several different contexts, consider breaking up the function into smaller and more manageable pieces.

</div>

### Reference Arguments

<div class="summary">

All parameters passed by reference must be labeled `const`.

</div>

<div class="stylebody">

<div class="definition">

In C, if a function needs to modify a variable, the parameter must use a pointer, eg `int foo(int *pval)`. In C++, the function can alternatively declare a reference parameter: `int foo(int &val)`.

</div>

<div class="pros">

Defining a parameter as reference avoids ugly code like `(*pval)++`. Necessary for some applications like copy constructors. Makes it clear, unlike with pointers, that a null pointer is not a possible value.

</div>

<div class="cons">

References can be confusing, as they have value syntax but pointer semantics.

</div>

<div class="decision">

Within function parameter lists all references must be `const`:

<pre>void Foo(const string &in, string *out);
</pre>

In fact it is a very strong convention in Google code that input arguments are values or `const` references while output arguments are pointers. Input parameters may be `const` pointers, but we never allow non-`const` reference parameters except when required by convention, e.g., `swap()`.

However, there are some instances where using `const T*` is preferable to `const T&` for input parameters. For example:

*   You want to pass in a null pointer.
*   The function saves a pointer or reference to the input.

Remember that most of the time input parameters are going to be specified as `const T&`. Using `const T*` instead communicates to the reader that the input is somehow treated differently. So if you choose `const T*` rather than `const T&`, do so for a concrete reason; otherwise it will likely confuse readers by making them look for an explanation that doesn't exist.

</div>

</div>

### Function Overloading

<div class="summary">

Use overloaded functions (including constructors) only if a reader looking at a call site can get a good idea of what is happening without having to first figure out exactly which overload is being called.

</div>

<div class="stylebody">

<div class="definition">

You may write a function that takes a `const string&` and overload it with another that takes `const char*`.

<pre>class MyClass {
 public:
  void Analyze(const string &text);
  void Analyze(const char *text, size_t textlen);
};
</pre>

</div>

<div class="pros">

Overloading can make code more intuitive by allowing an identically-named function to take different arguments. It may be necessary for templatized code, and it can be convenient for Visitors.

</div>

<div class="cons">

If a function is overloaded by the argument types alone, a reader may have to understand C++'s complex matching rules in order to tell what's going on. Also many people are confused by the semantics of inheritance if a derived class overrides only some of the variants of a function.

</div>

<div class="decision">

If you want to overload a function, consider qualifying the name with some information about the arguments, e.g., `AppendString()`, `AppendInt()` rather than just `Append()`. If you are overloading a function to support variable number of arguments of the same type, consider making it take a `std::vector` so that the user can use an [initializer list](#Braced_Initializer_List) to specify the arguments.

</div>

</div>

### Default Arguments

<div class="summary">

Default arguments are allowed on non-virtual functions when the default is guaranteed to always have the same value. Follow the same restrictions as for [function overloading](#Function_Overloading), and prefer overloaded functions if the readability gained with default arguments doesn't outweigh the downsides below.

</div>

<div class="stylebody">

<div class="pros">

Often you have a function that uses default values, but occasionally you want to override the defaults. Default parameters allow an easy way to do this without having to define many functions for the rare exceptions. Compared to overloading the function, default arguments have a cleaner syntax, with less boilerplate and a clearer distinction between 'required' and 'optional' arguments.

</div>

<div class="cons">

Defaulted arguments are another way to achieve the semantics of overloaded functions, so all the [reasons not to overload functions](#Function_Overloading) apply.

The defaults for arguments in a virtual function call are determined by the static type of the target object, and there's no guarantee that all overrides of a given function declare the same defaults.

Default parameters are re-evaluated at each call site, which can bloat the generated code. Readers may also expect the default's value to be fixed at the declaration instead of varying at each call.

Function pointers are confusing in the presence of default arguments, since the function signature often doesn't match the call signature. Adding function overloads avoids these problems.

</div>

<div class="decision">

Default arguments are banned on virtual functions, where they don't work properly, and in cases where the specified default might not evaluate to the same value depending on when it was evaluated. (For example, don't write `void f(int n = counter++);`.)

In some other cases, default arguments can improve the readability of their function declarations enough to overcome the downsides above, so they are allowed. When in doubt, use overloads.

</div>

</div>

### Trailing Return Type Syntax

<div class="summary">

Use trailing return types only where using the ordinary syntax (leading return types) is impractical or much less readable.

</div>

<div class="definition">

C++ allows two different forms of function declarations. In the older form, the return type appears before the function name. For example:

<pre>int foo(int x);
</pre>

The new form, introduced in C++11, uses the `auto` keyword before the function name and a trailing return type after the argument list. For example, the declaration above could equivalently be written:

<pre>auto foo(int x) -> int;
</pre>

The trailing return type is in the function's scope. This doesn't make a difference for a simple case like `int` but it matters for more complicated cases, like types declared in class scope or types written in terms of the function parameters.

</div>

<div class="stylebody">

<div class="pros">

Trailing return types are the only way to explicitly specify the return type of a [lambda expression](#Lambda_expressions). In some cases the compiler is able to deduce a lambda's return type, but not in all cases. Even when the compiler can deduce it automatically, sometimes specifying it explicitly would be clearer for readers.

Sometimes it's easier and more readable to specify a return type after the function's parameter list has already appeared. This is particularly true when the return type depends on template parameters. For example:

<pre>template <class T, class U> auto add(T t, U u) -> decltype(t + u);</pre>

versus

<pre>template <class T, class U> decltype(declval<T&>() + declval<U&>()) add(T t, U u);</pre>

</div>

<div class="cons">

Trailing return type syntax is relatively new and it has no analogue in C++-like languages like C and Java, so some readers may find it unfamiliar.

Existing code bases have an enormous number of function declarations that aren't going to get changed to use the new syntax, so the realistic choices are using the old syntax only or using a mixture of the two. Using a single version is better for uniformity of style.

</div>

<div class="decision">

In most cases, continue to use the older style of function declaration where the return type goes before the function name. Use the new trailing-return-type form only in cases where it's required (such as lambdas) or where, by putting the type after the function's parameter list, it allows you to write the type in a much more readable way. The latter case should be rare; it's mostly an issue in fairly complicated template code, which is [discouraged in most cases](#Template_metaprogramming).

</div>

</div>

## Google-Specific Magic

There are various tricks and utilities that we use to make C++ code more robust, and various ways we use C++ that may differ from what you see elsewhere.

### Ownership and Smart Pointers

<div class="summary">

Prefer to have single, fixed owners for dynamically allocated objects. Prefer to transfer ownership with smart pointers.

</div>

<div class="stylebody">

<div class="definition">

"Ownership" is a bookkeeping technique for managing dynamically allocated memory (and other resources). The owner of a dynamically allocated object is an object or function that is responsible for ensuring that it is deleted when no longer needed. Ownership can sometimes be shared, in which case the last owner is typically responsible for deleting it. Even when ownership is not shared, it can be transferred from one piece of code to another.

"Smart" pointers are classes that act like pointers, e.g. by overloading the `*` and `->` operators. Some smart pointer types can be used to automate ownership bookkeeping, to ensure these responsibilities are met. [`std::unique_ptr`](http://en.cppreference.com/w/cpp/memory/unique_ptr) is a smart pointer type introduced in C++11, which expresses exclusive ownership of a dynamically allocated object; the object is deleted when the `std::unique_ptr` goes out of scope. It cannot be copied, but can be _moved_ to represent ownership transfer. [`std::shared_ptr`](http://en.cppreference.com/w/cpp/memory/shared_ptr) is a smart pointer type that expresses shared ownership of a dynamically allocated object. `std::shared_ptr`s can be copied; ownership of the object is shared among all copies, and the object is deleted when the last `std::shared_ptr` is destroyed.

</div>

<div class="pros">

*   It's virtually impossible to manage dynamically allocated memory without some sort of ownership logic.
*   Transferring ownership of an object can be cheaper than copying it (if copying it is even possible).
*   Transferring ownership can be simpler than 'borrowing' a pointer or reference, because it reduces the need to coordinate the lifetime of the object between the two users.
*   Smart pointers can improve readability by making ownership logic explicit, self-documenting, and unambiguous.
*   Smart pointers can eliminate manual ownership bookkeeping, simplifying the code and ruling out large classes of errors.
*   For const objects, shared ownership can be a simple and efficient alternative to deep copying.

</div>

<div class="cons">

*   Ownership must be represented and transferred via pointers (whether smart or plain). Pointer semantics are more complicated than value semantics, especially in APIs: you have to worry not just about ownership, but also aliasing, lifetime, and mutability, among other issues.
*   The performance costs of value semantics are often overestimated, so the performance benefits of ownership transfer might not justify the readability and complexity costs.
*   APIs that transfer ownership force their clients into a single memory management model.
*   Code using smart pointers is less explicit about where the resource releases take place.
*   `std::unique_ptr` expresses ownership transfer using C++11's move semantics, which are relatively new and may confuse some programmers.
*   Shared ownership can be a tempting alternative to careful ownership design, obfuscating the design of a system.
*   Shared ownership requires explicit bookkeeping at run-time, which can be costly.
*   In some cases (e.g. cyclic references), objects with shared ownership may never be deleted.
*   Smart pointers are not perfect substitutes for plain pointers.

</div>

<div class="decision">

If dynamic allocation is necessary, prefer to keep ownership with the code that allocated it. If other code needs access to the object, consider passing it a copy, or passing a pointer or reference without transferring ownership. Prefer to use `std::unique_ptr` to make ownership transfer explicit. For example:

<pre>std::unique_ptr<Foo> FooFactory();
void FooConsumer(std::unique_ptr<Foo> ptr);
</pre>

Do not design your code to use shared ownership without a very good reason. One such reason is to avoid expensive copy operations, but you should only do this if the performance benefits are significant, and the underlying object is immutable (i.e. `std::shared_ptr<const Foo>`). If you do use shared ownership, prefer to use `std::shared_ptr`.

Never use `std::auto_ptr`. Instead, use `std::unique_ptr`.

</div>

</div>

### cpplint

<div class="summary">

Use `cpplint.py` to detect style errors.

</div>

<div class="stylebody">

`cpplint.py` is a tool that reads a source file and identifies many style errors. It is not perfect, and has both false positives and false negatives, but it is still a valuable tool. False positives can be ignored by putting `// NOLINT` at the end of the line or `// NOLINTNEXTLINE` in the previous line.

Some projects have instructions on how to run `cpplint.py` from their project tools. If the project you are contributing to does not, you can download [`cpplint.py`](https://raw.githubusercontent.com/google/styleguide/gh-pages/cpplint/cpplint.py) separately.

</div>

## Other C++ Features

### Rvalue References

<div class="summary">

Use rvalue references only to define move constructors and move assignment operators, or for perfect forwarding.

</div>

<div class="stylebody">

<div class="definition">

Rvalue references are a type of reference that can only bind to temporary objects. The syntax is similar to traditional reference syntax. For example, `void f(string&& s);` declares a function whose argument is an rvalue reference to a string.

</div>

<div class="pros">

*   Defining a move constructor (a constructor taking an rvalue reference to the class type) makes it possible to move a value instead of copying it. If `v1` is a `std::vector<string>`, for example, then `auto v2(std::move(v1))` will probably just result in some simple pointer manipulation instead of copying a large amount of data. In some cases this can result in a major performance improvement.
*   Rvalue references make it possible to write a generic function wrapper that forwards its arguments to another function, and works whether or not its arguments are temporary objects. (This is sometimes called "perfect forwarding".)
*   Rvalue references make it possible to implement types that are movable but not copyable, which can be useful for types that have no sensible definition of copying but where you might still want to pass them as function arguments, put them in containers, etc.
*   `std::move` is necessary to make effective use of some standard-library types, such as `std::unique_ptr`.

</div>

<div class="cons">

*   Rvalue references are a relatively new feature (introduced as part of C++11), and not yet widely understood. Rules like reference collapsing, and automatic synthesis of move constructors, are complicated.

</div>

<div class="decision">

Use rvalue references only to define move constructors and move assignment operators (as described in [Copyable and Movable Types](#Copyable_Movable_Types)) and, in conjunction with `[std::forward](http://en.cppreference.com/w/cpp/utility/forward)`, to support perfect forwarding. You may use `std::move` to express moving a value from one object to another rather than copying it.

</div>

</div>

### Friends

<div class="summary">

We allow use of `friend` classes and functions, within reason.

</div>

<div class="stylebody">

Friends should usually be defined in the same file so that the reader does not have to look in another file to find uses of the private members of a class. A common use of `friend` is to have a `FooBuilder` class be a friend of `Foo` so that it can construct the inner state of `Foo` correctly, without exposing this state to the world. In some cases it may be useful to make a unittest class a friend of the class it tests.

Friends extend, but do not break, the encapsulation boundary of a class. In some cases this is better than making a member public when you want to give only one other class access to it. However, most classes should interact with other classes solely through their public members.

</div>

### Exceptions

<div class="summary">

We do not use C++ exceptions.

</div>

<div class="stylebody">

<div class="pros">

*   Exceptions allow higher levels of an application to decide how to handle "can't happen" failures in deeply nested functions, without the obscuring and error-prone bookkeeping of error codes.
*   Exceptions are used by most other modern languages. Using them in C++ would make it more consistent with Python, Java, and the C++ that others are familiar with.
*   Some third-party C++ libraries use exceptions, and turning them off internally makes it harder to integrate with those libraries.
*   Exceptions are the only way for a constructor to fail. We can simulate this with a factory function or an `Init()` method, but these require heap allocation or a new "invalid" state, respectively.
*   Exceptions are really handy in testing frameworks.

</div>

<div class="cons">

*   When you add a `throw` statement to an existing function, you must examine all of its transitive callers. Either they must make at least the basic exception safety guarantee, or they must never catch the exception and be happy with the program terminating as a result. For instance, if `f()` calls `g()` calls `h()`, and `h` throws an exception that `f` catches, `g` has to be careful or it may not clean up properly.
*   More generally, exceptions make the control flow of programs difficult to evaluate by looking at code: functions may return in places you don't expect. This causes maintainability and debugging difficulties. You can minimize this cost via some rules on how and where exceptions can be used, but at the cost of more that a developer needs to know and understand.
*   Exception safety requires both RAII and different coding practices. Lots of supporting machinery is needed to make writing correct exception-safe code easy. Further, to avoid requiring readers to understand the entire call graph, exception-safe code must isolate logic that writes to persistent state into a "commit" phase. This will have both benefits and costs (perhaps where you're forced to obfuscate code to isolate the commit). Allowing exceptions would force us to always pay those costs even when they're not worth it.
*   Turning on exceptions adds data to each binary produced, increasing compile time (probably slightly) and possibly increasing address space pressure.
*   The availability of exceptions may encourage developers to throw them when they are not appropriate or recover from them when it's not safe to do so. For example, invalid user input should not cause exceptions to be thrown. We would need to make the style guide even longer to document these restrictions!

</div>

<div class="decision">

On their face, the benefits of using exceptions outweigh the costs, especially in new projects. However, for existing code, the introduction of exceptions has implications on all dependent code. If exceptions can be propagated beyond a new project, it also becomes problematic to integrate the new project into existing exception-free code. Because most existing C++ code at Google is not prepared to deal with exceptions, it is comparatively difficult to adopt new code that generates exceptions.

Given that Google's existing code is not exception-tolerant, the costs of using exceptions are somewhat greater than the costs in a new project. The conversion process would be slow and error-prone. We don't believe that the available alternatives to exceptions, such as error codes and assertions, introduce a significant burden.

Our advice against using exceptions is not predicated on philosophical or moral grounds, but practical ones. Because we'd like to use our open-source projects at Google and it's difficult to do so if those projects use exceptions, we need to advise against exceptions in Google open-source projects as well. Things would probably be different if we had to do it all over again from scratch.

This prohibition also applies to the exception-related features added in C++11, such as `noexcept`, `std::exception_ptr`, and `std::nested_exception`.

There is an [exception](#Windows_Code) to this rule (no pun intended) for Windows code.

</div>

</div>

### Run-Time Type Information (RTTI)

<div class="summary">

Avoid using Run Time Type Information (RTTI).

</div>

<div class="stylebody">

<div class="definition">

RTTI allows a programmer to query the C++ class of an object at run time. This is done by use of `typeid` or `dynamic_cast`.

</div>

<div class="cons">

Querying the type of an object at run-time frequently means a design problem. Needing to know the type of an object at runtime is often an indication that the design of your class hierarchy is flawed.

Undisciplined use of RTTI makes code hard to maintain. It can lead to type-based decision trees or switch statements scattered throughout the code, all of which must be examined when making further changes.

</div>

<div class="pros">

The standard alternatives to RTTI (described below) require modification or redesign of the class hierarchy in question. Sometimes such modifications are infeasible or undesirable, particularly in widely-used or mature code.

RTTI can be useful in some unit tests. For example, it is useful in tests of factory classes where the test has to verify that a newly created object has the expected dynamic type. It is also useful in managing the relationship between objects and their mocks.

RTTI is useful when considering multiple abstract objects. Consider

<pre>bool Base::Equal(Base* other) = 0;
bool Derived::Equal(Base* other) {
  Derived* that = dynamic_cast<Derived*>(other);
  if (that == NULL)
    return false;
  ...
}
</pre>

</div>

<div class="decision">

RTTI has legitimate uses but is prone to abuse, so you must be careful when using it. You may use it freely in unittests, but avoid it when possible in other code. In particular, think twice before using RTTI in new code. If you find yourself needing to write code that behaves differently based on the class of an object, consider one of the following alternatives to querying the type:

*   Virtual methods are the preferred way of executing different code paths depending on a specific subclass type. This puts the work within the object itself.
*   If the work belongs outside the object and instead in some processing code, consider a double-dispatch solution, such as the Visitor design pattern. This allows a facility outside the object itself to determine the type of class using the built-in type system.

When the logic of a program guarantees that a given instance of a base class is in fact an instance of a particular derived class, then a `dynamic_cast` may be used freely on the object. Usually one can use a `static_cast` as an alternative in such situations.

Decision trees based on type are a strong indication that your code is on the wrong track.

<pre class="badcode">if (typeid(*data) == typeid(D1)) {
  ...
} else if (typeid(*data) == typeid(D2)) {
  ...
} else if (typeid(*data) == typeid(D3)) {
...
</pre>

Code such as this usually breaks when additional subclasses are added to the class hierarchy. Moreover, when properties of a subclass change, it is difficult to find and modify all the affected code segments.

Do not hand-implement an RTTI-like workaround. The arguments against RTTI apply just as much to workarounds like class hierarchies with type tags. Moreover, workarounds disguise your true intent.

</div>

</div>

### Casting

<div class="summary">

Use C++-style casts like `static_cast<float>(double_value)`, or brace initialization for conversion of arithmetic types like `int64 y = int64{1} << 42`. Do not use cast formats like `int y = (int)x` or `int y = int(x)` (but the latter is okay when invoking a constructor of a class type).

</div>

<div class="stylebody">

<div class="definition">

C++ introduced a different cast system from C that distinguishes the types of cast operations.

</div>

<div class="pros">

The problem with C casts is the ambiguity of the operation; sometimes you are doing a _conversion_ (e.g., `(int)3.5`) and sometimes you are doing a _cast_ (e.g., `(int)"hello"`). Brace initialization and C++ casts can often help avoid this ambiguity. Additionally, C++ casts are more visible when searching for them.

</div>

<div class="cons">

The C++-style cast syntax is verbose and cumbersome.

</div>

<div class="decision">

Do not use C-style casts. Instead, use these C++-style casts when explicit type conversion is necessary.

*   Use brace initialization to convert arithmetic types (e.g. `int64{x}`). This is the safest approach because code will not compile if conversion can result in information loss. The syntax is also concise.
*   Use `static_cast` as the equivalent of a C-style cast that does value conversion, when you need to explicitly up-cast a pointer from a class to its superclass, or when you need to explicitly cast a pointer from a superclass to a subclass. In this last case, you must be sure your object is actually an instance of the subclass.
*   Use `const_cast` to remove the `const` qualifier (see [const](#Use_of_const)).
*   Use `reinterpret_cast` to do unsafe conversions of pointer types to and from integer and other pointer types. Use this only if you know what you are doing and you understand the aliasing issues.

See the [RTTI section](#Run-Time_Type_Information__RTTI_) for guidance on the use of `dynamic_cast`.

</div>

</div>

### Streams

<div class="summary">

Use streams where appropriate, and stick to "simple" usages.

</div>

<div class="stylebody">

<div class="definition">

Streams are the standard I/O abstraction in C++, as exemplified by the standard header `<iostream>`. They are widely used in Google code, but only for debug logging and test diagnostics.

</div>

<div class="pros">

The `<<` and `>>` stream operators provide an API for formatted I/O that is easily learned, portable, reusable, and extensible. `printf`, by contrast, doesn't even support `string`, to say nothing of user-defined types, and is very difficult to use portably. `printf` also obliges you to choose among the numerous slightly different versions of that function, and navigate the dozens of conversion specifiers.

Streams provide first-class support for console I/O via `std::cin`, `std::cout`, `std::cerr`, and `std::clog`. The C APIs do as well, but are hampered by the need to manually buffer the input.

</div>

<div class="cons">

*   Stream formatting can be configured by mutating the state of the stream. Such mutations are persistent, so the behavior of your code can be affected by the entire previous history of the stream, unless you go out of your way to restore it to a known state every time other code might have touched it. User code can not only modify the built-in state, it can add new state variables and behaviors through a registration system.
*   It is difficult to precisely control stream output, due to the above issues, the way code and data are mixed in streaming code, and the use of operator overloading (which may select a different overload than you expect).
*   The practice of building up output through chains of `<<` operators interferes with internationalization, because it bakes word order into the code, and streams' support for localization is [flawed](http://www.boost.org/doc/libs/1_48_0/libs/locale/doc/html/rationale.html#rationale_why).
*   The streams API is subtle and complex, so programmers must develop experience with it in order to use it effectively. However, streams were historically banned in Google code (except for logging and diagnostics), so Google engineers tend not to have that experience. Consequently, streams-based code is likely to be less readable and maintainable by Googlers than code based on more familiar abstractions.
*   Resolving the many overloads of `<<` is extremely costly for the compiler. When used pervasively in a large code base, it can consume as much as 20% of the parsing and semantic analysis time.

</div>

<div class="decision">

Use streams only when they are the best tool for the job. This is typically the case when the I/O is ad-hoc, local, human-readable, and targeted at other developers rather than end-users. Be consistent with the code around you, and with the codebase as a whole; if there's an established tool for your problem, use that tool instead.

Avoid using streams for I/O that faces external users or handles untrusted data. Instead, find and use the appropriate templating libraries to handle issues like internationalization, localization, and security hardening.

If you do use streams, avoid the stateful parts of the streams API (other than error state), such as `imbue()`, `xalloc()`, and `register_callback()`. Use explicit formatting functions rather than stream manipulators or formatting flags to control formatting details such as number base, precision, or padding.

Overload `<<` as a streaming operator for your type only if your type represents a value, and `<<` writes out a human-readable string representation of that value. Avoid exposing implementation details in the output of `<<`; if you need to print object internals for debugging, use named functions instead (a method named `DebugString()` is the most common convention).

</div>

</div>

### Preincrement and Predecrement

<div class="summary">

Use prefix form (`++i`) of the increment and decrement operators with iterators and other template objects.

</div>

<div class="stylebody">

<div class="definition">

When a variable is incremented (`++i` or `i++`) or decremented (`--i` or `i--`) and the value of the expression is not used, one must decide whether to preincrement (decrement) or postincrement (decrement).

</div>

<div class="pros">

When the return value is ignored, the "pre" form (`++i`) is never less efficient than the "post" form (`i++`), and is often more efficient. This is because post-increment (or decrement) requires a copy of `i` to be made, which is the value of the expression. If `i` is an iterator or other non-scalar type, copying `i` could be expensive. Since the two types of increment behave the same when the value is ignored, why not just always pre-increment?

</div>

<div class="cons">

The tradition developed, in C, of using post-increment when the expression value is not used, especially in `for` loops. Some find post-increment easier to read, since the "subject" (`i`) precedes the "verb" (`++`), just like in English.

</div>

<div class="decision">

For simple scalar (non-object) values there is no reason to prefer one form and we allow either. For iterators and other template types, use pre-increment.

</div>

</div>

### Use of const

<div class="summary">

Use `const` whenever it makes sense. With C++11, `constexpr` is a better choice for some uses of const.

</div>

<div class="stylebody">

<div class="definition">

Declared variables and parameters can be preceded by the keyword `const` to indicate the variables are not changed (e.g., `const int foo`). Class functions can have the `const` qualifier to indicate the function does not change the state of the class member variables (e.g., `class Foo { int Bar(char c) const; };`).

</div>

<div class="pros">

Easier for people to understand how variables are being used. Allows the compiler to do better type checking, and, conceivably, generate better code. Helps people convince themselves of program correctness because they know the functions they call are limited in how they can modify your variables. Helps people know what functions are safe to use without locks in multi-threaded programs.

</div>

<div class="cons">

`const` is viral: if you pass a `const` variable to a function, that function must have `const` in its prototype (or the variable will need a `const_cast`). This can be a particular problem when calling library functions.

</div>

<div class="decision">

`const` variables, data members, methods and arguments add a level of compile-time type checking; it is better to detect errors as soon as possible. Therefore we strongly recommend that you use `const` whenever it makes sense to do so:

*   If a function guarantees that it will not modify an argument passed by reference or by pointer, the corresponding function parameter should be a reference-to-const (`const T&`) or pointer-to-const (`const T*`), respectively.
*   Declare methods to be `const` whenever possible. Accessors should almost always be `const`. Other methods should be const if they do not modify any data members, do not call any non-`const` methods, and do not return a non-`const` pointer or non-`const` reference to a data member.
*   Consider making data members `const` whenever they do not need to be modified after construction.

The `mutable` keyword is allowed but is unsafe when used with threads, so thread safety should be carefully considered first.

</div>

<div class="stylepoint_subsection">

#### Where to put the const

Some people favor the form `int const *foo` to `const int* foo`. They argue that this is more readable because it's more consistent: it keeps the rule that `const` always follows the object it's describing. However, this consistency argument doesn't apply in codebases with few deeply-nested pointer expressions since most `const` expressions have only one `const`, and it applies to the underlying value. In such cases, there's no consistency to maintain. Putting the `const` first is arguably more readable, since it follows English in putting the "adjective" (`const`) before the "noun" (`int`).

That said, while we encourage putting `const` first, we do not require it. But be consistent with the code around you!

</div>

</div>

### Use of constexpr

<div class="summary">

In C++11, use `constexpr` to define true constants or to ensure constant initialization.

</div>

<div class="stylebody">

<div class="definition">

Some variables can be declared `constexpr` to indicate the variables are true constants, i.e. fixed at compilation/link time. Some functions and constructors can be declared `constexpr` which enables them to be used in defining a `constexpr` variable.

</div>

<div class="pros">

Use of `constexpr` enables definition of constants with floating-point expressions rather than just literals; definition of constants of user-defined types; and definition of constants with function calls.

</div>

<div class="cons">

Prematurely marking something as constexpr may cause migration problems if later on it has to be downgraded. Current restrictions on what is allowed in constexpr functions and constructors may invite obscure workarounds in these definitions.

</div>

<div class="decision">

`constexpr` definitions enable a more robust specification of the constant parts of an interface. Use `constexpr` to specify true constants and the functions that support their definitions. Avoid complexifying function definitions to enable their use with `constexpr`. Do not use `constexpr` to force inlining.

</div>

</div>

### Integer Types

<div class="summary">

Of the built-in C++ integer types, the only one used is `int`. If a program needs a variable of a different size, use a precise-width integer type from `<stdint.h>`, such as `int16_t`. If your variable represents a value that could ever be greater than or equal to 2^31 (2GiB), use a 64-bit type such as `int64_t`. Keep in mind that even if your value won't ever be too large for an `int`, it may be used in intermediate calculations which may require a larger type. When in doubt, choose a larger type.

</div>

<div class="stylebody">

<div class="definition">

C++ does not specify the sizes of its integer types. Typically people assume that `short` is 16 bits, `int` is 32 bits, `long` is 32 bits and `long long` is 64 bits.

</div>

<div class="pros">

Uniformity of declaration.

</div>

<div class="cons">

The sizes of integral types in C++ can vary based on compiler and architecture.

</div>

<div class="decision">

`<stdint.h>` defines types like `int16_t`, `uint32_t`, `int64_t`, etc. You should always use those in preference to `short`, `unsigned long long` and the like, when you need a guarantee on the size of an integer. Of the C integer types, only `int` should be used. When appropriate, you are welcome to use standard types like `size_t` and `ptrdiff_t`.

We use `int` very often, for integers we know are not going to be too big, e.g., loop counters. Use plain old `int` for such things. You should assume that an `int` is at least 32 bits, but don't assume that it has more than 32 bits. If you need a 64-bit integer type, use `int64_t` or `uint64_t`.

For integers we know can be "big", use `int64_t`.

You should not use the unsigned integer types such as `uint32_t`, unless there is a valid reason such as representing a bit pattern rather than a number, or you need defined overflow modulo 2^N. In particular, do not use unsigned types to say a number will never be negative. Instead, use assertions for this.

If your code is a container that returns a size, be sure to use a type that will accommodate any possible usage of your container. When in doubt, use a larger type rather than a smaller type.

Use care when converting integer types. Integer conversions and promotions can cause non-intuitive behavior.

</div>

<div class="stylepoint_subsection">

#### On Unsigned Integers

Some people, including some textbook authors, recommend using unsigned types to represent numbers that are never negative. This is intended as a form of self-documentation. However, in C, the advantages of such documentation are outweighed by the real bugs it can introduce. Consider:

<pre>for (unsigned int i = foo.Length()-1; i >= 0; --i) ...
</pre>

This code will never terminate! Sometimes gcc will notice this bug and warn you, but often it will not. Equally bad bugs can occur when comparing signed and unsigned variables. Basically, C's type-promotion scheme causes unsigned types to behave differently than one might expect.

So, document that a variable is non-negative using assertions. Don't use an unsigned type.

</div>

</div>

### 64-bit Portability

<div class="summary">

Code should be 64-bit and 32-bit friendly. Bear in mind problems of printing, comparisons, and structure alignment.

</div>

<div class="stylebody">

*   `printf()` specifiers for some types are not cleanly portable between 32-bit and 64-bit systems. C99 defines some portable format specifiers. Unfortunately, MSVC 7.1 does not understand some of these specifiers and the standard is missing a few, so we have to define our own ugly versions in some cases (in the style of the standard include file `inttypes.h`):

    <div>

    <pre>// printf macros for size_t, in the style of inttypes.h
    #ifdef _LP64
    #define __PRIS_PREFIX "z"
    #else
    #define __PRIS_PREFIX
    #endif

    // Use these macros after a % in a printf format string
    // to get correct 32/64 bit behavior, like this:
    // size_t size = records.size();
    // printf("%" PRIuS "\n", size);

    #define PRIdS __PRIS_PREFIX "d"
    #define PRIxS __PRIS_PREFIX "x"
    #define PRIuS __PRIS_PREFIX "u"
    #define PRIXS __PRIS_PREFIX "X"
    #define PRIoS __PRIS_PREFIX "o"
      </pre>

    </div>

    <table border="1" summary="portable printf specifiers">

    <tbody>

    <tr align="center">

    <th>Type</th>

    <th>DO NOT use</th>

    <th>DO use</th>

    <th>Notes</th>

    </tr>

    <tr align="center">

    <td>`void *` (or any pointer)</td>

    <td>`%lx`</td>

    <td>`%p`</td>

    <td></td>

    </tr>

    <tr align="center">

    <td>`int64_t`</td>

    <td>`%qd`, `%lld`</td>

    <td>`%" PRId64 "`</td>

    <td></td>

    </tr>

    <tr align="center">

    <td>`uint64_t`</td>

    <td>`%qu`, `%llu`, `%llx`</td>

    <td>`%" PRIu64 "`, `%" PRIx64 "`</td>

    <td></td>

    </tr>

    <tr align="center">

    <td>`size_t`</td>

    <td>`%u`</td>

    <td>`%" PRIuS "`, `%" PRIxS "`</td>

    <td>C99 specifies `%zu`</td>

    </tr>

    <tr align="center">

    <td>`ptrdiff_t`</td>

    <td>`%d`</td>

    <td>`%" PRIdS "`</td>

    <td>C99 specifies `%td`</td>

    </tr>

    </tbody>

    </table>

    Note that the `PRI*` macros expand to independent strings which are concatenated by the compiler. Hence if you are using a non-constant format string, you need to insert the value of the macro into the format, rather than the name. Note also that spaces are required around the macro identifier to separate it from the string literal. It is still possible, as usual, to include length specifiers, etc., after the `%` when using the `PRI*` macros. So, e.g. `printf("x = %30" PRIuS "\n", x)` would expand on 32-bit Linux to `printf("x = %30" "u" "\n", x)`, which the compiler will treat as `printf("x = %30u\n", x)`.

*   Remember that `sizeof(void *)` != `sizeof(int)`. Use `intptr_t` if you want a pointer-sized integer.
*   You may need to be careful with structure alignments, particularly for structures being stored on disk. Any class/structure with a `int64_t`/`uint64_t` member will by default end up being 8-byte aligned on a 64-bit system. If you have such structures being shared on disk between 32-bit and 64-bit code, you will need to ensure that they are packed the same on both architectures. Most compilers offer a way to alter structure alignment. For gcc, you can use `__attribute__((packed))`. MSVC offers `#pragma pack()` and `__declspec(align())`.
*   Use the `LL` or `ULL` suffixes as needed to create 64-bit constants. For example:

    <pre>int64_t my_value = 0x123456789LL;
    uint64_t my_mask = 3ULL << 48;
    </pre>

</div>

### Preprocessor Macros

<div class="summary">

Avoid defining macros, especially in headers; prefer inline functions, enums, and `const` variables. Name macros with a project-specific prefix. Do not use macros to define pieces of a C++ API.

</div>

<div class="stylebody">

Macros mean that the code you see is not the same as the code the compiler sees. This can introduce unexpected behavior, especially since macros have global scope.

The problems introduced by macros are especially severe when they are used to define pieces of a C++ API, and still more so for public APIs. Every error message from the compiler when developers incorrectly use that interface now must explain how the macros formed the interface. Refactoring and analysis tools have a dramatically harder time updating the interface. As a consequence, we specifically disallow using macros in this way. For example, avoid patterns like:

<pre class="badcode">class WOMBAT_TYPE(Foo) {
  // ...

 public:
  EXPAND_PUBLIC_WOMBAT_API(Foo)

  EXPAND_WOMBAT_COMPARISONS(Foo, ==, <)
};
</pre>

Luckily, macros are not nearly as necessary in C++ as they are in C. Instead of using a macro to inline performance-critical code, use an inline function. Instead of using a macro to store a constant, use a `const` variable. Instead of using a macro to "abbreviate" a long variable name, use a reference. Instead of using a macro to conditionally compile code ... well, don't do that at all (except, of course, for the `#define` guards to prevent double inclusion of header files). It makes testing much more difficult.

Macros can do things these other techniques cannot, and you do see them in the codebase, especially in the lower-level libraries. And some of their special features (like stringifying, concatenation, and so forth) are not available through the language proper. But before using a macro, consider carefully whether there's a non-macro way to achieve the same result. If you need to use a macro to define an interface, contact your project leads to request a waiver of this rule.

The following usage pattern will avoid many problems with macros; if you use macros, follow it whenever possible:

*   Don't define macros in a `.h` file.
*   `#define` macros right before you use them, and `#undef` them right after.
*   Do not just `#undef` an existing macro before replacing it with your own; instead, pick a name that's likely to be unique.
*   Try not to use macros that expand to unbalanced C++ constructs, or at least document that behavior well.
*   Prefer not using `##` to generate function/class/variable names.

Exporting macros from headers (i.e. defining them in a header without `#undef`ing them before the end of the header) is extremely strongly discouraged. If you do export a macro from a header, it must have a globally unique name. To achieve this, it must be named with a prefix consisting of your project's namespace name (but upper case).

</div>

### 0 and nullptr/NULL

<div class="summary">

Use `0` for integers, `0.0` for reals, `nullptr` (or `NULL`) for pointers, and `'\0'` for chars.

</div>

<div class="stylebody">

Use `0` for integers and `0.0` for reals. This is not controversial.

For pointers (address values), there is a choice between `0`, `NULL`, and `nullptr`. For projects that allow C++11 features, use `nullptr`. For C++03 projects, we prefer `NULL` because it looks like a pointer. In fact, some C++ compilers provide special definitions of `NULL` which enable them to give useful warnings, particularly in situations where `sizeof(NULL)` is not equal to `sizeof(0)`.

Use `'\0'` for chars. This is the correct type and also makes code more readable.

</div>

### sizeof

<div class="summary">

Prefer `sizeof(<var>varname</var>)` to `sizeof(<var>type</var>)`.

</div>

<div class="stylebody">

Use `sizeof(<var>varname</var>)` when you take the size of a particular variable. `sizeof(<var>varname</var>)` will update appropriately if someone changes the variable type either now or later. You may use `sizeof(<var>type</var>)` for code unrelated to any particular variable, such as code that manages an external or internal data format where a variable of an appropriate C++ type is not convenient.

<pre>Struct data;
memset(&data, 0, sizeof(data));
</pre>

<pre class="badcode">memset(&data, 0, sizeof(Struct));
</pre>

<pre>if (raw_size < sizeof(int)) {
  LOG(ERROR) << "compressed record not big enough for count: " << raw_size;
  return false;
}
</pre>

</div>

### auto

<div class="summary">

Use `auto` to avoid type names that are noisy, obvious, or unimportant - cases where the type doesn't aid in clarity for the reader. Continue to use manifest type declarations when it helps readability.

</div>

<div class="stylebody">

<div class="pros">

*   C++ type names can be long and cumbersome, especially when they involve templates or namespaces.
*   When a C++ type name is repeated within a single declaration or a small code region, the repetition may not be aiding readability.
*   It is sometimes safer to let the type be specified by the type of the initialization expression, since that avoids the possibility of unintended copies or type conversions.

</div>

<div class="cons">

Sometimes code is clearer when types are manifest, especially when a variable's initialization depends on things that were declared far away. In expressions like:

<pre class="badcode">auto foo = x.add_foo();
auto i = y.Find(key);
</pre>

it may not be obvious what the resulting types are if the type of `y` isn't very well known, or if `y` was declared many lines earlier.

Programmers have to understand the difference between `auto` and `const auto&` or they'll get copies when they didn't mean to.

If an `auto` variable is used as part of an interface, e.g. as a constant in a header, then a programmer might change its type while only intending to change its value, leading to a more radical API change than intended.

</div>

<div class="decision">

`auto` is permitted when it increases readability, particularly as described below. Never initialize an `auto`-typed variable with a braced initializer list.

Specific cases where `auto` is allowed or encouraged:

*   (Encouraged) For iterators and other long/cluttery type names, particularly when the type is clear from context (calls to `find`, `begin`, or `end` for instance).
*   (Allowed) When the type is clear from local context (in the same expression or within a few lines). Initialization of a pointer or smart pointer with calls to `new` commonly falls into this category, as does use of `auto` in a range-based loop over a container whose type is spelled out nearby.
*   (Allowed) When the type doesn't matter because it isn't being used for anything other than equality comparison.
*   (Encouraged) When iterating over a map with a range-based loop (because it is often assumed that the correct type is `std::pair<KeyType, ValueType>` whereas it is actually `std::pair<const KeyType, ValueType>`). This is particularly well paired with local `key` and `value` aliases for `.first` and `.second` (often const-ref).

    <pre class="code">for (const auto& item : some_map) {
      const KeyType& key = item.first;
      const ValType& value = item.second;
      // The rest of the loop can now just refer to key and value,
      // a reader can see the types in question, and we've avoided
      // the too-common case of extra copies in this iteration.
    }
    </pre>

</div>

</div>

### Braced Initializer List

<div class="summary">

You may use braced initializer lists.

</div>

<div class="stylebody">

In C++03, aggregate types (arrays and structs with no constructor) could be initialized with braced initializer lists.

<pre>struct Point { int x; int y; };
Point p = {1, 2};
</pre>

In C++11, this syntax was generalized, and any object type can now be created with a braced initializer list, known as a _braced-init-list_ in the C++ grammar. Here are a few examples of its use.

<pre>// Vector takes a braced-init-list of elements.
std::vector<string> v{"foo", "bar"};

// Basically the same, ignoring some small technicalities.
// You may choose to use either form.
std::vector<string> v = {"foo", "bar"};

// Usable with 'new' expressions.
auto p = new vector<string>{"foo", "bar"};

// A map can take a list of pairs. Nested braced-init-lists work.
std::map<int, string> m = {{1, "one"}, {2, "2"}};

// A braced-init-list can be implicitly converted to a return type.
std::vector<int> test_function() { return {1, 2, 3}; }

// Iterate over a braced-init-list.
for (int i : {-1, -2, -3}) {}

// Call a function using a braced-init-list.
void TestFunction2(std::vector<int> v) {}
TestFunction2({1, 2, 3});
</pre>

A user-defined type can also define a constructor and/or assignment operator that take `std::initializer_list<T>`, which is automatically created from _braced-init-list_:

<pre>class MyType {
 public:
  // std::initializer_list references the underlying init list.
  // It should be passed by value.
  MyType(std::initializer_list<int> init_list) {
    for (int i : init_list) append(i);
  }
  MyType& operator=(std::initializer_list<int> init_list) {
    clear();
    for (int i : init_list) append(i);
  }
};
MyType m{2, 3, 5, 7};
</pre>

Finally, brace initialization can also call ordinary constructors of data types, even if they do not have `std::initializer_list<T>` constructors.

<pre>double d{1.23};
// Calls ordinary constructor as long as MyOtherType has no
// std::initializer_list constructor.
class MyOtherType {
 public:
  explicit MyOtherType(string);
  MyOtherType(int, string);
};
MyOtherType m = {1, "b"};
// If the constructor is explicit, you can't use the "= {}" form.
MyOtherType m{"b"};
</pre>

Never assign a _braced-init-list_ to an auto local variable. In the single element case, what this means can be confusing.

<pre class="badcode">auto d = {1.23};        // d is a std::initializer_list<double>
</pre>

<pre>auto d = double{1.23};  // Good -- d is a double, not a std::initializer_list.
</pre>

See [Braced_Initializer_List_Format](#Braced_Initializer_List_Format) for formatting.

</div>

### Lambda expressions

<div class="summary">

Use lambda expressions where appropriate. Prefer explicit captures when the lambda will escape the current scope.

</div>

<div class="stylebody">

<div class="definition">

Lambda expressions are a concise way of creating anonymous function objects. They're often useful when passing functions as arguments. For example:

<pre>std::sort(v.begin(), v.end(), [](int x, int y) {
  return Weight(x) < Weight(y);
});
</pre>

They further allow capturing variables from the enclosing scope either explicitly by name, or implicitly using a default capture. Explicit captures require each variable to be listed, as either a value or reference capture:

<pre>int weight = 3;
int sum = 0;
// Captures `weight` by value and `sum` by reference.
std::for_each(v.begin(), v.end(), [weight, &sum](int x) {
  sum += weight * x;
});
</pre>

Default captures implicitly capture any variable referenced in the lambda body, including `this` if any members are used:

<pre>const std::vector<int> lookup_table = ...;
std::vector<int> indices = ...;
// Captures `lookup_table` by reference, sorts `indices` by the value
// of the associated element in `lookup_table`.
std::sort(indices.begin(), indices.end(), [&](int a, int b) {
  return lookup_table[a] < lookup_table[b];
});
</pre>

Lambdas were introduced in C++11 along with a set of utilities for working with function objects, such as the polymorphic wrapper `std::function`.

</div>

<div class="pros">

*   Lambdas are much more concise than other ways of defining function objects to be passed to STL algorithms, which can be a readability improvement.
*   Appropriate use of default captures can remove redundancy and highlight important exceptions from the default.
*   Lambdas, `std::function`, and `std::bind` can be used in combination as a general purpose callback mechanism; they make it easy to write functions that take bound functions as arguments.

</div>

<div class="cons">

*   Variable capture in lambdas can be a source of dangling-pointer bugs, particularly if a lambda escapes the current scope.
*   Default captures by value can be misleading because they do not prevent dangling-pointer bugs. Capturing a pointer by value doesn't cause a deep copy, so it often has the same lifetime issues as capture by reference. This is especially confusing when capturing 'this' by value, since the use of 'this' is often implicit.
*   It's possible for use of lambdas to get out of hand; very long nested anonymous functions can make code harder to understand.

</div>

<div class="decision">

*   Use lambda expressions where appropriate, with formatting as described [below](#Formatting_Lambda_Expressions).
*   Prefer explicit captures if the lambda may escape the current scope. For example, instead of:

    <pre class="badcode">{
      Foo foo;
      ...
      executor->Schedule([&] { Frobnicate(foo); })
      ...
    }
    // BAD! The fact that the lambda makes use of a reference to `foo` and
    // possibly `this` (if `Frobnicate` is a member function) may not be
    // apparent on a cursory inspection. If the lambda is invoked after
    // the function returns, that would be bad, because both `foo`
    // and the enclosing object could have been destroyed.
    </pre>

    prefer to write:

    <pre>{
      Foo foo;
      ...
      executor->Schedule([&foo] { Frobnicate(foo); })
      ...
    }
    // BETTER - The compile will fail if `Frobnicate` is a member
    // function, and it's clearer that `foo` is dangerously captured by
    // reference.
    </pre>

*   Use default capture by reference ([&]) only when the lifetime of the lambda is obviously shorter than any potential captures.
*   Use default capture by value ([=]) only as a means of binding a few variables for a short lambda, where the set of captured variables is obvious at a glance. Prefer not to write long or complex lambdas with default capture by value.
*   Keep unnamed lambdas short. If a lambda body is more than maybe five lines long, prefer to give the lambda a name, or to use a named function instead of a lambda.
*   Specify the return type of the lambda explicitly if that will make it more obvious to readers, as with [`auto`](#auto).

</div>

</div>

### Template metaprogramming

<div class="summary">

Avoid complicated template programming.

</div>

<div class="stylebody">

<div class="definition">

Template metaprogramming refers to a family of techniques that exploit the fact that the C++ template instantiation mechanism is Turing complete and can be used to perform arbitrary compile-time computation in the type domain.

</div>

<div class="pros">

Template metaprogramming allows extremely flexible interfaces that are type safe and high performance. Facilities like [Google Test](https://code.google.com/p/googletest/), `std::tuple`, `std::function`, and Boost.Spirit would be impossible without it.

</div>

<div class="cons">

The techniques used in template metaprogramming are often obscure to anyone but language experts. Code that uses templates in complicated ways is often unreadable, and is hard to debug or maintain.

Template metaprogramming often leads to extremely poor compiler time error messages: even if an interface is simple, the complicated implementation details become visible when the user does something wrong.

Template metaprogramming interferes with large scale refactoring by making the job of refactoring tools harder. First, the template code is expanded in multiple contexts, and it's hard to verify that the transformation makes sense in all of them. Second, some refactoring tools work with an AST that only represents the structure of the code after template expansion. It can be difficult to automatically work back to the original source construct that needs to be rewritten.

</div>

<div class="decision">

Template metaprogramming sometimes allows cleaner and easier-to-use interfaces than would be possible without it, but it's also often a temptation to be overly clever. It's best used in a small number of low level components where the extra maintenance burden is spread out over a large number of uses.

Think twice before using template metaprogramming or other complicated template techniques; think about whether the average member of your team will be able to understand your code well enough to maintain it after you switch to another project, or whether a non-C++ programmer or someone casually browsing the code base will be able to understand the error messages or trace the flow of a function they want to call. If you're using recursive template instantiations or type lists or metafunctions or expression templates, or relying on SFINAE or on the `sizeof` trick for detecting function overload resolution, then there's a good chance you've gone too far.

If you use template metaprogramming, you should expect to put considerable effort into minimizing and isolating the complexity. You should hide metaprogramming as an implementation detail whenever possible, so that user-facing headers are readable, and you should make sure that tricky code is especially well commented. You should carefully document how the code is used, and you should say something about what the "generated" code looks like. Pay extra attention to the error messages that the compiler emits when users make mistakes. The error messages are part of your user interface, and your code should be tweaked as necessary so that the error messages are understandable and actionable from a user point of view.

</div>

</div>

### Boost

<div class="summary">

Use only approved libraries from the Boost library collection.

</div>

<div class="stylebody">

<div class="definition">

The [Boost library collection](https://www.boost.org/) is a popular collection of peer-reviewed, free, open-source C++ libraries.

</div>

<div class="pros">

Boost code is generally very high-quality, is widely portable, and fills many important gaps in the C++ standard library, such as type traits and better binders.

</div>

<div class="cons">

Some Boost libraries encourage coding practices which can hamper readability, such as metaprogramming and other advanced template techniques, and an excessively "functional" style of programming.

</div>

<div class="decision">

<div>

In order to maintain a high level of readability for all contributors who might read and maintain code, we only allow an approved subset of Boost features. Currently, the following libraries are permitted:

*   [Call Traits](https://www.boost.org/libs/utility/call_traits.htm) from `boost/call_traits.hpp`
*   [Compressed Pair](https://www.boost.org/libs/utility/compressed_pair.htm) from `boost/compressed_pair.hpp`
*   [The Boost Graph Library (BGL)](https://www.boost.org/libs/graph/) from `boost/graph`, except serialization (`adj_list_serialize.hpp`) and parallel/distributed algorithms and data structures (`boost/graph/parallel/*` and `boost/graph/distributed/*`).
*   [Property Map](https://www.boost.org/libs/property_map/) from `boost/property_map`, except parallel/distributed property maps (`boost/property_map/parallel/*`).
*   [Iterator](https://www.boost.org/libs/iterator/) from `boost/iterator`
*   The part of [Polygon](https://www.boost.org/libs/polygon/) that deals with Voronoi diagram construction and doesn't depend on the rest of Polygon: `boost/polygon/voronoi_builder.hpp`, `boost/polygon/voronoi_diagram.hpp`, and `boost/polygon/voronoi_geometry_type.hpp`
*   [Bimap](https://www.boost.org/libs/bimap/) from `boost/bimap`
*   [Statistical Distributions and Functions](https://www.boost.org/libs/math/doc/html/dist.html) from `boost/math/distributions`
*   [Special Functions](https://www.boost.org/libs/math/doc/html/special.html) from `boost/math/special_functions`
*   [Multi-index](https://www.boost.org/libs/multi_index/) from `boost/multi_index`
*   [Heap](https://www.boost.org/libs/heap/) from `boost/heap`
*   The flat containers from [Container](https://www.boost.org/libs/container/): `boost/container/flat_map`, and `boost/container/flat_set`
*   [Intrusive](https://www.boost.org/libs/intrusive/) from `boost/intrusive`.
*   [The `boost/sort` library](https://www.boost.org/libs/sort/).
*   [Preprocessor](https://www.boost.org/libs/preprocessor/) from `boost/preprocessor`.

We are actively considering adding other Boost features to the list, so this list may be expanded in the future.

</div>

The following libraries are permitted, but their use is discouraged because they've been superseded by standard libraries in C++11:

*   [Array](https://www.boost.org/libs/array/) from `boost/array.hpp`: use [`std::array`](http://en.cppreference.com/w/cpp/container/array) instead.
*   [Pointer Container](https://www.boost.org/libs/ptr_container/) from `boost/ptr_container`: use containers of [`std::unique_ptr`](http://en.cppreference.com/w/cpp/memory/unique_ptr) instead.

</div>

</div>

### std::hash

<div class="summary">

Do not define specializations of `std::hash`.

</div>

<div class="stylebody">

<div class="definition">

`std::hash<T>` is the function object that the C++11 hash containers use to hash keys of type `T`, unless the user explicitly specifies a different hash function. For example, `std::unordered_map<int, string>` is a hash map that uses `std::hash<int>` to hash its keys, whereas `std::unordered_map<int, string, MyIntHash>` uses `MyIntHash`.

`std::hash` is defined for all integral, floating-point, pointer, and `enum` types, as well as some standard library types such as `string` and `unique_ptr`. Users can enable it to work for their own types by defining specializations of it for those types.

</div>

<div class="pros">

`std::hash` is easy to use, and simplifies the code since you don't have to name it explicitly. Specializing `std::hash` is the standard way of specifying how to hash a type, so it's what outside resources will teach, and what new engineers will expect.

</div>

<div class="cons">

`std::hash` is hard to specialize. It requires a lot of boilerplate code, and more importantly, it combines responsibility for identifying the hash inputs with responsibility for executing the hashing algorithm itself. The type author has to be responsible for the former, but the latter requires expertise that a type author usually doesn't have, and shouldn't need. The stakes here are high because low-quality hash functions can be security vulnerabilities, due to the emergence of [hash flooding attacks](https://emboss.github.io/blog/2012/12/14/breaking-murmur-hash-flooding-dos-reloaded/).

Even for experts, `std::hash` specializations are inordinately difficult to implement correctly for compound types, because the implementation cannot recursively call `std::hash` on data members. High-quality hash algorithms maintain large amounts of internal state, and reducing that state to the `size_t` bytes that `std::hash` returns is usually the slowest part of the computation, so it should not be done more than once.

Due to exactly that issue, `std::hash` does not work with `std::pair` or `std::tuple`, and the language does not allow us to extend it to support them.

</div>

<div class="decision">

You can use `std::hash` with the types that it supports "out of the box", but do not specialize it to support additional types. If you need a hash table with a key type that `std::hash` does not support, consider using legacy hash containers (e.g. `hash_map`) for now; they use a different default hasher, which is unaffected by this prohibition.

If you want to use the standard hash containers anyway, you will need to specify a custom hasher for the key type, e.g.

<pre>std::unordered_map<MyKeyType, Value, MyKeyTypeHasher> my_map;
</pre>

Consult with the type's owners to see if there is an existing hasher that you can use; otherwise work with them to provide one, or roll your own.

We are planning to provide a hash function that can work with any type, using a new customization mechanism that doesn't have the drawbacks of `std::hash`.

</div>

</div>

### C++11

<div class="summary">

Use libraries and language extensions from C++11 when appropriate. Consider portability to other environments before using C++11 features in your project.

</div>

<div class="stylebody">

<div class="definition">

C++11 contains [significant changes](https://en.wikipedia.org/wiki/C%2B%2B11) both to the language and libraries.

</div>

<div class="pros">

C++11 was the official standard until august 2014, and is supported by most C++ compilers. It standardizes some common C++ extensions that we use already, allows shorthands for some operations, and has some performance and safety improvements.

</div>

<div class="cons">

The C++11 standard is substantially more complex than its predecessor (1,300 pages versus 800 pages), and is unfamiliar to many developers. The long-term effects of some features on code readability and maintenance are unknown. We cannot predict when its various features will be implemented uniformly by tools that may be of interest, particularly in the case of projects that are forced to use older versions of tools.

As with [Boost](#Boost), some C++11 extensions encourage coding practices that hamper readability—for example by removing checked redundancy (such as type names) that may be helpful to readers, or by encouraging template metaprogramming. Other extensions duplicate functionality available through existing mechanisms, which may lead to confusion and conversion costs.

</div>

<div class="decision">

C++11 features may be used unless specified otherwise. In addition to what's described in the rest of the style guide, the following C++11 features may not be used:

*   Compile-time rational numbers (`<ratio>`), because of concerns that it's tied to a more template-heavy interface style.
*   The `<cfenv>` and `<fenv.h>` headers, because many compilers do not support those features reliably.
*   Ref-qualifiers on member functions, such as `void X::Foo() &` or `void X::Foo() &&`, because of concerns that they're an overly obscure feature.

</div>

</div>

### Nonstandard Extensions

<div class="summary">

Nonstandard extensions to C++ may not be used unless otherwise specified.

</div>

<div class="stylebody">

<div class="definition">

Compilers support various extensions that are not part of standard C++. Such extensions include GCC's `__attribute__`, intrinsic functions such as `__builtin_prefetch`, designated initializers (e.g. `Foo f = {.field = 3}`), inline assembly, `__COUNTER__`, `__PRETTY_FUNCTION__`, compound statement expressions (e.g. `foo = ({ int x; Bar(&x); x })`, variable-length arrays and `alloca()`, and the `a?:b` syntax.

</div>

<div class="pros">

*   Nonstandard extensions may provide useful features that do not exist in standard C++. For example, some people think that designated initializers are more readable than standard C++ features like constructors.
*   Important performance guidance to the compiler can only be specified using extensions.

</div>

<div class="cons">

*   Nonstandard extensions do not work in all compilers. Use of nonstandard extensions reduces portability of code.
*   Even if they are supported in all targeted compilers, the extensions are often not well-specified, and there may be subtle behavior differences between compilers.
*   Nonstandard extensions add to the language features that a reader must know to understand the code.

</div>

<div class="decision">

Do not use nonstandard extensions. You may use portability wrappers that are implemented using nonstandard extensions, so long as those wrappers are provided by a designated project-wide portability header.

</div>

</div>

### Aliases

<div class="summary">

Public aliases are for the benefit of an API's user, and should be clearly documented.

</div>

<div class="stylebody">

<div class="definition">

There are several ways to create names that are aliases of other entities:

<pre>typedef Foo Bar;
using Bar = Foo;
using other_namespace::Foo;
</pre>

Like other declarations, aliases declared in a header file are part of that header's public API unless they're in a function definition, in the private portion of a class, or in an explicitly-marked internal namespace. Aliases in such areas or in .cc files are implementation details (because client code can't refer to them), and are not restricted by this rule.

</div>

<div class="pros">

*   Aliases can improve readability by simplifying a long or complicated name.
*   Aliases can reduce duplication by naming in one place a type used repeatedly in an API, which _might_ make it easier to change the type later.

</div>

<div class="cons">

*   When placed in a header where client code can refer to them, aliases increase the number of entities in that header's API, increasing its complexity.
*   Clients can easily rely on unintended details of public aliases, making changes difficult.
*   It can be tempting to create a public alias that is only intended for use in the implementation, without considering its impact on the API, or on maintainability.
*   Aliases can create risk of name collisions
*   Aliases can reduce readability by giving a familiar construct an unfamiliar name
*   Type aliases can create an unclear API contract: it is unclear whether the alias is guaranteed to be identical to the type it aliases, to have the same API, or only to be usable in specified narrow ways

</div>

<div class="decision">

Don't put an alias in your public API just to save typing in the implementation; do so only if you intend it to be used by your clients.

When defining a public alias, document the intent of the new name, including whether it is guaranteed to always be the same as the type it's currently aliased to, or whether a more limited compatibility is intended. This lets the user know whether they can treat the types as substitutable or whether more specific rules must be followed, and can help the implementation retain some degree of freedom to change the alias.

Don't put namespace aliases in your public API. (See also [Namespaces](#Namespaces)).

For example, these aliases document how they are intended to be used in client code:

<pre>namespace a {
// Used to store field measurements. DataPoint may change from Bar* to some internal type.
// Client code should treat it as an opaque pointer.
using DataPoint = foo::bar::Bar*;

// A set of measurements. Just an alias for user convenience.
using TimeSeries = std::unordered_set<DataPoint, std::hash<DataPoint>, DataPointComparator>;
}  // namespace a
</pre>

These aliases don't document intended use, and half of them aren't meant for client use:

<pre class="badcode">namespace a {
// Bad: none of these say how they should be used.
using DataPoint = foo::bar::Bar*;
using std::unordered_set;  // Bad: just for local convenience
using std::hash;           // Bad: just for local convenience
typedef unordered_set<DataPoint, hash<DataPoint>, DataPointComparator> TimeSeries;
}  // namespace a
</pre>

However, local convenience aliases are fine in function definitions, private sections of classes, explicitly marked internal namespaces, and in .cc files:

<pre>// In a .cc file
using std::unordered_set;
</pre>

</div>

</div>

## Naming

The most important consistency rules are those that govern naming. The style of a name immediately informs us what sort of thing the named entity is: a type, a variable, a function, a constant, a macro, etc., without requiring us to search for the declaration of that entity. The pattern-matching engine in our brains relies a great deal on these naming rules.

Naming rules are pretty arbitrary, but we feel that consistency is more important than individual preferences in this area, so regardless of whether you find them sensible or not, the rules are the rules.

### General Naming Rules

<div class="summary">

Names should be descriptive; avoid abbreviation.

</div>

<div class="stylebody">

Give as descriptive a name as possible, within reason. Do not worry about saving horizontal space as it is far more important to make your code immediately understandable by a new reader. Do not use abbreviations that are ambiguous or unfamiliar to readers outside your project, and do not abbreviate by deleting letters within a word.

<pre>int price_count_reader;    // No abbreviation.
int num_errors;            // "num" is a widespread convention.
int num_dns_connections;   // Most people know what "DNS" stands for.
</pre>

<pre class="badcode">int n;                     // Meaningless.
int nerr;                  // Ambiguous abbreviation.
int n_comp_conns;          // Ambiguous abbreviation.
int wgc_connections;       // Only your group knows what this stands for.
int pc_reader;             // Lots of things can be abbreviated "pc".
int cstmr_id;              // Deletes internal letters.
</pre>

Note that certain universally-known abbreviations are OK, such as `i` for an iteration variable and `T` for a template parameter.

Template parameters should follow the naming style for their category: type template parameters should follow the rules for [type names](#Type_Names), and non-type template parameters should follow the rules for [variable names](#Variable_Names).

</div>

### File Names

<div class="summary">

Filenames should be all lowercase and can include underscores (`_`) or dashes (`-`). Follow the convention that your project uses. If there is no consistent local pattern to follow, prefer "_".

</div>

<div class="stylebody">

Examples of acceptable file names:

*   `my_useful_class.cc`
*   `my-useful-class.cc`
*   `myusefulclass.cc`
*   `myusefulclass_test.cc // _unittest and _regtest are deprecated.`

C++ files should end in `.cc` and header files should end in `.h`. Files that rely on being textually included at specific points should end in `.inc` (see also the section on [self-contained headers](#Self_contained_Headers)).

Do not use filenames that already exist in `/usr/include`, such as `db.h`.

In general, make your filenames very specific. For example, use `http_server_logs.h` rather than `logs.h`. A very common case is to have a pair of files called, e.g., `foo_bar.h` and `foo_bar.cc`, defining a class called `FooBar`.

Inline functions must be in a `.h` file. If your inline functions are very short, they should go directly into your `.h` file.

</div>

### Type Names

<div class="summary">

Type names start with a capital letter and have a capital letter for each new word, with no underscores: `MyExcitingClass`, `MyExcitingEnum`.

</div>

<div class="stylebody">

The names of all types — classes, structs, type aliases, enums, and type template parameters — have the same naming convention. Type names should start with a capital letter and have a capital letter for each new word. No underscores. For example:

<pre>// classes and structs
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// typedefs
typedef hash_map<UrlTableProperties *, string> PropertiesMap;

// using aliases
using PropertiesMap = hash_map<UrlTableProperties *, string>;

// enums
enum UrlTableErrors { ...
</pre>

</div>

### Variable Names

<div class="summary">

The names of variables (including function parameters) and data members are all lowercase, with underscores between words. Data members of classes (but not structs) additionally have trailing underscores. For instance: `a_local_variable`, `a_struct_data_member`, `a_class_data_member_`.

</div>

<div class="stylebody">

#### Common Variable names

For example:

<pre>string table_name;  // OK - uses underscore.
string tablename;   // OK - all lowercase.
</pre>

<pre class="badcode">string tableName;   // Bad - mixed case.
</pre>

#### Class Data Members

Data members of classes, both static and non-static, are named like ordinary nonmember variables, but with a trailing underscore.

<pre>class TableInfo {
  ...
 private:
  string table_name_;  // OK - underscore at end.
  string tablename_;   // OK.
  static Pool<TableInfo>* pool_;  // OK.
};
</pre>

#### Struct Data Members

Data members of structs, both static and non-static, are named like ordinary nonmember variables. They do not have the trailing underscores that data members in classes have.

<pre>struct UrlTableProperties {
  string name;
  int num_entries;
  static Pool<UrlTableProperties>* pool;
};
</pre>

See [Structs vs. Classes](#Structs_vs._Classes) for a discussion of when to use a struct versus a class.

</div>

### Constant Names

<div class="summary">

Variables declared constexpr or const, and whose value is fixed for the duration of the program, are named with a leading "k" followed by mixed case. For example:

</div>

<pre>const int kDaysInAWeek = 7;
</pre>

<div class="stylebody">

All such variables with static storage duration (i.e. statics and globals, see [Storage Duration](http://en.cppreference.com/w/cpp/language/storage_duration#Storage_duration) for details) should be named this way. This convention is optional for variables of other storage classes, e.g. automatic variables, otherwise the usual variable naming rules apply.

</div>

### Function Names

<div class="summary">

Regular functions have mixed case; accessors and mutators may be named like variables.

</div>

<div class="stylebody">

Ordinarily, functions should start with a capital letter and have a capital letter for each new word (a.k.a. "[Camel Case](https://en.wikipedia.org/wiki/Camel_case)" or "Pascal case"). Such names should not have underscores. Prefer to capitalize acronyms as single words (i.e. `StartRpc()`, not `StartRPC()`).

<pre>AddTableEntry()
DeleteUrl()
OpenFileOrDie()
</pre>

(The same naming rule applies to class- and namespace-scope constants that are exposed as part of an API and that are intended to look like functions, because the fact that they're objects rather than functions is an unimportant implementation detail.)

Accessors and mutators (get and set functions) may be named like variables. These often correspond to actual member variables, but this is not required. For example, `int count()` and `void set_count(int count)`.

</div>

### Namespace Names

<div class="summary">Namespace names are all lower-case. Top-level namespace names are based on the project name . Avoid collisions between nested namespaces and well-known top-level namespaces.</div>

<div class="stylebody">

The name of a top-level namespace should usually be the name of the project or team whose code is contained in that namespace. The code in that namespace should usually be in a directory whose basename matches the namespace name (or subdirectories thereof).

Keep in mind that the [rule against abbreviated names](#General_Naming_Rules) applies to namespaces just as much as variable names. Code inside the namespace seldom needs to mention the namespace name, so there's usually no particular need for abbreviation anyway.

Avoid nested namespaces that match well-known top-level namespaces. Collisions between namespace names can lead to surprising build breaks because of name lookup rules. In particular, do not create any nested `std` namespaces. Prefer unique project identifiers (`websearch::index`, `websearch::index_util`) over collision-prone names like `websearch::util`.

For `internal` namespaces, be wary of other code being added to the same `internal` namespace causing a collision (internal helpers within a team tend to be related and may lead to collisions). In such a situation, using the filename to make a unique internal name is helpful (`websearch::index::frobber_internal` for use in `frobber.h`)

</div>

### Enumerator Names

<div class="summary">

Enumerators (for both scoped and unscoped enums) should be named _either_ like [constants](#Constant_Names) or like [macros](#Macro_Names): either `kEnumName` or `ENUM_NAME`.

</div>

<div class="stylebody">

Preferably, the individual enumerators should be named like [constants](#Constant_Names). However, it is also acceptable to name them like [macros](#Macro_Names). The enumeration name, `UrlTableErrors` (and `AlternateUrlTableErrors`), is a type, and therefore mixed case.

<pre>enum UrlTableErrors {
  kOK = 0,
  kErrorOutOfMemory,
  kErrorMalformedInput,
};
enum AlternateUrlTableErrors {
  OK = 0,
  OUT_OF_MEMORY = 1,
  MALFORMED_INPUT = 2,
};
</pre>

Until January 2009, the style was to name enum values like [macros](#Macro_Names). This caused problems with name collisions between enum values and macros. Hence, the change to prefer constant-style naming was put in place. New code should prefer constant-style naming if possible. However, there is no reason to change old code to use constant-style names, unless the old names are actually causing a compile-time problem.

</div>

### Macro Names

<div class="summary">

You're not really going to [define a macro](#Preprocessor_Macros), are you? If you do, they're like this: `MY_MACRO_THAT_SCARES_SMALL_CHILDREN`.

</div>

<div class="stylebody">

Please see the [description of macros](#Preprocessor_Macros); in general macros should _not_ be used. However, if they are absolutely needed, then they should be named with all capitals and underscores.

<pre>#define ROUND(x) ...
#define PI_ROUNDED 3.0
</pre>

</div>

### Exceptions to Naming Rules

<div class="summary">

If you are naming something that is analogous to an existing C or C++ entity then you can follow the existing naming convention scheme.

</div>

<div class="stylebody">

<dl>

<dt>`bigopen()`</dt>

<dd>function name, follows form of `open()`</dd>

<dt>`uint`</dt>

<dd>`typedef`</dd>

<dt>`bigpos`</dt>

<dd>`struct` or `class`, follows form of `pos`</dd>

<dt>`sparse_hash_map`</dt>

<dd>STL-like entity; follows STL naming conventions</dd>

<dt>`LONGLONG_MAX`</dt>

<dd>a constant, as in `INT_MAX`</dd>

</dl>

</div>

## Comments

Though a pain to write, comments are absolutely vital to keeping our code readable. The following rules describe what you should comment and where. But remember: while comments are very important, the best code is self-documenting. Giving sensible names to types and variables is much better than using obscure names that you must then explain through comments.

When writing your comments, write for your audience: the next contributor who will need to understand your code. Be generous — the next one may be you!

### Comment Style

<div class="summary">

Use either the `//` or `/* */` syntax, as long as you are consistent.

</div>

<div class="stylebody">

You can use either the `//` or the `/* */` syntax; however, `//` is _much_ more common. Be consistent with how you comment and what style you use where.

</div>

### File Comments

<div class="summary">

Start each file with license boilerplate.

File comments describe the contents of a file. If a file declares, implements, or tests exactly one abstraction that is documented by a comment at the point of declaration, file comments are not required. All other files must have file comments.

</div>

<div class="stylebody">

#### Legal Notice and Author Line

Every file should contain license boilerplate. Choose the appropriate boilerplate for the license used by the project (for example, Apache 2.0, BSD, LGPL, GPL).

If you make significant changes to a file with an author line, consider deleting the author line.

#### File Contents

If a `.h` declares multiple abstractions, the file-level comment should broadly describe the contents of the file, and how the abstractions are related. A 1 or 2 sentence file-level comment may be sufficient. The detailed documentation about individual abstractions belongs with those abstractions, not at the file level.

Do not duplicate comments in both the `.h` and the `.cc`. Duplicated comments diverge.

</div>

### Class Comments

<div class="summary">

Every non-obvious class declaration should have an accompanying comment that describes what it is for and how it should be used.

</div>

<div class="stylebody">

<pre>// Iterates over the contents of a GargantuanTable.
// Example:
//    GargantuanTableIterator* iter = table->NewIterator();
//    for (iter->Seek("foo"); !iter->done(); iter->Next()) {
//      process(iter->key(), iter->value());
//    }
//    delete iter;
class GargantuanTableIterator {
  ...
};
</pre>

The class comment should provide the reader with enough information to know how and when to use the class, as well as any additional considerations necessary to correctly use the class. Document the synchronization assumptions the class makes, if any. If an instance of the class can be accessed by multiple threads, take extra care to document the rules and invariants surrounding multithreaded use.

The class comment is often a good place for a small example code snippet demonstrating a simple and focused usage of the class.

When sufficiently separated (e.g. `.h` and `.cc` files), comments describing the use of the class should go together with its interface definition; comments about the class operation and implementation should accompany the implementation of the class's methods.

</div>

### Function Comments

<div class="summary">

Declaration comments describe use of the function (when it is non-obvious); comments at the definition of a function describe operation.

</div>

<div class="stylebody">

#### Function Declarations

Almost every function declaration should have comments immediately preceding it that describe what the function does and how to use it. These comments may be omitted only if the function is simple and obvious (e.g. simple accessors for obvious properties of the class). These comments should be descriptive ("Opens the file") rather than imperative ("Open the file"); the comment describes the function, it does not tell the function what to do. In general, these comments do not describe how the function performs its task. Instead, that should be left to comments in the function definition.

Types of things to mention in comments at the function declaration:

*   What the inputs and outputs are.
*   For class member functions: whether the object remembers reference arguments beyond the duration of the method call, and whether it will free them or not.
*   If the function allocates memory that the caller must free.
*   Whether any of the arguments can be a null pointer.
*   If there are any performance implications of how a function is used.
*   If the function is re-entrant. What are its synchronization assumptions?

Here is an example:

<pre>// Returns an iterator for this table.  It is the client's
// responsibility to delete the iterator when it is done with it,
// and it must not use the iterator once the GargantuanTable object
// on which the iterator was created has been deleted.
//
// The iterator is initially positioned at the beginning of the table.
//
// This method is equivalent to:
//    Iterator* iter = table->NewIterator();
//    iter->Seek("");
//    return iter;
// If you are going to immediately seek to another place in the
// returned iterator, it will be faster to use NewIterator()
// and avoid the extra seek.
Iterator* GetIterator() const;
</pre>

However, do not be unnecessarily verbose or state the completely obvious. Notice below that it is not necessary to say "returns false otherwise" because this is implied.

<pre>// Returns true if the table cannot hold any more entries.
bool IsTableFull();
</pre>

When documenting function overrides, focus on the specifics of the override itself, rather than repeating the comment from the overridden function. In many of these cases, the override needs no additional documentation and thus no comment is required.

When commenting constructors and destructors, remember that the person reading your code knows what constructors and destructors are for, so comments that just say something like "destroys this object" are not useful. Document what constructors do with their arguments (for example, if they take ownership of pointers), and what cleanup the destructor does. If this is trivial, just skip the comment. It is quite common for destructors not to have a header comment.

#### Function Definitions

If there is anything tricky about how a function does its job, the function definition should have an explanatory comment. For example, in the definition comment you might describe any coding tricks you use, give an overview of the steps you go through, or explain why you chose to implement the function in the way you did rather than using a viable alternative. For instance, you might mention why it must acquire a lock for the first half of the function but why it is not needed for the second half.

Note you should _not_ just repeat the comments given with the function declaration, in the `.h` file or wherever. It's okay to recapitulate briefly what the function does, but the focus of the comments should be on how it does it.

</div>

### Variable Comments

<div class="summary">

In general the actual name of the variable should be descriptive enough to give a good idea of what the variable is used for. In certain cases, more comments are required.

</div>

<div class="stylebody">

#### Class Data Members

The purpose of each class data member (also called an instance variable or member variable) must be clear. If there are any invariants (special values, relationships between members, lifetime requirements) not clearly expressed by the type and name, they must be commented. However, if the type and name suffice (`int num_events_;`), no comment is needed.

In particular, add comments to describe the existence and meaning of sentinel values, such as nullptr or -1, when they are not obvious. For example:

<pre>private:
 // Used to bounds-check table accesses. -1 means
 // that we don't yet know how many entries the table has.
 int num_total_entries_;
</pre>

#### Global Variables

All global variables should have a comment describing what they are, what they are used for, and (if unclear) why it needs to be global. For example:

<pre>// The total number of tests cases that we run through in this regression test.
const int kNumTestCases = 6;
</pre>

</div>

### Implementation Comments

<div class="summary">

In your implementation you should have comments in tricky, non-obvious, interesting, or important parts of your code.

</div>

<div class="stylebody">

#### Explanatory Comments

Tricky or complicated code blocks should have comments before them. Example:

<pre>// Divide result by two, taking into account that x
// contains the carry from the add.
for (int i = 0; i < result->size(); i++) {
  x = (x << 8) + (*result)[i];
  (*result)[i] = x >> 1;
  x &= 1;
}
</pre>

#### Line Comments

Also, lines that are non-obvious should get a comment at the end of the line. These end-of-line comments should be separated from the code by 2 spaces. Example:

<pre>// If we have enough memory, mmap the data portion too.
mmap_budget = max<int64>(0, mmap_budget - index_->length());
if (mmap_budget >= data_size_ && !MmapData(mmap_chunk_bytes, mlock))
  return;  // Error already logged.
</pre>

Note that there are both comments that describe what the code is doing, and comments that mention that an error has already been logged when the function returns.

If you have several comments on subsequent lines, it can often be more readable to line them up:

<pre>DoSomething();                  // Comment here so the comments line up.
DoSomethingElseThatIsLonger();  // Two spaces between the code and the comment.
{ // One space before comment when opening a new scope is allowed,
  // thus the comment lines up with the following comments and code.
  DoSomethingElse();  // Two spaces before line comments normally.
}
std::vector<string> list{
                    // Comments in braced lists describe the next element...
                    "First item",
                    // .. and should be aligned appropriately.
                    "Second item"};
DoSomething(); /* For trailing block comments, one space is fine. */
</pre>

#### Function Argument Comments

When the meaning of a function argument is nonobvious, consider one of the following remedies:

*   If the argument is a literal constant, and the same constant is used in multiple function calls in a way that tacitly assumes they're the same, you should use a named constant to make that constraint explicit, and to guarantee that it holds.
*   Consider changing the function signature to replace a `bool` argument with an `enum` argument. This will make the argument values self-describing.
*   For functions that have several configuration options, consider defining a single class or struct to hold all the options , and pass an instance of that. This approach has several advantages. Options are referenced by name at the call site, which clarifies their meaning. It also reduces function argument count, which makes function calls easier to read and write. As an added benefit, you don't have to change call sites when you add another option.
*   Replace large or complex nested expressions with named variables.
*   As a last resort, use comments to clarify argument meanings at the call site.

Consider the following example:

<pre class="badcode">// What are these arguments?
const DecimalNumber product = CalculateProduct(values, 7, false, nullptr);
</pre>

versus:

<pre>ProductOptions options;
options.set_precision_decimals(7);
options.set_use_cache(ProductOptions::kDontUseCache);
const DecimalNumber product =
    CalculateProduct(values, options, /*completion_callback=*/nullptr);
</pre>

#### Don'ts

Do not state the obvious. In particular, don't literally describe what code does, unless the behavior is nonobvious to a reader who understands C++ well. Instead, provide higher level comments that describe _why_ the code does what it does, or make the code self describing.

Compare this:

<pre class="badcode">// Find the element in the vector.  <-- Bad: obvious!
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
</pre>

To this:

<pre>// Process "element" unless it was already processed.
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
</pre>

Self-describing code doesn't need a comment. The comment from the example above would be obvious:

<pre>if (!IsAlreadyProcessed(element)) {
  Process(element);
}
</pre>

</div>

### Punctuation, Spelling and Grammar

<div class="summary">

Pay attention to punctuation, spelling, and grammar; it is easier to read well-written comments than badly written ones.

</div>

<div class="stylebody">

Comments should be as readable as narrative text, with proper capitalization and punctuation. In many cases, complete sentences are more readable than sentence fragments. Shorter comments, such as comments at the end of a line of code, can sometimes be less formal, but you should be consistent with your style.

Although it can be frustrating to have a code reviewer point out that you are using a comma when you should be using a semicolon, it is very important that source code maintain a high level of clarity and readability. Proper punctuation, spelling, and grammar help with that goal.

</div>

### TODO Comments

<div class="summary">

Use `TODO` comments for code that is temporary, a short-term solution, or good-enough but not perfect.

</div>

<div class="stylebody">

`TODO`s should include the string `TODO` in all caps, followed by the name, e-mail address, bug ID, or other identifier of the person or issue with the best context about the problem referenced by the `TODO`. The main purpose is to have a consistent `TODO` that can be searched to find out how to get more details upon request. A `TODO` is not a commitment that the person referenced will fix the problem. Thus when you create a `TODO` with a name, it is almost always your name that is given.

<div>

<pre>// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke) change this to use relations.
// TODO(bug 12345): remove the "Last visitors" feature
</pre>

</div>

If your `TODO` is of the form "At a future date do something" make sure that you either include a very specific date ("Fix by November 2005") or a very specific event ("Remove this code when all clients can handle XML responses.").

</div>

### Deprecation Comments

<div class="summary">

Mark deprecated interface points with `DEPRECATED` comments.

</div>

<div class="stylebody">

You can mark an interface as deprecated by writing a comment containing the word `DEPRECATED` in all caps. The comment goes either before the declaration of the interface or on the same line as the declaration.

After the word `DEPRECATED`, write your name, e-mail address, or other identifier in parentheses.

A deprecation comment must include simple, clear directions for people to fix their callsites. In C++, you can implement a deprecated function as an inline function that calls the new interface point.

Marking an interface point `DEPRECATED` will not magically cause any callsites to change. If you want people to actually stop using the deprecated facility, you will have to fix the callsites yourself or recruit a crew to help you.

New code should not contain calls to deprecated interface points. Use the new interface point instead. If you cannot understand the directions, find the person who created the deprecation and ask them for help using the new interface point.

</div>

## Formatting

Coding style and formatting are pretty arbitrary, but a project is much easier to follow if everyone uses the same style. Individuals may not agree with every aspect of the formatting rules, and some of the rules may take some getting used to, but it is important that all project contributors follow the style rules so that they can all read and understand everyone's code easily.

To help you format code correctly, we've created a [settings file for emacs](https://raw.githubusercontent.com/google/styleguide/gh-pages/google-c-style.el).

### Line Length

<div class="summary">

Each line of text in your code should be at most 80 characters long.

</div>

<div class="stylebody">

We recognize that this rule is controversial, but so much existing code already adheres to it, and we feel that consistency is important.

<div class="pros">

Those who favor this rule argue that it is rude to force them to resize their windows and there is no need for anything longer. Some folks are used to having several code windows side-by-side, and thus don't have room to widen their windows in any case. People set up their work environment assuming a particular maximum window width, and 80 columns has been the traditional standard. Why change it?

</div>

<div class="cons">

Proponents of change argue that a wider line can make code more readable. The 80-column limit is an hidebound throwback to 1960s mainframes; modern equipment has wide screens that can easily show longer lines.

</div>

<div class="decision">

80 characters is the maximum.

Comment lines can be longer than 80 characters if it is not feasible to split them without harming readability, ease of cut and paste or auto-linking -- e.g. if a line contains an example command or a literal URL longer than 80 characters.

A raw-string literal may have content that exceeds 80 characters. Except for test code, such literals should appear near the top of a file.

An `#include` statement with a long path may exceed 80 columns.

You needn't be concerned about [header guards](#The__define_Guard) that exceed the maximum length.

</div>

</div>

### Non-ASCII Characters

<div class="summary">

Non-ASCII characters should be rare, and must use UTF-8 formatting.

</div>

<div class="stylebody">

You shouldn't hard-code user-facing text in source, even English, so use of non-ASCII characters should be rare. However, in certain cases it is appropriate to include such words in your code. For example, if your code parses data files from foreign sources, it may be appropriate to hard-code the non-ASCII string(s) used in those data files as delimiters. More commonly, unittest code (which does not need to be localized) might contain non-ASCII strings. In such cases, you should use UTF-8, since that is an encoding understood by most tools able to handle more than just ASCII.

Hex encoding is also OK, and encouraged where it enhances readability — for example, `"\xEF\xBB\xBF"`, or, even more simply, `u8"\uFEFF"`, is the Unicode zero-width no-break space character, which would be invisible if included in the source as straight UTF-8.

Use the `u8` prefix to guarantee that a string literal containing `\uXXXX` escape sequences is encoded as UTF-8. Do not use it for strings containing non-ASCII characters encoded as UTF-8, because that will produce incorrect output if the compiler does not interpret the source file as UTF-8\.

You shouldn't use the C++11 `char16_t` and `char32_t` character types, since they're for non-UTF-8 text. For similar reasons you also shouldn't use `wchar_t` (unless you're writing code that interacts with the Windows API, which uses `wchar_t` extensively).

</div>

### Spaces vs. Tabs

<div class="summary">

Use only spaces, and indent 2 spaces at a time.

</div>

<div class="stylebody">

We use spaces for indentation. Do not use tabs in your code. You should set your editor to emit spaces when you hit the tab key.

</div>

### Function Declarations and Definitions

<div class="summary">

Return type on the same line as function name, parameters on the same line if they fit. Wrap parameter lists which do not fit on a single line as you would wrap arguments in a [function call](#Function_Calls).

</div>

<div class="stylebody">

Functions look like this:

<pre>ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
  DoSomething();
  ...
}
</pre>

If you have too much text to fit on one line:

<pre>ReturnType ClassName::ReallyLongFunctionName(Type par_name1, Type par_name2,
                                             Type par_name3) {
  DoSomething();
  ...
}
</pre>

or if you cannot fit even the first parameter:

<pre>ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
    Type par_name1,  // 4 space indent
    Type par_name2,
    Type par_name3) {
  DoSomething();  // 2 space indent
  ...
}
</pre>

Some points to note:

*   Choose good parameter names.
*   Parameter names may be omitted only if the parameter is unused and its purpose is obvious.
*   If you cannot fit the return type and the function name on a single line, break between them.
*   If you break after the return type of a function declaration or definition, do not indent.
*   The open parenthesis is always on the same line as the function name.
*   There is never a space between the function name and the open parenthesis.
*   There is never a space between the parentheses and the parameters.
*   The open curly brace is always on the end of the last line of the function declaration, not the start of the next line.
*   The close curly brace is either on the last line by itself or on the same line as the open curly brace.
*   There should be a space between the close parenthesis and the open curly brace.
*   All parameters should be aligned if possible.
*   Default indentation is 2 spaces.
*   Wrapped parameters have a 4 space indent.

Unused parameters that are obvious from context may be omitted:

<pre>class Foo {
 public:
  Foo(Foo&&);
  Foo(const Foo&);
  Foo& operator=(Foo&&);
  Foo& operator=(const Foo&);
};
</pre>

Unused parameters that might not be obvious should comment out the variable name in the function definition:

<pre>class Shape {
 public:
  virtual void Rotate(double radians) = 0;
};

class Circle : public Shape {
 public:
  void Rotate(double radians) override;
};

void Circle::Rotate(double /*radians*/) {}
</pre>

<pre class="badcode">// Bad - if someone wants to implement later, it's not clear what the
// variable means.
void Circle::Rotate(double) {}
</pre>

Attributes, and macros that expand to attributes, appear at the very beginning of the function declaration or definition, before the return type:

<pre>MUST_USE_RESULT bool IsOK();
</pre>

</div>

### Lambda Expressions

<div class="summary">

Format parameters and bodies as for any other function, and capture lists like other comma-separated lists.

</div>

<div class="stylebody">

For by-reference captures, do not leave a space between the ampersand (&) and the variable name.

<pre>int x = 0;
auto x_plus_n = [&x](int n) -> int { return x + n; }
</pre>

Short lambdas may be written inline as function arguments.

<pre>std::set<int> blacklist = {7, 8, 9};
std::vector<int> digits = {3, 9, 1, 8, 4, 7, 1};
digits.erase(std::remove_if(digits.begin(), digits.end(), [&blacklist](int i) {
               return blacklist.find(i) != blacklist.end();
             }),
             digits.end());
</pre>

</div>

### Function Calls

<div class="summary">

Either write the call all on a single line, wrap the arguments at the parenthesis, or start the arguments on a new line indented by four spaces and continue at that 4 space indent. In the absence of other considerations, use the minimum number of lines, including placing multiple arguments on each line where appropriate.

</div>

<div class="stylebody">

Function calls have the following format:

<pre>bool result = DoSomething(argument1, argument2, argument3);
</pre>

If the arguments do not all fit on one line, they should be broken up onto multiple lines, with each subsequent line aligned with the first argument. Do not add spaces after the open paren or before the close paren:

<pre>bool result = DoSomething(averyveryveryverylongargument1,
                          argument2, argument3);
</pre>

Arguments may optionally all be placed on subsequent lines with a four space indent:

<pre>if (...) {
  ...
  ...
  if (...) {
    bool result = DoSomething(
        argument1, argument2,  // 4 space indent
        argument3, argument4);
    ...
  }
</pre>

Put multiple arguments on a single line to reduce the number of lines necessary for calling a function unless there is a specific readability problem. Some find that formatting with strictly one argument on each line is more readable and simplifies editing of the arguments. However, we prioritize for the reader over the ease of editing arguments, and most readability problems are better addressed with the following techniques.

If having multiple arguments in a single line decreases readability due to the complexity or confusing nature of the expressions that make up some arguments, try creating variables that capture those arguments in a descriptive name:

<pre>int my_heuristic = scores[x] * y + bases[x];
bool result = DoSomething(my_heuristic, x, y, z);
</pre>

Or put the confusing argument on its own line with an explanatory comment:

<pre>bool result = DoSomething(scores[x] * y + bases[x],  // Score heuristic.
                          x, y, z);
</pre>

If there is still a case where one argument is significantly more readable on its own line, then put it on its own line. The decision should be specific to the argument which is made more readable rather than a general policy.

Sometimes arguments form a structure that is important for readability. In those cases, feel free to format the arguments according to that structure:

<pre>// Transform the widget by a 3x3 matrix.
my_widget.Transform(x1, x2, x3,
                    y1, y2, y3,
                    z1, z2, z3);
</pre>

</div>

### Braced Initializer List Format

<div class="summary">

Format a [braced initializer list](#Braced_Initializer_List) exactly like you would format a function call in its place.

</div>

<div class="stylebody">

If the braced list follows a name (e.g. a type or variable name), format as if the `{}` were the parentheses of a function call with that name. If there is no name, assume a zero-length name.

<pre>// Examples of braced init list on a single line.
return {foo, bar};
functioncall({foo, bar});
std::pair<int, int> p{foo, bar};

// When you have to wrap.
SomeFunction(
    {"assume a zero-length name before {"},
    some_other_function_parameter);
SomeType variable{
    some, other, values,
    {"assume a zero-length name before {"},
    SomeOtherType{
        "Very long string requiring the surrounding breaks.",
        some, other values},
    SomeOtherType{"Slightly shorter string",
                  some, other, values}};
SomeType variable{
    "This is too long to fit all in one line"};
MyType m = {  // Here, you could also break before {.
    superlongvariablename1,
    superlongvariablename2,
    {short, interior, list},
    {interiorwrappinglist,
     interiorwrappinglist2}};
</pre>

</div>

### Conditionals

<div class="summary">

Prefer no spaces inside parentheses. The `if` and `else` keywords belong on separate lines.

</div>

<div class="stylebody">

There are two acceptable formats for a basic conditional statement. One includes spaces between the parentheses and the condition, and one does not.

The most common form is without spaces. Either is fine, but _be consistent_. If you are modifying a file, use the format that is already present. If you are writing new code, use the format that the other files in that directory or project use. If in doubt and you have no personal preference, do not add the spaces.

<pre>if (condition) {  // no spaces inside parentheses
  ...  // 2 space indent.
} else if (...) {  // The else goes on the same line as the closing brace.
  ...
} else {
  ...
}
</pre>

If you prefer you may add spaces inside the parentheses:

<pre>if ( condition ) {  // spaces inside parentheses - rare
  ...  // 2 space indent.
} else {  // The else goes on the same line as the closing brace.
  ...
}
</pre>

Note that in all cases you must have a space between the `if` and the open parenthesis. You must also have a space between the close parenthesis and the curly brace, if you're using one.

<pre class="badcode">if(condition) {   // Bad - space missing after IF.
if (condition){   // Bad - space missing before {.
if(condition){    // Doubly bad.
</pre>

<pre>if (condition) {  // Good - proper space after IF and before {.
</pre>

Short conditional statements may be written on one line if this enhances readability. You may use this only when the line is brief and the statement does not use the `else` clause.

<pre>if (x == kFoo) return new Foo();
if (x == kBar) return new Bar();
</pre>

This is not allowed when the if statement has an `else`:

<pre class="badcode">// Not allowed - IF statement on one line when there is an ELSE clause
if (x) DoThis();
else DoThat();
</pre>

In general, curly braces are not required for single-line statements, but they are allowed if you like them; conditional or loop statements with complex conditions or statements may be more readable with curly braces. Some projects require that an `if` must always always have an accompanying brace.

<pre>if (condition)
  DoSomething();  // 2 space indent.

if (condition) {
  DoSomething();  // 2 space indent.
}
</pre>

However, if one part of an `if`-`else` statement uses curly braces, the other part must too:

<pre class="badcode">// Not allowed - curly on IF but not ELSE
if (condition) {
  foo;
} else
  bar;

// Not allowed - curly on ELSE but not IF
if (condition)
  foo;
else {
  bar;
}
</pre>

<pre>// Curly braces around both IF and ELSE required because
// one of the clauses used braces.
if (condition) {
  foo;
} else {
  bar;
}
</pre>

</div>

### Loops and Switch Statements

<div class="summary">

Switch statements may use braces for blocks. Annotate non-trivial fall-through between cases. Braces are optional for single-statement loops. Empty loop bodies should use empty braces or `continue`.

</div>

<div class="stylebody">

`case` blocks in `switch` statements can have curly braces or not, depending on your preference. If you do include curly braces they should be placed as shown below.

If not conditional on an enumerated value, switch statements should always have a `default` case (in the case of an enumerated value, the compiler will warn you if any values are not handled). If the default case should never execute, simply `assert`:

<div>

<pre>switch (var) {
  case 0: {  // 2 space indent
    ...      // 4 space indent
    break;
  }
  case 1: {
    ...
    break;
  }
  default: {
    assert(false);
  }
}
</pre>

</div>

Braces are optional for single-statement loops.

<pre>for (int i = 0; i < kSomeNumber; ++i)
  printf("I love you\n");

for (int i = 0; i < kSomeNumber; ++i) {
  printf("I take it back\n");
}
</pre>

Empty loop bodies should use an empty pair of braces or `continue`, but not a single semicolon.

<pre>while (condition) {
  // Repeat test until it returns false.
}
for (int i = 0; i < kSomeNumber; ++i) {}  // Good - one newline is also OK.
while (condition) continue;  // Good - continue indicates no logic.
</pre>

<pre class="badcode">while (condition);  // Bad - looks like part of do/while loop.
</pre>

</div>

### Pointer and Reference Expressions

<div class="summary">

No spaces around period or arrow. Pointer operators do not have trailing spaces.

</div>

<div class="stylebody">

The following are examples of correctly-formatted pointer and reference expressions:

<pre>x = *p;
p = &x;
x = r.y;
x = r->y;
</pre>

Note that:

*   There are no spaces around the period or arrow when accessing a member.
*   Pointer operators have no space after the `*` or `&`.

When declaring a pointer variable or argument, you may place the asterisk adjacent to either the type or to the variable name:

<pre>// These are fine, space preceding.
char *c;
const string &str;

// These are fine, space following.
char* c;
const string& str;
</pre>

It is allowed (if unusual) to declare multiple variables in the same declaration, but it is disallowed if any of those have pointer or reference decorations. Such declarations are easily misread.

<pre>// Fine if helpful for readability.
int x, y;
</pre>

<pre class="badcode">int x, *y;  // Disallowed - no & or * in multiple declaration
char * c;  // Bad - spaces on both sides of *
const string & str;  // Bad - spaces on both sides of &
</pre>

You should do this consistently within a single file, so, when modifying an existing file, use the style in that file.

</div>

### Boolean Expressions

<div class="summary">

When you have a boolean expression that is longer than the [standard line length](#Line_Length), be consistent in how you break up the lines.

</div>

<div class="stylebody">

In this example, the logical AND operator is always at the end of the lines:

<pre>if (this_one_thing > this_other_thing &&
    a_third_thing == a_fourth_thing &&
    yet_another && last_one) {
  ...
}
</pre>

Note that when the code wraps in this example, both of the `&&` logical AND operators are at the end of the line. This is more common in Google code, though wrapping all operators at the beginning of the line is also allowed. Feel free to insert extra parentheses judiciously because they can be very helpful in increasing readability when used appropriately. Also note that you should always use the punctuation operators, such as `&&` and `~`, rather than the word operators, such as `and` and `compl`.

</div>

### Return Values

<div class="summary">

Do not needlessly surround the `return` expression with parentheses.

</div>

<div class="stylebody">

Use parentheses in `return expr;` only where you would use them in `x = expr;`.

<pre>return result;                  // No parentheses in the simple case.
// Parentheses OK to make a complex expression more readable.
return (some_long_condition &&
        another_condition);
</pre>

<pre class="badcode">return (value);                // You wouldn't write var = (value);
return(result);                // return is not a function!
</pre>

</div>

### Variable and Array Initialization

<div class="summary">

Your choice of `=`, `()`, or `{}`.

</div>

<div class="stylebody">

You may choose between `=`, `()`, and `{}`; the following are all correct:

<pre>int x = 3;
int x(3);
int x{3};
string name = "Some Name";
string name("Some Name");
string name{"Some Name"};
</pre>

Be careful when using a braced initialization list `{...}` on a type with an `std::initializer_list` constructor. A nonempty _braced-init-list_ prefers the `std::initializer_list` constructor whenever possible. Note that empty braces `{}` are special, and will call a default constructor if available. To force the non-`std::initializer_list` constructor, use parentheses instead of braces.

<pre>std::vector<int> v(100, 1);  // A vector of 100 1s.
std::vector<int> v{100, 1};  // A vector of 100, 1.
</pre>

Also, the brace form prevents narrowing of integral types. This can prevent some types of programming errors.

<pre>int pi(3.14);  // OK -- pi == 3.
int pi{3.14};  // Compile error: narrowing conversion.
</pre>

</div>

### Preprocessor Directives

<div class="summary">

The hash mark that starts a preprocessor directive should always be at the beginning of the line.

</div>

<div class="stylebody">

Even when preprocessor directives are within the body of indented code, the directives should start at the beginning of the line.

<pre>// Good - directives at beginning of line
  if (lopsided_score) {
#if DISASTER_PENDING      // Correct -- Starts at beginning of line
    DropEverything();
# if NOTIFY               // OK but not required -- Spaces after #
    NotifyClient();
# endif
#endif
    BackToNormal();
  }
</pre>

<pre class="badcode">// Bad - indented directives
  if (lopsided_score) {
    #if DISASTER_PENDING  // Wrong!  The "#if" should be at beginning of line
    DropEverything();
    #endif                // Wrong!  Do not indent "#endif"
    BackToNormal();
  }
</pre>

</div>

### Class Format

<div class="summary">

Sections in `public`, `protected` and `private` order, each indented one space.

</div>

<div class="stylebody">

The basic format for a class definition (lacking the comments, see [Class Comments](#Class_Comments) for a discussion of what comments are needed) is:

<pre>class MyClass : public OtherClass {
 public:      // Note the 1 space indent!
  MyClass();  // Regular 2 space indent.
  explicit MyClass(int var);
  ~MyClass() {}

  void SomeFunction();
  void SomeFunctionThatDoesNothing() {
  }

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

 private:
  bool SomeInternalFunction();

  int some_var_;
  int some_other_var_;
};
</pre>

Things to note:

*   Any base class name should be on the same line as the subclass name, subject to the 80-column limit.
*   The `public:`, `protected:`, and `private:` keywords should be indented one space.
*   Except for the first instance, these keywords should be preceded by a blank line. This rule is optional in small classes.
*   Do not leave a blank line after these keywords.
*   The `public` section should be first, followed by the `protected` and finally the `private` section.
*   See [Declaration Order](#Declaration_Order) for rules on ordering declarations within each of these sections.

</div>

### Constructor Initializer Lists

<div class="summary">

Constructor initializer lists can be all on one line or with subsequent lines indented four spaces.

</div>

<div class="stylebody">

The acceptable formats for initializer lists are:

<pre>// When everything fits on one line:
MyClass::MyClass(int var) : some_var_(var) {
  DoSomething();
}

// If the signature and initializer list are not all on one line,
// you must wrap before the colon and indent 4 spaces:
MyClass::MyClass(int var)
    : some_var_(var), some_other_var_(var + 1) {
  DoSomet
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
<title>Google C++ Style Guide</title>
<link rel="stylesheet" type="text/css" href="include/styleguide.css">
<script language="javascript" src="include/styleguide.js"></script>
<link rel="shortcut icon" type="image/x-icon" href="https://www.google.com/favicon.ico" />
</head>
<body onload="initStyleGuide();">
<div id="content">
<h1>Google C++ Style Guide</h1>
<div class="horizontal_toc" id="tocDiv"></div>

<div class="main_body">

<h2 class="ignoreLink" id="Background">Background</h2>

<p>C++ is one of the main development languages  used by
many of Google's open-source projects. As every C++
programmer knows, the language has many powerful features, but
this power brings with it complexity, which in turn can make
code more bug-prone and harder to read and maintain.</p>

<p>The goal of this guide is to manage this complexity by
describing in detail the dos and don'ts of writing C++ code.
These rules exist to
keep  the code base manageable while still allowing
coders to use C++ language features productively.</p>

<p><em>Style</em>, also known as readability, is what we call
the conventions that govern our C++ code. The term Style is a
bit of a misnomer, since these conventions cover far more than
just source file formatting.</p>

<p>
Most open-source projects developed by
Google conform to the requirements in this guide.
</p>





<p>Note that this guide is not a C++ tutorial: we assume that
the reader is familiar with the language. </p>

<h3 id="Goals">Goals of the Style Guide</h3>
<div class="stylebody">
<p>Why do we have this document?</p>

<p>There are a few core goals that we believe this guide should
serve. These are the fundamental <b>why</b>s that
underlie all of the individual rules. By bringing these ideas to
the fore, we hope to ground discussions and make it clearer to our
broader community why the rules are in place and why particular
decisions have been made. If you understand what goals each rule is
serving, it should be clearer to everyone when a rule may be waived
(some can be), and what sort of argument or alternative would be
necessary to change a rule in the guide.</p>

<p>The goals of the style guide as we currently see them are as follows:</p>
<dl>
<dt>Style rules should pull their weight</dt>
<dd>The benefit of a style rule
must be large enough to justify asking all of our engineers to
remember it. The benefit is measured relative to the codebase we would
get without the rule, so a rule against a very harmful practice may
still have a small benefit if people are unlikely to do it
anyway. This principle mostly explains the rules we don&#8217;t have, rather
than the rules we do: for example, <code>goto</code> contravenes many
of the following principles, but is already vanishingly rare, so the Style
Guide doesn&#8217;t discuss it.</dd>

<dt>Optimize for the reader, not the writer</dt>
<dd>Our codebase (and most individual components submitted to it) is
expected to continue for quite some time. As a result, more time will
be spent reading most of our code than writing it. We explicitly
choose to optimize for the experience of our average software engineer
reading, maintaining, and debugging code in our codebase rather than
ease when writing said code.  "Leave a trace for the reader" is a
particularly common sub-point of this principle: When something
surprising or unusual is happening in a snippet of code (for example,
transfer of pointer ownership), leaving textual hints for the reader
at the point of use is valuable (<code>std::unique_ptr</code>
demonstrates the ownership transfer unambiguously at the call
site). </dd>

<dt>Be consistent with existing code</dt>
<dd>Using one style consistently through our codebase lets us focus on
other (more important) issues. Consistency also allows for
automation: tools that format your code or adjust
your <code>#include</code>s only work properly when your code is
consistent with the expectations of the tooling. In many cases, rules
that are attributed to "Be Consistent" boil down to "Just pick one and
stop worrying about it"; the potential value of allowing flexibility
on these points is outweighed by the cost of having people argue over
them. </dd>

<dt>Be consistent with the broader C++ community when appropriate</dt>
<dd>Consistency with the way other organizations use C++ has value for
the same reasons as consistency within our code base. If a feature in
the C++ standard solves a problem, or if some idiom is widely known
and accepted, that's an argument for using it. However, sometimes
standard features and idioms are flawed, or were just designed without
our codebase's needs in mind. In those cases (as described below) it's
appropriate to constrain or ban standard features.  In some cases we
prefer a homegrown or third-party library over a library defined in
the C++ Standard, either out of perceived superiority or insufficient
value to transition the codebase to the standard interface.</dd>

<dt>Avoid surprising or dangerous constructs</dt>
<dd>C++ has features that are more surprising or dangerous than one
might think at a glance. Some style guide restrictions are in place to
prevent falling into these pitfalls. There is a high bar for style
guide waivers on such restrictions, because waiving such rules often
directly risks compromising program correctness.
</dd>

<dt>Avoid constructs that our average C++ programmer would find tricky
or hard to maintain</dt>
<dd>C++ has features that may not be generally appropriate because of
the complexity they introduce to the code. In widely used
code, it may be more acceptable to use
trickier language constructs, because any benefits of more complex
implementation are multiplied widely by usage, and the cost in understanding
the complexity does not need to be paid again when working with new
portions of the codebase. When in doubt, waivers to rules of this type
can be sought by asking 
your project leads. This is specifically
important for our codebase because code ownership and team membership
changes over time: even if everyone that works with some piece of code
currently understands it, such understanding is not guaranteed to hold a
few years from now.</dd>

<dt>Be mindful of our scale</dt>
<dd>With a codebase of 100+ million lines and thousands of engineers,
some mistakes and simplifications for one engineer can become costly
for many. For instance it's particularly important to
avoid polluting the global namespace: name collisions across a
codebase of hundreds of millions of lines are difficult to work with
and hard to avoid if everyone puts things into the global
namespace.</dd>

<dt>Concede to optimization when necessary</dt>
<dd>Performance optimizations can sometimes be necessary and
appropriate, even when they conflict with the other principles of this
document.</dd>
</dl>

<p>The intent of this document is to provide maximal guidance with
reasonable restriction. As always, common sense and good taste should
prevail. By this we specifically refer to the established conventions
of the entire Google C++ community, not just your personal preferences
or those of your team. Be skeptical about and reluctant to use
clever or unusual constructs: the absence of a prohibition is not the
same as a license to proceed.  Use your judgment, and if you are
unsure, please don't hesitate to ask your project leads to get additional
input.</p>

</div>

 

<h2 id="Header_Files">Header Files</h2>

<p>In general, every <code>.cc</code> file should have an
associated <code>.h</code> file. There are some common
exceptions, such as  unittests and
small <code>.cc</code> files containing just a
<code>main()</code> function.</p>

<p>Correct use of header files can make a huge difference to
the readability, size and performance of your code.</p>

<p>The following rules will guide you through the various
pitfalls of using header files.</p>

<a id="The_-inl.h_Files"></a>
<h3 id="Self_contained_Headers">Self-contained Headers</h3>

<div class="summary">
<p>Header files should be self-contained (compile on their own) and
end in <code>.h</code>.  Non-header files that are meant for inclusion
should end in <code>.inc</code> and be used sparingly.</p>
</div> 

<div class="stylebody">
<p>All header files should be self-contained. Users and refactoring
tools should not have to adhere to special conditions to include the
header. Specifically, a header should
have <a href="#The__define_Guard">header guards</a> and include all
other headers it needs.</p>

<p>Prefer placing the definitions for template and inline functions in
the same file as their declarations.  The definitions of these
constructs must be included into every <code>.cc</code> file that uses
them, or the program may fail to link in some build configurations.  If
declarations and definitions are in different files, including the
former should transitively include the latter.  Do not move these
definitions to separately included header files (<code>-inl.h</code>);
this practice was common in the past, but is no longer allowed.</p>

<p>As an exception, a template that is explicitly instantiated for
all relevant sets of template arguments, or that is a private
implementation detail of a class, is allowed to be defined in the one
and only <code>.cc</code> file that instantiates the template.</p>

<p>There are rare cases where a file designed to be included is not
self-contained.  These are typically intended to be included at unusual
locations, such as the middle of another file.  They might not
use <a href="#The__define_Guard">header guards</a>, and might not include
their prerequisites.  Name such files with the <code>.inc</code>
extension.  Use sparingly, and prefer self-contained headers when
possible.</p>

</div> 

<h3 id="The__define_Guard">The #define Guard</h3>

<div class="summary">
<p>All header files should have <code>#define</code> guards to
prevent multiple inclusion. The format of the symbol name
should be
<code><i>&lt;PROJECT&gt;</i>_<i>&lt;PATH&gt;</i>_<i>&lt;FILE&gt;</i>_H_</code>.</p>
</div> 

<div class="stylebody">



<p>To guarantee uniqueness, they should
be based on the full path in a project's source tree. For
example, the file <code>foo/src/bar/baz.h</code> in
project <code>foo</code> should have the following
guard:</p>

<pre>#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_
</pre>




</div> 

<h3 id="Forward_Declarations">Forward Declarations</h3>

<div class="summary">
  <p>Avoid using forward declarations where possible.
  Just <code>#include</code> the headers you need.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>A "forward declaration" is a declaration of a class,
function, or template without an associated definition.</p>
</div>

<div class="pros">
<ul>
  <li>Forward declarations can save compile time, as
  <code>#include</code>s force the compiler to open
  more files and process more input.</li>

  <li>Forward declarations can save on unnecessary
  recompilation. <code>#include</code>s can force
  your code to be recompiled more often, due to unrelated
  changes in the header.</li>
</ul>
</div>

<div class="cons">
<ul>
  <li>Forward declarations can hide a dependency, allowing
  user code to skip necessary recompilation when headers
  change.</li>

  <li>A forward declaration may be broken by subsequent
  changes to the library. Forward declarations of functions
  and templates can prevent the header owners from making
  otherwise-compatible changes to their APIs, such as
  widening a parameter type, adding a template parameter
  with a default value, or migrating to a new namespace.</li>

  <li>Forward declaring symbols from namespace
  <code>std::</code> yields undefined behavior.</li>

  <li>It can be difficult to determine whether a forward
  declaration or a full <code>#include</code> is needed.
  Replacing an <code>#include</code> with a forward
  declaration can silently change the meaning of
  code:
      <pre>      // b.h:
      struct B {};
      struct D : B {};

      // good_user.cc:
      #include "b.h"
      void f(B*);
      void f(void*);
      void test(D* x) { f(x); }  // calls f(B*)
      </pre>
  If the <code>#include</code> was replaced with forward
  decls for <code>B</code> and <code>D</code>,
  <code>test()</code> would call <code>f(void*)</code>.
  </li>

  <li>Forward declaring multiple symbols from a header
  can be more verbose than simply
  <code>#include</code>ing the header.</li>

  <li>Structuring code to enable forward declarations
  (e.g. using pointer members instead of object members)
  can make the code slower and more complex.</li>

  
</ul>
</div>

<div class="decision">
<ul>
  <li>Try to avoid forward declarations of entities
  defined in another project.</li>

  <li>When using a function declared in a header file,
  always <code>#include</code> that header.</li>

  <li>When using a class template, prefer to
  <code>#include</code> its header file.</li>
</ul>

<p>Please see <a href="#Names_and_Order_of_Includes">Names and Order
of Includes</a> for rules about when to #include a header.</p>
</div>

</div> 

<h3 id="Inline_Functions">Inline Functions</h3>

<div class="summary">
<p>Define functions inline only when they are small, say, 10
lines or fewer.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>You can declare functions in a way that allows the compiler to expand
them inline rather than calling them through the usual
function call mechanism.</p>
</div>

<div class="pros">
<p>Inlining a function can generate more efficient object
code, as long as the inlined function is small. Feel free
to inline accessors and mutators, and other short,
performance-critical functions.</p>
</div>

<div class="cons">
<p>Overuse of inlining can actually make programs slower.
Depending on a function's size, inlining it can cause the
code size to increase or decrease. Inlining a very small
accessor function will usually decrease code size while
inlining a very large function can dramatically increase
code size. On modern processors smaller code usually runs
faster due to better use of the instruction cache.</p>
</div>

<div class="decision">
<p>A decent rule of thumb is to not inline a function if
it is more than 10 lines long. Beware of destructors,
which are often longer than they appear because of
implicit member- and base-destructor calls!</p>

<p>Another useful rule of thumb: it's typically not cost
effective to inline functions with loops or switch
statements (unless, in the common case, the loop or
switch statement is never executed).</p>

<p>It is important to know that functions are not always
inlined even if they are declared as such; for example,
virtual and recursive functions are not normally inlined.
Usually recursive functions should not be inline. The
main reason for making a virtual function inline is to
place its definition in the class, either for convenience
or to document its behavior, e.g., for accessors and
mutators.</p>
</div> 

</div> 

<h3 id="Names_and_Order_of_Includes">Names and Order of Includes</h3>

<div class="summary">
<p>Use standard order for readability and to avoid hidden
dependencies: Related header, C library, C++ library,  other libraries'
<code>.h</code>, your project's <code>.h</code>.</p>
</div>

<div class="stylebody">
<p>
All of a project's header files should be
listed as descendants of the project's source
directory without use of UNIX directory shortcuts
<code>.</code> (the current directory) or <code>..</code>
(the parent directory). For example,

<code>google-awesome-project/src/base/logging.h</code>
should be included as:</p>

<pre>#include "base/logging.h"
</pre>

<p>In <code><var>dir/foo</var>.cc</code> or
<code><var>dir/foo_test</var>.cc</code>, whose main
purpose is to implement or test the stuff in
<code><var>dir2/foo2</var>.h</code>, order your includes
as follows:</p>

<ol>
  <li><code><var>dir2/foo2</var>.h</code>.</li>

  <li>C system files.</li>

  <li>C++ system files.</li>

  <li>Other libraries' <code>.h</code>
  files.</li>

  <li>
  Your project's <code>.h</code>
  files.</li>
</ol>

<p>With the preferred ordering, if
<code><var>dir2/foo2</var>.h</code> omits any necessary
includes, the build of <code><var>dir/foo</var>.cc</code>
or <code><var>dir/foo</var>_test.cc</code> will break.
Thus, this rule ensures that build breaks show up first
for the people working on these files, not for innocent
people in other packages.</p>

<p><code><var>dir/foo</var>.cc</code> and
<code><var>dir2/foo2</var>.h</code> are usually in the same
directory (e.g. <code>base/basictypes_test.cc</code> and
<code>base/basictypes.h</code>), but may sometimes be in different
directories too.</p>



<p>Within each section the includes should be ordered
alphabetically. Note that older code might not conform to
this rule and should be fixed when convenient.</p>

<p>You should include all the headers that define the symbols you rely
upon, except in the unusual case of <a href="#Forward_Declarations">forward
declaration</a>. If you rely on symbols from <code>bar.h</code>,
don't count on the fact that you included <code>foo.h</code> which
(currently) includes <code>bar.h</code>: include <code>bar.h</code>
yourself, unless <code>foo.h</code> explicitly demonstrates its intent
to provide you the symbols of <code>bar.h</code>.  However, any
includes present in the related header do not need to be included
again in the related <code>cc</code> (i.e., <code>foo.cc</code> can
rely on <code>foo.h</code>'s includes).</p>

<p>For example, the includes in

<code>google-awesome-project/src/foo/internal/fooserver.cc</code>
might look like this:</p>


<pre>#include "foo/server/fooserver.h"

#include &lt;sys/types.h&gt;
#include &lt;unistd.h&gt;

#include &lt;hash_map&gt;
#include &lt;vector&gt;

#include "base/basictypes.h"
#include "base/commandlineflags.h"
#include "foo/server/bar.h"
</pre>

<p class="exception">Sometimes, system-specific code needs
conditional includes. Such code can put conditional
includes after other includes. Of course, keep your
system-specific code small and localized. Example:</p>

<pre>#include "foo/public/fooserver.h"

#include "base/port.h"  // For LANG_CXX11.

#ifdef LANG_CXX11
#include &lt;initializer_list&gt;
#endif  // LANG_CXX11
</pre>

</div> 

<h2 id="Scoping">Scoping</h2>

<h3 id="Namespaces">Namespaces</h3>

<div class="summary">
<p>With few exceptions, place code in a namespace. Namespaces
should have unique names based on the project name, and possibly
its path. Do not use <i>using-directives</i> (e.g.
<code>using namespace foo</code>). Do not use
inline namespaces. For unnamed namespaces, see
<a href="#Unnamed_Namespaces_and_Static_Variables">Unnamed Namespaces and
Static Variables</a>.
</p></div>

<div class="stylebody">

<div class="definition">
<p>Namespaces subdivide the global scope
into distinct, named scopes, and so are useful for preventing
name collisions in the global scope.</p>
</div>

<div class="pros">

<p>Namespaces provide a method for preventing name conflicts
in large programs while allowing most code to use reasonably
short names.</p>

<p>For example, if two different projects have a class
<code>Foo</code> in the global scope, these symbols may
collide at compile time or at runtime. If each project
places their code in a namespace, <code>project1::Foo</code>
and <code>project2::Foo</code> are now distinct symbols that
do not collide, and code within each project's namespace
can continue to refer to <code>Foo</code> without the prefix.</p>

<p>Inline namespaces automatically place their names in
the enclosing scope. Consider the following snippet, for
example:</p>

<pre>namespace X {
inline namespace Y {
  void foo();
}  // namespace Y
}  // namespace X
</pre>

<p>The expressions <code>X::Y::foo()</code> and
<code>X::foo()</code> are interchangeable. Inline
namespaces are primarily intended for ABI compatibility
across versions.</p>
</div>

<div class="cons">

<p>Namespaces can be confusing, because they complicate
the mechanics of figuring out what definition a name refers
to.</p>

<p>Inline namespaces, in particular, can be confusing
because names aren't actually restricted to the namespace
where they are declared. They are only useful as part of
some larger versioning policy.</p>

<p>In some contexts, it's necessary to repeatedly refer to
symbols by their fully-qualified names. For deeply-nested
namespaces, this can add a lot of clutter.</p>
</div>

<div class="decision">

<p>Namespaces should be used as follows:</p>

<ul>
  <li>Follow the rules on <a href="#Namespace_Names">Namespace Names</a>.
  </li><li>Terminate namespaces with comments as shown in the given examples.
  </li><li>

  <p>Namespaces wrap the entire source file after
  includes,  
  <a href="https://gflags.github.io/gflags/">
  gflags</a> definitions/declarations
  and forward declarations of classes from other namespaces.</p>

<pre>// In the .h file
namespace mynamespace {

// All declarations are within the namespace scope.
// Notice the lack of indentation.
class MyClass {
 public:
  ...
  void Foo();
};

}  // namespace mynamespace
</pre>

<pre>// In the .cc file
namespace mynamespace {

// Definition of functions is within scope of the namespace.
void MyClass::Foo() {
  ...
}

}  // namespace mynamespace
</pre>

  <p>More complex <code>.cc</code> files might have additional details,
  like flags or using-declarations.</p>

<pre>#include "a.h"

DEFINE_FLAG(bool, someflag, false, "dummy flag");

namespace a {

using ::foo::bar;

...code for a...         // Code goes against the left margin.

}  // namespace a
</pre>
  </li>

  

  <li>Do not declare anything in namespace
  <code>std</code>, including forward declarations of
  standard library classes. Declaring entities in
  namespace <code>std</code> is undefined behavior, i.e.,
  not portable. To declare entities from the standard
  library, include the appropriate header file.</li>

  <li><p>You may not use a <i>using-directive</i>
  to make all names from a namespace available.</p>

<pre class="badcode">// Forbidden -- This pollutes the namespace.
using namespace foo;
</pre>
  </li>

  <li><p>Do not use <i>Namespace aliases</i> at namespace scope
  in header files except in explicitly marked
  internal-only namespaces, because anything imported into a namespace
  in a header file becomes part of the public
  API exported by that file.</p>

<pre>// Shorten access to some commonly used names in .cc files.
namespace baz = ::foo::bar::baz;
</pre>

<pre>// Shorten access to some commonly used names (in a .h file).
namespace librarian {
namespace impl {  // Internal, not part of the API.
namespace sidetable = ::pipeline_diagnostics::sidetable;
}  // namespace impl

inline void my_inline_function() {
  // namespace alias local to a function (or method).
  namespace baz = ::foo::bar::baz;
  ...
}
}  // namespace librarian
</pre>

  </li><li>Do not use inline namespaces.</li>
</ul>
</div>
</div>

<h3 id="Unnamed_Namespaces_and_Static_Variables">Unnamed Namespaces and Static
Variables</h3>

<div class="summary">
<p>When definitions in a <code>.cc</code> file do not need to be
referenced outside that file, place them in an unnamed
namespace or declare them <code>static</code>. Do not use either
of these constructs in <code>.h</code> files.
</p></div>

<div class="stylebody">

<div class="definition">
<p>All declarations can be given internal linkage by placing them in
unnamed namespaces, and functions and variables can be given internal linkage by
declaring them <code>static</code>. This means that anything you're declaring
can't be accessed from another file. If a different file declares something
with the same name, then the two entities are completely independent.</p>
</div>

<div class="decision">

<p>Use of internal linkage in <code>.cc</code> files is encouraged
for all code that does not need to be referenced elsewhere.
Do not use internal linkage in <code>.h</code> files.</p>

<p>Format unnamed namespaces like named namespaces. In the
  terminating comment, leave the namespace name empty:</p>

<pre>namespace {
...
}  // namespace
</pre>
</div>
</div>

<h3 id="Nonmember,_Static_Member,_and_Global_Functions">Nonmember, Static Member, and Global Functions</h3>

<div class="summary">
<p>Prefer placing nonmember functions in a namespace; use completely global
functions rarely. Prefer grouping functions with a namespace instead of
using a class as if it were a namespace. Static methods of a class should
generally be closely related to instances of the class or the class's static
data.</p>
</div>

 <div class="stylebody">

 <div class="pros">
 <p>Nonmember and static member functions can be useful in
 some situations. Putting nonmember functions in a
 namespace avoids polluting the global namespace.</p>
 </div>

<div class="cons">
<p>Nonmember and static member functions may make more sense
as members of a new class, especially if they access
external resources or have significant dependencies.</p>
</div>

<div class="decision">
<p>Sometimes it is useful to define a
function not bound to a class instance. Such a function
can be either a static member or a nonmember function.
Nonmember functions should not depend on external
variables, and should nearly always exist in a namespace.
Rather than creating classes only to group static member
functions which do not share static data, use
<a href="#Namespaces">namespaces</a> instead. For a header
<code>myproject/foo_bar.h</code>, for example, write</p>
<pre>namespace myproject {
namespace foo_bar {
void Function1();
void Function2();
}  // namespace foo_bar
}  // namespace myproject
</pre>
<p>instead of</p>
<pre class="badcode">namespace myproject {
class FooBar {
 public:
  static void Function1();
  static void Function2();
};
}  // namespace myproject
</pre>

<p>If you define a nonmember function and it is only
needed in its <code>.cc</code> file, use
<a href="#Unnamed_Namespaces_and_Static_Variables">internal linkage</a> to limit
its scope.</p>
</div>

</div> 

<h3 id="Local_Variables">Local Variables</h3>

<div class="summary">
<p>Place a function's variables in the narrowest scope
possible, and initialize variables in the declaration.</p>
</div>

<div class="stylebody">

<p>C++ allows you to declare variables anywhere in a
function. We encourage you to declare them in as local a
scope as possible, and as close to the first use as
possible. This makes it easier for the reader to find the
declaration and see what type the variable is and what it
was initialized to. In particular, initialization should
be used instead of declaration and assignment, e.g.:</p>

<pre class="badcode">int i;
i = f();      // Bad -- initialization separate from declaration.
</pre>

<pre>int j = g();  // Good -- declaration has initialization.
</pre>

<pre class="badcode">std::vector&lt;int&gt; v;
v.push_back(1);  // Prefer initializing using brace initialization.
v.push_back(2);
</pre>

<pre>std::vector&lt;int&gt; v = {1, 2};  // Good -- v starts initialized.
</pre>

<p>Variables needed for <code>if</code>, <code>while</code>
and <code>for</code> statements should normally be declared
within those statements, so that such variables are confined
to those scopes.  E.g.:</p>

<pre>while (const char* p = strchr(str, '/')) str = p + 1;
</pre>

<p>There is one caveat: if the variable is an object, its
constructor is invoked every time it enters scope and is
created, and its destructor is invoked every time it goes
out of scope.</p>

<pre class="badcode">// Inefficient implementation:
for (int i = 0; i &lt; 1000000; ++i) {
  Foo f;  // My ctor and dtor get called 1000000 times each.
  f.DoSomething(i);
}
</pre>

<p>It may be more efficient to declare such a variable
used in a loop outside that loop:</p>

<pre>Foo f;  // My ctor and dtor get called once each.
for (int i = 0; i &lt; 1000000; ++i) {
  f.DoSomething(i);
}
</pre>

</div> 

<h3 id="Static_and_Global_Variables">Static and Global Variables</h3>

<div class="summary">
  <p>Variables of class type with <a href="http://en.cppreference.com/w/cpp/language/storage_duration#Storage_duration">
    static storage duration</a> are forbidden: they cause hard-to-find bugs due
  to indeterminate order of construction and destruction. However, such
  variables are allowed if they are <code>constexpr</code>: they have no
  dynamic initialization or destruction.</p>
</div>

<div class="stylebody">

<p>Objects with static storage duration, including global
variables, static variables, static class member
variables, and function static variables, must be Plain
Old Data (POD): only ints, chars, floats, or pointers, or
arrays/structs of POD.</p>

<p>The order in which class constructors and initializers
for static variables are called is only partially
specified in C++ and can even change from build to build,
which can cause bugs that are difficult to find.
Therefore in addition to banning globals of class type,
we do not allow non-local static variables to be initialized
with the result of a function, unless that function (such
as getenv(), or getpid()) does not itself depend on any
other globals. However, a static POD variable within
function scope may be initialized with the result of a
function, since its initialization order is well-defined
and does not occur until control passes through its
declaration.</p>

<p>Likewise, global and static variables are destroyed
when the program terminates, regardless of whether the
termination is by returning from <code>main()</code> or
by calling <code>exit()</code>. The order in which
destructors are called is defined to be the reverse of
the order in which the constructors were called. Since
constructor order is indeterminate, so is destructor
order. For example, at program-end time a static variable
might have been destroyed, but code still running
&#8212; perhaps in another thread
&#8212; tries to access it and fails. Or the
destructor for a static <code>string</code> variable
might be run prior to the destructor for another variable
that contains a reference to that string.</p>

<p>One way to alleviate the destructor problem is to
terminate the program by calling
<code>quick_exit()</code> instead of <code>exit()</code>.
The difference is that <code>quick_exit()</code> does not
invoke destructors and does not invoke any handlers that
were registered by calling <code>atexit()</code>. If you
have a handler that needs to run when a program
terminates via <code>quick_exit()</code> (flushing logs,
for example), you can register it using
<code>at_quick_exit()</code>. (If you have a handler that
needs to run at both <code>exit()</code> and
<code>quick_exit()</code>, you need to register it in
both places.)</p>

<p>As a result we only allow static variables to contain
POD data. This rule completely disallows
<code>std::vector</code> (use C arrays instead), or
<code>string</code> (use <code>const char []</code>).</p>



<p>If you need a static or global
variable of a class type, consider initializing a pointer
(which will never be freed), from either your main()
function or from pthread_once(). Note that this must be a
raw pointer, not a "smart" pointer, since the smart
pointer's destructor will have the order-of-destructor
issue that we are trying to avoid.</p>





</div> 

<h2 id="Classes">Classes</h2>

<p>Classes are the fundamental unit of code in C++. Naturally,
we use them extensively. This section lists the main dos and
don'ts you should follow when writing a class.</p>

<h3 id="Doing_Work_in_Constructors">Doing Work in Constructors</h3>

<div class="summary">
<p>Avoid virtual method calls in constructors, and avoid
initialization that can fail if you can't signal an error.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>It is possible to perform arbitrary initialization in the body
of the constructor.</p>
</div>

<div class="pros">
<ul>
  <li>No need to worry about whether the class has been initialized or
  not.</li>

  <li>Objects that are fully initialized by constructor call can
  be <code>const</code> and may also be easier to use with standard containers
  or algorithms.</li>
</ul>

</div>

<div class="cons">
<ul>
  <li>If the work calls virtual functions, these calls
  will not get dispatched to the subclass
  implementations. Future modification to your class can
  quietly introduce this problem even if your class is
  not currently subclassed, causing much confusion.</li>

  <li>There is no easy way for constructors to signal errors, short of
  crashing the program (not always appropriate) or using exceptions
  (which are <a href="#Exceptions">forbidden</a>).</li>

  <li>If the work fails, we now have an object whose initialization
  code failed, so it may be an unusual state requiring a <code>bool
  IsValid()</code> state checking mechanism (or similar) which is easy
  to forget to call.</li>

  <li>You cannot take the address of a constructor, so whatever work
  is done in the constructor cannot easily be handed off to, for
  example, another thread.</li>
</ul>
</div>


<div class="decision">
<p>Constructors should never call virtual functions. If appropriate
for your code
,
terminating the program may be an appropriate error handling
response. Otherwise, consider a factory function
or <code>Init()</code> method. Avoid <code>Init()</code> methods on objects with
no other states that affect which public methods may be called
(semi-constructed objects of this form are particularly hard to work
with correctly).</p>
</div>

</div> 

<a id="Explicit_Constructors"></a>
<h3 id="Implicit_Conversions">Implicit Conversions</h3>

<div class="summary">
<p>Do not define implicit conversions. Use the <code>explicit</code>
keyword for conversion operators and single-argument
constructors.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>Implicit conversions allow an
object of one type (called the <dfn>source type</dfn>) to
be used where a different type (called the <dfn>destination
type</dfn>) is expected, such as when passing an
<code>int</code> argument to a function that takes a
<code>double</code> parameter.</p>

<p>In addition to the implicit conversions defined by the language,
users can define their own, by adding appropriate members to the
class definition of the source or destination type. An implicit
conversion in the source type is defined by a type conversion operator
named after the destination type (e.g. <code>operator
bool()</code>). An implicit conversion in the destination
type is defined by a constructor that can take the source type as
its only argument (or only argument with no default value).</p>

<p>The <code>explicit</code> keyword can be applied to a constructor
or (since C++11) a conversion operator, to ensure that it can only be
used when the destination type is explicit at the point of use,
e.g. with a cast. This applies not only to implicit conversions, but to
C++11's list initialization syntax:</p>
<pre>class Foo {
  explicit Foo(int x, double y);
  ...
};

void Func(Foo f);
</pre>
<pre class="badcode">Func({42, 3.14});  // Error
</pre>
This kind of code isn't technically an implicit conversion, but the
language treats it as one as far as <code>explicit</code> is concerned.
</div>

<div class="pros">
<ul>
<li>Implicit conversions can make a type more usable and
    expressive by eliminating the need to explicitly name a type
    when it's obvious.</li>
<li>Implicit conversions can be a simpler alternative to
    overloading.</li>
<li>List initialization syntax is a concise and expressive
    way of initializing objects.</li>
</ul>
</div>

<div class="cons">
<ul>
<li>Implicit conversions can hide type-mismatch bugs, where the
    destination type does not match the user's expectation, or
    the user is unaware that any conversion will take place.</li>

<li>Implicit conversions can make code harder to read, particularly
    in the presence of overloading, by making it less obvious what
    code is actually getting called.</li>

<li>Constructors that take a single argument may accidentally
    be usable as implicit type conversions, even if they are not
    intended to do so.</li>

<li>When a single-argument constructor is not marked
    <code>explicit</code>, there's no reliable way to tell whether
    it's intended to define an implicit conversion, or the author
    simply forgot to mark it.</li>

<li>It's not always clear which type should provide the conversion,
    and if they both do, the code becomes ambiguous.</li>

<li>List initialization can suffer from the same problems if
    the destination type is implicit, particularly if the
    list has only a single element.</li>
</ul>
</div>

<div class="decision">
<p>Type conversion operators, and constructors that are
callable with a single argument, must be marked
<code>explicit</code> in the class definition. As an
exception, copy and move constructors should not be
<code>explicit</code>, since they do not perform type
conversion. Implicit conversions can sometimes be necessary and
appropriate for types that are designed to transparently wrap other
types. In that case, contact 
your project leads to request
a waiver of this rule.</p>

<p>Constructors that cannot be called with a single argument
should usually omit <code>explicit</code>. Constructors that
take a single <code>std::initializer_list</code> parameter should
also omit <code>explicit</code>, in order to support copy-initialization
(e.g. <code>MyType m = {1, 2};</code>).</p>
</div>

</div> 

<h3 id="Copyable_Movable_Types">Copyable and Movable Types</h3>
<a id="Copy_Constructors"></a>
<div class="summary">
<p>Support copying and/or moving if these operations are clear and meaningful
for your type. Otherwise, disable the implicitly generated special functions
that perform copies and moves.
</p></div>

<div class="stylebody">

<div class="definition">
<p>A copyable type allows its objects to be initialized or assigned
from any other object of the same type, without changing the value of the source.
For user-defined types, the copy behavior is defined by the copy
constructor and the copy-assignment operator.
<code>string</code> is an example of a copyable type.</p>

<p>A movable type is one that can be initialized and assigned
from temporaries (all copyable types are therefore movable).
<code>std::unique_ptr&lt;int&gt;</code> is an example of a movable but not
copyable type. For user-defined types, the move behavior is defined by the move
constructor and the move-assignment operator.</p>

<p>The copy/move constructors can be implicitly invoked by the compiler
in some situations, e.g. when passing objects by value.</p>
</div>

<div class="pros">
<p>Objects of copyable and movable types can be passed and returned by value,
which makes APIs simpler, safer, and more general. Unlike when passing objects
by pointer or reference, there's no risk of confusion over ownership,
lifetime, mutability, and similar issues, and no need to specify them in the
contract. It also prevents non-local interactions between the client and the
implementation, which makes them easier to understand, maintain, and optimize by
the compiler. Further, such objects can be used with generic APIs that
require pass-by-value, such as most containers, and they allow for additional
flexibility in e.g., type composition.</p>

<p>Copy/move constructors and assignment operators are usually
easier to define correctly than alternatives
like <code>Clone()</code>, <code>CopyFrom()</code> or <code>Swap()</code>,
because they can be generated by the compiler, either implicitly or
with <code>= default</code>.  They are concise, and ensure
that all data members are copied. Copy and move
constructors are also generally more efficient, because they don't
require heap allocation or separate initialization and assignment
steps, and they're eligible for optimizations such as

<a href="http://en.cppreference.com/w/cpp/language/copy_elision">
copy elision</a>.</p>

<p>Move operations allow the implicit and efficient transfer of
resources out of rvalue objects. This allows a plainer coding style
in some cases.</p>
</div>

<div class="cons">
<p>Some types do not need to be copyable, and providing copy
operations for such types can be confusing, nonsensical, or outright
incorrect. Types representing singleton objects (<code>Registerer</code>),
objects tied to a specific scope (<code>Cleanup</code>), or closely coupled to
object identity (<code>Mutex</code>) cannot be copied meaningfully.
Copy operations for base class types that are to be used
polymorphically are hazardous, because use of them can lead to
<a href="https://en.wikipedia.org/wiki/Object_slicing">object slicing</a>.
Defaulted or carelessly-implemented copy operations can be incorrect, and the
resulting bugs can be confusing and difficult to diagnose.</p>

<p>Copy constructors are invoked implicitly, which makes the
invocation easy to miss. This may cause confusion for programmers used to
languages where pass-by-reference is conventional or mandatory. It may also
encourage excessive copying, which can cause performance problems.</p>
</div>

<div class="decision">

<p>Provide the copy and move operations if their meaning is clear to a casual
user and the copying/moving does not incur unexpected costs. If you define a
copy or move constructor, define the corresponding assignment operator, and
vice-versa. If your type is copyable, do not define move operations unless they
are significantly more efficient than the corresponding copy operations. If your
type is not copyable, but the correctness of a move is obvious to users of the
type, you may make the type move-only by defining both of the move operations.
</p>

<p>If your type provides copy operations, it is recommended that you design
your class so that the default implementation of those operations is correct.
Remember to review the correctness of any defaulted operations as you would any
other code, and to document that your class is copyable and/or cheaply movable
if that's an API guarantee.</p>

<pre class="badcode">class Foo {
 public:
  Foo(Foo&amp;&amp; other) : field_(other.field) {}
  // Bad, defines only move constructor, but not operator=.

 private:
  Field field_;
};
</pre>

<p>Due to the risk of slicing, avoid providing an assignment
operator or public copy/move constructor for a class that's
intended to be derived from (and avoid deriving from a class
with such members). If your base class needs to be
copyable, provide a public virtual <code>Clone()</code>
method, and a protected copy constructor that derived classes
can use to implement it.</p>

<p>If you do not want to support copy/move operations on your type,
explicitly disable them using <code>= delete</code> in
the <code>public:</code> section:</p>

<pre class="code">// MyClass is neither copyable nor movable.
MyClass(const MyClass&amp;) = delete;
MyClass&amp; operator=(const MyClass&amp;) = delete;
</pre>

<p></p>

</div> 
</div> 

<h3 id="Structs_vs._Classes">Structs vs. Classes</h3>

<div class="summary">
<p>Use a <code>struct</code> only for passive objects that
      carry data; everything else is a <code>class</code>.</p>
</div>

<div class="stylebody">

<p>The <code>struct</code> and <code>class</code>
keywords behave almost identically in C++. We add our own
semantic meanings to each keyword, so you should use the
appropriate keyword for the data-type you're
defining.</p>

<p><code>structs</code> should be used for passive
objects that carry data, and may have associated
constants, but lack any functionality other than
access/setting the data members. The accessing/setting of
fields is done by directly accessing the fields rather
than through method invocations. Methods should not
provide behavior but should only be used to set up the
data members, e.g., constructor, destructor,
<code>Initialize()</code>, <code>Reset()</code>,
<code>Validate()</code>.</p>

<p>If more functionality is required, a
<code>class</code> is more appropriate. If in doubt, make
it a <code>class</code>.</p>

<p>For consistency with STL, you can use
<code>struct</code> instead of <code>class</code> for
functors and traits.</p>

<p>Note that member variables in structs and classes have
<a href="#Variable_Names">different naming rules</a>.</p>

</div> 

<h3 id="Inheritance">Inheritance</h3>

<div class="summary">
<p>Composition is often more appropriate than inheritance.
When using inheritance, make it <code>public</code>.</p>
</div>

<div class="stylebody">

<div class="definition">
<p> When a sub-class
inherits from a base class, it includes the definitions
of all the data and operations that the parent base class
defines. In practice, inheritance is used in two major
ways in C++: implementation inheritance, in which actual
code is inherited by the child, and
<a href="#Interfaces">interface inheritance</a>, in which
only method names are inherited.</p>
</div>

<div class="pros">
<p>Implementation inheritance reduces code size by re-using
the base class code as it specializes an existing type.
Because inheritance is a compile-time declaration, you
and the compiler can understand the operation and detect
errors. Interface inheritance can be used to
programmatically enforce that a class expose a particular
API. Again, the compiler can detect errors, in this case,
when a class does not define a necessary method of the
API.</p>
</div>

<div class="cons">
<p>For implementation inheritance, because the code
implementing a sub-class is spread between the base and
the sub-class, it can be more difficult to understand an
implementation. The sub-class cannot override functions
that are not virtual, so the sub-class cannot change
implementation. The base class may also define some data
members, so that specifies physical layout of the base
class.</p>
</div>

<div class="decision">

<p>All inheritance should be <code>public</code>. If you
want to do private inheritance, you should be including
an instance of the base class as a member instead.</p>

<p>Do not overuse implementation inheritance. Composition
is often more appropriate. Try to restrict use of
inheritance to the "is-a" case: <code>Bar</code>
subclasses <code>Foo</code> if it can reasonably be said
that <code>Bar</code> "is a kind of"
<code>Foo</code>.</p>

<p>Make your destructor <code>virtual</code> if
necessary. If your class has virtual methods, its
destructor  should be virtual.</p>

<p>Limit the use of <code>protected</code> to those
member functions that might need to be accessed from
subclasses. Note that <a href="#Access_Control">data
members should be private</a>.</p>

<p>Explicitly annotate overrides of virtual functions
or virtual destructors with an <code>override</code>
or (less frequently) <code>final</code> specifier.
Older (pre-C++11) code will use the
<code>virtual</code> keyword as an inferior
alternative annotation. For clarity, use exactly one of
<code>override</code>, <code>final</code>, or
<code>virtual</code> when declaring an override.
Rationale: A function or destructor marked
<code>override</code> or <code>final</code> that is
not an override of a base class virtual function will
not compile, and this helps catch common errors. The
specifiers serve as documentation; if no specifier is
present, the reader has to check all ancestors of the
class in question to determine if the function or
destructor is virtual or not.</p>
</div>

</div> 

<h3 id="Multiple_Inheritance">Multiple Inheritance</h3>

<div class="summary">
<p>Only very rarely is multiple implementation inheritance
actually useful. We allow multiple inheritance only when at
most one of the base classes has an implementation; all
other base classes must be <a href="#Interfaces">pure
interface</a> classes tagged with the
<code>Interface</code> suffix.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>Multiple inheritance allows a sub-class to have more than
one base class. We distinguish between base classes that are
<em>pure interfaces</em> and those that have an
<em>implementation</em>.</p>
</div>

<div class="pros">
<p>Multiple implementation inheritance may let you re-use
even more code than single inheritance (see <a href="#Inheritance">Inheritance</a>).</p>
</div>

<div class="cons">
<p>Only very rarely is multiple <em>implementation</em>
inheritance actually useful. When multiple implementation
inheritance seems like the solution, you can usually find
a different, more explicit, and cleaner solution.</p>
</div>

<div class="decision">
<p> Multiple inheritance is allowed only when all
superclasses, with the possible exception of the first one,
are <a href="#Interfaces">pure interfaces</a>. In order to
ensure that they remain pure interfaces, they must end with
the <code>Interface</code> suffix.</p>
</div>

<div class="note">
<p>There is an <a href="#Windows_Code">exception</a> to
this rule on Windows.</p>
</div>

</div> 

<h3 id="Interfaces">Interfaces</h3>

<div class="summary">
<p>Classes that satisfy certain conditions are allowed, but
not required, to end with an <code>Interface</code> suffix.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>A class is a pure interface if it meets the following
requirements:</p>

<ul>
  <li>It has only public pure virtual ("<code>=
  0</code>") methods and static methods (but see below
  for destructor).</li>

  <li>It may not have non-static data members.</li>

  <li>It need not have any constructors defined. If a
  constructor is provided, it must take no arguments and
  it must be protected.</li>

  <li>If it is a subclass, it may only be derived from
  classes that satisfy these conditions and are tagged
  with the <code>Interface</code> suffix.</li>
</ul>

<p>An interface class can never be directly instantiated
because of the pure virtual method(s) it declares. To
make sure all implementations of the interface can be
destroyed correctly, the interface must also declare a
virtual destructor (in an exception to the first rule,
this should not be pure). See Stroustrup, <cite>The C++
Programming Language</cite>, 3rd edition, section 12.4
for details.</p>
</div>

<div class="pros">
<p>Tagging a class with the <code>Interface</code> suffix
lets others know that they must not add implemented
methods or non static data members. This is particularly
important in the case of <a href="#Multiple_Inheritance">multiple inheritance</a>.
Additionally, the interface concept is already
well-understood by Java programmers.</p>
</div>

<div class="cons">
<p>The <code>Interface</code> suffix lengthens the class
name, which can make it harder to read and understand.
Also, the interface property may be considered an
implementation detail that shouldn't be exposed to
clients.</p>
</div>

<div class="decision">
<p>A class may end
with <code>Interface</code> only if it meets the above
requirements. We do not require the converse, however:
classes that meet the above requirements are not required
to end with <code>Interface</code>.</p>
</div>

</div> 

<h3 id="Operator_Overloading">Operator Overloading</h3>

<div class="summary">
<p>Overload operators judiciously. Do not create user-defined literals.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>C++ permits user code to
<a href="http://en.cppreference.com/w/cpp/language/operators">declare
overloaded versions of the built-in operators</a> using the
<code>operator</code> keyword, so long as one of the parameters
is a user-defined type. The <code>operator</code> keyword also
permits user code to define new kinds of literals using
<code>operator""</code>, and to define type-conversion functions
such as <code>operator bool()</code>.</p>
</div>

<div class="pros">
<p>Operator overloading can make code more concise and
intuitive by enabling user-defined types to behave the same
as built-in types. Overloaded operators are the idiomatic names
for certain operations (e.g. <code>==</code>, <code>&lt;</code>,
<code>=</code>, and <code>&lt;&lt;</code>), and adhering to
those conventions can make user-defined types more readable
and enable them to interoperate with libraries that expect
those names.</p>

<p>User-defined literals are a very concise notation for
creating objects of user-defined types.</p>
</div>

<div class="cons">
<ul>
  <li>Providing a correct, consistent, and unsurprising
  set of operator overloads requires some care, and failure
  to do so can lead to confusion and bugs.</li>

  <li>Overuse of operators can lead to obfuscated code,
  particularly if the overloaded operator's semantics
  don't follow convention.</li>

  <li>The hazards of function overloading apply just as
  much to operator overloading, if not more so.</li>

  <li>Operator overloads can fool our intuition into
  thinking that expensive operations are cheap, built-in
  operations.</li>

  <li>Finding the call sites for overloaded operators may
  require a search tool that's aware of C++ syntax, rather
  than e.g. grep.</li>

  <li>If you get the argument type of an overloaded operator
  wrong, you may get a different overload rather than a
  compiler error. For example, <code>foo &lt; bar</code>
  may do one thing, while <code>&amp;foo &lt; &amp;bar</code>
  does something totally different.</li>

  <li>Certain operator overloads are inherently hazardous.
  Overloading unary <code>&amp;</code> can cause the same
  code to have different meanings depending on whether
  the overload declaration is visible. Overloads of
  <code>&amp;&amp;</code>, <code>||</code>, and <code>,</code>
  (comma) cannot match the evaluation-order semantics of the
  built-in operators.</li>

  <li>Operators are often defined outside the class,
  so there's a risk of different files introducing
  different definitions of the same operator. If both
  definitions are linked into the same binary, this results
  in undefined behavior, which can manifest as subtle
  run-time bugs.</li>

  <li>User-defined literals allow the creation of new
  syntactic forms that are unfamiliar even to experienced C++
  programmers.</li>
</ul>
</div>

<div class="decision">
<p>Define overloaded operators only if their meaning is
obvious, unsurprising, and consistent with the corresponding
built-in operators. For example, use <code>|</code> as a
bitwise- or logical-or, not as a shell-style pipe.</p>

<p>Define operators only on your own types. More precisely,
define them in the same headers, .cc files, and namespaces
as the types they operate on. That way, the operators are available
wherever the type is, minimizing the risk of multiple
definitions. If possible, avoid defining operators as templates,
because they must satisfy this rule for any possible template
arguments. If you define an operator, also define
any related operators that make sense, and make sure they
are defined consistently. For example, if you overload
<code>&lt;</code>, overload all the comparison operators,
and make sure <code>&lt;</code> and <code>&gt;</code> never
return true for the same arguments.</p>

<p>Prefer to define non-modifying binary operators as
non-member functions. If a binary operator is defined as a
class member, implicit conversions will apply to the
right-hand argument, but not the left-hand one. It will
confuse your users if <code>a &lt; b</code> compiles but
<code>b &lt; a</code> doesn't.</p>

<p>Don't go out of your way to avoid defining operator
overloads. For example, prefer to define <code>==</code>,
<code>=</code>, and <code>&lt;&lt;</code>, rather than
<code>Equals()</code>, <code>CopyFrom()</code>, and
<code>PrintTo()</code>. Conversely, don't define
operator overloads just because other libraries expect
them. For example, if your type doesn't have a natural
ordering, but you want to store it in a <code>std::set</code>,
use a custom comparator rather than overloading
<code>&lt;</code>.</p>

<p>Do not overload <code>&amp;&amp;</code>, <code>||</code>,
<code>,</code> (comma), or unary <code>&amp;</code>. Do not overload
<code>operator""</code>, i.e. do not introduce user-defined
literals.</p>

<p>Type conversion operators are covered in the section on
<a href="#Implicit_Conversions">implicit conversions</a>.
The <code>=</code> operator is covered in the section on
<a href="#Copy_Constructors">copy constructors</a>. Overloading
<code>&lt;&lt;</code> for use with streams is covered in the
section on <a href="#Streams">streams</a>. See also the rules on
<a href="#Function_Overloading">function overloading</a>, which
apply to operator overloading as well.</p>
</div>

</div> 

<h3 id="Access_Control">Access Control</h3>

<div class="summary">
<p> Make data members <code>private</code>, unless they are
<code>static const</code> (and follow the <a href="#Constant_Names">
naming convention for constants</a>). For technical
reasons, we allow data members of a test fixture class to
be <code>protected</code> when using


<a href="https://github.com/google/googletest">Google
Test</a>).</p>
</div>

<h3 id="Declaration_Order">Declaration Order</h3>

<div class="summary">
<p>Group similar declarations together, placing public parts
earlier.</p>
</div>

<div class="stylebody">

<p>A class definition should usually start with a
<code>public:</code> section, followed by
<code>protected:</code>, then <code>private:</code>.  Omit
sections that would be empty.</p>

<p>Within each section, generally prefer grouping similar
kinds of declarations together, and generally prefer the
following order: types (including <code>typedef</code>,
<code>using</code>, and nested structs and classes),
constants, factory functions, constructors, assignment
operators, destructor, all other methods, data members.</p>

<p>Do not put large method definitions inline in the
class definition. Usually, only trivial or
performance-critical, and very short, methods may be
defined inline. See <a href="#Inline_Functions">Inline
Functions</a> for more details.</p>

</div> 

<h2 id="Functions">Functions</h2>

<h3 id="Function_Parameter_Ordering">Parameter Ordering</h3>

<div class="summary">
<p>When defining a function, parameter order is: inputs, then
outputs.</p>
</div>

<div class="stylebody">
<p>Parameters to C/C++ functions are either input to the
function, output from the function, or both. Input
parameters are usually values or <code>const</code>
references, while output and input/output parameters will
be pointers to non-<code>const</code>. When ordering
function parameters, put all input-only parameters before
any output parameters. In particular, do not add new
parameters to the end of the function just because they
are new; place new input-only parameters before the
output parameters.</p>

<p>This is not a hard-and-fast rule. Parameters that are
both input and output (often classes/structs) muddy the
waters, and, as always, consistency with related
functions may require you to bend the rule.</p>

</div> 

<h3 id="Write_Short_Functions">Write Short Functions</h3>

<div class="summary">
<p>Prefer small and focused functions.</p>
</div>

<div class="stylebody">
<p>We recognize that long functions are sometimes
appropriate, so no hard limit is placed on functions
length. If a function exceeds about 40 lines, think about
whether it can be broken up without harming the structure
of the program.</p>

<p>Even if your long function works perfectly now,
someone modifying it in a few months may add new
behavior. This could result in bugs that are hard to
find. Keeping your functions short and simple makes it
easier for other people to read and modify your code.</p>

<p>You could find long and complicated functions when
working with 
some code. Do not be
intimidated by modifying existing code: if working with
such a function proves to be difficult, you find that
errors are hard to debug, or you want to use a piece of
it in several different contexts, consider breaking up
the function into smaller and more manageable pieces.</p>

</div> 

<h3 id="Reference_Arguments">Reference Arguments</h3>

<div class="summary">
<p>All parameters passed by reference must be labeled
<code>const</code>.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>In C, if a
function needs to modify a variable, the parameter must
use a pointer, eg <code>int foo(int *pval)</code>. In
C++, the function can alternatively declare a reference
parameter: <code>int foo(int &amp;val)</code>.</p>
</div>

<div class="pros">
<p>Defining a parameter as reference avoids ugly code like
<code>(*pval)++</code>. Necessary for some applications
like copy constructors. Makes it clear, unlike with
pointers, that a null pointer is not a possible
value.</p>
</div>

<div class="cons">
<p>References can be confusing, as they have value syntax
but pointer semantics.</p>
</div>

<div class="decision">
<p>Within function parameter lists all references must be
<code>const</code>:</p>

<pre>void Foo(const string &amp;in, string *out);
</pre>

<p>In fact it is a very strong convention in Google code
that input arguments are values or <code>const</code>
references while output arguments are pointers. Input
parameters may be <code>const</code> pointers, but we
never allow non-<code>const</code> reference parameters
except when required by convention, e.g.,
<code>swap()</code>.</p>

<p>However, there are some instances where using
<code>const T*</code> is preferable to <code>const
T&amp;</code> for input parameters. For example:</p>

<ul>
  <li>You want to pass in a null pointer.</li>

  <li>The function saves a pointer or reference to the
  input.</li>
</ul>

<p> Remember that most of the time input
parameters are going to be specified as <code>const
T&amp;</code>. Using <code>const T*</code> instead
communicates to the reader that the input is somehow
treated differently. So if you choose <code>const
T*</code> rather than <code>const T&amp;</code>, do so
for a concrete reason; otherwise it will likely confuse
readers by making them look for an explanation that
doesn't exist.</p>
</div>

</div> 

<h3 id="Function_Overloading">Function Overloading</h3>

<div class="summary">
<p>Use overloaded functions (including constructors) only if a
reader looking at a call site can get a good idea of what
is happening without having to first figure out exactly
which overload is being called.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>You may write a function that takes a <code>const
string&amp;</code> and overload it with another that
takes <code>const char*</code>.</p>

<pre>class MyClass {
 public:
  void Analyze(const string &amp;text);
  void Analyze(const char *text, size_t textlen);
};
</pre>
</div>

<div class="pros">
<p>Overloading can make code more intuitive by allowing an
identically-named function to take different arguments.
It may be necessary for templatized code, and it can be
convenient for Visitors.</p>
</div>

<div class="cons">
<p>If a function is overloaded by the argument types alone,
a reader may have to understand C++'s complex matching
rules in order to tell what's going on. Also many people
are confused by the semantics of inheritance if a derived
class overrides only some of the variants of a
function.</p>
</div>

<div class="decision">
<p>If you want to overload a function, consider qualifying
the name with some information about the arguments, e.g.,
<code>AppendString()</code>, <code>AppendInt()</code>
rather than just <code>Append()</code>. If you are
overloading a function to support variable number of
arguments of the same type, consider making it take a
<code>std::vector</code> so that the user can use an
<a href="#Braced_Initializer_List">initializer list
</a> to specify the arguments.</p>
</div>

</div> 

<h3 id="Default_Arguments">Default Arguments</h3>

<div class="summary">
<p>Default arguments are allowed on non-virtual functions
when the default is guaranteed to always have the same
value. Follow the same restrictions as for <a href="#Function_Overloading">function overloading</a>, and
prefer overloaded functions if the readability gained with
default arguments doesn't outweigh the downsides below.</p>
</div>

<div class="stylebody">

<div class="pros">
<p>Often you have a function that uses default values, but
occasionally you want to override the defaults. Default
parameters allow an easy way to do this without having to
define many functions for the rare exceptions. Compared
to overloading the function, default arguments have a
cleaner syntax, with less boilerplate and a clearer
distinction between 'required' and 'optional'
arguments.</p>
</div>

<div class="cons">
<p>Defaulted arguments are another way to achieve the
semantics of overloaded functions, so all the <a href="#Function_Overloading">reasons not to overload
functions</a> apply.</p>

<p>The defaults for arguments in a virtual function call are
determined by the static type of the target object, and
there's no guarantee that all overrides of a given function
declare the same defaults.</p>

<p>Default parameters are re-evaluated at each call site,
which can bloat the generated code. Readers may also expect
the default's value to be fixed at the declaration instead
of varying at each call.</p>

<p>Function pointers are confusing in the presence of
default arguments, since the function signature often
doesn't match the call signature. Adding
function overloads avoids these problems.</p>
</div>

<div class="decision">
<p>Default arguments are banned on virtual functions, where
they don't work properly, and in cases where the specified
default might not evaluate to the same value depending on
when it was evaluated. (For example, don't write <code>void
f(int n = counter++);</code>.)</p>

<p>In some other cases, default arguments can improve the
readability of their function declarations enough to
overcome the downsides above, so they are allowed. When in
doubt, use overloads.</p>
</div>

</div> 

<h3 id="trailing_return">Trailing Return Type Syntax</h3>
<div class="summary">
<p>Use trailing return types only where using the ordinary syntax (leading
  return types) is impractical or much less readable.</p>
</div>

<div class="definition">
<p>C++ allows two different forms of function declarations. In the older
  form, the return type appears before the function name. For example:</p>
<pre>int foo(int x);
</pre>
<p>The new form, introduced in C++11, uses the <code>auto</code>
  keyword before the function name and a trailing return type after
  the argument list. For example, the declaration above could
  equivalently be written:</p>
<pre>auto foo(int x) -&gt; int;
</pre>
<p>The trailing return type is in the function's scope. This doesn't
  make a difference for a simple case like <code>int</code> but it matters
  for more complicated cases, like types declared in class scope or
  types written in terms of the function parameters.</p>
</div>

<div class="stylebody">
<div class="pros">
<p>Trailing return types are the only way to explicitly specify the
  return type of a <a href="#Lambda_expressions">lambda expression</a>.
  In some cases the compiler is able to deduce a lambda's return type,
  but not in all cases. Even when the compiler can deduce it automatically,
  sometimes specifying it explicitly would be clearer for readers.
</p>
<p>Sometimes it's easier and more readable to specify a return type
  after the function's parameter list has already appeared. This is
  particularly true when the return type depends on template parameters.
  For example:</p>
  <pre>template &lt;class T, class U&gt; auto add(T t, U u) -&gt; decltype(t + u);</pre>
  versus
  <pre>template &lt;class T, class U&gt; decltype(declval&lt;T&amp;&gt;() + declval&lt;U&amp;&gt;()) add(T t, U u);</pre>
</div>

<div class="cons">
<p>Trailing return type syntax is relatively new and it has no
  analogue in C++-like languages like C and Java, so some readers may
  find it unfamiliar.</p>
<p>Existing code bases have an enormous number of function
  declarations that aren't going to get changed to use the new syntax,
  so the realistic choices are using the old syntax only or using a mixture
  of the two. Using a single version is better for uniformity of style.</p>
</div>

<div class="decision">
<p>In most cases, continue to use the older style of function
  declaration where the return type goes before the function name.
  Use the new trailing-return-type form only in cases where it's
  required (such as lambdas) or where, by putting the type after the
  function's parameter list, it allows you to write the type in a much
  more readable way. The latter case should be rare; it's mostly an
  issue in fairly complicated template code, which is
  <a href="#Template_metaprogramming">discouraged in most cases</a>.</p>

</div> 
</div> 

<h2 id="Google-Specific_Magic">Google-Specific Magic</h2>



<p>There are various tricks and utilities that
we use to make C++ code more robust, and various ways we use
C++ that may differ from what you see elsewhere.</p>

 

<h3 id="Ownership_and_Smart_Pointers">Ownership and Smart Pointers</h3>

<div class="summary">
<p>Prefer to have single, fixed owners for dynamically
allocated objects. Prefer to transfer ownership with smart
pointers.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>"Ownership" is a bookkeeping technique for managing
dynamically allocated memory (and other resources). The
owner of a dynamically allocated object is an object or
function that is responsible for ensuring that it is
deleted when no longer needed. Ownership can sometimes be
shared, in which case the last owner is typically
responsible for deleting it. Even when ownership is not
shared, it can be transferred from one piece of code to
another.</p>

<p>"Smart" pointers are classes that act like pointers,
e.g. by overloading the <code>*</code> and
<code>-&gt;</code> operators. Some smart pointer types
can be used to automate ownership bookkeeping, to ensure
these responsibilities are met.
<a href="http://en.cppreference.com/w/cpp/memory/unique_ptr">
<code>std::unique_ptr</code></a> is a smart pointer type
introduced in C++11, which expresses exclusive ownership
of a dynamically allocated object; the object is deleted
when the <code>std::unique_ptr</code> goes out of scope.
It cannot be copied, but can be <em>moved</em> to
represent ownership transfer.
<a href="http://en.cppreference.com/w/cpp/memory/shared_ptr">
<code>std::shared_ptr</code></a> is a smart pointer type
that expresses shared ownership of
a dynamically allocated object. <code>std::shared_ptr</code>s
can be copied; ownership of the object is shared among
all copies, and the object is deleted when the last
<code>std::shared_ptr</code> is destroyed. </p>
</div>

<div class="pros">
<ul>
  <li>It's virtually impossible to manage dynamically
  allocated memory without some sort of ownership
  logic.</li>

  <li>Transferring ownership of an object can be cheaper
  than copying it (if copying it is even possible).</li>

  <li>Transferring ownership can be simpler than
  'borrowing' a pointer or reference, because it reduces
  the need to coordinate the lifetime of the object
  between the two users.</li>

  <li>Smart pointers can improve readability by making
  ownership logic explicit, self-documenting, and
  unambiguous.</li>

  <li>Smart pointers can eliminate manual ownership
  bookkeeping, simplifying the code and ruling out large
  classes of errors.</li>

  <li>For const objects, shared ownership can be a simple
  and efficient alternative to deep copying.</li>
</ul>
</div>

<div class="cons">
<ul>
  <li>Ownership must be represented and transferred via
  pointers (whether smart or plain). Pointer semantics
  are more complicated than value semantics, especially
  in APIs: you have to worry not just about ownership,
  but also aliasing, lifetime, and mutability, among
  other issues.</li>

  <li>The performance costs of value semantics are often
  overestimated, so the performance benefits of ownership
  transfer might not justify the readability and
  complexity costs.</li>

  <li>APIs that transfer ownership force their clients
  into a single memory management model.</li>

  <li>Code using smart pointers is less explicit about
  where the resource releases take place.</li>

  <li><code>std::unique_ptr</code> expresses ownership
  transfer using C++11's move semantics, which are
  relatively new and may confuse some programmers.</li>

  <li>Shared ownership can be a tempting alternative to
  careful ownership design, obfuscating the design of a
  system.</li>

  <li>Shared ownership requires explicit bookkeeping at
  run-time, which can be costly.</li>

  <li>In some cases (e.g. cyclic references), objects
  with shared ownership may never be deleted.</li>

  <li>Smart pointers are not perfect substitutes for
  plain pointers.</li>
</ul>
</div>

<div class="decision">
<p>If dynamic allocation is necessary, prefer to keep
ownership with the code that allocated it. If other code
needs access to the object, consider passing it a copy,
or passing a pointer or reference without transferring
ownership. Prefer to use <code>std::unique_ptr</code> to
make ownership transfer explicit. For example:</p>

<pre>std::unique_ptr&lt;Foo&gt; FooFactory();
void FooConsumer(std::unique_ptr&lt;Foo&gt; ptr);
</pre>



<p>Do not design your code to use shared ownership
without a very good reason. One such reason is to avoid
expensive copy operations, but you should only do this if
the performance benefits are significant, and the
underlying object is immutable (i.e.
<code>std::shared_ptr&lt;const Foo&gt;</code>).  If you
do use shared ownership, prefer to use
<code>std::shared_ptr</code>.</p>

<p>Never use <code>std::auto_ptr</code>. Instead, use
<code>std::unique_ptr</code>.</p>
</div> 

</div> 

<h3 id="cpplint">cpplint</h3>

<div class="summary">
<p>Use <code>cpplint.py</code>
to detect style errors.</p>
</div>

<div class="stylebody">

<p><code>cpplint.py</code>
is a tool that reads a source file and identifies many
style errors. It is not perfect, and has both false
positives and false negatives, but it is still a valuable
tool. False positives can be ignored by putting <code>//
NOLINT</code> at the end of the line or
<code>// NOLINTNEXTLINE</code> in the previous line.</p>



<p>Some projects have instructions on
how to run <code>cpplint.py</code> from their project
tools. If the project you are contributing to does not,
you can download
<a href="https://raw.githubusercontent.com/google/styleguide/gh-pages/cpplint/cpplint.py">
<code>cpplint.py</code></a> separately.</p>

</div> 

 

<h2 id="Other_C++_Features">Other C++ Features</h2>

<h3 id="Rvalue_references">Rvalue References</h3>

<div class="summary">
<p>Use rvalue references only to define move constructors and move assignment
operators, or for perfect forwarding.
</p>
</div>

<div class="stylebody">

<div class="definition">
<p> Rvalue references
are a type of reference that can only bind to temporary
objects. The syntax is similar to traditional reference
syntax. For example, <code>void f(string&amp;&amp;
s);</code> declares a function whose argument is an
rvalue reference to a string.</p>
</div>

<div class="pros">
<ul>
  <li>Defining a move constructor (a constructor taking
  an rvalue reference to the class type) makes it
  possible to move a value instead of copying it. If
  <code>v1</code> is a <code>std::vector&lt;string&gt;</code>,
  for example, then <code>auto v2(std::move(v1))</code>
  will probably just result in some simple pointer
  manipulation instead of copying a large amount of data.
  In some cases this can result in a major performance
  improvement.</li>

  <li>Rvalue references make it possible to write a
  generic function wrapper that forwards its arguments to
  another function, and works whether or not its
  arguments are temporary objects. (This is sometimes called
  "perfect forwarding".)</li>

  <li>Rvalue references make it possible to implement
  types that are movable but not copyable, which can be
  useful for types that have no sensible definition of
  copying but where you might still want to pass them as
  function arguments, put them in containers, etc.</li>

  <li><code>std::move</code> is necessary to make
  effective use of some standard-library types, such as
  <code>std::unique_ptr</code>.</li>
</ul>
</div>

<div class="cons">
<ul>
  <li>Rvalue references are a relatively new feature
  (introduced as part of C++11), and not yet widely
  understood. Rules like reference collapsing, and
  automatic synthesis of move constructors, are
  complicated.</li>
</ul>
</div>

<div class="decision">
  <p>Use rvalue references only to define move constructors and move assignment
  operators (as described in <a href="#Copyable_Movable_Types">Copyable and
  Movable Types</a>) and, in conjunction with <code><a href="http://en.cppreference.com/w/cpp/utility/forward">std::forward</a></code>,
to support perfect forwarding.  You may use <code>std::move</code> to express
moving a value from one object to another rather than copying it. </p>
</div>

</div> 

<h3 id="Friends">Friends</h3>

<div class="summary">
<p>We allow use of <code>friend</code> classes and functions,
within reason.</p>
</div>

<div class="stylebody">

<p>Friends should usually be defined in the same file so
that the reader does not have to look in another file to
find uses of the private members of a class. A common use
of <code>friend</code> is to have a
<code>FooBuilder</code> class be a friend of
<code>Foo</code> so that it can construct the inner state
of <code>Foo</code> correctly, without exposing this
state to the world. In some cases it may be useful to
make a unittest class a friend of the class it tests.</p>

<p>Friends extend, but do not break, the encapsulation
boundary of a class. In some cases this is better than
making a member public when you want to give only one
other class access to it. However, most classes should
interact with other classes solely through their public
members.</p>

</div> 

<h3 id="Exceptions">Exceptions</h3>

<div class="summary">
<p>We do not use C++ exceptions.</p>
</div>

<div class="stylebody">

<div class="pros">
<ul>
  <li>Exceptions allow higher levels of an application to
  decide how to handle "can't happen" failures in deeply
  nested functions, without the obscuring and error-prone
  bookkeeping of error codes.</li>

  

  <li>Exceptions are used by most other
  modern languages. Using them in C++ would make it more
  consistent with Python, Java, and the C++ that others
  are familiar with.</li>

  <li>Some third-party C++ libraries use exceptions, and
  turning them off internally makes it harder to
  integrate with those libraries.</li>

  <li>Exceptions are the only way for a constructor to
  fail. We can simulate this with a factory function or
  an <code>Init()</code> method, but these require heap
  allocation or a new "invalid" state, respectively.</li>

  <li>Exceptions are really handy in testing
  frameworks.</li>
</ul>
</div>

<div class="cons">
<ul>
  <li>When you add a <code>throw</code> statement to an
  existing function, you must examine all of its
  transitive callers. Either they must make at least the
  basic exception safety guarantee, or they must never
  catch the exception and be happy with the program
  terminating as a result. For instance, if
  <code>f()</code> calls <code>g()</code> calls
  <code>h()</code>, and <code>h</code> throws an
  exception that <code>f</code> catches, <code>g</code>
  has to be careful or it may not clean up properly.</li>

  <li>More generally, exceptions make the control flow of
  programs difficult to evaluate by looking at code:
  functions may return in places you don't expect. This
  causes maintainability and debugging difficulties. You
  can minimize this cost via some rules on how and where
  exceptions can be used, but at the cost of more that a
  developer needs to know and understand.</li>

  <li>Exception safety requires both RAII and different
  coding practices. Lots of supporting machinery is
  needed to make writing correct exception-safe code
  easy. Further, to avoid requiring readers to understand
  the entire call graph, exception-safe code must isolate
  logic that writes to persistent state into a "commit"
  phase. This will have both benefits and costs (perhaps
  where you're forced to obfuscate code to isolate the
  commit). Allowing exceptions would force us to always
  pay those costs even when they're not worth it.</li>

  <li>Turning on exceptions adds data to each binary
  produced, increasing compile time (probably slightly)
  and possibly increasing address space pressure.
  </li>

  <li>The availability of exceptions may encourage
  developers to throw them when they are not appropriate
  or recover from them when it's not safe to do so. For
  example, invalid user input should not cause exceptions
  to be thrown. We would need to make the style guide
  even longer to document these restrictions!</li>
</ul>
</div>

<div class="decision">
<p>On their face, the benefits of using exceptions
outweigh the costs, especially in new projects. However,
for existing code, the introduction of exceptions has
implications on all dependent code. If exceptions can be
propagated beyond a new project, it also becomes
problematic to integrate the new project into existing
exception-free code. Because most existing C++ code at
Google is not prepared to deal with exceptions, it is
comparatively difficult to adopt new code that generates
exceptions.</p>

<p>Given that Google's existing code is not
exception-tolerant, the costs of using exceptions are
somewhat greater than the costs in a new project. The
conversion process would be slow and error-prone. We
don't believe that the available alternatives to
exceptions, such as error codes and assertions, introduce
a significant burden. </p>

<p>Our advice against using exceptions is not predicated
on philosophical or moral grounds, but practical ones.
 Because we'd like to use our open-source
projects at Google and it's difficult to do so if those
projects use exceptions, we need to advise against
exceptions in Google open-source projects as well.
Things would probably be different if we had to do it all
over again from scratch.</p>

<p>This prohibition also applies to the exception-related
features added in C++11, such as <code>noexcept</code>,
<code>std::exception_ptr</code>, and
<code>std::nested_exception</code>.</p>

<p>There is an <a href="#Windows_Code">exception</a> to
this rule (no pun intended) for Windows code.</p>
</div>

</div> 

<h3 id="Run-Time_Type_Information__RTTI_">Run-Time Type
Information (RTTI)</h3>

<div class="summary">
<p>Avoid using Run Time Type Information (RTTI).</p>
</div>

<div class="stylebody">

<div class="definition">
<p> RTTI allows a
programmer to query the C++ class of an object at run
time. This is done by use of <code>typeid</code> or
<code>dynamic_cast</code>.</p>
</div>

<div class="cons">
<p>Querying the type of an object at run-time frequently
means a design problem. Needing to know the type of an
object at runtime is often an indication that the design
of your class hierarchy is flawed.</p>

<p>Undisciplined use of RTTI makes code hard to maintain.
It can lead to type-based decision trees or switch
statements scattered throughout the code, all of which
must be examined when making further changes.</p>
</div>

<div class="pros">
<p>The standard alternatives to RTTI (described below)
require modification or redesign of the class hierarchy
in question. Sometimes such modifications are infeasible
or undesirable, particularly in widely-used or mature
code.</p>

<p>RTTI can be useful in some unit tests. For example, it
is useful in tests of factory classes where the test has
to verify that a newly created object has the expected
dynamic type. It is also useful in managing the
relationship between objects and their mocks.</p>

<p>RTTI is useful when considering multiple abstract
objects. Consider</p>

<pre>bool Base::Equal(Base* other) = 0;
bool Derived::Equal(Base* other) {
  Derived* that = dynamic_cast&lt;Derived*&gt;(other);
  if (that == NULL)
    return false;
  ...
}
</pre>
</div>

<div class="decision">
<p>RTTI has legitimate uses but is prone to abuse, so you
must be careful when using it. You may use it freely in
unittests, but avoid it when possible in other code. In
particular, think twice before using RTTI in new code. If
you find yourself needing to write code that behaves
differently based on the class of an object, consider one
of the following alternatives to querying the type:</p>

<ul>
  <li>Virtual methods are the preferred way of executing
  different code paths depending on a specific subclass
  type. This puts the work within the object itself.</li>

  <li>If the work belongs outside the object and instead
  in some processing code, consider a double-dispatch
  solution, such as the Visitor design pattern. This
  allows a facility outside the object itself to
  determine the type of class using the built-in type
  system.</li>
</ul>

<p>When the logic of a program guarantees that a given
instance of a base class is in fact an instance of a
particular derived class, then a
<code>dynamic_cast</code> may be used freely on the
object.  Usually one
can use a <code>static_cast</code> as an alternative in
such situations.</p>

<p>Decision trees based on type are a strong indication
that your code is on the wrong track.</p>

<pre class="badcode">if (typeid(*data) == typeid(D1)) {
  ...
} else if (typeid(*data) == typeid(D2)) {
  ...
} else if (typeid(*data) == typeid(D3)) {
...
</pre>

<p>Code such as this usually breaks when additional
subclasses are added to the class hierarchy. Moreover,
when properties of a subclass change, it is difficult to
find and modify all the affected code segments.</p>

<p>Do not hand-implement an RTTI-like workaround. The
arguments against RTTI apply just as much to workarounds
like class hierarchies with type tags. Moreover,
workarounds disguise your true intent.</p>
</div>

</div> 

<h3 id="Casting">Casting</h3>

<div class="summary">
<p>Use C++-style casts
like <code>static_cast&lt;float&gt;(double_value)</code>, or brace
initialization for conversion of arithmetic types like
<code>int64 y = int64{1} &lt;&lt; 42</code>. Do not use
cast formats like
<code>int y = (int)x</code> or <code>int y = int(x)</code> (but the latter
is okay when invoking a constructor of a class type).</p>
</div>

<div class="stylebody">

<div class="definition">
<p> C++ introduced a
different cast system from C that distinguishes the types
of cast operations.</p>
</div>

<div class="pros">
<p>The problem with C casts is the ambiguity of the operation;
sometimes you are doing a <em>conversion</em>
(e.g., <code>(int)3.5</code>) and sometimes you are doing
a <em>cast</em> (e.g., <code>(int)"hello"</code>). Brace
initialization and C++ casts can often help avoid this
ambiguity. Additionally, C++ casts are more visible when searching for
them.</p>
</div>

<div class="cons">
<p>The C++-style cast syntax is verbose and cumbersome.</p>
</div>

<div class="decision">
<p>Do not use C-style casts. Instead, use these C++-style casts when
explicit type conversion is necessary. </p>

<ul>
  <li>Use brace initialization to convert arithmetic types
  (e.g. <code>int64{x}</code>).  This is the safest approach because code
  will not compile if conversion can result in information loss.  The
  syntax is also concise.</li>

  

  <li>Use <code>static_cast</code> as the equivalent of a C-style cast
  that does value conversion, when you need to
  explicitly up-cast a pointer from a class to its superclass, or when
  you need to explicitly cast a pointer from a superclass to a
  subclass.  In this last case, you must be sure your object is
  actually an instance of the subclass.</li>

   

  <li>Use <code>const_cast</code> to remove the
  <code>const</code> qualifier (see <a href="#Use_of_const">const</a>).</li>

  <li>Use <code>reinterpret_cast</code> to do unsafe
  conversions of pointer types to and from integer and
  other pointer types. Use this only if you know what you
  are doing and you understand the aliasing issues.
  </li>

  
</ul>

<p>See the <a href="#Run-Time_Type_Information__RTTI_">
RTTI section</a> for guidance on the use of
<code>dynamic_cast</code>.</p>
</div>

</div> 

<h3 id="Streams">Streams</h3>

<div class="summary">
<p>Use streams where appropriate, and stick to "simple"
usages.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>Streams are the standard I/O abstraction in C++, as
exemplified by the standard header <code>&lt;iostream&gt;</code>.
They are widely used in Google code, but only for debug logging
and test diagnostics.</p>
</div>

<div class="pros">
<p>The <code>&lt;&lt;</code> and <code>&gt;&gt;</code>
stream operators provide an API for formatted I/O that
is easily learned, portable, reusable, and extensible.
<code>printf</code>, by contrast, doesn't even support
<code>string</code>, to say nothing of user-defined types,
and is very difficult to use portably.
<code>printf</code> also obliges you to choose among the
numerous slightly different versions of that function,
and navigate the dozens of conversion specifiers.</p>

<p>Streams provide first-class support for console I/O
via <code>std::cin</code>, <code>std::cout</code>,
<code>std::cerr</code>, and <code>std::clog</code>.
The C APIs do as well, but are hampered by the need to
manually buffer the input. </p>
</div>

<div class="cons">
<ul>
<li>Stream formatting can be configured by mutating the
state of the stream. Such mutations are persistent, so
the behavior of your code can be affected by the entire
previous history of the stream, unless you go out of your
way to restore it to a known state every time other code
might have touched it. User code can not only modify the
built-in state, it can add new state variables and behaviors
through a registration system.</li>

<li>It is difficult to precisely control stream output, due
to the above issues, the way code and data are mixed in
streaming code, and the use of operator overloading (which
may select a different overload than you expect).</li>

<li>The practice of building up output through chains
of <code>&lt;&lt;</code> operators interferes with
internationalization, because it bakes word order into the
code, and streams' support for localization is <a href="http://www.boost.org/doc/libs/1_48_0/libs/locale/doc/html/rationale.html#rationale_why">
flawed</a>.</li>





<li>The streams API is subtle and complex, so programmers must
develop experience with it in order to use it effectively.
However, streams were historically banned in Google code (except
for logging and diagnostics), so Google engineers tend not to
have that experience. Consequently, streams-based code is likely
to be less readable and maintainable by Googlers than code based
on more familiar abstractions.</li>

<li>Resolving the many overloads of <code>&lt;&lt;</code> is
extremely costly for the compiler. When used pervasively in a
large code base, it can consume as much as 20% of the parsing
and semantic analysis time.</li>
</ul>
</div>

<div class="decision">
<p>Use streams only when they are the best tool for the job.
This is typically the case when the I/O is ad-hoc, local,
human-readable, and targeted at other developers rather than
end-users. Be consistent with the code around you, and with the
codebase as a whole; if there's an established tool for
your problem, use that tool instead. </p>

<p>Avoid using streams for I/O that faces external users or
handles untrusted data. Instead, find and use the appropriate
templating libraries to handle issues like internationalization,
localization, and security hardening.</p>

<p>If you do use streams, avoid the stateful parts of the
streams API (other than error state), such as <code>imbue()</code>,
<code>xalloc()</code>, and <code>register_callback()</code>.
Use explicit formatting functions  rather than
stream manipulators or formatting flags to control formatting
details such as number base, precision, or padding.</p>

<p>Overload <code>&lt;&lt;</code> as a streaming operator
for your type only if your type represents a value, and
<code>&lt;&lt;</code> writes out a human-readable string
representation of that value. Avoid exposing implementation
details in the output of <code>&lt;&lt;</code>; if you need to print
object internals for debugging, use named functions instead
(a method named <code>DebugString()</code> is the most common
convention).</p>
</div>

</div> 

<h3 id="Preincrement_and_Predecrement">Preincrement and Predecrement</h3>

<div class="summary">
<p>Use prefix form (<code>++i</code>) of the increment and
decrement operators with iterators and other template
objects.</p>
</div>

<div class="stylebody">

<div class="definition">
<p> When a variable
is incremented (<code>++i</code> or <code>i++</code>) or
decremented (<code>--i</code> or <code>i--</code>) and
the value of the expression is not used, one must decide
whether to preincrement (decrement) or postincrement
(decrement).</p>
</div>

<div class="pros">
<p>When the return value is ignored, the "pre" form
(<code>++i</code>) is never less efficient than the
"post" form (<code>i++</code>), and is often more
efficient. This is because post-increment (or decrement)
requires a copy of <code>i</code> to be made, which is
the value of the expression. If <code>i</code> is an
iterator or other non-scalar type, copying <code>i</code>
could be expensive. Since the two types of increment
behave the same when the value is ignored, why not just
always pre-increment?</p>
</div>

<div class="cons">
<p>The tradition developed, in C, of using post-increment
when the expression value is not used, especially in
<code>for</code> loops. Some find post-increment easier
to read, since the "subject" (<code>i</code>) precedes
the "verb" (<code>++</code>), just like in English.</p>
</div>

<div class="decision">
<p> For simple scalar
(non-object) values there is no reason to prefer one form
and we allow either. For iterators and other template
types, use pre-increment.</p>
</div>

</div> 

<h3 id="Use_of_const">Use of const</h3>

<div class="summary">
<p>Use <code>const</code> whenever it makes sense. With C++11,
<code>constexpr</code> is a better choice for some uses of
const.</p>
</div>

<div class="stylebody">

<div class="definition">
<p> Declared variables and parameters can be preceded
by the keyword <code>const</code> to indicate the variables
are not changed (e.g., <code>const int foo</code>). Class
functions can have the <code>const</code> qualifier to
indicate the function does not change the state of the
class member variables (e.g., <code>class Foo { int
Bar(char c) const; };</code>).</p>
</div>

<div class="pros">
<p>Easier for people to understand how variables are being
used. Allows the compiler to do better type checking,
and, conceivably, generate better code. Helps people
convince themselves of program correctness because they
know the functions they call are limited in how they can
modify your variables. Helps people know what functions
are safe to use without locks in multi-threaded
programs.</p>
</div>

<div class="cons">
<p><code>const</code> is viral: if you pass a
<code>const</code> variable to a function, that function
must have <code>const</code> in its prototype (or the
variable will need a <code>const_cast</code>). This can
be a particular problem when calling library
functions.</p>
</div>

<div class="decision">
<p><code>const</code> variables, data members, methods
and arguments add a level of compile-time type checking;
it is better to detect errors as soon as possible.
Therefore we strongly recommend that you use
<code>const</code> whenever it makes sense to do so:</p>

<ul>
  <li>If a function guarantees that it will not modify an argument
  passed by reference or by pointer, the corresponding function parameter
  should be a reference-to-const (<code>const T&amp;</code>) or
  pointer-to-const (<code>const T*</code>), respectively.</li>

  <li>Declare methods to be <code>const</code> whenever
  possible. Accessors should almost always be
  <code>const</code>. Other methods should be const if
  they do not modify any data members, do not call any
  non-<code>const</code> methods, and do not return a
  non-<code>const</code> pointer or
  non-<code>const</code> reference to a data member.</li>

  <li>Consider making data members <code>const</code>
  whenever they do not need to be modified after
  construction.</li>
</ul>

<p>The <code>mutable</code> keyword is allowed but is
unsafe when used with threads, so thread safety should be
carefully considered first.</p>
</div>

<div class="stylepoint_subsection">
<h4>Where to put the const</h4>

<p>Some people favor the form <code>int const *foo</code>
to <code>const int* foo</code>. They argue that this is
more readable because it's more consistent: it keeps the
rule that <code>const</code> always follows the object
it's describing. However, this consistency argument
doesn't apply in codebases with few deeply-nested pointer
expressions since most <code>const</code> expressions
have only one <code>const</code>, and it applies to the
underlying value. In such cases, there's no consistency
to maintain. Putting the <code>const</code> first is
arguably more readable, since it follows English in
putting the "adjective" (<code>const</code>) before the
"noun" (<code>int</code>).</p>

<p>That said, while we encourage putting
<code>const</code> first, we do not require it. But be
consistent with the code around you!</p>
</div>

</div> 

<h3 id="Use_of_constexpr">Use of constexpr</h3>

<div class="summary">
<p>In C++11, use <code>constexpr</code> to define true
constants or to ensure constant initialization.</p>
</div>

<div class="stylebody">

<div class="definition">
<p> Some variables can be declared <code>constexpr</code>
to indicate the variables are true constants, i.e. fixed at
compilation/link time. Some functions and constructors
can be declared <code>constexpr</code> which enables them
to be used in defining a <code>constexpr</code>
variable.</p>
</div>

<div class="pros">
<p>Use of <code>constexpr</code> enables definition of
constants with floating-point expressions rather than
just literals; definition of constants of user-defined
types; and definition of constants with function
calls.</p>
</div>

<div class="cons">
<p>Prematurely marking something as constexpr may cause
migration problems if later on it has to be downgraded.
Current restrictions on what is allowed in constexpr
functions and constructors may invite obscure workarounds
in these definitions.</p>
</div>

<div class="decision">
<p><code>constexpr</code> definitions enable a more
robust specification of the constant parts of an
interface. Use <code>constexpr</code> to specify true
constants and the functions that support their
definitions. Avoid complexifying function definitions to
enable their use with <code>constexpr</code>. Do not use
<code>constexpr</code> to force inlining.</p>
</div>

</div> 

<h3 id="Integer_Types">Integer Types</h3>

<div class="summary">
<p>Of the built-in C++ integer types, the only one used
 is
<code>int</code>. If a program needs a variable of a
different size, use 
a precise-width integer type from
<code>&lt;stdint.h&gt;</code>, such as
<code>int16_t</code>. If your variable represents a
value that could ever be greater than or equal to 2^31
(2GiB), use a 64-bit type such as
<code>int64_t</code>.
Keep in mind that even if your value won't ever be too large
for an <code>int</code>, it may be used in intermediate
calculations which may require a larger type. When in doubt,
choose a larger type.</p>
</div>

<div class="stylebody">

<div class="definition">
<p> C++ does not specify the sizes of its integer types.
Typically people assume that <code>short</code> is 16 bits,
<code>int</code> is 32 bits, <code>long</code> is 32 bits
and <code>long long</code> is 64 bits.</p>
</div>

<div class="pros">
<p>Uniformity of declaration.</p>
</div>

<div class="cons">
<p>The sizes of integral types in C++ can vary based on
compiler and architecture.</p>
</div>

<div class="decision">

<p>
<code>&lt;stdint.h&gt;</code> defines types
like <code>int16_t</code>, <code>uint32_t</code>,
<code>int64_t</code>, etc. You should always use
those in preference to <code>short</code>, <code>unsigned
long long</code> and the like, when you need a guarantee
on the size of an integer. Of the C integer types, only
<code>int</code> should be used. When appropriate, you
are welcome to use standard types like
<code>size_t</code> and <code>ptrdiff_t</code>.</p>

<p>We use <code>int</code> very often, for integers we
know are not going to be too big, e.g., loop counters.
Use plain old <code>int</code> for such things. You
should assume that an <code>int</code> is

at least 32 bits, but don't
assume that it has more than 32 bits. If you need a 64-bit
integer type, use
<code>int64_t</code>
or
<code>uint64_t</code>.</p>

<p>For integers we know can be "big",
 use
<code>int64_t</code>.
</p>

<p>You should not use the unsigned integer types such as
<code>uint32_t</code>, unless there is a valid
reason such as representing a bit pattern rather than a
number, or you need defined overflow modulo 2^N. In
particular, do not use unsigned types to say a number
will never be negative. Instead, use 
assertions for this.</p>



<p>If your code is a container that returns a size, be
sure to use a type that will accommodate any possible
usage of your container. When in doubt, use a larger type
rather than a smaller type.</p>

<p>Use care when converting integer types. Integer
conversions and promotions can cause non-intuitive
behavior. </p>
</div>

<div class="stylepoint_subsection">

<h4>On Unsigned Integers</h4>

<p>Some people, including some textbook authors,
recommend using unsigned types to represent numbers that
are never negative. This is intended as a form of
self-documentation. However, in C, the advantages of such
documentation are outweighed by the real bugs it can
introduce. Consider:</p>

<pre>for (unsigned int i = foo.Length()-1; i &gt;= 0; --i) ...
</pre>

<p>This code will never terminate! Sometimes gcc will
notice this bug and warn you, but often it will not.
Equally bad bugs can occur when comparing signed and
unsigned variables. Basically, C's type-promotion scheme
causes unsigned types to behave differently than one
might expect.</p>

<p>So, document that a variable is non-negative using
assertions. Don't use an unsigned
type.</p>
</div>

</div> 

<h3 id="64-bit_Portability">64-bit Portability</h3>

<div class="summary">
<p>Code should be 64-bit and 32-bit friendly. Bear in mind
problems of printing, comparisons, and structure alignment.</p>
</div>

<div class="stylebody">

<ul>
  <li>
  <p><code>printf()</code> specifiers for some types
  are not cleanly portable between 32-bit and 64-bit
  systems. C99 defines some portable format specifiers.
  Unfortunately, MSVC 7.1 does not understand some of
  these specifiers and the standard is missing a few,
  so we 
  have to define our own ugly versions in some cases
   (in the style of the standard include file
  <code>inttypes.h</code>):</p>

  <div>
  <pre>// printf macros for size_t, in the style of inttypes.h
#ifdef _LP64
#define __PRIS_PREFIX "z"
#else
#define __PRIS_PREFIX
#endif

// Use these macros after a % in a printf format string
// to get correct 32/64 bit behavior, like this:
// size_t size = records.size();
// printf("%" PRIuS "\n", size);

#define PRIdS __PRIS_PREFIX "d"
#define PRIxS __PRIS_PREFIX "x"
#define PRIuS __PRIS_PREFIX "u"
#define PRIXS __PRIS_PREFIX "X"
#define PRIoS __PRIS_PREFIX "o"
  </pre>
  </div> 

  <table border="1" summary="portable printf specifiers">
  <tbody><tr align="center">
    <th>Type</th>
    <th>DO NOT use</th>
    <th>DO use</th>
    <th>Notes</th>
  </tr>

  <tr align="center">
    <td><code>void *</code> (or any pointer)</td>
    <td><code>%lx</code></td>
    <td><code>%p</code></td>
    <td></td>
  </tr>

  

  <tr align="center">
    <td><code>int64_t</code></td>
    <td><code>%qd</code>, <code>%lld</code></td>
    <td><code>%" PRId64 "</code></td>
    <td></td>
  </tr>

  

  <tr align="center">
    <td><code>uint64_t</code></td>
    <td><code>%qu</code>, <code>%llu</code>,
                  <code>%llx</code></td>
    <td><code>%" PRIu64 "</code>,
                  <code>%" PRIx64 "</code></td>
    <td></td>
  </tr>

  

  <tr align="center">
    <td><code>size_t</code></td>
    <td><code>%u</code></td>
    <td><code>%" PRIuS "</code>, <code>%" PRIxS "</code></td>
    <td>
    C99 specifies <code>%zu</code></td>
  </tr>

  <tr align="center">
    <td><code>ptrdiff_t</code></td>
    <td><code>%d</code></td>
    <td><code>%" PRIdS "</code></td>
    <td>
    C99 specifies <code>%td</code></td>
  </tr>

  
  </tbody></table>

  <p>Note that the <code>PRI*</code> macros expand to
  independent strings which are concatenated by the
  compiler. Hence if you are using a non-constant
  format string, you need to insert the value of the
  macro into the format, rather than the name. Note also
  that spaces are required around the macro identifier to
  separate it from the string literal. It is
  still possible, as usual, to include length
  specifiers, etc., after the <code>%</code> when using
  the <code>PRI*</code> macros. So, e.g.
  <code>printf("x = %30" PRIuS "\n", x)</code> would
  expand on 32-bit Linux to <code>printf("x = %30" "u"
  "\n", x)</code>, which the compiler will treat as
  <code>printf("x = %30u\n", x)</code>.</p>

  
  </li>

  <li>Remember that <code>sizeof(void *)</code> !=
  <code>sizeof(int)</code>. Use <code>intptr_t</code> if
  you want a pointer-sized integer.</li>

  <li>You may need to be careful with structure
  alignments, particularly for structures being stored on
  disk. Any class/structure with a 
  <code>int64_t</code>/<code>uint64_t</code>
  member will by default end up being 8-byte aligned on a
  64-bit system. If you have such structures being shared
  on disk between 32-bit and 64-bit code, you will need
  to ensure that they are packed the same on both
  architectures. 
  Most compilers offer a way to
  alter structure alignment. For gcc, you can use
  <code>__attribute__((packed))</code>. MSVC offers
  <code>#pragma pack()</code> and
  <code>__declspec(align())</code>.</li>

  <li>
  <p>Use the <code>LL</code> or <code>ULL</code>
  suffixes as needed to create 64-bit constants. For
  example:</p>


<pre>int64_t my_value = 0x123456789LL;
uint64_t my_mask = 3ULL &lt;&lt; 48;
</pre>
  </li>
</ul>

</div> 

<h3 id="Preprocessor_Macros">Preprocessor Macros</h3>

<div class="summary">
<p>Avoid defining macros, especially in headers; prefer
inline functions, enums, and <code>const</code> variables.
Name macros with a project-specific prefix. Do not use
macros to define pieces of a C++ API.</p>
</div>

<div class="stylebody">

<p>Macros mean that the code you see is not the same as
the code the compiler sees. This can introduce unexpected
behavior, especially since macros have global scope.</p>

<p>The problems introduced by macros are especially severe
when they are used to define pieces of a C++ API,
and still more so for public APIs. Every error message from
the compiler when developers incorrectly use that interface
now must explain how the macros formed the interface.
Refactoring and analysis tools have a dramatically harder
time updating the interface. As a consequence, we
specifically disallow using macros in this way.
For example, avoid patterns like:</p>

<pre class="badcode">class WOMBAT_TYPE(Foo) {
  // ...

 public:
  EXPAND_PUBLIC_WOMBAT_API(Foo)

  EXPAND_WOMBAT_COMPARISONS(Foo, ==, &lt;)
};
</pre>

<p>Luckily, macros are not nearly as necessary in C++ as
they are in C. Instead of using a macro to inline
performance-critical code, use an inline function.
Instead of using a macro to store a constant, use a
<code>const</code> variable. Instead of using a macro to
"abbreviate" a long variable name, use a reference.
Instead of using a macro to conditionally compile code
... well, don't do that at all (except, of course, for
the <code>#define</code> guards to prevent double
inclusion of header files). It makes testing much more
difficult.</p>

<p>Macros can do things these other techniques cannot,
and you do see them in the codebase, especially in the
lower-level libraries. And some of their special features
(like stringifying, concatenation, and so forth) are not
available through the language proper. But before using a
macro, consider carefully whether there's a non-macro way
to achieve the same result. If you need to use a macro to
define an interface, contact 
your project leads to request
a waiver of this rule.</p>

<p>The following usage pattern will avoid many problems
with macros; if you use macros, follow it whenever
possible:</p>

<ul>
  <li>Don't define macros in a <code>.h</code> file.</li>

  <li><code>#define</code> macros right before you use
  them, and <code>#undef</code> them right after.</li>

  <li>Do not just <code>#undef</code> an existing macro
  before replacing it with your own; instead, pick a name
  that's likely to be unique.</li>

  <li>Try not to use macros that expand to unbalanced C++
  constructs, or at least document that behavior
  well.</li>

  <li>Prefer not using <code>##</code> to generate
  function/class/variable names.</li>
</ul>

<p>Exporting macros from headers (i.e. defining them in a header
without <code>#undef</code>ing them before the end of the header)
is extremely strongly discouraged. If you do export a macro from a
header, it must have a globally unique name. To achieve this, it
must be named with a prefix consisting of your project's namespace
name (but upper case). </p>

</div> 

<h3 id="0_and_nullptr/NULL">0 and nullptr/NULL</h3>

<div class="summary">
<p>Use <code>0</code> for integers, <code>0.0</code> for
reals, <code>nullptr</code> (or <code>NULL</code>) for
pointers, and <code>'\0'</code> for chars.</p>
</div>

<div class="stylebody">

<p>Use <code>0</code> for integers and <code>0.0</code>
for reals. This is not controversial.</p>

<p> For
pointers (address values), there is a choice between
<code>0</code>, <code>NULL</code>, and
<code>nullptr</code>. For projects that allow C++11
features, use <code>nullptr</code>. For C++03 projects,
we prefer <code>NULL</code> because it looks like a
pointer. In fact, some C++ compilers provide special
definitions of <code>NULL</code> which enable them to
give useful warnings, particularly in situations where
<code>sizeof(NULL)</code> is not equal to
<code>sizeof(0)</code>.</p>

<p>Use <code>'\0'</code> for chars. This is the correct
type and also makes code more readable.</p>

</div> 

<h3 id="sizeof">sizeof</h3>

<div class="summary">
<p>Prefer <code>sizeof(<var>varname</var>)</code> to
<code>sizeof(<var>type</var>)</code>.</p>
</div>

<div class="stylebody">

<p>Use <code>sizeof(<var>varname</var>)</code> when you
take the size of a particular variable.
<code>sizeof(<var>varname</var>)</code> will update
appropriately if someone changes the variable type either
now or later. You may use
<code>sizeof(<var>type</var>)</code> for code unrelated
to any particular variable, such as code that manages an
external or internal data format where a variable of an
appropriate C++ type is not convenient.</p>

<pre>Struct data;
memset(&amp;data, 0, sizeof(data));
</pre>

<pre class="badcode">memset(&amp;data, 0, sizeof(Struct));
</pre>

<pre>if (raw_size &lt; sizeof(int)) {
  LOG(ERROR) &lt;&lt; "compressed record not big enough for count: " &lt;&lt; raw_size;
  return false;
}
</pre>

</div> 

<h3 id="auto">auto</h3>

<div class="summary">
<p>Use <code>auto</code> to avoid type names that are noisy, obvious,
or unimportant - cases where the type doesn't aid in clarity for the
reader. Continue to use manifest type declarations when it helps
readability.</p>
</div>

<div class="stylebody">

<div class="pros">
<p>
</p><ul>
<li>C++ type names can be long and cumbersome, especially when they
involve templates or namespaces.</li>
<li>When a C++ type name is repeated within a single declaration or a
small code region, the repetition may not be aiding readability.</li>
<li>It is sometimes safer to let the type be specified by the type of
the initialization expression, since that avoids the possibility of
unintended copies or type conversions.</li>
</ul>
</div>
<div class="cons">

<p>Sometimes code is clearer when types are manifest,
especially when a variable's initialization depends on
things that were declared far away. In expressions
like:</p>

<pre class="badcode">auto foo = x.add_foo();
auto i = y.Find(key);
</pre>

<p>it may not be obvious what the resulting types are if the type
of <code>y</code> isn't very well known, or if <code>y</code> was
declared many lines earlier.</p>

<p>Programmers have to understand the difference between
<code>auto</code> and <code>const auto&amp;</code> or
they'll get copies when they didn't mean to.</p>

<p>If an <code>auto</code> variable is used as part of an
interface, e.g. as a constant in a header, then a
programmer might change its type while only intending to
change its value, leading to a more radical API change
than intended.</p>
</div>

<div class="decision">

<p><code>auto</code> is permitted when it increases readability,
particularly as described below. Never initialize an <code>auto</code>-typed
variable with a braced initializer list.</p>

<p>Specific cases where <code>auto</code> is allowed or encouraged:
</p><ul>
<li>(Encouraged) For iterators and other long/cluttery type names, particularly
when the type is clear from context (calls
to <code>find</code>, <code>begin</code>, or <code>end</code> for
instance).</li>
<li>(Allowed) When the type is clear from local context (in the same expression
or within a few lines).  Initialization of a pointer or smart pointer
with calls
to <code>new</code> 
commonly falls into this category, as does use of <code>auto</code> in
a range-based loop over a container whose type is spelled out
nearby.</li>
<li>(Allowed) When the type doesn't matter because it isn't being used for
anything other than equality comparison.</li>
<li>(Encouraged) When iterating over a map with a range-based loop
(because it is often assumed that the correct type
is <code>std::pair&lt;KeyType, ValueType&gt;</code> whereas it is actually
<code>std::pair&lt;const KeyType, ValueType&gt;</code>). This is
particularly well paired with local <code>key</code>
and <code>value</code> aliases for <code>.first</code>
and <code>.second</code> (often const-ref).
<pre class="code">for (const auto&amp; item : some_map) {
  const KeyType&amp; key = item.first;
  const ValType&amp; value = item.second;
  // The rest of the loop can now just refer to key and value,
  // a reader can see the types in question, and we've avoided
  // the too-common case of extra copies in this iteration.
}
</pre>
</li>
</ul>

</div>

</div> 

<h3 id="Braced_Initializer_List">Braced Initializer List</h3>

<div class="summary">
<p>You may use braced initializer lists.</p>
</div>

<div class="stylebody">

<p>In C++03, aggregate types (arrays and structs with no
constructor) could be initialized with braced initializer lists.</p>

<pre>struct Point { int x; int y; };
Point p = {1, 2};
</pre>

<p>In C++11, this syntax was generalized, and any object type can now
be created with a braced initializer list, known as a
<i>braced-init-list</i> in the C++ grammar. Here are a few examples
of its use.</p>

<pre>// Vector takes a braced-init-list of elements.
std::vector&lt;string&gt; v{"foo", "bar"};

// Basically the same, ignoring some small technicalities.
// You may choose to use either form.
std::vector&lt;string&gt; v = {"foo", "bar"};

// Usable with 'new' expressions.
auto p = new vector&lt;string&gt;{"foo", "bar"};

// A map can take a list of pairs. Nested braced-init-lists work.
std::map&lt;int, string&gt; m = {{1, "one"}, {2, "2"}};

// A braced-init-list can be implicitly converted to a return type.
std::vector&lt;int&gt; test_function() { return {1, 2, 3}; }

// Iterate over a braced-init-list.
for (int i : {-1, -2, -3}) {}

// Call a function using a braced-init-list.
void TestFunction2(std::vector&lt;int&gt; v) {}
TestFunction2({1, 2, 3});
</pre>

<p>A user-defined type can also define a constructor and/or assignment operator
that take <code>std::initializer_list&lt;T&gt;</code>, which is automatically
created from <i>braced-init-list</i>:</p>

<pre>class MyType {
 public:
  // std::initializer_list references the underlying init list.
  // It should be passed by value.
  MyType(std::initializer_list&lt;int&gt; init_list) {
    for (int i : init_list) append(i);
  }
  MyType&amp; operator=(std::initializer_list&lt;int&gt; init_list) {
    clear();
    for (int i : init_list) append(i);
  }
};
MyType m{2, 3, 5, 7};
</pre>

<p>Finally, brace initialization can also call ordinary
constructors of data types, even if they do not have
<code>std::initializer_list&lt;T&gt;</code> constructors.</p>

<pre>double d{1.23};
// Calls ordinary constructor as long as MyOtherType has no
// std::initializer_list constructor.
class MyOtherType {
 public:
  explicit MyOtherType(string);
  MyOtherType(int, string);
};
MyOtherType m = {1, "b"};
// If the constructor is explicit, you can't use the "= {}" form.
MyOtherType m{"b"};
</pre>

<p>Never assign a <i>braced-init-list</i> to an auto
local variable. In the single element case, what this
means can be confusing.</p>

<pre class="badcode">auto d = {1.23};        // d is a std::initializer_list&lt;double&gt;
</pre>

<pre>auto d = double{1.23};  // Good -- d is a double, not a std::initializer_list.
</pre>

<p>See <a href="#Braced_Initializer_List_Format">Braced_Initializer_List_Format</a> for formatting.</p>

</div> 

<h3 id="Lambda_expressions">Lambda expressions</h3>

<div class="summary">
<p>Use lambda expressions where appropriate. Prefer explicit captures
when the lambda will escape the current scope.</p>
</div>

<div class="stylebody">

<div class="definition">

<p> Lambda expressions are a concise way of creating anonymous
function objects. They're often useful when passing
functions as arguments. For example:</p>

<pre>std::sort(v.begin(), v.end(), [](int x, int y) {
  return Weight(x) &lt; Weight(y);
});
</pre>

<p> They further allow capturing variables from the enclosing scope either
explicitly by name, or implicitly using a default capture. Explicit captures
require each variable to be listed, as
either a value or reference capture:</p>

<pre>int weight = 3;
int sum = 0;
// Captures `weight` by value and `sum` by reference.
std::for_each(v.begin(), v.end(), [weight, &amp;sum](int x) {
  sum += weight * x;
});
</pre>


Default captures implicitly capture any variable referenced in the
lambda body, including <code>this</code> if any members are used:

<pre>const std::vector&lt;int&gt; lookup_table = ...;
std::vector&lt;int&gt; indices = ...;
// Captures `lookup_table` by reference, sorts `indices` by the value
// of the associated element in `lookup_table`.
std::sort(indices.begin(), indices.end(), [&amp;](int a, int b) {
  return lookup_table[a] &lt; lookup_table[b];
});
</pre>

<p>Lambdas were introduced in C++11 along with a set of utilities
for working with function objects, such as the polymorphic
wrapper <code>std::function</code>.
</p>
</div>

<div class="pros">
<ul>
  <li>Lambdas are much more concise than other ways of
   defining function objects to be passed to STL
   algorithms, which can be a readability
   improvement.</li>

  <li>Appropriate use of default captures can remove
    redundancy and highlight important exceptions from
    the default.</li>

   <li>Lambdas, <code>std::function</code>, and
   <code>std::bind</code> can be used in combination as a
   general purpose callback mechanism; they make it easy
   to write functions that take bound functions as
   arguments.</li>
</ul>
</div>

<div class="cons">
<ul>
  <li>Variable capture in lambdas can be a source of dangling-pointer
  bugs, particularly if a lambda escapes the current scope.</li>

  <li>Default captures by value can be misleading because they do not prevent
  dangling-pointer bugs. Capturing a pointer by value doesn't cause a deep
  copy, so it often has the same lifetime issues as capture by reference.
  This is especially confusing when capturing 'this' by value, since the use
  of 'this' is often implicit.</li>

  <li>It's possible for use of lambdas to get out of
  hand; very long nested anonymous functions can make
  code harder to understand.</li>

</ul>
</div>

<div class="decision">
<ul>
<li>Use lambda expressions where appropriate, with formatting as
described <a href="#Formatting_Lambda_Expressions">below</a>.</li>
<li>Prefer explicit captures if the lambda may escape the current scope.
For example, instead of:
<pre class="badcode">{
  Foo foo;
  ...
  executor-&gt;Schedule([&amp;] { Frobnicate(foo); })
  ...
}
// BAD! The fact that the lambda makes use of a reference to `foo` and
// possibly `this` (if `Frobnicate` is a member function) may not be
// apparent on a cursory inspection. If the lambda is invoked after
// the function returns, that would be bad, because both `foo`
// and the enclosing object could have been destroyed.
</pre>
prefer to write:
<pre>{
  Foo foo;
  ...
  executor-&gt;Schedule([&amp;foo] { Frobnicate(foo); })
  ...
}
// BETTER - The compile will fail if `Frobnicate` is a member
// function, and it's clearer that `foo` is dangerously captured by
// reference.
</pre>
</li>
<li>Use default capture by reference ([&amp;]) only when the
lifetime of the lambda is obviously shorter than any potential
captures.
</li>
<li>Use default capture by value ([=]) only as a means of binding a
few variables for a short lambda, where the set of captured
variables is obvious at a glance. Prefer not to write long or
complex lambdas with default capture by value.
</li>
<li>Keep unnamed lambdas short.  If a lambda body is more than
maybe five lines long, prefer to give the lambda a name, or to
use a named function instead of a lambda.</li>
<li>Specify the return type of the lambda explicitly if that will
make it more obvious to readers, as with
<a href="#auto"><code>auto</code></a>.</li>

</ul>
</div>

</div> 

<h3 id="Template_metaprogramming">Template metaprogramming</h3>
<div class="summary">
<p>Avoid complicated template programming.</p>
</div>

<div class="stylebody">

<div class="definition">
<p>Template metaprogramming refers to a family of techniques that
exploit the fact that the C++ template instantiation mechanism is
Turing complete and can be used to perform arbitrary compile-time
computation in the type domain.</p>
</div>

<div class="pros">
<p>Template metaprogramming allows extremely flexible interfaces that
are type safe and high performance. Facilities like

<a href="https://code.google.com/p/googletest/">Google Test</a>,
<code>std::tuple</code>, <code>std::function</code>, and
Boost.Spirit would be impossible without it.</p>
</div>

<div class="cons">
<p>The techniques used in template metaprogramming are often obscure
to anyone but language experts. Code that uses templates in
complicated ways is often unreadable, and is hard to debug or
maintain.</p>

<p>Template metaprogramming often leads to extremely poor compiler
time error messages: even if an interface is simple, the complicated
implementation details become visible when the user does something
wrong.</p>

<p>Template metaprogramming interferes with large scale refactoring by
making the job of refactoring tools harder. First, the template code
is expanded in multiple contexts, and it's hard to verify that the
transformation makes sense in all of them. Second, some refactoring
tools work with an AST that only represents the structure of the code
after template expansion. It can be difficult to automatically work
back to the original source construct that needs to be
rewritten.</p>
</div>

<div class="decision">
<p>Template metaprogramming sometimes allows cleaner and easier-to-use
interfaces than would be possible without it, but it's also often a
temptation to be overly clever. It's best used in a small number of
low level components where the extra maintenance burden is spread out
over a large number of uses.</p>

<p>Think twice before using template metaprogramming or other
complicated template techniques; think about whether the average
member of your team will be able to understand your code well enough
to maintain it after you switch to another project, or whether a
non-C++ programmer or someone casually browsing the code base will be
able to understand the error messages or trace the flow of a function
they want to call.  If you're using recursive template instantiations
or type lists or metafunctions or expression templates, or relying on
SFINAE or on the <code>sizeof</code> trick for detecting function
overload resolution, then there's a good chance you've gone too
far.</p>

<p>If you use template metaprogramming, you should expect to put
considerable effort into minimizing and isolating the complexity. You
should hide metaprogramming as an implementation detail whenever
possible, so that user-facing headers are readable, and you should
make sure that tricky code is especially well commented. You should
carefully document how the code is used, and you should say something
about what the "generated" code looks like. Pay extra attention to the
error messages that the compiler emits when users make mistakes.  The
error messages are part of your user interface, and your code should
be tweaked as necessary so that the error messages are understandable
and actionable from a user point of view.</p>

</div> 
</div> 


<h3 id="Boost">Boost</h3>

<div class="summary">
<p>Use only approved libraries from the Boost library
collection.</p>
</div>

<div class="stylebody">

<div class="definition">
<p> The
<a href="https://www.boost.org/">
Boost library collection</a> is a popular collection of
peer-reviewed, free, open-source C++ libraries.</p>
</div>

<div class="pros">
<p>Boost code is generally very high-quality, is widely
portable, and fills many important gaps in the C++
standard library, such as type traits and better binders.</p>
</div>

<div class="cons">
<p>Some Boost libraries encourage coding practices which can
hamper readability, such as metaprogramming and other
advanced template techniques, and an excessively
"functional" style of programming. </p>
</div>

<div class="decision">

 

<div>
<p>In order to maintain a high level of readability for
all contributors who might read and maintain code, we
only allow an approved subset of Boost features.
Currently, the following libraries are permitted:</p>

<ul>
  <li>
  <a href="https://www.boost.org/libs/utility/call_traits.htm">
  Call Traits</a> from <code>boost/call_traits.hpp</code></li>

  <li><a href="https://www.boost.org/libs/utility/compressed_pair.htm">
  Compressed Pair</a> from  <code>boost/compressed_pair.hpp</code></li>

  <li><a href="https://www.boost.org/libs/graph/">
  The Boost Graph Library (BGL)</a> from <code>boost/graph</code>,
  except serialization (<code>adj_list_serialize.hpp</code>) and
   parallel/distributed algorithms and data structures
   (<code>boost/graph/parallel/*</code> and
   <code>boost/graph/distributed/*</code>).</li>

  <li><a href="https://www.boost.org/libs/property_map/">
  Property Map</a> from <code>boost/property_map</code>, except
  parallel/distributed property maps (<code>boost/property_map/parallel/*</code>).</li>

  <li><a href="https://www.boost.org/libs/iterator/">
  Iterator</a> from <code>boost/iterator</code></li>

  <li>The part of <a href="https://www.boost.org/libs/polygon/">
  Polygon</a> that deals with Voronoi diagram
  construction and doesn't depend on the rest of
  Polygon:
  <code>boost/polygon/voronoi_builder.hpp</code>,
  <code>boost/polygon/voronoi_diagram.hpp</code>, and
  <code>boost/polygon/voronoi_geometry_type.hpp</code></li>

  <li><a href="https://www.boost.org/libs/bimap/">
  Bimap</a> from <code>boost/bimap</code></li>

  <li><a href="https://www.boost.org/libs/math/doc/html/dist.html">
  Statistical Distributions and Functions</a> from
  <code>boost/math/distributions</code></li>

  <li><a href="https://www.boost.org/libs/math/doc/html/special.html">
  Special Functions</a> from <code>boost/math/special_functions</code></li>

  <li><a href="https://www.boost.org/libs/multi_index/">
  Multi-index</a> from <code>boost/multi_index</code></li>

  <li><a href="https://www.boost.org/libs/heap/">
  Heap</a> from <code>boost/heap</code></li>

  <li>The flat containers from
  <a href="https://www.boost.org/libs/container/">Container</a>:
  <code>boost/container/flat_map</code>, and
  <code>boost/container/flat_set</code></li>

  <li><a href="https://www.boost.org/libs/intrusive/">Intrusive</a>
  from <code>boost/intrusive</code>.</li>

  <li><a href="https://www.boost.org/libs/sort/">The
  <code>boost/sort</code> library</a>.</li>

  <li><a href="https://www.boost.org/libs/preprocessor/">Preprocessor</a>
  from <code>boost/preprocessor</code>.</li>
</ul>

<p>We are actively considering adding other Boost
features to the list, so this list may be expanded in
the future.</p>
</div> 

<p>The following libraries are permitted, but their use
is discouraged because they've been superseded by
standard libraries in C++11:</p>

<ul>
  <li><a href="https://www.boost.org/libs/array/">
  Array</a> from <code>boost/array.hpp</code>: use
  <a href="http://en.cppreference.com/w/cpp/container/array">
   <code>std::array</code></a> instead.</li>

   <li><a href="https://www.boost.org/libs/ptr_container/">
   Pointer Container</a> from <code>boost/ptr_container</code>: use containers of
   <a href="http://en.cppreference.com/w/cpp/memory/unique_ptr">
   <code>std::unique_ptr</code></a> instead.</li>
</ul>
</div> 

</div> 

 

<h3 id="std_hash">std::hash</h3>

<div class="summary">
<p>Do not define specializations of <code>std::hash</code>.</p>
</div>

<div class="stylebody">

<div class="definition">
<p><code>std::hash&lt;T&gt;</code> is the function object that the
C++11 hash containers use to hash keys of type <code>T</code>,
unless the user explicitly specifies a different hash function. For
example, <code>std::unordered_map&lt;int, string&gt;</code> is a hash
map that uses <code>std::hash&lt;int&gt;</code> to hash its keys,
whereas <code>std::unordered_map&lt;int, string, MyIntHash&gt;</code>
uses <code>MyIntHash</code>.</p>

<p><code>std::hash</code> is defined for all integral, floating-point,
pointer, and <code>enum</code> types, as well as some standard library
types such as <code>string</code> and <code>unique_ptr</code>. Users
can enable it to work for their own types by defining specializations
of it for those types.</p>
</div>

<div class="pros">
<p><code>std::hash</code> is easy to use, and simplifies the code
since you don't have to name it explicitly. Specializing
<code>std::hash</code> is the standard way of specifying how to
hash a type, so it's what outside resources will teach, and what
new engineers will expect.</p>
</div>

<div class="cons">
<p><code>std::hash</code> is hard to specialize. It requires a lot
of boilerplate code, and more importantly, it combines responsibility
for identifying the hash inputs with responsibility for executing the
hashing algorithm itself. The type author has to be responsible for
the former, but the latter requires expertise that a type author
usually doesn't have, and shouldn't need. The stakes here are high
because low-quality hash functions can be security vulnerabilities,
due to the emergence of
<a href="https://emboss.github.io/blog/2012/12/14/breaking-murmur-hash-flooding-dos-reloaded/">
hash flooding attacks</a>.</p>

<p>Even for experts, <code>std::hash</code> specializations are
inordinately difficult to implement correctly for compound types,
because the implementation cannot recursively call <code>std::hash</code>
on data members. High-quality hash algorithms maintain large
amounts of internal state, and reducing that state to the
<code>size_t</code> bytes that <code>std::hash</code>
returns is usually the slowest part of the computation, so it
should not be done more than once.</p>

<p>Due to exactly that issue, <code>std::hash</code> does not work
with <code>std::pair</code> or <code>std::tuple</code>, and the
language does not allow us to extend it to support them.</p>
</div>

<div class="decision">
<p>You can use <code>std::hash</code> with the types that it supports
"out of the box", but do not specialize it to support additional types.
If you need a hash table with a key type that <code>std::hash</code>
does not support, consider using legacy hash containers (e.g.
<code>hash_map</code>) for now; they use a different default hasher,
which is unaffected by this prohibition.</p>

<p>If you want to use the standard hash containers anyway, you will
need to specify a custom hasher for the key type, e.g.</p>
<pre>std::unordered_map&lt;MyKeyType, Value, MyKeyTypeHasher&gt; my_map;
</pre><p>
Consult with the type's owners to see if there is an existing hasher
that you can use; otherwise work with them to provide one,
 or roll your own.</p>

<p>We are planning to provide a hash function that can work with any type,
using a new customization mechanism that doesn't have the drawbacks of
<code>std::hash</code>.</p>
</div>

</div>  

<h3 id="C++11">C++11</h3>

<div class="summary">
<p>Use libraries and language extensions from C++11 when appropriate.
Consider portability to other environments
before using C++11 features in your
project. </p>

</div>

<div class="stylebody">

<div class="definition">
<p> C++11 contains <a href="https://en.wikipedia.org/wiki/C%2B%2B11">
significant changes</a> both to the language and
libraries. </p>
</div>

<div class="pros">
<p>C++11 was the official standard until august 2014, and
is supported by most C++ compilers. It standardizes
some common C++ extensions that we use already, allows
shorthands for some operations, and has some performance
and safety improvements.</p>
</div>

<div class="cons">
<p>The C++11 standard is substantially more complex than
its predecessor (1,300 pages versus 800 pages), and is
unfamiliar to many developers. The long-term effects of
some features on code readability and maintenance are
unknown. We cannot predict when its various features will
be implemented uniformly by tools that may be of
interest, particularly in the case of projects that are
forced to use older versions of tools.</p>

<p>As with <a href="#Boost">Boost</a>, some C++11
extensions encourage coding practices that hamper
readability&#8212;for example by removing
checked redundancy (such as type names) that may be
helpful to readers, or by encouraging template
metaprogramming. Other extensions duplicate functionality
available through existing mechanisms, which may lead to confusion
and conversion costs.</p>


</div>

<div class="decision">

<p>C++11 features may be used unless specified otherwise.
In addition to what's described in the rest of the style
guide, the following C++11 features may not be used:</p>

<ul>
  

  

  

  

  <li>Compile-time rational numbers
  (<code>&lt;ratio&gt;</code>), because of concerns that
  it's tied to a more template-heavy interface
  style.</li>

  <li>The <code>&lt;cfenv&gt;</code> and
  <code>&lt;fenv.h&gt;</code> headers, because many
  compilers do not support those features reliably.</li>

  <li>Ref-qualifiers on member functions, such as <code>void X::Foo()
    &amp;</code> or <code>void X::Foo() &amp;&amp;</code>, because of concerns
    that they're an overly obscure feature.</li>

  

  
</ul>
</div>

</div> 

<h3 id="Nonstandard_Extensions">Nonstandard Extensions</h3>

<div class="summary">
<p>Nonstandard extensions to C++ may not be used unless otherwise specified.</p>
</div>
<div class="stylebody">
<div class="definition">
<p>Compilers support various extensions that are not part of standard C++. Such
  extensions include GCC's <code>__attribute__</code>, intrinsic functions such
  as <code>__builtin_prefetch</code>, designated initializers (e.g.
  <code>Foo f = {.field = 3}</code>), inline assembly, <code>__COUNTER__</code>,
  <code>__PRETTY_FUNCTION__</code>, compound statement expressions (e.g.
  <code>foo = ({ int x; Bar(&amp;x); x })</code>, variable-length arrays and
  <code>alloca()</code>, and the <code>a?:b</code> syntax.</p>
</div>

<div class="pros">
  <ul>
    <li>Nonstandard extensions may provide useful features that do not exist
      in standard C++. For example, some people think that designated
      initializers are more readable than standard C++ features like
      constructors.</li>
    <li>Important performance guidance to the compiler can only be specified
      using extensions.</li>
  </ul>
</div>

<div class="cons">
  <ul>
    <li>Nonstandard extensions do not work in all compilers. Use of nonstandard
      extensions reduces portability of code.</li>
    <li>Even if they are supported in all targeted compilers, the extensions
      are often not well-specified, and there may be subtle behavior differences
      between compilers.</li>
    <li>Nonstandard extensions add to the language features that a reader must
      know to understand the code.</li>
  </ul>
</div>

<div class="decision">
<p>Do not use nonstandard extensions. You may use portability wrappers that
  are implemented using nonstandard extensions, so long as those wrappers
  
  are provided by a designated project-wide
  portability header.</p>
</div>
</div> 

<h3 id="Aliases">Aliases</h3>

<div class="summary">
<p>Public aliases are for the benefit of an API's user, and should be clearly documented.</p>
</div>
<div class="stylebody">
<div class="definition">
<p>There are several ways to create names that are aliases of other entities:</p>
<pre>typedef Foo Bar;
using Bar = Foo;
using other_namespace::Foo;
</pre>

  <p>Like other declarations, aliases declared in a header file are part of that
  header's public API unless they're in a function definition, in the private portion of a class,
  or in an explicitly-marked internal namespace. Aliases in such areas or in .cc files are
  implementation details (because client code can't refer to them), and are not restricted by this
  rule.</p>
</div>

<div class="pros">
  <ul>
    <li>Aliases can improve readability by simplifying a long or complicated name.</li>
    <li>Aliases can reduce duplication by naming in one place a type used repeatedly in an API,
      which <em>might</em> make it easier to change the type later.
    </li>
  </ul>
</div>

<div class="cons">
  <ul>
    <li>When placed in a header where client code can refer to them, aliases increase the
      number of entities in that header's API, increasing its complexity.</li>
    <li>Clients can easily rely on unintended details of public aliases, making
      changes difficult.</li>
    <li>It can be tempting to create a public alias that is only intended for use
      in the implementation, without considering its impact on the API, or on maintainability.</li>
    <li>Aliases can create risk of name collisions</li>
    <li>Aliases can reduce readability by giving a familiar construct an unfamiliar name</li>
    <li>Type aliases can create an unclear API contract:
      it is unclear whether the alias is guaranteed to be identical to the type it aliases,
      to have the same API, or only to be usable in specified narrow ways</li>
  </ul>
</div>

<div class="decision">
<p>Don't put an alias in your public API just to save typing in the implementation;
  do so only if you intend it to be used by your clients.</p>
<p>When defining a public alias, document the intent of
the new name, including whether it is guaranteed to always be the same as the type
it's currently aliased to, or whether a more limited compatibility is
intended. This lets the user know whether they can treat the types as
substitutable or whether more specific rules must be followed, and can help the
implementation retain some degree of freedom to change the alias.</p>
<p>Don't put namespace aliases in your public API. (See also <a href="#Namespaces">Namespaces</a>).
</p>

<p>For example, these aliases document how they are intended to be used in client code:</p>
<pre>namespace a {
// Used to store field measurements. DataPoint may change from Bar* to some internal type.
// Client code should treat it as an opaque pointer.
using DataPoint = foo::bar::Bar*;

// A set of measurements. Just an alias for user convenience.
using TimeSeries = std::unordered_set&lt;DataPoint, std::hash&lt;DataPoint&gt;, DataPointComparator&gt;;
}  // namespace a
</pre>

<p>These aliases don't document intended use, and half of them aren't meant for client use:</p>

<pre class="badcode">namespace a {
// Bad: none of these say how they should be used.
using DataPoint = foo::bar::Bar*;
using std::unordered_set;  // Bad: just for local convenience
using std::hash;           // Bad: just for local convenience
typedef unordered_set&lt;DataPoint, hash&lt;DataPoint&gt;, DataPointComparator&gt; TimeSeries;
}  // namespace a
</pre>

<p>However, local convenience aliases are fine in function definitions, private sections of
  classes, explicitly marked internal namespaces, and in .cc files:</p>

<pre>// In a .cc file
using std::unordered_set;
</pre>

</div>
</div> 

<h2 id="Naming">Naming</h2>

<p>The most important consistency rules are those that govern
naming. The style of a name immediately informs us what sort of
thing the named entity is: a type, a variable, a function, a
constant, a macro, etc., without requiring us to search for the
declaration of that entity. The pattern-matching engine in our
brains relies a great deal on these naming rules.
</p>

<p>Naming rules are pretty arbitrary, but
 we feel that
consistency is more important than individual preferences in this
area, so regardless of whether you find them sensible or not,
the rules are the rules.</p>

<h3 id="General_Naming_Rules">General Naming Rules</h3>

<div class="summary">
<p>Names should be descriptive; avoid abbreviation.</p>
</div>

<div class="stylebody">
<p>Give as descriptive a name as possible, within reason.
Do not worry about saving horizontal space as it is far
more important to make your code immediately
understandable by a new reader. Do not use abbreviations
that are ambiguous or unfamiliar to readers outside your
project, and do not abbreviate by deleting letters within
a word.</p>

<pre>int price_count_reader;    // No abbreviation.
int num_errors;            // "num" is a widespread convention.
int num_dns_connections;   // Most people know what "DNS" stands for.
</pre>

<pre class="badcode">int n;                     // Meaningless.
int nerr;                  // Ambiguous abbreviation.
int n_comp_conns;          // Ambiguous abbreviation.
int wgc_connections;       // Only your group knows what this stands for.
int pc_reader;             // Lots of things can be abbreviated "pc".
int cstmr_id;              // Deletes internal letters.
</pre>

<p>Note that certain universally-known abbreviations are OK, such as
<code>i</code> for an iteration variable and <code>T</code> for a
template parameter.</p>

<p>Template parameters should follow the naming style for their
category: type template parameters should follow the rules for
<a href="#Type_Names">type names</a>, and non-type template
parameters should follow the rules for <a href="#Variable_Names">
variable names</a>.

</p></div> 

<h3 id="File_Names">File Names</h3>

<div class="summary">
<p>Filenames should be all lowercase and can include
underscores (<code>_</code>) or dashes (<code>-</code>).
Follow the convention that your
 
project uses. If there is no consistent
local pattern to follow, prefer "_".</p>
</div>

<div class="stylebody">

<p>Examples of acceptable file names:</p>

<ul>
  <li><code>my_useful_class.cc</code></li>
  <li><code>my-useful-class.cc</code></li>
  <li><code>myusefulclass.cc</code></li>
  <li><code>myusefulclass_test.cc // _unittest and _regtest are deprecated.</code></li>
</ul>

<p>C++ files should end in <code>.cc</code> and header files should end in
<code>.h</code>. Files that rely on being textually included at specific points
should end in <code>.inc</code> (see also the section on
<a href="#Self_contained_Headers">self-contained headers</a>).</p>

<p>Do not use filenames that already exist in
<code>/usr/include</code>, such as <code>db.h</code>.</p>

<p>In general, make your filenames very specific. For
example, use <code>http_server_logs.h</code> rather than
<code>logs.h</code>. A very common case is to have a pair
of files called, e.g., <code>foo_bar.h</code> and
<code>foo_bar.cc</code>, defining a class called
<code>FooBar</code>.</p>

<p>Inline functions must be in a <code>.h</code> file. If
your inline functions are very short, they should go
directly into your <code>.h</code> file. </p>

</div> 

<h3 id="Type_Names">Type Names</h3>

<div class="summary">
<p>Type names start with a capital letter and have a capital
letter for each new word, with no underscores:
<code>MyExcitingClass</code>, <code>MyExcitingEnum</code>.</p>
</div>

<div class="stylebody">

<p>The names of all types &#8212; classes, structs, type aliases,
enums, and type template parameters &#8212; have the same naming convention.
Type names should start with a capital letter and have a capital letter
for each new word. No underscores. For example:</p>

<pre>// classes and structs
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// typedefs
typedef hash_map&lt;UrlTableProperties *, string&gt; PropertiesMap;

// using aliases
using PropertiesMap = hash_map&lt;UrlTableProperties *, string&gt;;

// enums
enum UrlTableErrors { ...
</pre>

</div> 

<h3 id="Variable_Names">Variable Names</h3>

<div class="summary">
<p>The names of variables (including function parameters) and data members are
all lowercase, with underscores between words. Data members of classes (but not
structs) additionally have trailing underscores. For instance:
<code>a_local_variable</code>, <code>a_struct_data_member</code>,
<code>a_class_data_member_</code>.</p>
</div>

<div class="stylebody">

<h4 class="stylepoint_subsection">Common Variable names</h4>

<p>For example:</p>

<pre>string table_name;  // OK - uses underscore.
string tablename;   // OK - all lowercase.
</pre>

<pre class="badcode">string tableName;   // Bad - mixed case.
</pre>

<h4 class="stylepoint_subsection">Class Data Members</h4>

<p>Data members of classes, both static and non-static, are
named like ordinary nonmember variables, but with a
trailing underscore.</p>

<pre>class TableInfo {
  ...
 private:
  string table_name_;  // OK - underscore at end.
  string tablename_;   // OK.
  static Pool&lt;TableInfo&gt;* pool_;  // OK.
};
</pre>

<h4 class="stylepoint_subsection">Struct Data Members</h4>

<p>Data members of structs, both static and non-static,
are named like ordinary nonmember variables. They do not have
the trailing underscores that data members in classes have.</p>

<pre>struct UrlTableProperties {
  string name;
  int num_entries;
  static Pool&lt;UrlTableProperties&gt;* pool;
};
</pre>


<p>See <a href="#Structs_vs._Classes">Structs vs.
Classes</a> for a discussion of when to use a struct
versus a class.</p>

</div> 

<h3 id="Constant_Names">Constant Names</h3>

<div class="summary">
  <p>Variables declared constexpr or const, and whose value is fixed for
  the duration of the program, are named with a leading "k" followed
  by mixed case.  For example:</p>
</div>

<pre>const int kDaysInAWeek = 7;
</pre>

<div class="stylebody">

  <p>All such variables with static storage duration (i.e. statics and globals,
  see <a href="http://en.cppreference.com/w/cpp/language/storage_duration#Storage_duration">
    Storage Duration</a> for details) should be named this way.  This
  convention is optional for variables of other storage classes, e.g. automatic
  variables, otherwise the usual variable naming rules apply.</p><p>

</p></div> 

<h3 id="Function_Names">Function Names</h3>

<div class="summary">
<p>Regular functions have mixed case; accessors and mutators may be named
like variables.</p>
</div>

<div class="stylebody">

<p>Ordinarily, functions should start with a capital letter and have a
capital letter for each new word
(a.k.a. "<a href="https://en.wikipedia.org/wiki/Camel_case">Camel
Case</a>" or "Pascal case"). Such names should not have
underscores. Prefer to capitalize acronyms as single words
(i.e. <code>StartRpc()</code>, not <code>StartRPC()</code>).</p>

<pre>AddTableEntry()
DeleteUrl()
OpenFileOrDie()
</pre>

<p>(The same naming rule applies to class- and namespace-scope
constants that are exposed as part of an API and that are intended to look
like functions, because the fact that they're
objects rather than functions is an unimportant implementation detail.)</p>

<p>Accessors and mutators (get and set functions) may be named like
variables. These often correspond to actual member variables, but this is
not required. For example, <code>int count()</code> and <code>void
set_count(int count)</code>.</p>

</div> 

<h3 id="Namespace_Names">Namespace Names</h3>

<div class="summary">
Namespace names are all lower-case. Top-level namespace names are
based on the project name
. Avoid collisions
between nested namespaces and well-known top-level namespaces.
</div>

<div class="stylebody">
<p>The name of a top-level namespace should usually be the
name of the project or team whose code is contained in that
namespace. The code in that namespace should usually be in
a directory whose basename matches the namespace name (or
subdirectories thereof).</p>





<p>Keep in mind that the <a href="#General_Naming_Rules">rule
against abbreviated names</a> applies to namespaces just as much
as variable names. Code inside the namespace seldom needs to
mention the namespace name, so there's usually no particular need
for abbreviation anyway.</p>

<p>Avoid nested namespaces that match well-known top-level
namespaces. Collisions between namespace names can lead to surprising
build breaks because of name lookup rules. In particular, do not
create any nested <code>std</code> namespaces. Prefer unique project
identifiers
(<code>websearch::index</code>, <code>websearch::index_util</code>)
over collision-prone names like <code>websearch::util</code>.</p>

<p>For <code>internal</code> namespaces, be wary of other code being
added to the same <code>internal</code> namespace causing a collision
(internal helpers within a team tend to be related and may lead to
collisions). In such a situation, using the filename to make a unique
internal name is helpful
(<code>websearch::index::frobber_internal</code> for use
in <code>frobber.h</code>)</p>

</div> 

<h3 id="Enumerator_Names">Enumerator Names</h3>

<div class="summary">
<p>Enumerators (for both scoped and unscoped enums) should be named <i>either</i> like
<a href="#Constant_Names">constants</a> or like
<a href="#Macro_Names">macros</a>: either <code>kEnumName</code> or
<code>ENUM_NAME</code>.</p>
</div>

<div class="stylebody">

<p>Preferably, the individual enumerators should be named
like <a href="#Constant_Names">constants</a>. However, it
is also acceptable to name them like
<a href="#Macro_Names">macros</a>.  The enumeration name,
<code>UrlTableErrors</code> (and
<code>AlternateUrlTableErrors</code>), is a type, and
therefore mixed case.</p>

<pre>enum UrlTableErrors {
  kOK = 0,
  kErrorOutOfMemory,
  kErrorMalformedInput,
};
enum AlternateUrlTableErrors {
  OK = 0,
  OUT_OF_MEMORY = 1,
  MALFORMED_INPUT = 2,
};
</pre>

<p>Until January 2009, the style was to name enum values
like <a href="#Macro_Names">macros</a>. This caused
problems with name collisions between enum values and
macros. Hence, the change to prefer constant-style naming
was put in place. New code should prefer constant-style
naming if possible. However, there is no reason to change
old code to use constant-style names, unless the old
names are actually causing a compile-time problem.</p>



</div> 

<h3 id="Macro_Names">Macro Names</h3>

<div class="summary">
<p>You're not really going to <a href="#Preprocessor_Macros">
define a macro</a>, are you? If you do, they're like this:
<code>MY_MACRO_THAT_SCARES_SMALL_CHILDREN</code>.</p>
</div>

<div class="stylebody">

<p>Please see the <a href="#Preprocessor_Macros">description
of macros</a>; in general macros should <em>not</em> be used.
However, if they are absolutely needed, then they should be
named with all capitals and underscores.</p>

<pre>#define ROUND(x) ...
#define PI_ROUNDED 3.0
</pre>

</div> 

<h3 id="Exceptions_to_Naming_Rules">Exceptions to Naming Rules</h3>

<div class="summary">
<p>If you are naming something that is analogous to an
existing C or C++ entity then you can follow the existing
naming convention scheme.</p>
</div>

<div class="stylebody">

<dl>
  <dt><code>bigopen()</code></dt>
  <dd>function name, follows form of <code>open()</code></dd>

  <dt><code>uint</code></dt>
  <dd><code>typedef</code></dd>

  <dt><code>bigpos</code></dt>
  <dd><code>struct</code> or <code>class</code>, follows
  form of <code>pos</code></dd>

  <dt><code>sparse_hash_map</code></dt>
  <dd>STL-like entity; follows STL naming conventions</dd>

  <dt><code>LONGLONG_MAX</code></dt>
  <dd>a constant, as in <code>INT_MAX</code></dd>
</dl>

</div> 

<h2 id="Comments">Comments</h2>

<p>Though a pain to write, comments are absolutely vital to
keeping our code readable. The following rules describe what
you should comment and where. But remember: while comments are
very important, the best code is self-documenting. Giving
sensible names to types and variables is much better than using
obscure names that you must then explain through comments.</p>

<p>When writing your comments, write for your audience: the
next 
contributor who will need to
understand your code. Be generous &#8212; the next
one may be you!</p>

<h3 id="Comment_Style">Comment Style</h3>

<div class="summary">
<p>Use either the <code>//</code> or <code>/* */</code>
syntax, as long as you are consistent.</p>
</div>

<div class="stylebody">

<p>You can use either the <code>//</code> or the <code>/*
*/</code> syntax; however, <code>//</code> is
<em>much</em> more common. Be consistent with how you
comment and what style you use where.</p>

</div> 

<h3 id="File_Comments">File Comments</h3>

<div class="summary">
<p>Start each file with license boilerplate.</p>

<p>File comments describe the contents of a file. If a file declares,
implements, or tests exactly one abstraction that is documented by a comment
at the point of declaration, file comments are not required. All other files
must have file comments.</p>

</div>

<div class="stylebody">

<h4 class="stylepoint_subsection">Legal Notice and Author
Line</h4>



<p>Every file should contain license
boilerplate. Choose the appropriate boilerplate for the
license used by the project (for example, Apache 2.0,
BSD, LGPL, GPL).</p>

<p>If you make significant changes to a file with an
author line, consider deleting the author line.</p>

<h4 class="stylepoint_subsection">File Contents</h4>

<p>If a <code>.h</code> declares multiple abstractions, the file-level comment
should broadly describe the contents of the file, and how the abstractions are
related. A 1 or 2 sentence file-level comment may be sufficient. The detailed
documentation about individual abstractions belongs with those abstractions,
not at the file level.</p>

<p>Do not duplicate comments in both the <code>.h</code> and the
<code>.cc</code>. Duplicated comments diverge.</p>

</div> 

<h3 id="Class_Comments">Class Comments</h3>

<div class="summary">
<p>Every non-obvious class declaration should have an accompanying
comment that describes what it is for and how it should be used.</p>
</div>

<div class="stylebody">

<pre>// Iterates over the contents of a GargantuanTable.
// Example:
//    GargantuanTableIterator* iter = table-&gt;NewIterator();
//    for (iter-&gt;Seek("foo"); !iter-&gt;done(); iter-&gt;Next()) {
//      process(iter-&gt;key(), iter-&gt;value());
//    }
//    delete iter;
class GargantuanTableIterator {
  ...
};
</pre>

<p>The class comment should provide the reader with enough information to know
how and when to use the class, as well as any additional considerations
necessary to correctly use the class. Document the synchronization assumptions
the class makes, if any. If an instance of the class can be accessed by
multiple threads, take extra care to document the rules and invariants
surrounding multithreaded use.</p>

<p>The class comment is often a good place for a small example code snippet
demonstrating a simple and focused usage of the class.</p>

<p>When sufficiently separated (e.g. <code>.h</code> and <code>.cc</code>
files), comments describing the use of the class should go together with its
interface definition; comments about the class operation and implementation
should accompany the implementation of the class's methods.</p>

</div> 

<h3 id="Function_Comments">Function Comments</h3>

<div class="summary">
<p>Declaration comments describe use of the function (when it is
non-obvious); comments at the definition of a function describe
operation.</p>
</div>

<div class="stylebody">

<h4 class="stylepoint_subsection">Function Declarations</h4>

<p>Almost every function declaration should have comments immediately
preceding it that describe what the function does and how to use
it. These comments may be omitted only if the function is simple and
obvious (e.g. simple accessors for obvious properties of the
class).  These comments should be descriptive ("Opens the file")
rather than imperative ("Open the file"); the comment describes the
function, it does not tell the function what to do. In general, these
comments do not describe how the function performs its task. Instead,
that should be left to comments in the function definition.</p>

<p>Types of things to mention in comments at the function
declaration:</p>

<ul>
  <li>What the inputs and outputs are.</li>

  <li>For class member functions: whether the object
  remembers reference arguments beyond the duration of
  the method call, and whether it will free them or
  not.</li>

  <li>If the function allocates memory that the caller
  must free.</li>

  <li>Whether any of the arguments can be a null
  pointer.</li>

  <li>If there are any performance implications of how a
  function is used.</li>

  <li>If the function is re-entrant. What are its
  synchronization assumptions?</li>
 </ul>

<p>Here is an example:</p>

<pre>// Returns an iterator for this table.  It is the client's
// responsibility to delete the iterator when it is done with it,
// and it must not use the iterator once the GargantuanTable object
// on which the iterator was created has been deleted.
//
// The iterator is initially positioned at the beginning of the table.
//
// This method is equivalent to:
//    Iterator* iter = table-&gt;NewIterator();
//    iter-&gt;Seek("");
//    return iter;
// If you are going to immediately seek to another place in the
// returned iterator, it will be faster to use NewIterator()
// and avoid the extra seek.
Iterator* GetIterator() const;
</pre>

<p>However, do not be unnecessarily verbose or state the
completely obvious. Notice below that it is not necessary
 to say "returns false otherwise" because this is
implied.</p>

<pre>// Returns true if the table cannot hold any more entries.
bool IsTableFull();
</pre>

<p>When documenting function overrides, focus on the
specifics of the override itself, rather than repeating
the comment from the overridden function.  In many of these
cases, the override needs no additional documentation and
thus no comment is required.</p>

<p>When commenting constructors and destructors, remember
that the person reading your code knows what constructors
and destructors are for, so comments that just say
something like "destroys this object" are not useful.
Document what constructors do with their arguments (for
example, if they take ownership of pointers), and what
cleanup the destructor does. If this is trivial, just
skip the comment. It is quite common for destructors not
to have a header comment.</p>

<h4 class="stylepoint_subsection">Function Definitions</h4>

<p>If there is anything tricky about how a function does
its job, the function definition should have an
explanatory comment. For example, in the definition
comment you might describe any coding tricks you use,
give an overview of the steps you go through, or explain
why you chose to implement the function in the way you
did rather than using a viable alternative. For instance,
you might mention why it must acquire a lock for the
first half of the function but why it is not needed for
the second half.</p>

<p>Note you should <em>not</em> just repeat the comments
given with the function declaration, in the
<code>.h</code> file or wherever. It's okay to
recapitulate briefly what the function does, but the
focus of the comments should be on how it does it.</p>

</div> 

<h3 id="Variable_Comments">Variable Comments</h3>

<div class="summary">
<p>In general the actual name of the variable should be
descriptive enough to give a good idea of what the variable
is used for. In certain cases, more comments are required.</p>
</div>

<div class="stylebody">

<h4 class="stylepoint_subsection">Class Data Members</h4>

<p>The purpose of each class data member (also called an instance
variable or member variable) must be clear. If there are any
invariants (special values, relationships between members, lifetime
requirements) not clearly expressed by the type and name, they must be
commented. However, if the type and name suffice (<code>int
num_events_;</code>), no comment is needed.</p>

<p>In particular, add comments to describe the existence and meaning
of sentinel values, such as nullptr or -1, when they are not
obvious. For example:</p>

<pre>private:
 // Used to bounds-check table accesses. -1 means
 // that we don't yet know how many entries the table has.
 int num_total_entries_;
</pre>

<h4 class="stylepoint_subsection">Global Variables</h4>

<p>All global variables should have a comment describing what they
are, what they are used for, and (if unclear) why it needs to be
global. For example:</p>

<pre>// The total number of tests cases that we run through in this regression test.
const int kNumTestCases = 6;
</pre>

</div> 

<h3 id="Implementation_Comments">Implementation Comments</h3>

<div class="summary">
<p>In your implementation you should have comments in tricky,
non-obvious, interesting, or important parts of your code.</p>
</div>

<div class="stylebody">

<h4 class="stylepoint_subsection">Explanatory Comments</h4>

<p>Tricky or complicated code blocks should have comments
before them. Example:</p>

<pre>// Divide result by two, taking into account that x
// contains the carry from the add.
for (int i = 0; i &lt; result-&gt;size(); i++) {
  x = (x &lt;&lt; 8) + (*result)[i];
  (*result)[i] = x &gt;&gt; 1;
  x &amp;= 1;
}
</pre>

<h4 class="stylepoint_subsection">Line Comments</h4>

<p>Also, lines that are non-obvious should get a comment
at the end of the line. These end-of-line comments should
be separated from the code by 2 spaces. Example:</p>

<pre>// If we have enough memory, mmap the data portion too.
mmap_budget = max&lt;int64&gt;(0, mmap_budget - index_-&gt;length());
if (mmap_budget &gt;= data_size_ &amp;&amp; !MmapData(mmap_chunk_bytes, mlock))
  return;  // Error already logged.
</pre>

<p>Note that there are both comments that describe what
the code is doing, and comments that mention that an
error has already been logged when the function
returns.</p>

<p>If you have several comments on subsequent lines, it
can often be more readable to line them up:</p>

<pre>DoSomething();                  // Comment here so the comments line up.
DoSomethingElseThatIsLonger();  // Two spaces between the code and the comment.
{ // One space before comment when opening a new scope is allowed,
  // thus the comment lines up with the following comments and code.
  DoSomethingElse();  // Two spaces before line comments normally.
}
std::vector&lt;string&gt; list{
                    // Comments in braced lists describe the next element...
                    "First item",
                    // .. and should be aligned appropriately.
                    "Second item"};
DoSomething(); /* For trailing block comments, one space is fine. */
</pre>

<h4 class="stylepoint_subsection">Function Argument Comments</h4>

<p>When the meaning of a function argument is nonobvious, consider
one of the following remedies:</p>

<ul>
  <li>If the argument is a literal constant, and the same constant is
  used in multiple function calls in a way that tacitly assumes they're
  the same, you should use a named constant to make that constraint
  explicit, and to guarantee that it holds.</li>

  <li>Consider changing the function signature to replace a <code>bool</code>
  argument with an <code>enum</code> argument. This will make the argument
  values self-describing.</li>

  <li>For functions that have several configuration options, consider
  defining a single class or struct to hold all the options
  ,
  and pass an instance of that.
  This approach has several advantages. Options are referenced by name
  at the call site, which clarifies their meaning. It also reduces
  function argument count, which makes function calls easier to read and
  write. As an added benefit, you don't have to change call sites when
  you add another option.
  </li>

  <li>Replace large or complex nested expressions with named variables.</li>

  <li>As a last resort, use comments to clarify argument meanings at the
  call site.</li>
</ul>

Consider the following example:

<pre class="badcode">// What are these arguments?
const DecimalNumber product = CalculateProduct(values, 7, false, nullptr);
</pre>

<p>versus:</p>

<pre>ProductOptions options;
options.set_precision_decimals(7);
options.set_use_cache(ProductOptions::kDontUseCache);
const DecimalNumber product =
    CalculateProduct(values, options, /*completion_callback=*/nullptr);
</pre>

<h4 class="stylepoint_subsection">Don'ts</h4>

<p>Do not state the obvious. In particular, don't literally describe what
code does, unless the behavior is nonobvious to a reader who understands
C++ well. Instead, provide higher level comments that describe <i>why</i>
the code does what it does, or make the code self describing.</p>

Compare this:

<pre class="badcode">// Find the element in the vector.  &lt;-- Bad: obvious!
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
</pre>

To this:

<pre>// Process "element" unless it was already processed.
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
</pre>

Self-describing code doesn't need a comment. The comment from
the example above would be obvious:

<pre>if (!IsAlreadyProcessed(element)) {
  Process(element);
}
</pre>

</div> 

<h3 id="Punctuation,_Spelling_and_Grammar">Punctuation, Spelling and Grammar</h3>

<div class="summary">
<p>Pay attention to punctuation, spelling, and grammar; it is
easier to read well-written comments than badly written
ones.</p>
</div>

<div class="stylebody">

<p>Comments should be as readable as narrative text, with
proper capitalization and punctuation. In many cases,
complete sentences are more readable than sentence
fragments. Shorter comments, such as comments at the end
of a line of code, can sometimes be less formal, but you
should be consistent with your style.</p>

<p>Although it can be frustrating to have a code reviewer
point out that you are using a comma when you should be
using a semicolon, it is very important that source code
maintain a high level of clarity and readability. Proper
punctuation, spelling, and grammar help with that
goal.</p>

</div> 

<h3 id="TODO_Comments">TODO Comments</h3>

<div class="summary">
<p>Use <code>TODO</code> comments for code that is temporary,
a short-term solution, or good-enough but not perfect.</p>
</div>

<div class="stylebody">

<p><code>TODO</code>s should include the string
<code>TODO</code> in all caps, followed by the

name, e-mail address, bug ID, or other
identifier
of the person or issue with the best context
about the problem referenced by the <code>TODO</code>. The
main purpose is to have a consistent <code>TODO</code> that
can be searched to find out how to get more details upon
request. A <code>TODO</code> is not a commitment that the
person referenced will fix the problem. Thus when you create
a <code>TODO</code> with a name, it is almost always your
name that is given.</p>



<div>
<pre>// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke) change this to use relations.
// TODO(bug 12345): remove the "Last visitors" feature
</pre>
</div>

<p>If your <code>TODO</code> is of the form "At a future
date do something" make sure that you either include a
very specific date ("Fix by November 2005") or a very
specific event ("Remove this code when all clients can
handle XML responses.").</p>

</div> 

<h3 id="Deprecation_Comments">Deprecation Comments</h3>

<div class="summary">
<p>Mark deprecated interface points with <code>DEPRECATED</code>
comments.</p>
</div>

<div class="stylebody">

<p>You can mark an interface as deprecated by writing a
comment containing the word <code>DEPRECATED</code> in
all caps. The comment goes either before the declaration
of the interface or on the same line as the
declaration.</p>



<p>After the word
<code>DEPRECATED</code>, write your name, e-mail address,
or other identifier in parentheses.</p>

<p>A deprecation comment must include simple, clear
directions for people to fix their callsites. In C++, you
can implement a deprecated function as an inline function
that calls the new interface point.</p>

<p>Marking an interface point <code>DEPRECATED</code>
will not magically cause any callsites to change. If you
want people to actually stop using the deprecated
facility, you will have to fix the callsites yourself or
recruit a crew to help you.</p>

<p>New code should not contain calls to deprecated
interface points. Use the new interface point instead. If
you cannot understand the directions, find the person who
created the deprecation and ask them for help using the
new interface point.</p>



</div> 

<h2 id="Formatting">Formatting</h2>

<p>Coding style and formatting are pretty arbitrary, but a

project is much easier to follow
if everyone uses the same style. Individuals may not agree with every
aspect of the formatting rules, and some of the rules may take
some getting used to, but it is important that all

project contributors follow the
style rules so that 
they can all read and understand
everyone's code easily.</p>



<p>To help you format code correctly, we've
created a
<a href="https://raw.githubusercontent.com/google/styleguide/gh-pages/google-c-style.el">
settings file for emacs</a>.</p>

<h3 id="Line_Length">Line Length</h3>

<div class="summary">
<p>Each line of text in your code should be at most 80
characters long.</p>
</div>

<div class="stylebody">



 <p>We recognize that this rule is
controversial, but so much existing code already adheres
to it, and we feel that consistency is important.</p>

<div class="pros">
<p>Those who favor  this rule
argue that it is rude to force them to resize
their windows and there is no need for anything longer.
Some folks are used to having several code windows
side-by-side, and thus don't have room to widen their
windows in any case. People set up their work environment
assuming a particular maximum window width, and 80
columns has been the traditional standard. Why change
it?</p>
</div>

<div class="cons">
<p>Proponents of change argue that a wider line can make
code more readable. The 80-column limit is an hidebound
throwback to 1960s mainframes;  modern equipment has wide screens that
can easily show longer lines.</p>
</div>

<div class="decision">
<p> 80 characters is the maximum.</p>

<p class="exception">Comment lines can be longer than 80
characters if it is not feasible to split them without
harming readability, ease of cut and paste or auto-linking
-- e.g. if a line contains an example command or a literal
URL longer than 80 characters.</p>

<p class="exception">A raw-string literal may have content
that exceeds 80 characters.  Except for test code, such literals
should appear near the top of a file.</p>

<p class="exception">An <code>#include</code> statement with a
long path may exceed 80 columns.</p>

<p class="exception">You needn't be concerned about
<a href="#The__define_Guard">header guards</a> that exceed
the maximum length. </p>
</div>

</div> 

<h3 id="Non-ASCII_Characters">Non-ASCII Characters</h3>

<div class="summary">
<p>Non-ASCII characters should be rare, and must use UTF-8
formatting.</p>
</div>

<div class="stylebody">

<p>You shouldn't hard-code user-facing text in source,
even English, so use of non-ASCII characters should be
rare. However, in certain cases it is appropriate to
include such words in your code. For example, if your
code parses data files from foreign sources, it may be
appropriate to hard-code the non-ASCII string(s) used in
those data files as delimiters. More commonly, unittest
code (which does not  need to be localized) might
contain non-ASCII strings. In such cases, you should use
UTF-8, since that is  an encoding
understood by most tools able to handle more than just
ASCII.</p>

<p>Hex encoding is also OK, and encouraged where it
enhances readability &#8212; for example,
<code>"\xEF\xBB\xBF"</code>, or, even more simply,
<code>u8"\uFEFF"</code>, is the Unicode zero-width
no-break space character, which would be invisible if
included in the source as straight UTF-8.</p>

<p>Use the <code>u8</code> prefix
to guarantee that a string literal containing
<code>\uXXXX</code> escape sequences is encoded as UTF-8.
Do not use it for strings containing non-ASCII characters
encoded as UTF-8, because that will produce incorrect
output if the compiler does not interpret the source file
as UTF-8. </p>

<p>You shouldn't use the C++11 <code>char16_t</code> and
<code>char32_t</code> character types, since they're for
non-UTF-8 text. For similar reasons you also shouldn't
use <code>wchar_t</code> (unless you're writing code that
interacts with the Windows API, which uses
<code>wchar_t</code> extensively).</p>

</div> 

<h3 id="Spaces_vs._Tabs">Spaces vs. Tabs</h3>

<div class="summary">
<p>Use only spaces, and indent 2 spaces at a time.</p>
</div>

<div class="stylebody">

<p>We use spaces for indentation. Do not use tabs in your
code. You should set your editor to emit spaces when you
hit the tab key.</p>

</div> 

<h3 id="Function_Declarations_and_Definitions">Function Declarations and Definitions</h3>

<div class="summary">
<p>Return type on the same line as function name, parameters
on the same line if they fit. Wrap parameter lists which do
not fit on a single line as you would wrap arguments in a
<a href="#Function_Calls">function call</a>.</p>
</div>

<div class="stylebody">

<p>Functions look like this:</p>


<pre>ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
  DoSomething();
  ...
}
</pre>

<p>If you have too much text to fit on one line:</p>

<pre>ReturnType ClassName::ReallyLongFunctionName(Type par_name1, Type par_name2,
                                             Type par_name3) {
  DoSomething();
  ...
}
</pre>

<p>or if you cannot fit even the first parameter:</p>

<pre>ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
    Type par_name1,  // 4 space indent
    Type par_name2,
    Type par_name3) {
  DoSomething();  // 2 space indent
  ...
}
</pre>

<p>Some points to note:</p>

<ul>
  <li>Choose good parameter names.</li>

  <li>Parameter names may be omitted only if the parameter is unused and its
  purpose is obvious.</li>

  <li>If you cannot fit the return type and the function
  name on a single line, break between them.</li>

  <li>If you break after the return type of a function
  declaration or definition, do not indent.</li>

  <li>The open parenthesis is always on the same line as
  the function name.</li>

  <li>There is never a space between the function name
  and the open parenthesis.</li>

  <li>There is never a space between the parentheses and
  the parameters.</li>

  <li>The open curly brace is always on the end of the last line of the function
  declaration, not the start of the next line.</li>

  <li>The close curly brace is either on the last line by
  itself or on the same line as the open curly brace.</li>

  <li>There should be a space between the close
  parenthesis and the open curly brace.</li>

  <li>All parameters should be aligned if possible.</li>

  <li>Default indentation is 2 spaces.</li>

  <li>Wrapped parameters have a 4 space indent.</li>
</ul>

<p>Unused parameters that are obvious from context may be omitted:</p>

<pre>class Foo {
 public:
  Foo(Foo&amp;&amp;);
  Foo(const Foo&amp;);
  Foo&amp; operator=(Foo&amp;&amp;);
  Foo&amp; operator=(const Foo&amp;);
};
</pre>

<p>Unused parameters that might not be obvious should comment out the variable
name in the function definition:</p>

<pre>class Shape {
 public:
  virtual void Rotate(double radians) = 0;
};

class Circle : public Shape {
 public:
  void Rotate(double radians) override;
};

void Circle::Rotate(double /*radians*/) {}
</pre>

<pre class="badcode">// Bad - if someone wants to implement later, it's not clear what the
// variable means.
void Circle::Rotate(double) {}
</pre>

<p>Attributes, and macros that expand to attributes, appear at the very
beginning of the function declaration or definition, before the
return type:</p>
<pre>MUST_USE_RESULT bool IsOK();
</pre>

</div> 

<h3 id="Formatting_Lambda_Expressions">Lambda Expressions</h3>

<div class="summary">
<p>Format parameters and bodies as for any other function, and capture
lists like other comma-separated lists.</p>
</div>

<div class="stylebody">
<p>For by-reference captures, do not leave a space between the
ampersand (&amp;) and the variable name.</p>
<pre>int x = 0;
auto x_plus_n = [&amp;x](int n) -&gt; int { return x + n; }
</pre>
<p>Short lambdas may be written inline as function arguments.</p>
<pre>std::set&lt;int&gt; blacklist = {7, 8, 9};
std::vector&lt;int&gt; digits = {3, 9, 1, 8, 4, 7, 1};
digits.erase(std::remove_if(digits.begin(), digits.end(), [&amp;blacklist](int i) {
               return blacklist.find(i) != blacklist.end();
             }),
             digits.end());
</pre>

</div> 

<h3 id="Function_Calls">Function Calls</h3>

<div class="summary">
<p>Either write the call all on a single line, wrap the
arguments at the parenthesis, or start the arguments on a new
line indented by four spaces and continue at that 4 space
indent. In the absence of other considerations, use the
minimum number of lines, including placing multiple arguments
on each line where appropriate.</p>
</div>

<div class="stylebody">

<p>Function calls have the following format:</p>
<pre>bool result = DoSomething(argument1, argument2, argument3);
</pre>

<p>If the arguments do not all fit on one line, they
should be broken up onto multiple lines, with each
subsequent line aligned with the first argument. Do not
add spaces after the open paren or before the close
paren:</p>
<pre>bool result = DoSomething(averyveryveryverylongargument1,
                          argument2, argument3);
</pre>

<p>Arguments may optionally all be placed on subsequent
lines with a four space indent:</p>
<pre>if (...) {
  ...
  ...
  if (...) {
    bool result = DoSomething(
        argument1, argument2,  // 4 space indent
        argument3, argument4);
    ...
  }
</pre>

<p>Put multiple arguments on a single line to reduce the
number of lines necessary for calling a function unless
there is a specific readability problem. Some find that
formatting with strictly one argument on each line is
more readable and simplifies editing of the arguments.
However, we prioritize for the reader over the ease of
editing arguments, and most readability problems are
better addressed with the following techniques.</p>

<p>If having multiple arguments in a single line decreases
readability due to the complexity or confusing nature of the
expressions that make up some arguments, try creating
variables that capture those arguments in a descriptive name:</p>
<pre>int my_heuristic = scores[x] * y + bases[x];
bool result = DoSomething(my_heuristic, x, y, z);
</pre>

<p>Or put the confusing argument on its own line with
an explanatory comment:</p>
<pre>bool result = DoSomething(scores[x] * y + bases[x],  // Score heuristic.
                          x, y, z);
</pre>

<p>If there is still a case where one argument is
significantly more readable on its own line, then put it on
its own line. The decision should be specific to the argument
which is made more readable rather than a general policy.</p>

<p>Sometimes arguments form a structure that is important
for readability. In those cases, feel free to format the
arguments according to that structure:</p>
<pre>// Transform the widget by a 3x3 matrix.
my_widget.Transform(x1, x2, x3,
                    y1, y2, y3,
                    z1, z2, z3);
</pre>

</div> 

<h3 id="Braced_Initializer_List_Format">Braced Initializer List Format</h3>

<div class="summary">
<p>Format a <a href="#Braced_Initializer_List">braced initializer list</a>
exactly like you would format a function call in its place.</p>
</div>

<div class="stylebody">

<p>If the braced list follows a name (e.g. a type or
variable name), format as if the <code>{}</code> were the
parentheses of a function call with that name. If there
is no name, assume a zero-length name.</p>

<pre>// Examples of braced init list on a single line.
return {foo, bar};
functioncall({foo, bar});
std::pair&lt;int, int&gt; p{foo, bar};

// When you have to wrap.
SomeFunction(
    {"assume a zero-length name before {"},
    some_other_function_parameter);
SomeType variable{
    some, other, values,
    {"assume a zero-length name before {"},
    SomeOtherType{
        "Very long string requiring the surrounding breaks.",
        some, other values},
    SomeOtherType{"Slightly shorter string",
                  some, other, values}};
SomeType variable{
    "This is too long to fit all in one line"};
MyType m = {  // Here, you could also break before {.
    superlongvariablename1,
    superlongvariablename2,
    {short, interior, list},
    {interiorwrappinglist,
     interiorwrappinglist2}};
</pre>

</div> 

<h3 id="Conditionals">Conditionals</h3>

<div class="summary">
<p>Prefer no spaces inside parentheses. The <code>if</code>
and <code>else</code> keywords belong on separate lines.</p>
</div>

<div class="stylebody">

<p>There are two acceptable formats for a basic
conditional statement. One includes spaces between the
parentheses and the condition, and one does not.</p>

<p>The most common form is without spaces. Either is
fine, but <em>be consistent</em>. If you are modifying a
file, use the format that is already present. If you are
writing new code, use the format that the other files in
that directory or project use. If in doubt and you have
no personal preference, do not add the spaces.</p>

<pre>if (condition) {  // no spaces inside parentheses
  ...  // 2 space indent.
} else if (...) {  // The else goes on the same line as the closing brace.
  ...
} else {
  ...
}
</pre>

<p>If you prefer you may add spaces inside the
parentheses:</p>

<pre>if ( condition ) {  // spaces inside parentheses - rare
  ...  // 2 space indent.
} else {  // The else goes on the same line as the closing brace.
  ...
}
</pre>

<p>Note that in all cases you must have a space between
the <code>if</code> and the open parenthesis. You must
also have a space between the close parenthesis and the
curly brace, if you're using one.</p>

<pre class="badcode">if(condition) {   // Bad - space missing after IF.
if (condition){   // Bad - space missing before {.
if(condition){    // Doubly bad.
</pre>

<pre>if (condition) {  // Good - proper space after IF and before {.
</pre>

<p>Short conditional statements may be written on one
line if this enhances readability. You may use this only
when the line is brief and the statement does not use the
<code>else</code> clause.</p>

<pre>if (x == kFoo) return new Foo();
if (x == kBar) return new Bar();
</pre>

<p>This is not allowed when the if statement has an
<code>else</code>:</p>

<pre class="badcode">// Not allowed - IF statement on one line when there is an ELSE clause
if (x) DoThis();
else DoThat();
</pre>

<p>In general, curly braces are not required for
single-line statements, but they are allowed if you like
them; conditional or loop statements with complex
conditions or statements may be more readable with curly
braces. Some 
projects require that an
<code>if</code> must always always have an accompanying
brace.</p>

<pre>if (condition)
  DoSomething();  // 2 space indent.

if (condition) {
  DoSomething();  // 2 space indent.
}
</pre>

<p>However, if one part of an
<code>if</code>-<code>else</code> statement uses curly
braces, the other part must too:</p>

<pre class="badcode">// Not allowed - curly on IF but not ELSE
if (condition) {
  foo;
} else
  bar;

// Not allowed - curly on ELSE but not IF
if (condition)
  foo;
else {
  bar;
}
</pre>

<pre>// Curly braces around both IF and ELSE required because
// one of the clauses used braces.
if (condition) {
  foo;
} else {
  bar;
}
</pre>

</div> 

<h3 id="Loops_and_Switch_Statements">Loops and Switch Statements</h3>

<div class="summary">
<p>Switch statements may use braces for blocks. Annotate
non-trivial fall-through between cases.
Braces are optional for single-statement loops.
Empty loop bodies should use empty braces or <code>continue</code>.</p>
</div>

<div class="stylebody">

<p><code>case</code> blocks in <code>switch</code>
statements can have curly braces or not, depending on
your preference. If you do include curly braces they
should be placed as shown below.</p>

<p>If not conditional on an enumerated value, switch
statements should always have a <code>default</code> case
(in the case of an enumerated value, the compiler will
warn you if any values are not handled). If the default
case should never execute, simply
<code>assert</code>:</p>

 

<div>
<pre>switch (var) {
  case 0: {  // 2 space indent
    ...      // 4 space indent
    break;
  }
  case 1: {
    ...
    break;
  }
  default: {
    assert(false);
  }
}
</pre>
</div> 





<p> Braces are optional for single-statement loops.</p>

<pre>for (int i = 0; i &lt; kSomeNumber; ++i)
  printf("I love you\n");

for (int i = 0; i &lt; kSomeNumber; ++i) {
  printf("I take it back\n");
}
</pre>


<p>Empty loop bodies should use an empty pair of braces or <code>continue</code>,
but not a single semicolon.</p>

<pre>while (condition) {
  // Repeat test until it returns false.
}
for (int i = 0; i &lt; kSomeNumber; ++i) {}  // Good - one newline is also OK.
while (condition) continue;  // Good - continue indicates no logic.
</pre>

<pre class="badcode">while (condition);  // Bad - looks like part of do/while loop.
</pre>

</div> 

<h3 id="Pointer_and_Reference_Expressions">Pointer and Reference Expressions</h3>

<div class="summary">
<p>No spaces around period or arrow. Pointer operators do not
have trailing spaces.</p>
</div>

<div class="stylebody">

<p>The following are examples of correctly-formatted
pointer and reference expressions:</p>

<pre>x = *p;
p = &amp;x;
x = r.y;
x = r-&gt;y;
</pre>

<p>Note that:</p>

<ul>
  <li>There are no spaces around the period or arrow when
  accessing a member.</li>

   <li>Pointer operators have no space after the
   <code>*</code> or <code>&amp;</code>.</li>
</ul>

<p>When declaring a pointer variable or argument, you may
place the asterisk adjacent to either the type or to the
variable name:</p>

<pre>// These are fine, space preceding.
char *c;
const string &amp;str;

// These are fine, space following.
char* c;
const string&amp; str;
</pre>

It is allowed (if unusual) to declare multiple variables in the same
declaration, but it is disallowed if any of those have pointer or
reference decorations. Such declarations are easily misread.
<pre>// Fine if helpful for readability.
int x, y;
</pre>
<pre class="badcode">int x, *y;  // Disallowed - no &amp; or * in multiple declaration
char * c;  // Bad - spaces on both sides of *
const string &amp; str;  // Bad - spaces on both sides of &amp;
</pre>

<p>You should do this consistently within a single
file,
so, when modifying an existing file, use the style in
that file.</p>

</div> 

<h3 id="Boolean_Expressions">Boolean Expressions</h3>

<div class="summary">
<p>When you have a boolean expression that is longer than the
<a href="#Line_Length">standard line length</a>, be
consistent in how you break up the lines.</p>
</div>

<div class="stylebody">

<p>In this example, the logical AND operator is always at
the end of the lines:</p>

<pre>if (this_one_thing &gt; this_other_thing &amp;&amp;
    a_third_thing == a_fourth_thing &amp;&amp;
    yet_another &amp;&amp; last_one) {
  ...
}
</pre>

<p>Note that when the code wraps in this example, both of
the <code>&amp;&amp;</code> logical AND operators are at
the end of the line. This is more common in Google code,
though wrapping all operators at the beginning of the
line is also allowed. Feel free to insert extra
parentheses judiciously because they can be very helpful
in increasing readability when used
appropriately. Also note that you should always use
the punctuation operators, such as
<code>&amp;&amp;</code> and <code>~</code>, rather than
the word operators, such as <code>and</code> and
<code>compl</code>.</p>

</div> 

<h3 id="Return_Values">Return Values</h3>

<div class="summary">
<p>Do not needlessly surround the <code>return</code>
expression with parentheses.</p>
</div>

<div class="stylebody">

<p>Use parentheses in <code>return expr;</code> only
where you would use them in <code>x = expr;</code>.</p>

<pre>return result;                  // No parentheses in the simple case.
// Parentheses OK to make a complex expression more readable.
return (some_long_condition &amp;&amp;
        another_condition);
</pre>

<pre class="badcode">return (value);                // You wouldn't write var = (value);
return(result);                // return is not a function!
</pre>

</div> 

 

<h3 id="Variable_and_Array_Initialization">Variable and Array Initialization</h3>

<div class="summary">
<p>Your choice of <code>=</code>, <code>()</code>, or
<code>{}</code>.</p>
</div>

<div class="stylebody">

<p>You may choose between <code>=</code>,
<code>()</code>, and <code>{}</code>; the following are
all correct:</p>

<pre>int x = 3;
int x(3);
int x{3};
string name = "Some Name";
string name("Some Name");
string name{"Some Name"};
</pre>

<p>Be careful when using a braced initialization list <code>{...}</code>
on a type with an <code>std::initializer_list</code> constructor.
A nonempty <i>braced-init-list</i> prefers the
<code>std::initializer_list</code> constructor whenever
possible. Note that empty braces <code>{}</code> are special, and
will call a default constructor if available. To force the
non-<code>std::initializer_list</code> constructor, use parentheses
instead of braces.</p>

<pre>std::vector&lt;int&gt; v(100, 1);  // A vector of 100 1s.
std::vector&lt;int&gt; v{100, 1};  // A vector of 100, 1.
</pre>

<p>Also, the brace form prevents narrowing of integral
types. This can prevent some types of programming
errors.</p>

<pre>int pi(3.14);  // OK -- pi == 3.
int pi{3.14};  // Compile error: narrowing conversion.
</pre>

</div> 

<h3 id="Preprocessor_Directives">Preprocessor Directives</h3>

<div class="summary">
<p>The hash mark that starts a preprocessor directive should
always be at the beginning of the line.</p>
</div>

<div class="stylebody">

<p>Even when preprocessor directives are within the body
of indented code, the directives should start at the
beginning of the line.</p>

<pre>// Good - directives at beginning of line
  if (lopsided_score) {
#if DISASTER_PENDING      // Correct -- Starts at beginning of line
    DropEverything();
# if NOTIFY               // OK but not required -- Spaces after #
    NotifyClient();
# endif
#endif
    BackToNormal();
  }
</pre>

<pre class="badcode">// Bad - indented directives
  if (lopsided_score) {
    #if DISASTER_PENDING  // Wrong!  The "#if" should be at beginning of line
    DropEverything();
    #endif                // Wrong!  Do not indent "#endif"
    BackToNormal();
  }
</pre>

</div> 

<h3 id="Class_Format">Class Format</h3>

<div class="summary">
<p>Sections in <code>public</code>, <code>protected</code> and
<code>private</code> order, each indented one space.</p>
</div>

<div class="stylebody">

<p>The basic format for a class definition (lacking the
comments, see <a href="#Class_Comments">Class
Comments</a> for a discussion of what comments are
needed) is:</p>

<pre>class MyClass : public OtherClass {
 public:      // Note the 1 space indent!
  MyClass();  // Regular 2 space indent.
  explicit MyClass(int var);
  ~MyClass() {}

  void SomeFunction();
  void SomeFunctionThatDoesNothing() {
  }

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

 private:
  bool SomeInternalFunction();

  int some_var_;
  int some_other_var_;
};
</pre>

<p>Things to note:</p>

<ul>
  <li>Any base class name should be on the same line as
  the subclass name, subject to the 80-column limit.</li>

  <li>The <code>public:</code>, <code>protected:</code>,
  and <code>private:</code> keywords should be indented
  one space.</li>

  <li>Except for the first instance, these keywords
  should be preceded by a blank line. This rule is
  optional in small classes.</li>

  <li>Do not leave a blank line after these
  keywords.</li>

  <li>The <code>public</code> section should be first,
  followed by the <code>protected</code> and finally the
  <code>private</code> section.</li>

  <li>See <a href="#Declaration_Order">Declaration
  Order</a> for rules on ordering declarations within
  each of these sections.</li>
</ul>

</div> 

<h3 id="Constructor_Initializer_Lists">Constructor Initializer Lists</h3>

<div class="summary">
<p>Constructor initializer lists can be all on one line or
with subsequent lines indented four spaces.</p>
</div>

<div class="stylebody">

<p>The acceptable formats for initializer lists are:</p>

<pre>// When everything fits on one line:
MyClass::MyClass(int var) : some_var_(var) {
  DoSomething();
}

// If the signature and initializer list are not all on one line,
// you must wrap before the colon and indent 4 spaces:
MyClass::MyClass(int var)
    : some_var_(var), some_other_var_(var + 1) {
  DoSomething();
}

// When the list spans multiple lines, put each member on its own line
// and align them:
MyClass::MyClass(int var)
    : some_var_(var),             // 4 space indent
      some_other_var_(var + 1) {  // lined up
  DoSomething();
}

// As with any other code block, the close curly can be on the same
// line as the open curly, if it fits.
MyClass::MyClass(int var)
    : some_var_(var) {}
</pre>

</div> 

<h3 id="Namespace_Formatting">Namespace Formatting</h3>

<div class="summary">
<p>The contents of namespaces are not indented.</p>
</div>

<div class="stylebody">

<p><a href="#Namespaces">Namespaces</a> do not add an
extra level of indentation. For example, use:</p>

<pre>namespace {

void foo() {  // Correct.  No extra indentation within namespace.
  ...
}

}  // namespace
</pre>

<p>Do not indent within a namespace:</p>

<pre class="badcode">namespace {

  // Wrong.  Indented when it should not be.
  void foo() {
    ...
  }

}  // namespace
</pre>

<p>When declaring nested namespaces, put each namespace
on its own line.</p>

<pre>namespace foo {
namespace bar {
</pre>

</div> 

<h3 id="Horizontal_Whitespace">Horizontal Whitespace</h3>

<div class="summary">
<p>Use of horizontal whitespace depends on location. Never put
trailing whitespace at the end of a line.</p>
</div>

<div class="stylebody">

<h4 class="stylepoint_subsection">General</h4>

<pre>void f(bool b) {  // Open braces should always have a space before them.
  ...
int i = 0;  // Semicolons usually have no space before them.
// Spaces inside braces for braced-init-list are optional.  If you use them,
// put them on both sides!
int x[] = { 0 };
int x[] = {0};

// Spaces around the colon in inheritance and initializer lists.
class Foo : public Bar {
 public:
  // For inline function implementations, put spaces between the braces
  // and the implementation itself.
  Foo(int b) : Bar(), baz_(b) {}  // No spaces inside empty braces.
  void Reset() { baz_ = 0; }  // Spaces separating braces from implementation.
  ...
</pre>

<p>Adding trailing whitespace can cause extra work for
others editing the same file, when they merge, as can
removing existing trailing whitespace. So: Don't
introduce trailing whitespace. Remove it if you're
already changing that line, or do it in a separate
clean-up 
operation (preferably when no-one
else is working on the file).</p>

<h4 class="stylepoint_subsection">Loops and Conditionals</h4>

<pre>if (b) {          // Space after the keyword in conditions and loops.
} else {          // Spaces around else.
}
while (test) {}   // There is usually no space inside parentheses.
switch (i) {
for (int i = 0; i &lt; 5; ++i) {
// Loops and conditions may have spaces inside parentheses, but this
// is rare.  Be consistent.
switch ( i ) {
if ( test ) {
for ( int i = 0; i &lt; 5; ++i ) {
// For loops always have a space after the semicolon.  They may have a space
// before the semicolon, but this is rare.
for ( ; i &lt; 5 ; ++i) {
  ...

// Range-based for loops always have a space before and after the colon.
for (auto x : counts) {
  ...
}
switch (i) {
  case 1:         // No space before colon in a switch case.
    ...
  case 2: break;  // Use a space after a colon if there's code after it.
</pre>

<h4 class="stylepoint_subsection">Operators</h4>

<pre>// Assignment operators always have spaces around them.
x = 0;

// Other binary operators usually have spaces around them, but it's
// OK to remove spaces around factors.  Parentheses should have no
// internal padding.
v = w * x + y / z;
v = w*x + y/z;
v = w * (x + z);

// No spaces separating unary operators and their arguments.
x = -5;
++x;
if (x &amp;&amp; !y)
  ...
</pre>

<h4 class="stylepoint_subsection">Templates and Casts</h4>

<pre>// No spaces inside the angle brackets (&lt; and &gt;), before
// &lt;, or between &gt;( in a cast
std::vector&lt;string&gt; x;
y = static_cast&lt;char*&gt;(x);

// Spaces between type and pointer are OK, but be consistent.
std::vector&lt;char *&gt; x;
</pre>

</div> 

<h3 id="Vertical_Whitespace">Vertical Whitespace</h3>

<div class="summary">
<p>Minimize use of vertical whitespace.</p>
</div>

<div class="stylebody">

<p>This is more a principle than a rule: don't use blank
lines when you don't have to. In particular, don't put
more than one or two blank lines between functions,
resist starting functions with a blank line, don't end
functions with a blank line, and be discriminating with
your use of blank lines inside functions.</p>

<p>The basic principle is: The more code that fits on one
screen, the easier it is to follow and understand the
control flow of the program. Of course, readability can
suffer from code being too dense as well as too spread
out, so use your judgement. But in general, minimize use
of vertical whitespace.</p>

<p>Some rules of thumb to help when blank lines may be
useful:</p>

<ul>
  <li>Blank lines at the beginning or end of a function
  very rarely help readability.</li>

  <li>Blank lines inside a chain of if-else blocks may
  well help readability.</li>
</ul>

</div> 

<h2 id="Exceptions_to_the_Rules">Exceptions to the Rules</h2>

<p>The coding conventions described above are mandatory.
However, like all good rules, these sometimes have exceptions,
which we discuss here.</p>

 

<div>
<h3 id="Existing_Non-conformant_Code">Existing Non-conformant Code</h3>

<div class="summary">
<p>You may diverge from the rules when dealing with code that
does not conform to this style guide.</p>
</div>

<div class="stylebody">

<p>If you find yourself modifying code that was written
to specifications other than those presented by this
guide, you may have to diverge from these rules in order
to stay consistent with the local conventions in that
code. If you are in doubt about how to do this, ask the
original author or the person currently responsible for
the code. Remember that <em>consistency</em> includes
local consistency, too.</p>

</div> 
</div> 

 

<h3 id="Windows_Code">Windows Code</h3>

<div class="summary">
<p> Windows
programmers have developed their own set of coding
conventions, mainly derived from the conventions in Windows
headers and other Microsoft code. We want to make it easy
for anyone to understand your code, so we have a single set
of guidelines for everyone writing C++ on any platform.</p>
</div>

<div class="stylebody">
<p>It is worth reiterating a few of the guidelines that
you might forget if you are used to the prevalent Windows
style:</p>

<ul>
  <li>Do not use Hungarian notation (for example, naming
  an integer <code>iNum</code>). Use the Google naming
  conventions, including the <code>.cc</code> extension
  for source files.</li>

  <li>Windows defines many of its own synonyms for
  primitive types, such as <code>DWORD</code>,
  <code>HANDLE</code>, etc. It is perfectly acceptable,
  and encouraged, that you use these types when calling
  Windows API functions. Even so, keep as close as you
  can to the underlying C++ types. For example, use
  <code>const TCHAR *</code> instead of
  <code>LPCTSTR</code>.</li>

  <li>When compiling with Microsoft Visual C++, set the
  compiler to warning level 3 or higher, and treat all
  warnings as errors.</li>

  <li>Do not use <code>#pragma once</code>; instead use
  the standard Google include guards. The path in the
  include guards should be relative to the top of your
  project tree.</li>

  <li>In fact, do not use any nonstandard extensions,
  like <code>#pragma</code> and <code>__declspec</code>,
  unless you absolutely must. Using
  <code>__declspec(dllimport)</code> and
  <code>__declspec(dllexport)</code> is allowed; however,
  you must use them through macros such as
  <code>DLLIMPORT</code> and <code>DLLEXPORT</code>, so
  that someone can easily disable the extensions if they
  share the code.</li>
</ul>

<p>However, there are just a few rules that we
occasionally need to break on Windows:</p>

<ul>
  <li>Normally we <a href="#Multiple_Inheritance">forbid
  the use of multiple implementation inheritance</a>;
  however, it is required when using COM and some ATL/WTL
  classes. You may use multiple implementation
  inheritance to implement COM or ATL/WTL classes and
  interfaces.</li>

  <li>Although you should not use exceptions in your own
  code, they are used extensively in the ATL and some
  STLs, including the one that comes with Visual C++.
  When using the ATL, you should define
  <code>_ATL_NO_EXCEPTIONS</code> to disable exceptions.
  You should investigate whether you can also disable
  exceptions in your STL, but if not, it is OK to turn on
  exceptions in the compiler. (Note that this is only to
  get the STL to compile. You should still not write
  exception handling code yourself.)</li>

  <li>The usual way of working with precompiled headers
  is to include a header file at the top of each source
  file, typically with a name like <code>StdAfx.h</code>
  or <code>precompile.h</code>. To make your code easier
  to share with other projects, avoid including this file
  explicitly (except in <code>precompile.cc</code>), and
  use the <code>/FI</code> compiler option to include the
  file automatically.</li>

  <li>Resource headers, which are usually named
  <code>resource.h</code> and contain only macros, do not
  need to conform to these style guidelines.</li>
</ul>

</div> 

<h2 class="ignoreLink">Parting Words</h2>

<p>Use common sense and <em>BE CONSISTENT</em>.</p>

<p>If you are editing code, take a few minutes to look at the
code around you and determine its style. If they use spaces
around their <code>if</code> clauses, you should, too. If their
comments have little boxes of stars around them, make your
comments have little boxes of stars around them too.</p>

<p>The point of having style guidelines is to have a common
vocabulary of coding so people can concentrate on what you are
saying, rather than on how you are saying it. We present global
style rules here so people know the vocabulary. But local style
is also important. If code you add to a file looks drastically
different from the existing code around it, the discontinuity
throws readers out of their rhythm when they go to read it. Try
to avoid this.</p>



<p>OK, enough writing about writing code; the code itself is much
more interesting. Have fun!</p>

<hr>

</div> 
</div>
</body>
</html>
hing();
}

// When the list spans multiple lines, put each member on its own line
// and align them:
MyClass::MyClass(int var)
    : some_var_(var),             // 4 space indent
      some_other_var_(var + 1) {  // lined up
  DoSomething();
}

// As with any other code block, the close curly can be on the same
// line as the open curly, if it fits.
MyClass::MyClass(int var)
    : some_var_(var) {}
</pre>

</div>

### Namespace Formatting

<div class="summary">

The contents of namespaces are not indented.

</div>

<div class="stylebody">

[Namespaces](#Namespaces) do not add an extra level of indentation. For example, use:

<pre>namespace {

void foo() {  // Correct.  No extra indentation within namespace.
  ...
}

}  // namespace
</pre>

Do not indent within a namespace:

<pre class="badcode">namespace {

  // Wrong.  Indented when it should not be.
  void foo() {
    ...
  }

}  // namespace
</pre>

When declaring nested namespaces, put each namespace on its own line.

<pre>namespace foo {
namespace bar {
</pre>

</div>

### Horizontal Whitespace

<div class="summary">

Use of horizontal whitespace depends on location. Never put trailing whitespace at the end of a line.

</div>

<div class="stylebody">

#### General

<pre>void f(bool b) {  // Open braces should always have a space before them.
  ...
int i = 0;  // Semicolons usually have no space before them.
// Spaces inside braces for braced-init-list are optional.  If you use them,
// put them on both sides!
int x[] = { 0 };
int x[] = {0};

// Spaces around the colon in inheritance and initializer lists.
class Foo : public Bar {
 public:
  // For inline function implementations, put spaces between the braces
  // and the implementation itself.
  Foo(int b) : Bar(), baz_(b) {}  // No spaces inside empty braces.
  void Reset() { baz_ = 0; }  // Spaces separating braces from implementation.
  ...
</pre>

Adding trailing whitespace can cause extra work for others editing the same file, when they merge, as can removing existing trailing whitespace. So: Don't introduce trailing whitespace. Remove it if you're already changing that line, or do it in a separate clean-up operation (preferably when no-one else is working on the file).

#### Loops and Conditionals

<pre>if (b) {          // Space after the keyword in conditions and loops.
} else {          // Spaces around else.
}
while (test) {}   // There is usually no space inside parentheses.
switch (i) {
for (int i = 0; i < 5; ++i) {
// Loops and conditions may have spaces inside parentheses, but this
// is rare.  Be consistent.
switch ( i ) {
if ( test ) {
for ( int i = 0; i < 5; ++i ) {
// For loops always have a space after the semicolon.  They may have a space
// before the semicolon, but this is rare.
for ( ; i < 5 ; ++i) {
  ...

// Range-based for loops always have a space before and after the colon.
for (auto x : counts) {
  ...
}
switch (i) {
  case 1:         // No space before colon in a switch case.
    ...
  case 2: break;  // Use a space after a colon if there's code after it.
</pre>

#### Operators

<pre>// Assignment operators always have spaces around them.
x = 0;

// Other binary operators usually have spaces around them, but it's
// OK to remove spaces around factors.  Parentheses should have no
// internal padding.
v = w * x + y / z;
v = w*x + y/z;
v = w * (x + z);

// No spaces separating unary operators and their arguments.
x = -5;
++x;
if (x && !y)
  ...
</pre>

#### Templates and Casts

<pre>// No spaces inside the angle brackets (< and >), before
// <, or between >( in a cast
std::vector<string> x;
y = static_cast<char*>(x);

// Spaces between type and pointer are OK, but be consistent.
std::vector<char *> x;
</pre>

</div>

### Vertical Whitespace

<div class="summary">

Minimize use of vertical whitespace.

</div>

<div class="stylebody">

This is more a principle than a rule: don't use blank lines when you don't have to. In particular, don't put more than one or two blank lines between functions, resist starting functions with a blank line, don't end functions with a blank line, and be discriminating with your use of blank lines inside functions.

The basic principle is: The more code that fits on one screen, the easier it is to follow and understand the control flow of the program. Of course, readability can suffer from code being too dense as well as too spread out, so use your judgement. But in general, minimize use of vertical whitespace.

Some rules of thumb to help when blank lines may be useful:

*   Blank lines at the beginning or end of a function very rarely help readability.
*   Blank lines inside a chain of if-else blocks may well help readability.

</div>

## Exceptions to the Rules

The coding conventions described above are mandatory. However, like all good rules, these sometimes have exceptions, which we discuss here.

<div>

### Existing Non-conformant Code

<div class="summary">

You may diverge from the rules when dealing with code that does not conform to this style guide.

</div>

<div class="stylebody">

If you find yourself modifying code that was written to specifications other than those presented by this guide, you may have to diverge from these rules in order to stay consistent with the local conventions in that code. If you are in doubt about how to do this, ask the original author or the person currently responsible for the code. Remember that _consistency_ includes local consistency, too.

</div>

</div>

### Windows Code

<div class="summary">

Windows programmers have developed their own set of coding conventions, mainly derived from the conventions in Windows headers and other Microsoft code. We want to make it easy for anyone to understand your code, so we have a single set of guidelines for everyone writing C++ on any platform.

</div>

<div class="stylebody">

It is worth reiterating a few of the guidelines that you might forget if you are used to the prevalent Windows style:

*   Do not use Hungarian notation (for example, naming an integer `iNum`). Use the Google naming conventions, including the `.cc` extension for source files.
*   Windows defines many of its own synonyms for primitive types, such as `DWORD`, `HANDLE`, etc. It is perfectly acceptable, and encouraged, that you use these types when calling Windows API functions. Even so, keep as close as you can to the underlying C++ types. For example, use `const TCHAR *` instead of `LPCTSTR`.
*   When compiling with Microsoft Visual C++, set the compiler to warning level 3 or higher, and treat all warnings as errors.
*   Do not use `#pragma once`; instead use the standard Google include guards. The path in the include guards should be relative to the top of your project tree.
*   In fact, do not use any nonstandard extensions, like `#pragma` and `__declspec`, unless you absolutely must. Using `__declspec(dllimport)` and `__declspec(dllexport)` is allowed; however, you must use them through macros such as `DLLIMPORT` and `DLLEXPORT`, so that someone can easily disable the extensions if they share the code.

However, there are just a few rules that we occasionally need to break on Windows:

*   Normally we [forbid the use of multiple implementation inheritance](#Multiple_Inheritance); however, it is required when using COM and some ATL/WTL classes. You may use multiple implementation inheritance to implement COM or ATL/WTL classes and interfaces.
*   Although you should not use exceptions in your own code, they are used extensively in the ATL and some STLs, including the one that comes with Visual C++. When using the ATL, you should define `_ATL_NO_EXCEPTIONS` to disable exceptions. You should investigate whether you can also disable exceptions in your STL, but if not, it is OK to turn on exceptions in the compiler. (Note that this is only to get the STL to compile. You should still not write exception handling code yourself.)
*   The usual way of working with precompiled headers is to include a header file at the top of each source file, typically with a name like `StdAfx.h` or `precompile.h`. To make your code easier to share with other projects, avoid including this file explicitly (except in `precompile.cc`), and use the `/FI` compiler option to include the file automatically.
*   Resource headers, which are usually named `resource.h` and contain only macros, do not need to conform to these style guidelines.

</div>

## Parting Words

Use common sense and _BE CONSISTENT_.

If you are editing code, take a few minutes to look at the code around you and determine its style. If they use spaces around their `if` clauses, you should, too. If their comments have little boxes of stars around them, make your comments have little boxes of stars around them too.

The point of having style guidelines is to have a common vocabulary of coding so people can concentrate on what you are saying, rather than on how you are saying it. We present global style rules here so people know the vocabulary. But local style is also important. If code you add to a file looks drastically different from the existing code around it, the discontinuity throws readers out of their rhythm when they go to read it. Try to avoid this.

OK, enough writing about writing code; the code itself is much more interesting. Have fun!

* * *

</div>

</div>






# Content to integrate into the guide above

From @algernon:

# [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)

One of the selling points of this guide is that we use the formatting anyway, and it is a thorough guide, covering pretty much all aspects of the code. However, it has a number of issues too, which prevents us from using it as-is.

First of all, it is written with traditional x86-64 libraries and applications in mind, where you control the build system, and so on. This can be easily seen when it talks about [names and order of includes](https://google.github.io/styleguide/cppguide.html#Names_and_Order_of_Includes). We can't name our things like that, because of limitation of the Arduino build system: we can't use `#include <Kaleidoscope/LED/Theme/Something.h>`, or `#include <Kaleidoscope/OneShot.h>`, because Arduino won't find the libraries then. We may be able to build a tool on top of it that would, but then we'd lose the ability to use the Arduino IDE.

The guide discourages static members too (though allows them), along with globals - while we build heavily on those to conserve space, to be friendlier to the end-user, among other things.

The aim of Google's style guide is to make the code better organized, and more understandable for other developers. Our aim is to make the code easier to use for the novice user, for whom their firmware may be the first program they ever create.

## Existing differences

The list below is a collection of issues where our code differs from the recommendation, and where adapting to the guide is not immediately an obvious win. There are other cases where our code differs which I don't list, when adapting to the guide is a no-brainer.

* The guide mandates [lowercase namespace names](https://google.github.io/styleguide/cppguide.html#Namespace_Names), while we (my plugins, mostly) use CamelCase. Lowercase makes sense, making it clear that a namespace is not a class, though.
* We use `#pragma once` instead of [`#define` guards](https://google.github.io/styleguide/cppguide.html#The__define_Guard), but that accomplishes the same thing. Nevertheless, we should use one or the other, so we'd either have to switch, or augment the guide with a note.
* We use preprocessor macros a lot, while the guide [discourages them](https://google.github.io/styleguide/cppguide.html#Preprocessor_Macros).
* We use plenty of non-standard language extensions, while the guide [does not allow them](https://google.github.io/styleguide/cppguide.html#Nonstandard_Extensions). We don't need to care much about portability, because we'll be using GCC anyway. (Or perhaps Clang, which pretty much supports all the same extensions, as far as we are concerned)
* Our naming rules differ: the guide [suggests](https://google.github.io/styleguide/cppguide.html#General_Naming_Rules) snake_case for variables, for example. It also uses [lowercase names for files](https://google.github.io/styleguide/cppguide.html#File_Names). These all stem from the same goal of not using CamelCase by default, it seems. Mind you, reserving CamelCase for classes, and snake_case for variables is not a terrible idea... It does go against Arduino practices as far as I see, though. Even when it comes to CamelCase, Arduino usually goes for `fooBar` for functions, while Google would use `FooBar`. I think the Arduino convention is better here, to distinguish between member functions and classes.

## Things not covered

The guide does not cover naming, in a sense that it only controls how names should look, and does not impose a naming convention otherwise. As in, it does not tell whether to use `addHook` or `hookAdd` (or rather, `AddHook` or `HookAdd`).

## Summary

There are differences between the usual Arduino way, and between Google's guide, but not too much. It feels like we could opt for following Google's guide, with a few exceptions added to cover our use-cases.

---
I looked at a few other guides, but a lot of them are old, or far less comprehensive than Google's one, so my suggestion would be to go with that, with the following exceptions, to be applied on top of it, overriding when in conflict:

### File layout conventions

* We are targeting modern Arduino, and as such, libraries should follow the [rev2.1](https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5:-Library-specification) layout and library specification.
* The main include header must have the same name as the library, and must be in the `src/` directory, as required by Arduino.
  + As we can't namespace public headers, their names should be prefixed with `Kaleidoscope-`.
  + Following Arduino conventions, library names are `CamelCase` - with the first letter being a capital letter too. Dashes may be used to denote namespaceing.

### Headers

* We are using `#pragma once` include guards in all public headers, not `#define` guards.
* Pre-processor macros are valid, and useful constructs, use them for helping the end-user define data structures, or static data to be stored in `PROGMEM`, where dynamic initialization is not an option.

### Non-standard features

* As our target compiler is GCC, use of non-standard extensions, such as compound statements is allowed, though not recommended, unless necessary for optimization purposes. Do it only if you must.

### Naming & organizing things

* Namespaces should be lowercase, and preferably contain no underscores - not top-level namespaces, anyway.
* Class names are `CamelCase`, with the first letter being capital too.
* Data members in classes are `snake_case`, with an underscore at the end if they are not public.
* Function members are `camelCase`, with the first letter being lowercase.
  + Accessors may be `snake_case`, but prefer making the data member public, if it is of a simple type. (For performance and size considerations).
  + Function members should have their verb part first, so `addHook` instead of `hookAdd`. Group with namespaces, if need be, otherwise arrange functions that belong together, together, separate from the rest (in source code).
  + Do not repeat the class or namespace in the member names.
* Variables are `snake_case`, like data members, unless they are global instances of various classes, in which case they are `CamelCase`.
* It is recommended to put the class in a namespace, and when declaring the global, use the same name, but outside of the namespace. For example: `extern Kaleidoscope::Plugin::OneShot OneShot`.

### Indentation and visual style

* Follow the `make astyle` recommendations, and the Google Style Guide.


For context, some of the reasoning behind my proposal:

* File conventions is pretty much what we have now. It is Arduino-compatible, and there's nothing wrong with it. It's just current practice codified.
* Headers & non-standard features similarly.

### Naming things

I think lower-case namespace names make sense, as a way to differentiate classes, global objects, and namespaces. At the moment, `Kaleidoscope` is a global object, `KaleidoscopePlugins` is a namespace, and this is confusing. `kaledioscope::Keyboard`, or `kaleidoscope::Kaleidoscope` as the class is clearer than `Kaleidoscope_`, and we can still have a global `Kaleidoscope` object, because the namespace would be `kaleidoscope`.

Thus:

```c++
namespace kaleidoscope {
  class Kaleidoscope {
  public:
    uint16_t some_variable;

    Kaleidoscope ();

    void addSomething (...);
    void removeSomething (...);

    SomeComplexType foo_bar(); // getter
    void foo_bar(SomeComplexType v); // setter

  private:
    SomeComplexType foo_bar_;
  };
};

extern kaleidoscope::Kaleidoscope Kaleidoscope;
```

This would allow us to get rid of the ugly `KaleidoscopePlugin` namespace. Using namespaces in general would make a lot of code look much nicer. They are not a big thing in the Arduino world, as far as I see, but they are great tools for clarity.

The above code demonstrates all the various ways to name things, and how they make it clearer what is what:

* All classes are namespaced, so start with a `namespace::` prefix when used.
* Classes are always namespaced, and are `CamelCase`.
* All global objects are `CamelCase`. Whether a symbol is a class or an instance is not immediately clear from the name, but the position makes it easy to figure out. As our focus is on the end-user who will rarely - if ever - have to care about classes, and works with objects most of the time, this is fine, and as such, we do not need a more visible distinction.
* Data members are `snake_case`, and have a trailing underscore when not public.
* Data members are public, if setters/getters would be simple assignments or returns, and when access to them from outside is required.
* Function members are `camelCase`, and start with a verb, different from class and global object names that start with a capital letter.
  + Except setter/getter methods, which follow the data member naming convention.

Due to size constraints, and with the goal of being easier for the novice user to get started, we use a lot of global objects, and prefer to avoid inheritance in user-facing APIs. The goal is that the end-user will not have to subclass anything, and that building on top of existing plugins is possible by composing them, as opposed to deriving from them.

-- from @obra:

11:42 <@obra> https://google.github.io/styleguide/cppguide.html#Inline_Functions - Our version of that is going 
              to be more along the lines of "explicitly inline functions when it saves compiled space. For the 
              most part, the compiler will do the right thing without hinting. It is sometimes acceptable to 
              force a function to not be inlined using the GCC extension void __attribute__ ((noinline)) foo() "
