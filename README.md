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
  %6 = overdub(%4, %5)
  %7 = quoted(%4, Arg, :(%2), %2)
  %8 = overdub(%4, %7)
  %9 = quoted(%4, Arg, :(%3), %3)
  %10 = overdub(%4, %9)
  %11 = quoted(%4, Const, Main.rand)
  %12 = quoted(%4, Call, :(%4), %11)
  %13 = overdub(%4, %12)
  %14 = quoted(%4, Const, <)
  %15 = quoted(%4, Call, :(%5), %14, %12, %9) 
  %16 = overdub(%4, %15)
  %17 = quoted(%4, CondBranch, 2, %15)
  %18 = quoted(%4, Return, %7)
  br 2 (%17) unless %16
  br 3 (%18)
2: (%19)
  %20 = overdub(%4, 19)
  %21 = quoted(%4, Const, +)
  %22 = quoted(%4, Const, 1)
  %23 = quoted(%4, Call, :(%6), %21, %7, %22)
  %24 = overdub(%4, %23)
  %25 = quoted(%4, Const, Main.geom)
  %26 = quoted(%4, Call, :(%7), %25, %23, %9)
  %27 = overdub(%4, %26)
  %28 = quoted(%4, Return, %27)
  br 3 (%28)
3 (%29):
  %30 = overdub(%4, %29)
  return %30
```

where `%4` is the context object.
