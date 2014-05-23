<!-- maruku -o incomplete.html incomplete.md -->

<style type="text/css">
pre>code { display: block; margin-left: 2em; }
code { white-space: pre-wrap; }
ins { text-decoration: none; font-weight: bold; background-color: #A0FFA0 }
del { text-decoration: line-through; background-color: #FFA0A0 }
</style>

<table><tbody>
<tr><th>Doc. no.:</th>	<td>Dnnnn</td></tr>
<tr><th>Date:</th>	<td>2014-5-23</td></tr>
<tr><th>Project:</th>	<td>Programming Language C++, Library Evolution Working Group</td></tr>
<tr><th>Reply-to:</th>	<td>Zhihao Yuan &lt;zy at miator dot net&gt;</td></tr>
</tbody></table>

# Minimal incomplete type support for standard containers

## Overview

In the previous version
([N3890](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2014/n3890.html))
of this paper, we explored the possibility to make all STL containers usable
in the recursive data structure definitions, such as

    struct Entry
    {
        std::list<Entry> messages;
        // ...
    };

Based on the discussion on the Issaquah meeting, we achieved the consensus to
processed with the approach -- "Containers of Incomplete Types", but limit
the scope to `std::vector`, `std::list`, and `std::forward_list`, as the
first step.

The approach itself is well-known shortly
after the C++98 standard being shipped`[1]`, is one of the main features
provided by Boost.Container`[2]` and libstdc++, and is receiving increasing
interests in libc++.  By limiting the scope, MSVC STL, libstdc++, and libc++
already support the proposed solution.


## Impact on the Standard

 1. Conditionally require the targeted containers to be complete types.

 2. Define "Allocator completeness requirements" as a dependency of 1).

 3. Require `std::allocator<T>` to unconditionally satisfy the "Allocator
    completeness requirements" defined in 2).


#### Allocator completeness requirements

A type `X` satisfies the Allocator completeness requirements if:

 - it is a complete type, and
 - `X::value_type` is defined and identical to `T`.


#### Sequence containers

A sequence container with an `Allocator` template parameter is a complete type
if:

 - the argument of `Allocator` satisfies the Allocator completeness
   requirements.


## References

`[1]` Austern, Matthew H.  _The Standard Librarian: Containers of Incomplete
      Types_.
      <http://www.drdobbs.com/the-standard-librarian-containers-of-inc/184403814>

`[2]` _Main features: Containers of Incomplete Types_.
      "Boost.Container"
      <http://www.boost.org/doc/libs/1_55_0/doc/html/container/main_features.html#container.main_features.containers_of_incomplete_types>


## Acknowledgments

Thanks to the library implementers who helped refine the idea, run my crappy
test code, and review the patches publicly and/or privately.
