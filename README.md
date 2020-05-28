# QuadraphonicTape.jl
Record more stuff!

## Background

During my work on [IRTracker.jl](https://github.com/phipsgabler/IRTracker.jl), I noticed that for my purposes -- dynamic tracking of computation graphs, including nested functions and control flow -- Cassette is too abstract, and IRTools is too low-level.  

I ended up with an implementation based on IRTools that essentially replicates part of Cassette's interface (`canrecur`, something like `overdub`, contexts), but for more than just function calls.

The goal of this project would be to factor out those parts of IRTracker into a reusable system. 

Currently, it exists only in my head, though.

## Draft

```
geom(n, β) = rand() < β ? n : geom(n + 1, β)

1: (%1, %2, %3)
  %4 = Main.rand()
  %5 = %4 < %3
  br 2 unless %5
  return %2
2:
  %6 = %2 + 1
  %7 = Main.geom(%6, %3)
  return %7
```

should transform into something like

```
1: (%1, %4, %2, %3)
  %5 = quoted(%4, Arg, :(%1), %1)
  %6 = quoted(%4, Arg, :(%2), %2)
  %7 = quoted(%4, Arg, :(%3), %3)
  %8 = quoted(%4, Const, Main.rand)
  %9 = overdub(%4, Call, %8)
  %10 = quoted(%4, Const, <)
  %11 = overdub(%4, Call, %10, %9, %7) 
  %12 = quoted(%4, CondBranch, 2, %11)
  %13 = quoted(%4, Return, %6)
  br 2 (%12) unless %11
  br 3 (%13)
2 (%14):
  %15 = quoted(%4, Arg, :(%14), %14)
  %16 = quoted(%4, Const, +)
  %17 = quoted(%4, Const, 1)
  %18 = overdub(%4, Call, %16, %6, %17)
  %19 = quoted(%4, Const, Main.geom)
  %20 = overdub(%4, Call, %19, %18, %7)
  %21 = quoted(%4, Return, %20)
  br 3 (%21)
3 (%22):
  %22 = overdub(%4, Return, %22)
  return %22
```

where `%4` is the context object.
