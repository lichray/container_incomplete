<!-- maruku -o incomplete.html incomplete.md -->

<style type="text/css">
pre>code { display: block; margin-left: 2em; }
code { white-space: pre-wrap; }
ins { text-decoration: none; font-weight: bold; background-color: #A0FFA0 }
del { text-decoration: line-through; background-color: #FFA0A0 }
</style>

<table><tbody>
<tr><th>Doc. no.:</th>	<td>Dnnnn</td></tr>
<tr><th>Date:</th>	<td>2014-1-19</td></tr>
<tr><th>Project:</th>	<td>Programming Language C++, Library Evolution Working Group</td></tr>
<tr><th>Reply-to:</th>	<td>Zhihao Yuan &lt;zy at miator dot net&gt;</td></tr>
</tbody></table>

# Container&lt;Incomplete Type&gt;

## Overview

    struct Entry
    {
        std::deque<Entry> messages;
        // ...
    };

The goal of this paper is to allow recursive data structure definitions
with STL containers, while to make the STL container instantiations well-formed
even when some of the template parameters are incomplete types (in contrast to
17.6.4.8/2) is an approach to achieve the goal without breaking any language
restrictions or existing practices.

The approach, namely "Containers of Incomplete Types", was well-known shortly
after the C++98 standard being shipped`[1]`, is one of the main features
provided by Boost.Container`[2]` and libstdc++, and is receiving increasing
interest in libc++.

Some workarounds have been discussed, including using a container of smart
pointers.  However, the proposed solution has the following significant
advantages:

 1. The value semantics of `value_type` is well-preserved;
 2. scoped allocators work out-of-box;
 3. such a container can be used in type erasure.


## Impact on the Standard

 1. An additional "Completeness" requirement for each category of the
    containers -- Sequence, Associative, Unordered associative, and Container
    adaptors, plus the Hash requirements and Allocator requirements.

 2. Require the unspecialized and transparent function objects (`std::less<T>`
    and `std::less<>`, for example, respectively) to be complete types
    unconditionally.

 3. Require `std::allocator<T>` to unconditionally satisfy the "Allocator
    completeness requirements" in 1).

2) and 3) are trivial to the existing implementations, but not 1).  Some
changes involve ABI breakages, and some changes result in less compile-time
checking, but run-time performance will not be affected (interestingly, the
situation is similar to that of the SCARY iterators`[3]`, as well as some
techniques we are using, plus some goals we want to achieve from a type
system's point of view).

Currently,

 * libc++ can fully satisfy the proposed solution after being patched (see
   [Sample Implementation](#sample_implementation), but with an ABI breakage
   on `std::deque`;

 * libstdc++ can satisfy the proposed solution with some untested changes made
   to the unordered associative containers;

 * MSVC STL perfers to perserve some compile-time constants for debugger
   visualization on `std::deque` (read as "the solution remains unknown"), and
   is interested in patching `std::list` and `std::forward_list`.


## Design Decisions

The proposed feature set is chosen in a "least common multiple" (of the
existing implementations) manner.  All the possible (no `std::array`) and
useful cases are conditionally covered, so that only the standard library
implementations may be affected, but not user code.  For example, if a
customized `MyLess` is not a complete type, the whole `std::set<T, MyLess>`
specialization is not well-formed.


## Technical Specifications

## Sample Implementation

The proposed solution has been implemented (as a llvm/libc++ fork) and well-
tested: <https://github.com/lichray/libcxx/tree/container_incomplete>


## References
