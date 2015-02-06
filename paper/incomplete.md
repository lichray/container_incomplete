<!-- maruku -o incomplete.html incomplete.md -->

<style type="text/css">
pre>code { display: block; margin-left: 2em; }
code { white-space: pre-wrap; }
ins { text-decoration: none; font-weight: bold; background-color: #A0FFA0 }
del { text-decoration: line-through; background-color: #FFA0A0 }
</style>

<table><tbody>
<tr><th>Doc. no.:</th>	<td>Nxxxx</td></tr>
<tr><th>Date:</th>	<td>2015-02-04</td></tr>
<tr><th>Revises:</th>	<td>N4056</td></tr>
<tr><th>Project:</th>	<td>Programming Language C++, Library Working Group</td></tr>
<tr><th>Reply-to:</th>	<td>Zhihao Yuan &lt;zy at miator dot net&gt;</td></tr>
</tbody></table>

# Minimal incomplete type support for standard containers, revision 2

## Changes since N4056

- Wording reworked.
- Rationale of the wording is added.

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


## Wording

This wording is relative to N4296.

New section 17.6.3.5.1 &#91;allocator.requirements.completeness&#93;:

> #### 17.6.3.5.1 Allocator completeness requirements &#91;allocator.requirements.completeness&#93;
>
> If `X` is an allocator class for type `T`, `X` additionally satisfies
> the allocator completeness requirements if:
>
>  - `T` may be an incomplete type, and
>  - `X` is a complete type, and
>  - all the member types of `allocator_traits<X>` specified in 20.7.8.1
>    other than `value_type` are complete types.

New paragraph in 20.7.9 &#91;default.allocator&#93;, before the synopsis, as
the first paragraph:

> All specializations of the default allocator satisfy the allocator
> completeness requirements (17.6.3.5.1).

New paragraph in 23.3.4.1 &#91;forwardlist.overview&#93;, as paragraph 4:

> An incomplete type `T` may be used when instantiating `forward_list` if the
> allocator satisfies the allocator completeness requirements (17.6.3.5.1).
> `T` shall be complete before any member of the resulting specialization of
> `forward_list` is referenced.

New paragraph in 23.3.5.1 &#91;list.overview&#93;, as paragraph 3:

> An incomplete type `T` may be used when instantiating `list` if the
> allocator satisfies the allocator completeness requirements (17.6.3.5.1).
> `T` shall be complete before any member of the resulting specialization of
> `list` is referenced.

New paragraph in 23.3.6.1 &#91;vector.overview&#93;, as paragraph 3:

> An incomplete type `T` may be used when instantiating `vector` if the
> allocator satisfies the allocator completeness requirements (17.6.3.5.1).
> `T` shall be complete before any member of the resulting specialization of
> `vector` is referenced.

## Wording Rationale

### Allocator completeness requirements

Let `X` be an allocator type for `T`.

"`T` may be an incomplete type" revokes &#91;res.on.function&#93;/2.5.
`T` could be a type which is not yet complete, or a type which cannot be
incomplete, e.g. `int`.

"`X` is a complete type" is unconditional regarding to the completeness
of `T`.  `X` could be `X<T>`, `X<T, Args...>`, or even a
non-template class `IntAllocator`.  The completeness is required because
a container may store an instance of the allocator.

"all the member types of `allocator_traits<X>`... other than `value_type`
are complete types" allows a container to query minimal information from
the allocator type to finish the class definition.  For a non-template
container, the class definition must not require a complete `T` if `T`
could be incomplete; for a specialization  the outcome of the rule is more
"flexible", since the separately-instantiated entities do not require a
complete `T` when the specialization is implicitly instantiated.

### Container incomplete type support

Let `C` be a sequence container.

"An incomplete type `T` may be used..." revokes &#91;res.on.function&#93;/2.5.
The rule is unconditional regarding to the completeness of `T`.  If the rule
is extended to cover associative/hash containers, the rule may be
unconditional regarding to the completeness of `K` and/or `V`.

"... if the allocator satisfies the allocator completeness requirements"
ensures that the specialization can be instantiated from a user's point of
view; extra template parameters or meta-programming within the class
definition must not make the instantiation ill-formed.
If the rule is extended to cover associative/hash
containers, the rule may be conditional regarding to the completeness
requirements on other template parameters, e.g. `Compare` and `Hash`, as well.

"`T` shall be complete before any member of the resulting specialization...
is referenced." avoids ill-formed programs caused by instantiating
separately-instantiated entities before `T` is complete.  It seldom happens
for a typical recursive definition use case, but if a user try the following,

    struct NT
    {
        C<T> ct;
    };

, he/she has a chance to reference, for example, the default constructor of
`C<T>` after the definition of `NT`, while this rule requires `T` to be
completed before the default constructor is referenced.

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

Special thanks go to Jonathan Wakely, who reviewed this paper many times,
and Richard Smith, who wrote the new wording.
