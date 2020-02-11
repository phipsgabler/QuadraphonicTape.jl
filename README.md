# QuadraphonicTape.jl
Record more stuff!

## Background

During my work on [IRTracker.jl](https://github.com/phipsgabler/IRTracker.jl), I noticed that for my purposes -- dynamic tracking of computation graphs, including nested functions and control flow -- Cassette is too abstract, and IRTools is too low-level.  

I ended up with an implementation based on IRTools that essentially replicates part of Cassette's interface (`canrecur`, something like `overdub`, contexts), but for more than just function calls.

The goal of this project would be to factor out those parts of IRTracker into a reusable system. 

Currently, it exists only in my head, though.
