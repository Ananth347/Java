# SEBI IT Exam — Real Questions & Answers (r/SebiGradeA)
> 69 actual questions from SEBI Grade A IT Paper | Study these thoroughly

---

## 📊 Topic-wise Count
| Topic | Questions |
|-------|-----------|
| Data Structures | 16 |
| Algorithms | 14 |
| OOP / Java / C++ | 13 |
| Python | 10 |
| String Manipulation | 8 |
| Graph / Trees | 8 |
| **Total** | **69** |

---

## 🔴 Section 1: Sorting Algorithms

**Q1. What is a 5-regular graph?**
**Answer:** 15 edges
> Handshaking lemma: edges = n × degree / 2. For 6 vertices with degree 5 → 6×5/2 = 15.

**Q2. Equal degree graph is called?**
**Answer:** Regular graph
> A graph where every vertex has the same degree is called a regular graph. k-regular = every vertex has degree k.

**Q3. AVL tree with height = 4 — minimum nodes?**
**Answer:** 12
> N(0)=1, N(1)=2, N(2)=4, N(3)=7, N(4) = N(3)+N(2)+1 = 7+4+1 = **12**
> Formula: N(h) = N(h-1) + N(h-2) + 1

**Q4. Bubble sort — number of comparisons (answer was 14)?**
**Answer:** 14 (for a specific input; worst case = n(n-1)/2)
> Bubble sort worst case O(n²), best case O(n) with optimized version.

**Q5. Sorting elements by adding one by one?**
**Answer:** Insertion sort
> Pick next unsorted element and insert it into the correct position in the sorted portion.

**Q6. Every pass we get the minimum element?**
**Answer:** Selection sort
> Each pass finds the minimum in unsorted portion and swaps it to the correct position.

**Q7. Best algorithm when order of equal elements should NOT change?**
**Answer:** Merge sort (stable sort)
> Stable sorts preserve relative order of equal elements: Merge sort, Insertion sort, Bubble sort.
> Unstable: Quick sort, Heap sort, Selection sort.

**Q8. Employee and designation order should be same — which sorting property?**
**Answer:** Stability
> Stability = equal elements maintain their original relative order after sorting.

**Q9. Quick sort worst case — when does it occur?**
**Answer:** When array is already sorted (pivot always becomes min or max)
> Worst case O(n²). Fix: randomized pivot or median-of-three.

---

## 🟠 Section 2: Data Structures

**Q10. Queue using two stacks — s2 is empty, what happens during dequeue?**
**Answer:** Pop all elements from s1 and push to s2, then pop from s2
> s1 = input stack (enqueue), s2 = output stack (dequeue). Transfer only when s2 is empty.

**Q11. Queue operation to delete elements?**
**Answer:** Dequeue
> Queue = FIFO. Enqueue = add at rear. Dequeue = remove from front. Java: poll() or remove().

**Q14. Best data structure for database indexing?**
**Answer:** B-tree (B+ tree preferred in RDBMS)
> B-tree: all keys in nodes. B+ tree: all data in leaves, better for range queries. Height = O(log n).

**Q19. Java 2D array — how to get number of rows?**
**Answer:** array.length
> `array.length` = rows. `array[0].length` = columns.
> Example: `int[][] arr = new int[3][4]` → arr.length = 3, arr[0].length = 4.

**Q20. BST to hashed conversion problem?**
**Answer:** Out of order (BST ordering property is lost)
> BST gives sorted inorder output. Hash tables have no ordering — conversion loses sorted property.

**Q21. Heap element deletion — heapify top?**
**Answer:** Parent is compared with left and right child, then swapped appropriately
> Delete root → replace with last element → heapify down (compare with both children, swap with larger for max-heap).

**Q22. Hash chaining — load factor means?**
**Answer:** Average number of keys in chains (α = n/m)
> n = number of keys, m = number of slots. Average search time = O(1 + α).

**Q30. Python list pop[0] time complexity?**
**Answer:** O(n) — all elements shift left after removal
> list.pop() from end = O(1). list.pop(0) = O(n). Use `collections.deque` for O(1) popleft().

**Q31. Number of edges in a tree of n vertices?**
**Answer:** n - 1
> A tree is a connected acyclic graph. Always n-1 edges. Full binary tree with L leaves has 2L-1 nodes.

**Q37. Queue implemented in Java in which class?**
**Answer:** LinkedList (implements Queue interface)
> Queue is an interface in java.util. Implemented by: LinkedList, ArrayDeque, PriorityQueue.

**Q54. Which hashing technique uses linked list?**
**Answer:** Separate chaining
> Each slot holds a linked list. Open addressing (linear/quadratic probing) uses the array itself.

**Q55. Nodes in strict/full binary tree when leaves = 8?**
**Answer:** 15 (formula: 2L - 1 = 2×8 - 1 = 15)
> Full binary tree: every node has 0 or 2 children. Internal nodes = L - 1 = 7.

**Q59. How are 1D array elements stored?**
**Answer:** Contiguous memory locations
> Enables O(1) random access. Contrast with linked list: non-contiguous, O(n) access.

**Q62. Time complexity of inserting at beginning of linked list?**
**Answer:** O(1)
> Create node → new.next = head → update head. Constant time regardless of list size.

**Q63. Time complexity of deleting last node of linked list?**
**Answer:** O(n)
> Must traverse to second-last node to update its next = null (singly linked list).

---

## 🟡 Section 3: Graph & Tree Traversals

**Q12. Code given — identify type of traversal (answer was postorder)?**
**Answer:** Postorder — Left → Right → Root
> Inorder: L→Root→R (sorted BST output). Preorder: Root→L→R. Postorder: L→R→Root (Root is LAST).

**Q13. Correct statement about traversal?**
**Answer:** Left subtree is traversed before right subtree
> True for all three traversals (in/pre/post). What changes is WHEN the root is visited.

**Q48. BFS uses which data structure?**
**Answer:** Queue
> BFS = Queue (FIFO, level-by-level). DFS = Stack (LIFO, or recursion = call stack).

**Q49. Best representation for dense graph?**
**Answer:** Adjacency matrix
> Dense graph → adjacency matrix O(V²) space, O(1) edge lookup. Sparse → adjacency list O(V+E).

**Q50. Recursion internally uses which data structure?**
**Answer:** Stack (call stack)
> Each recursive call pushed onto call stack. Stack overflow = recursion too deep.

**Q53. BFS traversal is unique for which structure?**
**Answer:** Linked list
> Linear structure has only one path — traversal is always unique regardless of algorithm.

**Q56. Preorder of BST: 16, 11, 13, 12, 17, 20, 21 — find postorder?**
**Answer:** 12, 13, 11, 21, 20, 17, 16
> Build BST from preorder → 16(root), 11(left), 13(right of 11), 12(left of 13), 17(right of 16), 20(right of 17), 21(right of 20).
> Postorder traversal: 12→13→11→21→20→17→16.

**Q1 (graph). 5-regular graph edges?**
**Answer:** 15 *(see Q1 above)*

---

## 🟢 Section 4: Algorithm Design Techniques

**Q25. Searching in a large sorted array — which algorithm?**
**Answer:** Exponential search
> Find range [1, 2, 4, 8...] then binary search within. Better when element is near start. O(log n).

**Q26. Searching in array with uniform key distribution?**
**Answer:** Interpolation search
> Estimates position by value. O(log log n) for uniform distribution, O(n) worst case.

**Q32. Which is NOT a DP problem?**
**Answer:** Prim's MST (it is Greedy, not DP)
> DP: LCS, 0-1 Knapsack, Coin change, Matrix chain. Greedy: Prim's, Kruskal's, Dijkstra, Huffman.

**Q33. Prim's algorithm approach?**
**Answer:** Pick minimum weight edge(i,j) where i is in tree and j is NOT in tree
> Prim's grows MST from a starting vertex. Kruskal's picks globally minimum edge without cycle.

**Q34. DP has which properties?**
**Answer:** Optimal substructure + Overlapping subproblems (both must hold)
> Greedy: only optimal substructure. D&C: optimal substructure but NO overlapping subproblems.

**Q35. Sudoku solving — which technique?**
**Answer:** Backtracking
> Try all possibilities, abandon paths that violate constraints. Also: N-Queens, Maze solving.

**Q36. N-Queens constraints?**
**Answer:** No two queens in same diagonal, row, or column
> N queens on N×N board — solved via backtracking.

**Q43. Naive pattern search time complexity?**
**Answer:** O(nm) — n = text length, m = pattern length
> Slide pattern over text, compare at each position. KMP = O(n+m). Better alternative.

**Q44. Pattern search based on hash value?**
**Answer:** Rabin-Karp algorithm
> Computes hash of pattern and each window. Uses rolling hash for O(1) window updates.

**Q46. Recurrence relation of merge sort?**
**Answer:** T(n) = 2T(n/2) + n → O(n log n)
> Master theorem: a=2, b=2, f(n)=n → Case 2 → O(n log n). Always O(n log n) — best/worst/average.

**Q66. Convert prefix to postfix: + p q - s t \***
**Answer:** p q + s t - \*
> Prefix *(+pq)(-st) = (p+q)*(s-t). Postfix = pq+ st- \*

**Q69. Time complexity of Strassen's matrix multiplication?**
**Answer:** O(n^2.81) — specifically n^log₂7 ≈ n^2.807
> Standard = O(n³). Strassen uses 7 multiplications instead of 8. Divide & Conquer approach.

---

## 🔵 Section 5: OOP / Java / C++

**Q15. Static parent method — what happens when called from child?**
**Answer:** Parent method is called (method hiding, NOT overriding)
> Static methods belong to class, not object. Dynamic dispatch doesn't apply to static methods.

**Q16. Java 8 backward compatibility feature?**
**Answer:** Default methods in interfaces
> Java 8 added `default` and `static` methods to interfaces — existing implementations don't break.

**Q17. Pure virtual functions in C++?**
**Answer:** Child class must implement (override) them
> Syntax: `virtual void func() = 0;` — makes class abstract. All concrete subclasses must implement.

**Q18. NOT a property of OOP?**
**Answer:** Data compilation
> OOP properties: Encapsulation, Abstraction, Inheritance, Polymorphism. Data compilation ≠ OOP.

**Q24. Java/C# does not support?**
**Answer:** Multiple inheritance (of classes)
> Java/C# avoid diamond problem by disallowing multiple class inheritance. Multiple interfaces = OK. C++ supports multiple inheritance.

**Q29. Data and methods placed in same location — which OOP concept?**
**Answer:** Encapsulation
> Encapsulation = binding data + methods together in a class with access control.
> Abstraction = hiding implementation details (showing only essential features).

**Q47. One base class inherited by multiple child classes?**
**Answer:** Hierarchical inheritance
> 5 types: Single, Multiple (C++ only), Multilevel, Hierarchical (1→many), Hybrid.

**Q51. Java anonymous inner class?**
**Answer:** Class with no name, declared and instantiated at the same time
> Used for one-time implementations of interfaces/abstract classes. Common in event handling.

**Q60. Which OOP principle helps better software design?**
**Answer:** High cohesion and loose coupling
> Cohesion = module does one thing well. Coupling = degree of dependency. Goal: high cohesion, LOW coupling.

**Q67. Method called automatically when creating new Python object?**
**Answer:** `__init__` (constructor/initializer)
> `__new__` creates the object. `__init__` initializes it. Java equivalent = constructor method.

**Q68. Java method for finding a substring?**
**Answer:** `substring(int start, int end)` — end is exclusive
> `"Hello".substring(1,4)` = `"ell"`. Python: `s[1:4]`. C++: `s.substr(1, 3)` (pos, length).

---

## 🟣 Section 6: String Manipulation

**Q38. String length in C?**
**Answer:** `strlen()` — from `<string.h>`
> Key C string functions: strlen, strcpy, strcmp, strcat, strchr, strstr.

**Q39. String length in C++?**
**Answer:** `size()` and `length()` — both identical for std::string
> C++ std::string has both. They are completely interchangeable.

**Q40. Substring finding in C++?**
**Answer:** `substr(pos, length)` — second param is LENGTH not end index
> `str.substr(2, 3)` = 3 chars starting at index 2. ⚠️ Java uses (start, end), C++ uses (start, length).

**Q41. First occurrence of character in C?**
**Answer:** `strchr(str, char)` — returns pointer to first occurrence, NULL if not found
> `strrchr()` for last occurrence. `strstr()` for substring search.

**Q42. First occurrence of string in Java?**
**Answer:** `indexOf()` — `s.indexOf("substring")`
> Java: indexOf (first), lastIndexOf (last). Python: find() or index(). C++: str.find().

**Q61. Python string operation in O(1) time?**
**Answer:** `len()` — Python stores string length as an attribute
> len() = O(1). Slicing = O(k). Search = O(n).

**Q65. s = "Programming", print(len(s) + s.index("m"))?**
**Answer:** 17
> len("Programming") = 11. "m" is at index 6 (P=0,r=1,o=2,g=3,r=4,a=5,m=6). 11 + 6 = **17**.

**Q68 (string context). Java substring method?**
**Answer:** `substring()` *(see OOP section Q68)*

---

## 🐍 Section 7: Python

**Q23. NumPy matrix multiplication operator?**
**Answer:** `A @ B` (@ operator, since Python 3.5)
> `A * B` = element-wise. `A @ B` = matrix multiplication = `np.matmul(A, B)`.

**Q27. s = 'abc', print(s[:-1])?**
**Answer:** `ab` (removes last character)
> s[-1] = 'c'. s[:-1] = everything except last. s[::-1] = 'cba' (reverse).

**Q28. Find substring from index 1 to 4th in Python?**
**Answer:** `S[1:5]` (end index is EXCLUSIVE — use 5 to include index 4)
> ⚠️ TRAP: "1 to 4th" = S[1:5] NOT S[1:4]. End index is always exclusive.

**Q52. Python triple quotes ''' ''' usage?**
**Answer:** Store multiline strings (acts like comment when unassigned, contains \n between lines)
> Python has no official multiline comment. Triple quotes = multiline string literal.

**Q57. data={} → json.dumps(data) → print(type(data))?**
**Answer:** `dictionary` (dict type — json.dumps returns new string, doesn't modify data)
> json.dumps() → JSON string. json.loads() → Python dict. Original variable unchanged.

**Q58. JSON string converted to Python object?**
**Answer:** `json.loads()`
> loads/dumps = for strings. load/dump (without 's') = for file objects.

**Q64. X=[1,2,3], Y=X.append(4), print(len(X))?**
**Answer:** 4
> append() modifies in place, returns None. So Y=None, X=[1,2,3,4], len(X)=4.
> ⚠️ TRAP: Y is None — calling Y.append() later would fail.

**Q65. len(s) + s.index("m") for s="Programming"?**
**Answer:** 17 *(see String section Q65)*

---

## ⚠️ Section 8: Common Traps & Confusions

| Trap | Wrong Assumption | Correct Answer |
|------|-----------------|----------------|
| Static method in Java | Child overrides parent's static | Method hiding — parent called |
| Python slicing "1 to 4th" | S[1:4] | S[1:5] — end is exclusive |
| json.dumps effect | Changes original variable type | Returns new string, original unchanged |
| Quick sort worst case | Random / reverse sorted array | Already sorted array |
| Prim's algorithm type | Dynamic Programming | Greedy |
| list.pop(0) | O(1) | O(n) — elements shift |
| len() in Python | O(n) scan | O(1) — stored as attribute |
| C++ substr 2nd param | End index (like Java) | Length of substring |
| Java multiple inheritance | Supported like C++ | NOT supported (use interfaces) |
| Full binary tree nodes | Unknown formula | 2L - 1 (L = leaves) |
| AVL min nodes height 4 | Calculated wrong | N(h)=N(h-1)+N(h-2)+1 = 12 |

---

## 📝 Quick Reference: String Functions by Language

| Operation | C | C++ | Java | Python |
|-----------|---|-----|------|--------|
| Length | strlen(s) | s.size() / s.length() | s.length() | len(s) |
| Substring | — | s.substr(pos, len) | s.substring(start, end) | s[start:end] |
| Find first char | strchr(s, c) | s.find(c) | s.indexOf(c) | s.find(c) |
| Compare | strcmp(s1,s2) | s1.compare(s2) | s1.equals(s2) | s1 == s2 |
| Concat | strcat(s1,s2) | s1 + s2 | s1.concat(s2) | s1 + s2 |

---

## 📝 Quick Reference: Algorithm Complexities

| Algorithm | Best | Average | Worst | Stable? |
|-----------|------|---------|-------|---------|
| Bubble sort | O(n) | O(n²) | O(n²) | ✅ Yes |
| Insertion sort | O(n) | O(n²) | O(n²) | ✅ Yes |
| Selection sort | O(n²) | O(n²) | O(n²) | ❌ No |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | ✅ Yes |
| Quick sort | O(n log n) | O(n log n) | O(n²) | ❌ No |
| Heap sort | O(n log n) | O(n log n) | O(n log n) | ❌ No |
| Binary search | O(1) | O(log n) | O(log n) | — |
| Naive pattern | O(n) | O(nm) | O(nm) | — |
| KMP | O(n+m) | O(n+m) | O(n+m) | — |
| Rabin-Karp | O(n+m) | O(n+m) | O(nm) | — |

---

*Source: r/SebiGradeA — real SEBI Grade A IT exam questions*
*Prepared for SEBI Phase I & Phase II IT Paper 2*
