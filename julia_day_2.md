### Challenges

**Find**
- The parallel computing part of the Julia manual. Specifically, read up on `@spawn` and `@everywhere`.
	- `@spawn`
		- Run something in parallel on another worker
		- **This macro is deprecated**; `@spawnat :any expr` should be used instead. Create a closure around an expression and run the closure asynchronously on process `p`. Return a [`Future`](https://docs.julialang.org/en/v1/stdlib/Future/#Future) to the result. If `p` is the quoted literal symbol `:any`, then the system will pick a processor to use automatically.
	- `@everywhere`
		- Run a line of code on all processes
		- Execute an expression under `Main` on all `procs`. Errors on any of the processes are collected into a [`CompositeException`](https://docs.julialang.org/en/v1/base/base/#Base.CompositeException) and thrown.
• The Wikipedia page on [multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch).

**Do (Easy):**
- Write a for loop that counts backward using Julia’s range notation.
```julia
julia> for a in reverse(1:10)
           println("$a")
       end

julia> for a in 10:-1:1
	   println("$a")
	   end
```
- Write an iteration over a multidimensional array like [1 2 3; 4 5 6; 7 8 9]. In what order does it get printed out?
```
1, 4, 7... - first column of the matrix
```

- Use pmap to take an array of trial counts and produce the number of heads found for each element.
```julia
julia> pmap(tries -> sum(rand(Bool, tries)), [2, 4, 6])
3-element Vector{Int64}:
 1
 3
 1
```

**Do (Medium):**
- Write a factorial function as a parallel for loop.
	- a factorial is the number of possible combinations with numbers less than or equal to that number

 Non-parallel:
```julia
julia> function factorial(number :: Int64)
           f = 1
           for n in 1:number
               f = f * n
           end
           println(f)
       end
factorial (generic function with 1 method)

julia> factorial(4)
24
```
Parallel:
```julia
@everywhere function p_factorial(number :: Int64)
   @distributed (*) for n in 1:number
	   n
   end
end
```
- Add a method for concat that can concatenate an integer with a matrix. `concat(5, [1 2; 3 4])` should produce [5 5 1 2; 5 5 3 4].
```julia
julia> function concat(x :: Int, A :: Matrix)
       to_concat = fill(x, size(A))
       hcat(to_concat, A)
       end
concat (generic function with 1 method)

julia> concat(5, [1 2; 3 4])
2×4 Matrix{Int64}:
 5  5  1  2
 5  5  3  4
```
- You can extend built-in functions with new methods too. Add a new method for + to make "jul" + "ia" work.
```julia
julia> import Base: +

julia> function +(a :: String, b :: String)
		  "$a$b"
       end
+ (generic function with 163 methods)

julia> "foo" + "bar"
"foobar"
```

**Do (Hard):**
-Parallel for loops dispatch loop bodies to other processes. Depending on the size of the loop body, this can have noticeable overhead. See if you can beat Julia’s parallel for loop version of pflip_coins by writing something using the lower-level primitives like @spawn or remotecall.

Parallel for loop:
```julia
julia> using Distributed

julia> @everwhere function pflip_coins(times)
           @distributed (+) for i = 1:times
               Int(rand(Bool))
           end
       end
julia> @time pflip_coins(1500000000)
  2.207267 seconds (20 allocations: 1.031 KiB)
750033506
```

Threading:
```julia
function flip_coins(times)
   count = 0
   for i = times
	   count += Int(rand(Bool))
   end
   count
end
       
function threaded_flip_coins(times)
  chunks = Iterators.partition(1:times, Int(trunc((length(1:times) / Threads.nthreads()))))
  tasks = map(chunks) do chunk
	  Threads.@spawn flip_coins(chunk)
  end
  chunk_sums = fetch.(tasks)
  return sum(chunk_sums)
end

julia> @time threaded_flip_coins(1500000000)
  0.586965 seconds (36 allocations: 2.281 KiB)
749975411
```
