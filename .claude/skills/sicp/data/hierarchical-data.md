# Hierarchical Data

Hierarchical data structures are fundamental to computer science. SICP introduces them through pairs and lists, then shows how to build complex structures like trees and graphs. This document covers the essentials of working with hierarchical data.

## Pairs as Building Blocks

The pair (or cons cell) is the fundamental data structure for building higher-level structures.

### Core Operations

| Operation | Purpose | Returns |
|-----------|---------|---------|
| `cons(a, b)` | Create a pair containing a and b | A pair |
| `car(pair)` | Extract the first element | First element |
| `cdr(pair)` | Extract the second element | Second element |

**Pseudocode Example:**
```
pair = cons(1, 2)
first = car(pair)      // Returns 1
second = cdr(pair)     // Returns 2

// Nested pairs
nested = cons(1, cons(2, cons(3, null)))
car(nested)            // Returns 1
car(cdr(nested))       // Returns 2
car(cdr(cdr(nested)))  // Returns 3
```

The elegance of pairs: complex structures emerge from nesting simple cons operations.

## Box-and-Pointer Notation

Box-and-pointer notation visualizes pair structures as diagrams:

```
cons(1, 2)
┌─┬─┐
│1│2│
└─┴─┘

cons(1, cons(2, 3))
┌─┬───┐
│1│ ·─┼──→ ┌─┬─┐
└─┴───┘     │2│3│
            └─┴─┘

List: cons(1, cons(2, cons(3, null)))
┌─┬──┐     ┌─┬──┐     ┌─┬──┐
│1│·─┼────→│2│·─┼────→│3│∅ │
└─┴──┘     └─┴──┘     └─┴──┘
```

- Box on left holds data
- Pointer on right points to next pair (or null for end of list)
- null (∅) marks the end of a list

## Lists as Chains of Pairs

A list is a chain of pairs where each cdr points to the next pair, and the final cdr is null.

### List Construction

**Pseudocode:**
```
// Direct cons construction
list = cons(1, cons(2, cons(3, null)))

// Equivalent representation
list = [1, 2, 3]

// Single element
single = cons(1, null)

// Empty list
empty = null
```

### Detecting List Structure

**Pseudocode:**
```
function is_list(x)
    return x is null OR (x is a pair AND is_list(cdr(x)))
end function

function is_empty(list)
    return list is null
end function
```

## List Operations

Core operations for working with lists built from pairs.

### Length

Get the number of elements in a list.

**Pseudocode:**
```
function length(list)
    if is_empty(list)
        return 0
    else
        return 1 + length(cdr(list))
    end if
end function

// Iterative version
function length_iter(list, acc)
    if is_empty(list)
        return acc
    else
        return length_iter(cdr(list), acc + 1)
    end if
end function
```

### Append

Combine two lists into one.

**Pseudocode:**
```
function append(list1, list2)
    if is_empty(list1)
        return list2
    else
        return cons(car(list1), append(cdr(list1), list2))
    end if
end function

// Example:
append([1, 2], [3, 4])  // Returns [1, 2, 3, 4]
```

### Reverse

Reverse the order of elements.

**Pseudocode:**
```
function reverse(list)
    return reverse_iter(list, null)
end function

function reverse_iter(list, acc)
    if is_empty(list)
        return acc
    else
        return reverse_iter(cdr(list), cons(car(list), acc))
    end if
end function

// Example:
reverse([1, 2, 3])  // Returns [3, 2, 1]
```

### Nth Element

Access the element at index n (0-indexed).

**Pseudocode:**
```
function nth(list, n)
    if n == 0
        return car(list)
    else
        return nth(cdr(list), n - 1)
    end if
end function

// Example:
nth([1, 2, 3], 0)  // Returns 1
nth([1, 2, 3], 2)  // Returns 3
```

### Map

Apply a function to each element, returning a new list.

**Pseudocode:**
```
function map(f, list)
    if is_empty(list)
        return null
    else
        return cons(f(car(list)), map(f, cdr(list)))
    end if
end function

// Example:
square(x) = x * x
map(square, [1, 2, 3])  // Returns [1, 4, 9]
```

### Filter

Keep only elements that satisfy a predicate.

**Pseudocode:**
```
function filter(predicate, list)
    if is_empty(list)
        return null
    else if predicate(car(list))
        return cons(car(list), filter(predicate, cdr(list)))
    else
        return filter(predicate, cdr(list))
    end if
end function

// Example:
is_even(x) = (x mod 2) == 0
filter(is_even, [1, 2, 3, 4, 5])  // Returns [2, 4]
```

### Fold (Reduce)

Accumulate a value by applying a function to each element.

**Pseudocode:**
```
function fold_left(f, acc, list)
    if is_empty(list)
        return acc
    else
        return fold_left(f, f(acc, car(list)), cdr(list))
    end if
end function

function fold_right(f, acc, list)
    if is_empty(list)
        return acc
    else
        return f(car(list), fold_right(f, acc, cdr(list)))
    end if
end function

// Example:
plus(a, b) = a + b
fold_left(plus, 0, [1, 2, 3, 4])  // Returns 10
```

## Trees as Nested Lists

Trees are represented as nested lists where each element can itself be a list.

### Tree Structure

A tree is either:
1. A leaf (an atomic value)
2. A list whose elements are subtrees

**Pseudocode:**
```
// A tree as nested lists
tree = [1, [2, 3], [4, [5, 6]]]

// Box-and-pointer visualization:
┌──┬──┐
│·─┼──┐
└─┬┘  │
  │   ▼
  │  ┌──┬──┐      ┌──┬──┐
  │  │1 │·─┼─────→│2 │3 │
  │  └──┴──┘      └──┴──┘
  │
  ▼
 ┌──┬──┐      ┌──┬──┐
 │·─┼──┼─────→│4 │·─┼───┐
 └──┴──┘      └──┴──┘   │
                         ▼
                      ┌──┬──┐
                      │5 │6 │
                      └──┴──┘
```

### Tree Predicates

**Pseudocode:**
```
function is_leaf(x)
    return x is not a list
end function

function is_tree(x)
    return is_list(x) AND all(is_tree, x)
end function

// Check if tree is empty
function is_empty_tree(tree)
    return is_empty(tree)
end function
```

## Tree Operations

### Count Leaves

Count all leaf nodes in a tree.

**Pseudocode:**
```
function count_leaves(tree)
    if is_empty(tree)
        return 0
    else if is_leaf(car(tree))
        return 1 + count_leaves(cdr(tree))
    else
        return count_leaves(car(tree)) + count_leaves(cdr(tree))
    end if
end function

// Example:
count_leaves([1, [2, 3], [4, [5, 6]]])  // Returns 6
```

### Scale Tree

Multiply all leaves by a factor.

**Pseudocode:**
```
function scale_tree(tree, factor)
    if is_empty(tree)
        return null
    else if is_leaf(car(tree))
        return cons(car(tree) * factor, scale_tree(cdr(tree), factor))
    else
        return cons(scale_tree(car(tree), factor), scale_tree(cdr(tree), factor))
    end if
end function

// Example:
scale_tree([1, [2, 3], [4, [5, 6]]], 2)
// Returns [2, [4, 6], [8, [10, 12]]]
```

### Map Tree

Apply a function to all leaves.

**Pseudocode:**
```
function map_tree(f, tree)
    if is_empty(tree)
        return null
    else if is_leaf(car(tree))
        return cons(f(car(tree)), map_tree(f, cdr(tree)))
    else
        return cons(map_tree(f, car(tree)), map_tree(f, cdr(tree)))
    end if
end function

// Example:
square(x) = x * x
map_tree(square, [1, [2, 3], [4, [5, 6]]])
// Returns [1, [4, 9], [16, [25, 36]]]
```

### Tree Fold

Accumulate a value from a tree structure.

**Pseudocode:**
```
function fold_tree(f, leaf_f, acc, tree)
    if is_empty(tree)
        return acc
    else if is_leaf(car(tree))
        next_acc = f(leaf_f(car(tree)), acc)
        return fold_tree(f, leaf_f, next_acc, cdr(tree))
    else
        subtree_acc = fold_tree(f, leaf_f, acc, car(tree))
        return fold_tree(f, leaf_f, subtree_acc, cdr(tree))
    end if
end function

// Example: sum all leaves
plus(a, b) = a + b
identity(x) = x
fold_tree(plus, identity, 0, [1, [2, 3], [4, [5, 6]]])
// Returns 21
```

## Sequences as Conventional Interfaces

Sequences (lists) provide a conventional interface for working with hierarchical data. This abstraction separates data structure from operations.

### Sequence Interface

| Operation | Behavior |
|-----------|----------|
| `first(seq)` | Get first element |
| `rest(seq)` | Get all but first element |
| `cons(item, seq)` | Prepend item to sequence |
| `is_empty(seq)` | Check if empty |
| `length(seq)` | Get sequence length |

### Benefits

1. **Uniformity**: Treat nested structures uniformly as sequences
2. **Composability**: Combine sequence operations (map, filter, fold)
3. **Abstraction**: Hide implementation details
4. **Interchangeability**: Swap list representations without changing code

### Pattern: Sequence Operations

**Pseudocode:**
```
// Flatten a tree to a sequence
function flatten(tree)
    if is_empty(tree)
        return null
    else if is_leaf(car(tree))
        return cons(car(tree), flatten(cdr(tree)))
    else
        return append(flatten(car(tree)), flatten(cdr(tree)))
    end if
end function

// Filter leaves by predicate
function filter_leaves(predicate, tree)
    leaves = flatten(tree)
    return filter(predicate, leaves)
end function

// Example:
is_even(x) = (x mod 2) == 0
filter_leaves(is_even, [1, [2, 3], [4, [5, 6]]])
// Returns [2, 4, 6]
```

## Summary Table

| Concept | Key Operations | Primary Use |
|---------|----------------|-------------|
| **Pairs** | cons, car, cdr | Building blocks for all structures |
| **Lists** | cons, car, cdr, map, filter, fold | Sequences of elements |
| **Trees** | is_leaf, count_leaves, map_tree, scale_tree | Hierarchical data with nesting |
| **Sequences** | first, rest, map, filter, fold | Uniform interface for iteration |

### Common Patterns

| Pattern | Implementation Strategy |
|---------|------------------------|
| **Accumulate over structure** | Use fold/reduce with accumulator |
| **Transform all elements** | Use map (works on any structure) |
| **Select elements** | Use filter or filter_leaves |
| **Flatten hierarchy** | Use flatten to create sequence |
| **Count elements** | Use fold with increment or count_leaves |
| **Access by index** | Use nth (linear time) |

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
