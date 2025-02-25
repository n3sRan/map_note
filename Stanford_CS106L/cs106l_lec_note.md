# Lecture Note

> Stanford FA19 & WI20 CS106L

## 1 Intro

### Goals of CS106L

1. Learn what features are out there in C++ and why they exist.
2. Become comfortable with reading C++ documentation.
3. Become familiar with the design philosophy of modern C++
4. **NOT**: memorize the syntax of C++

### Stages

1. C++  Fundamentals
2. Standard Template Library
3. OOP
4. Modern C++

### Why C++?

- fast
- lower-level control

### C++ History

Assembly

- Benefits
  - simple instruction
  - fast
  - complete control
- Drawbacks
  - requires lots of code to do simple tasks
  - hard to understand
  - unportable

Invention of C

- Benefits
  - fast and simple
  - cross-platform
- Weakness
  - no **object** or **class**
  - difficult to write **generic code**
  - **tedious** when writing large programs

C++

- upper 3 advantage
- **high level features**
- *backwards compatible with lower level languages*

Design Philosophy

- Only add features if they solve an actual problem
- Programmers should be free to choose their own style
- Compartmentalization is key, 将混乱的结构模块化
- Allow the programmer full control if they want it
- Don’t sacrifice performance except as a last resort
- Enforce safety at compile time whenever possible

## 2* Types and Structs

### C++ Basics

STL

- Tons of general functionality
- Built in classes like maps, sets, vectors
- Accessed through the namespace `std::`
- Extremely powerful and well-maintained

Namespaces

- declaration `using namespace std;` will automatically add `std::`
- won't be used at most of time, because it's not good style

### Types

Fundamental Types: int, char, float, double, bool

`std::string`

## 2 Stream

Streams provide a unified interface (统一接口) for interacting with external input.

why can chain?

流式运算符 `>>` `<<` 返回流本身

stringstream positioning functions

Four bits indicate the state of the stream.

- **Good** bit: ready for read/write.
- **Fail** bit: previous operation failed, all future operations frozen.
- **EOF** bit: previous operation reached the end of buffer content.
- **Bad** bit: external error, likely irrecoverable.

Manipulators

- `endl`
- `ws`
- `boolalpha`
- `hex`
- `setpercision`

`cin` is a nightmare

1. `cin` reads the entire line into the buffer but extracts whitespace-separated tokens.
2. Trash in the buffer will make `cin` not prompt the user for input at the right time.
3. When `cin` fails, all future `cin` operations fail too

solve: `getline`

## 3 Modern C++ Types

type aliases

`auto`

- When you don't care what the type is (iterators)
- When its type is clear from context (templates)
- When you don't know what the type is (lambdas)
- Don't use it unnecessarily for return types

## 4 Sequence Containers

Overview of STL

- super extensible
- efficient

Sequence Containers

- Provides access to **sequences** of elements.	
- Includes:
  - `std::vector<T>`
  - `std::deque<T>`
    - push front **faster**
    - access **slower**
  - `std::list<T>`
  - `std::array<T>`
  - `std::forward_list<T>`
- `vec.at(i)` throws an exception
- `vec[i]` causes UB

Container Adaptors

- Stack
  - Just limit the functionality of a vector/deque to only allow `push_back` and `pop_back`.
- Queue
  - Just limit the functionality of a deque to only allow `push_back` and `pop_front`.

Associative Containers

- Have **no idea of a sequence**
- Data is accessed using the **key** instead of indexes
- Includes:
  - `std::map<T1, T2>`
  - `std::set<T>`
  - `std::unorder_map<T1, T2>`
  - `std::unorder_set<T>`

## 5 Iterators

Essential iterator operations.

- Create iterator
- Dereference iterator to read value currently pointed to
- Advance iterator
- Compare against another iterator (especially `.end()` iterator)

Iterator Types

- All iterators share a few common traits:
  - Can be created from existing iterator
  - Can be advanced using `++`
  - Can be compared with `==` and `!=`
- Include (继承关系)
  1. Input
     - For sequential, single-pass input
     - Read only i.e. can only be dereferenced on right side of expression.
     - case: `find(), count()`
  2. Output
     - single-pass output, write only
     - case: `copy()`
  3. Forward
     - R/W, multiple-pass (两个迭代器`A`, `B`指向同一对象, 则`++A`和`++B`也指向同一对象)
  4. Bidirectional
     - 同Forward, 但可以使用自减`--`
     - case: `std::map`, `std::set`, `std::list`,`reverse()`
  5. Random access
     - `+`/`-`
     - case: `std::vector`,`std::deque`, `std::string`

## 6 Templates