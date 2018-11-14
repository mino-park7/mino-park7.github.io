---
layout: "post"
title: "Data Structure & Implements 2 - Linked List"
date: "2018-11-04 16:55"
category: "자료 구조"
tags: [c++, 자료 구조]
---

# 자료 구조

## 4. Linked List
### 4.1 정의
![Linked List](https://mino-park7.github.io/assets/images/2018/11/linked-list.png)
- A **Linked List** consist of many **nodes**
- A **LINK** contains **address** of (= points to) the **next node**.
- Address of nodes is ***not sequential*** (Address of nodes may *change* on run time)
  - Array와 가장 다른 점
- Size of a linked list is **not predefined**.
- There is a pointer pointing to the **first node** of a linked list.
- There is a special address value(=**NULL**) at the last node

### 4.2 Array와 Linked List의 차이점

#### 4.2.1 Array
- **Sequential** representation
- Insert & Delete : Inefficient due to data shift - **O(n)**
- Size of data must be **predefined**
- **Static** storage allocation (during compile time)

#### 4.2.2 Linked List
- **Non-sequential representation
- Insert & Delete : Efficient due to avoiding data shift - **O(1)**
- Size of data needs **not** to be **predefined**
- **Dynamic** storage allocation (during run time)

### 4.3 Allocation/Deallocation

- (Memory) Allocation
  - Get a currently available node.
  - **malloc** Function
  - ![linked list malloc](https://mino-park7.github.io/assets/images/2018/11/linked-list-malloc.png)
- (Memory) Deallocation
  - Return a no longer unused node
  - **free** function
  - ![linked list free](https://mino-park7.github.io/assets/images/2018/11/linked-list-free.png)

### 4.4 Mechanisms

#### 4.4.1 Simple Insertion

- To insert 'eat' **between** 'cat' and 'fat'
  - ![linked list simple insertion1](https://mino-park7.github.io/assets/images/2018/11/linked-list-simple-insertion1.png)
  - Newly allocate *a node* by a pointer variable `p`, and stroe 'eat' at the node's `DATA`.
  - Assign the `LINK` of `p` to *the node* after **"cat"**, which contains **"fat"**.
  - Assign the `LINK` of ***the node "cat"*** to `p`.
  - ![linked list simple insertion2](https://mino-park7.github.io/assets/images/2018/11/linked-list-simple-insertion2.png)

#### 4.4.2 Simple Deletion

- To delete 'fat' from the **list**
  - ![linked list deletion1](https://mino-park7.github.io/assets/images/2018/11/linked-list-deletion1.png)
  - Assign the `LINK` of **'fat'** node to the `LINK` of ***'cat'*** node
  - Return (*deallocate*) the 'fat' node by using `free`
  - ![linked list deletion2](https://mino-park7.github.io/assets/images/2018/11/linked-list-deletion2.png)

#### 4.4.3 Node Definition
- Define the node, named '`list_node`', by using **struct**
- Each `list_node` consist of two fields, `DATA` and `LINK`
- **list_ptr** is a *pointer type* pointing to the **next `list_node`**
- **prt** is a *pointer variable* defined on **list_ptr*, initially `NULL`
- ![linked list node definition](https://mino-park7.github.io/assets/images/2018/11/linked-list-node-definition.png)

### 4.5 Operators

#### 4.5.1 Basic Operations
- All the variables `p, q, r, ...` are **list_ptr** type
- Initialize Pointer Value:
  - `p = NULL`
- Move to the next node:
  - `p = p -> LINK`
  - ![p->LINK](https://mino-park7.github.io/assets/images/2018/11/p-link.png)
- Assign Pointer value : Sharing the same node
  - `q = p, q = r, ...`
- Compare Pointer Values : **only permit `==, !=`**
  - `if (p == q) then ...`
  - `if (p != q) then ...`
  - `if (p == NULL) then ...`
  - `if (p -> LINK == NULL) then ...`

#### 4.5.2 Insert
- To do this, **three variables*** are required.
  - `ptr` : pointing to the **first node** in a list
  - `ins` : pointing to the **insert position**
  - `new` : pointing to the **new node**
- **Two cases** are considered
  - case 1 : **list** is *empty* (i.e., `ptr` is `NULL`)
  - case 2 : **list** is not empty (i.e., `ptr`)
  - ![prt is null?](https://mino-park7.github.io/assets/images/2018/11/prt-is-null.png)
- ![linked list insert](https://mino-park7.github.io/assets/images/2018/11/linked-list-insert.png)

```c
void Insert (list_ptr *ptr, list_ptr ins) {
  list_ptr new;
  new = (list_ptr) malloc(sizeof(list_node));
  new -> data = 25;
  if (*ptr != NULL) { /*In case, list is not empty*/
    new -> link = ins -> link;
    ins -> link = new;
  }
  else { /*In case, list becomes empty*/
    new -> link = NULL;
    *ptr = new;
  }
}
```
![linked list insert2](https://mino-park7.github.io/assets/images/2018/11/linked-list-insert2.png)

#### 4.5.3 Delete

- To this end, **three variables** are also required.
  - `ptr` : pointing to the **first node** in a list
  - `del` : pointing to the **delete position**
  - `pre` : pointing to the **preceding node** w.r.t the delte node
- Two cases exist w.r.t the delete position
  - case 1 : the fisrt node in a **list** (i.e., **prev** is `NULL`)
  - case 2 : other nodes except the first node (i.e., **ptr** in `not NULL`)
  - ![linked list del](https://mino-park7.github.io/assets/images/2018/11/linked-list-del.png)

```c
void Delete(list_ptr *ptr, list_ptr pre, list_ptr del){
  if (pre != NULL){
    ///In case, the delete node is not the first node
    pre -> link = del -> link;
  }
  else{ ///In case, the delete node is the fisrt node
    *ptr = (*ptr) -> link;
    free(del); ///Deallocate the delete node
  }
}
```

#### 4.5.4 Traverse
![linked list traverse](https://mino-park7.github.io/assets/images/2018/11/linked-list-traverse.png)
