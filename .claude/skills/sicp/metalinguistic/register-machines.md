# Register Machines

A register machine is an abstract model of computation that explicitly represents control flow and memory operations. Unlike high-level languages, register machines expose the underlying mechanics of program execution: how data flows between storage locations, how control is passed between instructions, and how subroutines are called and returned.

## Register Machine Model

A register machine consists of three components:

### 1. Registers

Named storage locations that hold data values. Each register can store:
- Numbers
- Symbols
- Lists/vectors (as pointers)
- Other objects

```
Registers: {
  accumulator,
  env,
  val,
  unev,
  continue,
  arg1, arg2, ...
}
```

**Special registers:**
- `accumulator` (or `ac`) - Primary work register for computations
- `env` - Current environment (for variable lookup)
- `continue` - Return address for subroutines
- `val` - Result of evaluated expressions

### 2. Operations

Primitive actions that the machine can perform:
- Arithmetic: `+`, `-`, `*`, `/`
- Comparison: `=`, `<`, `>`, `<=`, `>=`
- List operations: `cons`, `car`, `cdr`
- Memory operations: `vector-ref`, `vector-set!`
- Type checking: `atom?`, `number?`, `list?`

```
Operation: function_call(arg1, arg2, ...)
Assigns result to designated register
```

### 3. Controller

A sequence of instructions that orchestrate:
- When operations execute
- Where results are stored
- Which instruction executes next (control flow)

```
controller_sequence:
  label1: (instruction)
          (instruction)
          (branch to label2 or label3)
  label2: (instruction)
          (goto label1)
```

## Data Paths and Controller

The data path is the set of registers and operations. The controller directs how data flows through the path.

### Example: Simple Arithmetic

```
Machine: Simple-Arithmetic

Registers: {
  a, b, result
}

Operations: {
  +: (a + b) → result,
  -: (a - b) → result
}

Controller:
  start:
    (load-constant 5 → a)
    (load-constant 3 → b)
    (perform +)
    (print result)
    (halt)
```

### Example: GCD (Greatest Common Divisor)

```
Machine: GCD-Machine

Registers: {
  a, b, temp, result
}

Operations: {
  =: (a = b) → boolean,
  remainder: (a mod b) → temp
}

Controller:
  start:
    (load-constant 18 → a)
    (load-constant 12 → b)
  test-equal:
    (test (= a b))
    (branch equal-case)
    (perform remainder)  // a mod b → temp
    (assign a ← b)
    (assign b ← temp)
    (goto test-equal)
  equal-case:
    (assign result ← a)
    (print result)
    (halt)
```

## Machine Language for Register Machines

### Instruction Types

**1. Assignment (Assign)**
```
(assign target ← source)

source can be:
  - A constant: (const value)
  - A register: (reg register-name)
  - Result of operation: (op operation-name) input1 input2 ...
```

**2. Conditional Branch (Test)**
```
(test (op predicate) input1 input2 ...)
(branch label-if-true)
```

**3. Unconditional Jump (Goto)**
```
(goto label)
```

**4. Label**
```
label-name:
```

**5. Perform (Execute operation for side effects)**
```
(perform (op operation) input1 input2 ...)
```

**6. Halt/Terminate**
```
(halt)
```

### Complete Machine Definition Syntax

```
define_machine(name, registers, operations, controller):
  
  registers: [register1, register2, ...]
  
  operations: {
    op-name: function_reference,
    op-name: lambda(x, y) -> x + y,
    ...
  }
  
  controller:
    label1:
      instruction1
      instruction2
      conditional_branch or goto label2
    label2:
      instruction3
      ...
```

## Subroutines and Stack

Subroutines enable code reuse. The key challenge: preserving the return address (where to resume after the subroutine completes).

### Return Address in Continue Register

```
Machine: With-Subroutines

Registers: {
  accumulator,
  n,
  continue
}

Operations: {
  +: (a + b),
  -: (a - b),
  =: (a = b)
}

Controller:
  main:
    (assign continue ← label after-fact)
    (assign n ← 5)
    (goto factorial)
    
  after-fact:
    (print accumulator)
    (halt)
  
  factorial:
    (test (= n 0))
    (branch base-case)
    // Recursive case: n! = n * (n-1)!
    (assign continue ← label after-n-minus-1)
    (assign n ← (- n 1))
    (goto factorial)
    
  after-n-minus-1:
    // accumulator now holds (n-1)!
    (assign accumulator ← (* n accumulator))
    (goto continue)
    
  base-case:
    (assign accumulator ← 1)
    (goto continue)
```

### The Problem with Naive Recursion

Without a stack, recursive calls overwrite the `continue` register, losing intermediate return addresses.

### Stack-Based Solution

```
Machine: Recursive-Factorial

Registers: {
  accumulator,
  n,
  continue,
  stack  // Points to stack memory
}

Operations: {
  push: (stack ← value; stack-ptr++),
  pop: (value ← stack; stack-ptr--),
  stack-ref: (value ← stack[offset])
}

Controller:
  main:
    (assign continue ← label done)
    (assign n ← 5)
    (goto factorial)
    
  done:
    (print accumulator)
    (halt)
  
  factorial:
    (test (= n 0))
    (branch base-case)
    
    // Save continue address and n before recursing
    (perform push-value continue)  // Save return address
    (perform push-value n)         // Save n
    
    (assign n ← (- n 1))
    (assign continue ← label after-recursive-call)
    (goto factorial)
    
  after-recursive-call:
    // accumulator holds (n-1)!
    // Restore n and continue
    (assign n ← pop-value)
    (assign continue ← pop-value)
    
    // Compute n! = n * (n-1)!
    (assign accumulator ← (* n accumulator))
    (goto continue)
    
  base-case:
    (assign accumulator ← 1)
    (goto continue)
```

## Recursive Machine (Factorial)

Complete factorial implementation demonstrating all register machine concepts:

```
Machine: Factorial-Engine

Registers: {
  accumulator,     // Holds intermediate results
  n,               // Current input
  continue,        // Return address
  stack-pointer,   // Points to next stack location
  memory           // Stack memory (conceptually)
}

Operations: {
  =: test-equal,
  -: subtract-one,
  *: multiply,
  push: save-to-stack,
  pop: restore-from-stack
}

Controller:
  start:
    (assign continue ← label print-result)
    (assign n ← 6)
    (assign stack-pointer ← 0)
    (goto fact-loop)
    
  fact-loop:
    (test (= n 0))
    (branch base-case)
    
    // Save state for next iteration
    (perform push)           // stack[sp] ← continue; sp++
    (assign continue ← label after-fact)
    (perform push)           // stack[sp] ← n; sp++
    
    (assign n ← (- n 1))
    (goto fact-loop)
    
  after-fact:
    (perform pop)            // n ← stack[--sp]
    (assign n ← popped-value)
    (perform pop)            // continue ← stack[--sp]
    (assign continue ← popped-value)
    
    (assign accumulator ← (* n accumulator))
    (goto continue)
    
  base-case:
    (assign accumulator ← 1)
    (goto continue)
    
  print-result:
    (perform print accumulator)
    (halt)
```

**Execution trace for n=3:**
1. Start: continue=print, n=3, ac=?
2. Push continue, assign n=2, goto fact-loop
3. Push continue, assign n=1, goto fact-loop
4. Push continue, assign n=0, goto fact-loop
5. Base case: ac=1
6. Pop continue, pop n=1, ac = 1*1 = 1
7. Pop continue, pop n=2, ac = 2*1 = 2
8. Pop continue, pop n=3, ac = 3*2 = 6
9. Print: 6

## Memory and Garbage Collection Basics

### Memory Organization

```
Memory Layout:
┌─────────────────────┐
│   Free Memory       │  (Managed by GC)
├─────────────────────┤
│   Stack             │  (LIFO, grows downward)
│   (Local vars,      │
│    return addrs)    │
├─────────────────────┤
│   Heap              │  (Objects, lists, vectors)
│   (Data structures) │
├─────────────────────┤
│   Code              │  (Instructions)
├─────────────────────┤
│   Constants         │
└─────────────────────┘
```

### Pointer-Based Data

```
Registers can hold pointers (memory addresses):

accumulator: 0x1000  // Points to a cons cell in heap
Operations:
  (perform car)      // Dereference: heap[0x1000].car
  (perform cdr)      // Dereference: heap[0x1000].cdr
```

### Mark-and-Sweep Garbage Collection

```
GC Algorithm:
1. Mark Phase:
   - Start from roots (registers, stack)
   - Follow pointers, mark all reachable objects
   
2. Sweep Phase:
   - Scan entire heap
   - Free unmarked objects
   - Add freed memory to free list

Implementation:
  gc():
    mark_phase()      // Mark reachable objects
    sweep_phase()     // Free unmarked objects
    compact_memory()  // Optional: defragment heap
```

## Compilation: Transforming Programs to Machine Code

### From High-Level to Machine Instructions

Compilation maps high-level constructs to register machine instructions.

**Example: Simple Expression**

High-level:
```
(define (square x) (* x x))
(square 5)
```

Compiled to machine code:
```
Controller:
  evaluate-square-call:
    (assign accumulator ← 5)           // Evaluate argument
    (assign continue ← label after-square)
    (goto square-procedure)
    
  after-square:
    // accumulator holds result (25)
    (print accumulator)
    
  square-procedure:
    // accumulator holds x
    (assign accumulator ← (* accumulator accumulator))
    (goto continue)
```

**Example: Conditional**

High-level:
```
(if (> x 0) (+ x 1) (- x 1))
```

Compiled:
```
Controller:
  evaluate-condition:
    (test (> accumulator 0))
    (branch positive-case)
    
  negative-case:
    (assign accumulator ← (- accumulator 1))
    (goto end-if)
    
  positive-case:
    (assign accumulator ← (+ accumulator 1))
    
  end-if:
    // Continue with rest of code
```

### Compilation Strategy: Evaluator Loop

Most compilers generate code that simulates an evaluator loop:

```
Machine: Universal-Evaluator

Registers: {
  exp,        // Expression to evaluate
  env,        // Environment (bindings)
  val,        // Value of last evaluation
  unev,       // Unevaluated expression list
  continue    // Return address
}

Operations: {
  eval,
  apply,
  lookup-variable-value,
  set-variable-value
}

Controller:
  eval:
    (test (self-evaluating? exp))
    (branch ev-self-eval)
    
    (test (variable? exp))
    (branch ev-variable)
    
    (test (quoted? exp))
    (branch ev-quoted)
    
    (test (assignment? exp))
    (branch ev-assignment)
    
    (test (definition? exp))
    (branch ev-definition)
    
    (test (if? exp))
    (branch ev-if)
    
    (test (procedure? exp))
    (branch ev-lambda)
    
    // Default: application (function call)
    (goto ev-application)
    
  ev-self-eval:
    (assign val ← (exp-value exp))
    (goto continue)
    
  ev-variable:
    (assign val ← (lookup-variable-value exp env))
    (goto continue)
    
  // ... other cases
```

## Lexical Addressing

Rather than looking up variables in environments at runtime, lexical addressing pre-computes their positions during compilation.

### Environment Structure

```
Environment representation:

(define (define-variable! var val frame)
  (set-cdr! frame
    (cons (cons var val)
          (cdr frame))))

(define (lookup-variable-value var env)
  (define (env-loop env)
    (define (scan vars vals)
      (cond ((null? vars) (env-loop (enclosing-environment env)))
            ((eq? var (car vars)) (car vals))
            (else (scan (cdr vars) (cdr vals)))))
    (let ((frame (first-frame env)))
      (scan (frame-variables frame)
            (frame-values frame))))
  (env-loop env))
```

### Lexical Address Computation

At compile time, analyze the structure of environments to determine variable depth and position.

```
Lexical Address Format:
  (depth position)
  
depth:    How many enclosing environments to skip (0 = current)
position: Which variable in that environment (0-indexed)

Example:
  (define (outer x)
    (define (middle y)
      (define (inner z)
        (+ x y z))))  // x has address (2, 0)
                      // y has address (1, 0)
                      // z has address (0, 0)
```

### Fast Lookup with Lexical Addressing

Instead of searching linked list of environments:

```
lookup-variable-value-fast(var, address):
  depth, position = address
  
  frame ← env
  for i in range(depth):
    frame ← enclosing-environment(frame)
  
  return frame[position]
```

### Compilation with Lexical Addressing

```
compile-variable(var, env, target, linkage):
  address ← find-lexical-address(var, env)
  depth, position = address
  
  return(
    assign target
      (list-ref
        (list-ref env depth)
        position)
  )
```

## Pseudocode Examples

### Example 1: Recursive Factorial with Explicit Control

```
define_machine(factorial_machine,
  registers: [n, accumulator, continue, stack_ptr],
  operations: {
    =: equals,
    -: decrement,
    *: multiply,
    push: push_to_stack,
    pop: pop_from_stack
  },
  controller:
    start:
      (assign continue ← label done)
      (assign n ← 5)
      (goto fact-loop)
    
    fact-loop:
      (test (= n 0))
      (branch base-case)
      
      (perform push)
      (assign continue ← label after-recursive-call)
      (perform push)
      (assign n ← (- n 1))
      (goto fact-loop)
    
    after-recursive-call:
      (perform pop)
      (assign n ← popped)
      (perform pop)
      (assign continue ← popped)
      (assign accumulator ← (* n accumulator))
      (goto continue)
    
    base-case:
      (assign accumulator ← 1)
      (goto continue)
    
    done:
      (halt)
)
```

### Example 2: List Processing (Summing a List)

```
define_machine(sum_list_machine,
  registers: [list_ptr, accumulator, continue],
  operations: {
    null?: is_empty_list,
    car: get_head,
    cdr: get_tail,
    +: add
  },
  controller:
    start:
      (assign accumulator ← 0)
      (assign list_ptr ← (const (1 2 3 4 5)))
      (goto sum_loop)
    
    sum_loop:
      (test (null? list_ptr))
      (branch sum_done)
      
      (assign accumulator
        ← (+ accumulator (car list_ptr)))
      (assign list_ptr ← (cdr list_ptr))
      (goto sum_loop)
    
    sum_done:
      (print accumulator)
      (halt)
)
```

### Example 3: Finding Maximum in List

```
define_machine(find_max_machine,
  registers: [list_ptr, max_val, accumulator],
  operations: {
    null?: is_empty,
    car: head,
    cdr: tail,
    >: greater_than
  },
  controller:
    start:
      (assign list_ptr ← (const (3 7 2 9 1)))
      (assign max_val ← (car list_ptr))
      (assign list_ptr ← (cdr list_ptr))
      (goto check_loop)
    
    check_loop:
      (test (null? list_ptr))
      (branch done)
      
      (assign accumulator ← (car list_ptr))
      (test (> accumulator max_val))
      (branch update_max)
      
    skip_update:
      (assign list_ptr ← (cdr list_ptr))
      (goto check_loop)
    
    update_max:
      (assign max_val ← accumulator)
      (goto skip_update)
    
    done:
      (print max_val)
      (halt)
)
```

### Example 4: Evaluator with Environment Lookup

```
define_machine(evaluator,
  registers: [exp, env, val, unev, continue],
  operations: {
    self_evaluating?: check_literal,
    variable?: check_variable,
    quoted?: check_quote,
    lookup_variable_value: env_lookup,
    apply_primitive_procedure: apply_builtin
  },
  controller:
    eval:
      (test (self_evaluating? exp))
      (branch ev_self_eval)
      
      (test (variable? exp))
      (branch ev_variable)
      
      (test (quoted? exp))
      (branch ev_quoted)
      
      // Default: assume it's an application
      (goto ev_application)
    
    ev_self_eval:
      (assign val ← (value_of exp))
      (goto continue)
    
    ev_variable:
      (assign val ← (lookup_variable_value exp env))
      (goto continue)
    
    ev_quoted:
      (assign val ← (quoted_text exp))
      (goto continue)
    
    ev_application:
      // Evaluate procedure and arguments
      (assign unev ← (operands exp))
      (assign exp ← (operator exp))
      (assign continue ← label ev_appl_did_operator)
      (goto eval)
    
    ev_appl_did_operator:
      (assign accumulator ← val)
      // Continue with argument evaluation...
)
```

## Summary Table

| Concept | Purpose | Key Elements |
|---------|---------|--------------|
| **Register** | Storage for data | Named locations, can hold any value |
| **Operation** | Primitive computation | Input → processing → output to register |
| **Controller** | Instruction sequencing | Labels, branches, conditional jumps |
| **Data Path** | Registers + Operations | Defines what machine can compute |
| **Assignment** | Store value in register | `(assign reg ← source)` |
| **Conditional Branch** | Change control flow | `(test predicate)` → `(branch label)` |
| **Goto** | Jump to instruction | Unconditional control transfer |
| **Subroutine** | Reusable code | Needs saved return address |
| **Stack** | LIFO memory | Saves/restores registers during recursion |
| **Return Address** | Where to resume | Stored in `continue` register or stack |
| **Lexical Address** | Variable location | Pre-computed (depth, position) pair |
| **Compilation** | Transform programs | High-level → register machine code |
| **Evaluator** | Universal machine | Interprets programs given via registers |
| **GC (Mark-Sweep)** | Memory management | Mark reachable objects, free unmarked |
| **Pointer** | Reference to heap | Register holds memory address |

## Key Insights

1. **Explicit Control**: Register machines make control flow visible (unlike automatic function calls in high-level languages)

2. **Data vs. Control**: Separating data operations from control flow clarifies what's happening at each step

3. **Stack Discipline**: Without automatic stack management, explicitly saving/restoring registers is critical for recursion

4. **Compilation Strategy**: Most compilers generate code structured like an interpreter loop, testing expression type at each step

5. **Optimization**: Lexical addressing shows how analysis at compile time eliminates expensive runtime lookups

6. **Memory Model**: Explicit registers, stack, and heap clarify where data lives and how it's accessed

7. **Universal Machines**: A single register machine architecture (with appropriate operations) can simulate any other machine

## References for Further Study

- Register machine simulation in Scheme/Lisp
- Implementing the evaluator as a register machine
- Compilation to machine instructions (real instruction sets)
- Garbage collection algorithms in depth
- Performance optimization through lexical analysis

---

*Based on concepts from "Structure and Interpretation of Computer Programs" by Abelson and Sussman.*
