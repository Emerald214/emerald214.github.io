---
layout: post
title: To Be Constant or Not to Be Constant
---

Naming a variable is an important job and you have to think twice before pressing down your key. Recently I've found that naming a constant depends on mutability of the variable and its elements (in case it's a collection). From that, we can confidently give it an appropriate name.

```
// Constant
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");

// Not a constant
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
```

Read more at [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html#s5.2.4-constant-names)