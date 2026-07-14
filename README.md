# Generalized Data Structures Library — C++

A type-agnostic linked list library implemented in C++ using templates.
Built to understand pointer-based data structure design from first principles —
not to wrap standard library containers, but to implement what those containers
do internally.

---

## Overview

This library implements four linked list variants as fully templated C++ classes.
Each variant exposes the same interface — insert, delete, display, count — so the
same operations work regardless of whether the list is singly or doubly linked,
linear or circular.

The `main()` function demonstrates the library with `int`, `char`, and `double`
instantiations, confirming that a single implementation handles any data type
without modification.

```cpp
SinglyLL<int>    integers;
SinglyLL<char>   characters;
SinglyCL<double> decimals;
DoublyLL<int>    bidirectional;
DoublyCL<int>    bidirectionalCircular;
```

---

## Data Structures

### Node Structures

Two node templates underpin all four list variants:

```cpp
// Used by SinglyLL and SinglyCL
template <class T>
struct NodeS {
    T data;
    struct NodeS<T> *next;
};

// Used by DoublyLL and DoublyCL
template <class T>
struct NodeD {
    T data;
    struct NodeD<T> *next;
    struct NodeD<T> *prev;
};
```

### List Classes

| Class | Structure | Links | Termination |
|---|---|---|---|
| `SinglyLL<T>` | Singly Linked Linear | `next` only | `last->next = NULL` |
| `SinglyCL<T>` | Singly Linked Circular | `next` only | `last->next = first` |
| `DoublyLL<T>` | Doubly Linked Linear | `next` + `prev` | `first->prev = NULL`, `last->next = NULL` |
| `DoublyCL<T>` | Doubly Linked Circular | `next` + `prev` | `last->next = first`, `first->prev = last` |

`SinglyLL` tracks only `first`. `SinglyCL`, `DoublyLL`, and `DoublyCL` each track
both `first` and `last` to support O(1) tail insertion.

---

## API Reference

Every class exposes the same seven operations:

```cpp
void Display();           // Print all elements in order
int  Count();             // Return number of elements

void InsertFirst(T data);           // Insert at head
void InsertLast(T data);            // Insert at tail
void InsertAtPos(T data, int pos);  // Insert at 1-based position

void DeleteFirst();           // Remove head node
void DeleteLast();            // Remove tail node
void DeleteAtPos(int pos);    // Remove node at 1-based position
```

### Behaviour Notes

- **InsertAtPos / DeleteAtPos** validate position bounds and exit with an error
  message if the position is out of range.
- **Circular variants** — `Display()` uses the `last->next == first` sentinel
  to detect end of traversal rather than checking for `NULL`.
- **DoublyLL DeleteFirst / DeleteLast** correctly nullify the orphaned `prev`
  or `next` pointer on the new boundary node.
- **DoublyCL** maintains both `next` and `prev` circular links on every insertion
  and deletion, including the single-node edge case.

---

## Architecture

```
DataStructureLibrary.cpp  (~1056 lines)
│
├── NodeS<T>          — single-link node template
├── NodeD<T>          — double-link node template
│
├── SinglyLL<T>       — linear singly linked list
│   ├── first
│   └── Insert / Delete / Display / Count
│
├── SinglyCL<T>       — circular singly linked list
│   ├── first, last
│   └── Insert / Delete / Display / Count
│
├── DoublyLL<T>       — linear doubly linked list
│   ├── first, last
│   └── Insert / Delete / Display / Count
│
├── DoublyCL<T>       — circular doubly linked list
│   ├── first, last
│   └── Insert / Delete / Display / Count
│
└── main()            — demonstrates all five classes
                        with int, char, and double types
```

---

## Project Structure

```
data-structures-library-cpp/
│
├── Program/
│   └── DataStructureLibrary.cpp   # Complete library implementation
│
└── README.md
```

---

## Build and Run

**Requirements:** GCC or G++ (C++11 or later)

```bash
# Clone the repository
git clone https://github.com/nirajnale/data-structures-library-cpp.git
cd data-structures-library-cpp/Program

# Compile
g++ DataStructureLibrary.cpp -o dsl

# Run
./dsl
```

**Windows:**
```bash
g++ DataStructureLibrary.cpp -o dsl.exe
dsl.exe
```

---

## Usage Example

```cpp
#include <iostream>
using namespace std;

// --- Usage with SinglyLL ---
SinglyLL<int> list;

list.InsertFirst(10);
list.InsertFirst(20);
list.InsertFirst(30);   // list: 30 -> 20 -> 10

list.InsertLast(100);   // list: 30 -> 20 -> 10 -> 100
list.InsertAtPos(55, 3); // list: 30 -> 20 -> 55 -> 10 -> 100

list.Display();
// Output: 30 | 20 | 55 | 10 | 100
cout << "Count: " << list.Count() << endl;
// Output: Count: 5

list.DeleteFirst();     // removes 30
list.DeleteLast();      // removes 100
list.DeleteAtPos(2);    // removes 55

list.Display();
// Output: 20 | 10

// --- Same interface works for any type ---
SinglyLL<char> chars;
chars.InsertFirst('C');
chars.InsertFirst('B');
chars.InsertFirst('A');
chars.Display();
// Output: A | B | C

SinglyCL<double> decimals;
decimals.InsertLast(3.14);
decimals.InsertLast(2.71);
decimals.Display();
// Circular: 3.14 | 2.71 (wraps back to first)
```

---

## Design Decisions

**Why templates instead of `void*`?**
Templates enforce type safety at compile time. A `SinglyLL<int>` cannot accidentally
store a `double`. With `void*`, the compiler cannot catch type errors and every
access requires an unsafe cast. Templates produce the same machine code efficiency
while preserving correctness guarantees.

**Why implement all four variants?**
Each variant represents a distinct structural trade-off:
- Linear vs. circular determines how traversal terminates and whether tail-to-head
  wrapping is possible without explicit tracking.
- Singly vs. doubly linked determines whether backward traversal and O(1) tail
  deletion are supported. Doubly linked adds memory overhead per node (one extra
  pointer) in exchange for bidirectional access.

**Why a single translation unit?**
Template definitions must be visible at the point of instantiation. Splitting
declarations (`.h`) from definitions (`.cpp`) requires explicit instantiation or
inclusion tricks. A single file avoids that complexity while keeping the focus on
the data structure implementations themselves.

---

## What This Demonstrates

- **C++ template programming** — a single class definition generates correct,
  type-safe code for any data type
- **Pointer-based data structure implementation** — manual node allocation,
  pointer manipulation, and memory management without standard library containers
- **Object-oriented design** — encapsulation of node internals, clean public API
- **Edge case handling** — empty list, single-node list, out-of-bounds position,
  and circular link maintenance on both ends
- **Structural trade-off awareness** — four distinct variants demonstrate how
  design choices (directionality, circularity) affect interface and implementation

---

## License

MIT
