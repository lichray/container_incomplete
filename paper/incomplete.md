<!-- maruku -o incomplete.html incomplete.md -->

<style type="text/css">
pre>code { display: block; margin-left: 2em; }
code { white-space: pre-wrap; }
ins { text-decoration: none; font-weight: bold; background-color: #A0FFA0 }
del { text-decoration: line-through; background-color: #FFA0A0 }
</style>

<table><tbody>
<tr><th>Doc. no.:</th>	<td>Nxxxx</td></tr>
<tr><th>Date:</th>	<td>2014-11-04</td></tr>
<tr><th>Project:</th>	<td>Programming Language C++, Library Working Group</td></tr>
<tr><th>Reply-to:</th>	<td>Zhihao Yuan &lt;zy at miator dot net&gt;</td></tr>
</tbody></table>

# Minimal incomplete type support for standard containers, revision 1

## Changes since N4056

- The wording for containers is reworked based on LWG's suggestion.
- Core term is being used in the normative text.

The per-container requirement is not refactored because:

1. The reworked wording is short enough to repeat, and too short to refactor.
2. The text needs to place a requirement on each template argument, while
   different categories of containers have different sets of template
   parameters, which makes the text not able to be as general as expected.

## Overview

In the original version
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

 2. Define "allocator completeness requirements" as a dependency of 1).

 3. Require `std::allocator<T>` to unconditionally satisfy the "allocator
    completeness requirements" defined in 2).

Note that no change is needed for `scoped_allocator_adaptor`, since the
allocators passed as template arguments are complete types if they satisfy 2).


## Wording

This wording is relative to N4140.

New section 17.6.3.5.1 &#91;allocator.requirements.completeness&#93;:

> #### 17.6.3.5.1 Allocator completeness requirements &#91;allocator.requirements.completeness&#93;
>
> A library component may want to query the information about an allocation
> model for type `T` even if `T` is an incomplete type.  In such a case, the
> library describes a set of requirements that can be placed on a type `X`,
> which is deemed to be an allocator class for type `T` after `T` is
> completed, so that the component can query for the minimal information
> about the allocation model without making the program ill-formed.
> *\[Note:* The behavior is still undefined (17.6.4.8) if the component is
> _odr-used_ before `T` becomes a complete type.
> *--end note\]*
>
> A type `X` satisfies the allocator completeness requirements if:
>
>  - it is a complete type, and
>  - defines a nested type `value_type` as a synonym for `T`, and
>  - for other nested types defined to satisfy the requirements
>    of Table 28, if any, the types are complete types.

New paragraph in 20.7.9 &#91;default.allocator&#93;, before the synopsis, as
the first paragraph:

> All specializations of the default allocator satisfy the allocator
> completeness requirements (17.6.3.5.1).

New paragraph in 23.3.4.1 &#91;forwardlist.overview&#93;, as paragraph 4:

> If an allocator type `A` satisfies the allocator completeness requirements
> (17.6.3.5.1), an incomplete type `T` may be used when instantiating
> `forward_list`, as long as `T` is completed before
> the specialization `forward_list<T, A>` is _odr-used_.

New paragraph in 23.3.5.1 &#91;list.overview&#93;, as paragraph 3:

> If an allocator type `A` satisfies the allocator completeness requirements
> (17.6.3.5.1), an incomplete type `T` may be used when instantiating
> `list`, as long as `T` is completed before
> the specialization `list<T, A>` is _odr-used_.

New paragraph in 23.3.6.1 &#91;vector.overview&#93;, as paragraph 3:

> If an allocator type `A` satisfies the allocator completeness requirements
> (17.6.3.5.1), an incomplete type `T` may be used when instantiating
> `vector`, as long as `T` is completed before
> the specialization `vector<T, A>` is _odr-used_.

## References

`[1]` Austern, Matthew H.  _The Standard Librarian: Containers of Incomplete
      Types_.
      <http://www.drdobbs.com/the-standard-librarian-containers-of-inc/184403814>

`[2]` _Main features: Containers of Incomplete Types_.
      "Boost.Container"
      <http://www.boost.org/doc/libs/1_55_0/doc/html/container/main_features.html#container.main_features.containers_of_incomplete_types>


## Acknowledgments

Thanks to the library implementers who helped refine the idea, run my
test code, and review the patches and the wording.

Special thanks go to Jonathan Wakely, who helped reviewing this paper
and evaluated the wording word by word, over and over.
