Optimizing program

Optimization tips:
Make sure there isn’t more than one mutable reference to an object and you use it by last in first out
Make sure you structure loops to take advantage of locality
Get rid of unnecessary function calls by placing outside of loop
Take advantage of multiple functional units in a processor by breaking computations in multiple units

Have multiple accumulators when dealing with parallesmn

High-level design. Choose appropriate algorithms and data structures for the problem at hand. Be especially vigilant to avoid algorithms or coding techniques that yield asymptotically poor performance. 

Basic coding principles. Avoid optimization blockers so that a compiler can generate efficient code. 
1. Eliminate excessive function calls. Move computations out of loops when possible. Consider selective compromises of program modularity to gain greater efficiency. 
2. Eliminate unnecessary memory references. Introduce temporary variables to hold intermediate results. Store a result in an array or global variable only when the final value has been computed. 

Low-level optimizations. Structure code to take advantage of the hardware capabilities. 
1. Unroll loops to reduce overhead and to enable further optimizations. 
2. Find ways to increase instruction-level parallelism by techniques such as multiple accumulators and reassociation.
3. Rewrite conditional operations in a functional style to enable compi- lation via conditional data transfers. 

A final word of advice to the reader is to be vigilant to avoid introducing errors as you rewrite programs in the interest of efficiency. It is very easy to make mistakes when introducing new variables, changing loop bounds, and making the code more complex overall. One useful technique is to use checking code to test each version of a function as it is being optimized, to ensure no bugs are introduced during this process. Checking code applies a series of tests to the new versions of a function and makes sure they yield the same results as the original. The set of test cases must become more extensive with highly optimized code, since there are more cases to consider. For example, checking code that uses loop unrolling requires testing for many different loop bounds to make sure it handles all of the different possible numbers of single-step iterations required at the end. 
