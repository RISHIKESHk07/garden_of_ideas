 - [Link](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator)

## MY FINDINGS:

- Introduces the thought about memory allocators and how simply looking it as a MALLOC API and calling it a day without looking at the actual impl of this API and how the code written is tied with its complexity of its impl is not a good approach 
- Mem Management is quite complex when it has complex relations to express and becomes hard for us to make sure we are handling it properly (mem leaks or we free every allocated mem manually ) .
- Lifetimes becomes quite hard to map out to make sure un forseen errors occur ,in-order to combat these we have various techniques like RAII , Garbage collector etc (at a compiler lvl or OS lvl) , which help we proper deconstruction of these allocated mem , though they come with their own flaws , (Garbage collector slows down execution as it needs to steal CPU time to make sure it does its job properly)
- A Few patterns we see in gen codebases:
- 1) Stack or Heap argument: Use Heap when you cannot use Stack kinda situation , this creates a alot of heap allocations without asking some imp qs about the obj created , how many ? , how long is its lifetime ? , dependency ? ... etc .
- 2) Lifetime Soup: Dependency creates further issues as we do not have a good way to represent these (a clear organizing rel) , they form a tangled graph
- 3)Religiously Freeing Memory: Remember mem leaks mean that piece mem cannot be used again throughout the process lifetime we are working with , Leaks are harmful in long-running programs with unbounded allocation , its not harmful in short lived programs but its not smthg we can ignore , Perfect Freeing mem on top of this makes this a tedious process and bug fixing.
- 4)Lifetime Beginning/Ending Asymmetry: Allocation is done in a localised manner but when we want to deconstruct we think in a bulk  scenario.
- The solution (implied by the text) is to design memory around **lifetimes and ownership**, not around individual allocations.
- Ryan 's further talks about how the abstractions hiding complexity does not help as its effect are seen in performance related problems and when dependency kind of situation occurs it becomes quite hard to work out a answer.

```
My approach, on the other hand, is this: instead of _assuming_ that malloc and free were the correct low-level operations, we can _change the memory allocation interface_—tweaking what the user and implementation agree on—to simplify the problem and eliminate many of the problems found in the traditional malloc and free style of memory management. 
```

- Stack allocator
	- With stack allocation, the idea is simple: multiple allocation lifetimes—all using a single block of memory—may be in-flight at a single time, but the _end_ of a lifetime may _never cross_ the beginning of another lifetime. This means that several _nested_ allocation lifetimes may exist, but it is not an entirely arbitrary timeline of overlapping allocation lifetimes (as in the case of `malloc` and `free`).
	- Because of the stack we can define a lifetime differently compared to worrying about a single malloc , and free them asymmetrically like the case 4) from above patterns
	- Imp point is that , we need the start of the new allocation to be never before end of current allocation 's deconstruction
	- The stack fails not because lifetimes are unclear, but because overlapping lifetimes across composable layers cannot be represented by stack nesting at all.
	- In the composability section , when application layer calls the underlying logic (parsing code) , it does not know exactly how much code is required by the internals , blindly allowing all allocations to occurs is not a solution 
- Arena allocator
```
This is the approach of the arena allocator: take the absurdly simple linear allocator, which offers lightning fast allocation and deallocation, eliminating per-allocation freeing requirements, first being proved out by the stack, and make that a formal allocator concept—the “arena”. Usage code can make as many arenas as necessary, and choose them at will for specific allocations.

By getting _just a bit more organized_ about which arena we choose for an allocation, we’ve freed ourselves from the burden of `free`ing all of our dynamic allocations. We’ve also made it much easier to, for instance, track memory usage in our application, or bucket all allocations for a particular purpose, which may be useful for debugging or performance—we now have a fairly obvious path to determine which arena a given allocation is _within_, and to free _all_ allocations in any arena we choose, irrespective of _who_ pushed _what_ onto the arena, _when_, and _in what order_.

A very high level description of an arena is “a handle to which allocations are bound”. When an allocation occurs, it is “bound” to an “arena handle”. This makes it easily expressible to, for instance, clear all allocations “bound” to an “arena handle”

```
- This solves the bulk cleanup code issue by easily removing the arena when we are done instead of worrying about dependency order & manual removal in malloc .
- And lifetimes overlapping is dealt by using the arena of our choice accordingly while in malloc we still need to make sure of this manually and correctly .
