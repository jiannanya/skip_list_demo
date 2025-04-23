# skip_list_demo

## 跳表原理

跳表（Skip List）是一种随机化的数据结构，它通过在链表的基础上增加多级索引来提高查找、插入和删除操作的效率。下面我会用示意图来详细解释跳表的结构和工作原理。

**基本链表结构**  
普通的链表是一种线性数据结构，每个节点包含一个数据元素和一个指向下一个节点的指针。查找操作需要从头节点开始，依次遍历每个节点，直到找到目标元素或者到达链表末尾。因此，查找的时间复杂度是 $O(n)$。

下面是一个简单的链表示意图，包含元素 1、2、3、4、5：
```plaintext
  +---+    +---+    +---+    +---+    +---+
  | 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 |
  +---+    +---+    +---+    +---+    +---+
```

**跳表的多级索引结构**  
跳表通过在链表的基础上增加多级索引来提高查找效率。每一级索引都是一个稀疏的链表，它跳过了一些节点，使得查找时可以更快地定位到目标元素。

假设我们有一个跳表，它有三级索引。最底层是原始的链表，每一级索引都是上一级索引的一半。下面是一个跳表示意图：
```plaintext
Level 2:  +---+             +---+             +---+
          | 1 | ----------> | 3 | ----------> | 5 |
          +---+             +---+             +---+
Level 1:  +---+       +---+ +---+       +---+ +---+
          | 1 | ----> | 2 | | 3 | ----> | 4 | | 5 |
          +---+       +---+ +---+       +---+ +---+
Level 0:  +---+ +---+ +---+ +---+ +---+ +---+ +---+
          | 1 | | 2 | | 3 | | 4 | | 5 | | 6 | | 7 |
          +---+ +---+ +---+ +---+ +---+ +---+ +---+
```

**查找操作**  
假设我们要在上面的跳表中查找元素 4。查找过程如下：
1. 从最高层（Level 2）的头节点开始，比较当前节点的值和目标值。由于 1 < 4，我们继续向右移动到节点 3。
2. 由于 3 < 4，我们继续向右移动，但发现没有更多的节点了，于是我们下降到下一层（Level 1）。
3. 在 Level 1 中，从节点 3 开始，由于 3 < 4，我们向右移动到节点 4，找到了目标元素。

整个查找过程的时间复杂度是 $O(\log n)$，因为每一级索引都跳过了大约一半的节点。

**插入操作**  
插入操作的步骤如下：
1. 首先进行查找操作，找到要插入的位置。
2. 随机决定新节点的层数。例如，假设新节点的层数为 2。
3. 在相应的层中插入新节点，并更新指针。

假设我们要插入元素 6，并且新节点的层数为 2。插入后的跳表示意图如下：
```plaintext
Level 2:  +---+             +---+             +---+             +---+
          | 1 | ----------> | 3 | ----------> | 5 | ----------> | 6 |
          +---+             +---+             +---+             +---+
Level 1:  +---+       +---+ +---+       +---+ +---+       +---+ +---+
          | 1 | ----> | 2 | | 3 | ----> | 4 | | 5 | ----> | 6 | | 7 |
          +---+       +---+ +---+       +---+ +---+       +---+ +---+
Level 0:  +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+
          | 1 | | 2 | | 3 | | 4 | | 5 | | 6 | | 7 | | 8 | | 9 | |10 |
          +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+
```

**删除操作**  
删除操作的步骤如下：
1. 首先进行查找操作，找到要删除的节点。
2. 在相应的层中删除该节点，并更新指针。
3. 如果某一层的节点被删除后为空，则删除该层。

假设我们要删除元素 4。删除后的跳表示意图如下：
```plaintext
Level 2:  +---+             +---+             +---+             +---+
          | 1 | ----------> | 3 | ----------> | 5 | ----------> | 6 |
          +---+             +---+             +---+             +---+
Level 1:  +---+       +---+ +---+       +---+ +---+       +---+ +---+
          | 1 | ----> | 2 | | 3 | ----> | 5 | | 6 | ----> | 7 | | 8 |
          +---+       +---+ +---+       +---+ +---+       +---+ +---+
Level 0:  +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+
          | 1 | | 2 | | 3 | | 5 | | 6 | | 7 | | 8 | | 9 | |10 | |11 |
          +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+
```

通过多级索引，跳表在平均情况下可以在 $O(\log n)$ 的时间复杂度内完成查找、插入和删除操作，同时保持了链表的动态性和简单性。

## 跳表代码说明

下面将对上述跳表（Skip List）的 C++ 实现代码进行详细讲解。

**整体概述**  
跳表是一种随机化的数据结构，它通过维护多层链表来实现高效的查找、插入和删除操作，平均时间复杂度为 $O(\log n)$。此代码包含了跳表节点类 `SkipListNode` 和跳表类 `SkipList`，并在 `main` 函数中展示了如何使用这些操作。

**代码详细解释**  

**常量和头文件部分**  
```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>

const int MAX_LEVEL = 16;
```
- `#include <iostream>`：用于输入输出操作。
- `#include <cstdlib>`：提供了随机数生成函数 `rand()` 等。
- `#include <ctime>`：用于获取当前时间，作为随机数种子。
- `const int MAX_LEVEL = 16`：定义了跳表的最大层数。

**跳表节点类 `SkipListNode`**  
```cpp
template <typename T>
class SkipListNode {
public:
    T key;
    SkipListNode** forward;

    SkipListNode(T k, int level) : key(k) {
        forward = new SkipListNode*[level + 1];
        for (int i = 0; i <= level; ++i) {
            forward[i] = nullptr;
        }
    }

    ~SkipListNode() {
        delete[] forward;
    }
};
```
- `template <typename T>`：使用模板类，使节点可以存储任意类型的数据。
- `T key`：存储节点的键值。
- `SkipListNode** forward`：一个指针数组，用于存储指向不同层级下一个节点的指针。
- `SkipListNode(T k, int level)`：构造函数，初始化节点的键值和 `forward` 数组。
- `~SkipListNode()`：析构函数，释放 `forward` 数组所占用的内存。

**跳表类 `SkipList`**  
```cpp
template <typename T>
class SkipList {
private:
    int level;
    SkipListNode<T>* head;

    int randomLevel() {
        int lvl = 0;
        while ((rand() % 2) == 0 && lvl < MAX_LEVEL) {
            lvl++;
        }
        return lvl;
    }

public:
    SkipList() : level(0) {
        head = new SkipListNode<T>(0, MAX_LEVEL);
        srand(time(0));
    }

    ~SkipList() {
        SkipListNode<T>* current = head;
        while (current != nullptr) {
            SkipListNode<T>* temp = current;
            current = current->forward[0];
            delete temp;
        }
    }

    void insert(T key) {
        SkipListNode<T>* update[MAX_LEVEL + 1];
        SkipListNode<T>* current = head;

        for (int i = level; i >= 0; --i) {
            while (current->forward[i] != nullptr && current->forward[i]->key < key) {
                current = current->forward[i];
            }
            update[i] = current;
        }

        current = current->forward[0];

        if (current == nullptr || current->key != key) {
            int newLevel = randomLevel();

            if (newLevel > level) {
                for (int i = level + 1; i <= newLevel; ++i) {
                    update[i] = head;
                }
                level = newLevel;
            }

            SkipListNode<T>* newNode = new SkipListNode<T>(key, newLevel);

            for (int i = 0; i <= newLevel; ++i) {
                newNode->forward[i] = update[i]->forward[i];
                update[i]->forward[i] = newNode;
            }
        }
    }

    bool search(T key) {
        SkipListNode<T>* current = head;

        for (int i = level; i >= 0; --i) {
            while (current->forward[i] != nullptr && current->forward[i]->key < key) {
                current = current->forward[i];
            }
        }

        current = current->forward[0];

        return (current != nullptr && current->key == key);
    }

    void remove(T key) {
        SkipListNode<T>* update[MAX_LEVEL + 1];
        SkipListNode<T>* current = head;

        for (int i = level; i >= 0; --i) {
            while (current->forward[i] != nullptr && current->forward[i]->key < key) {
                current = current->forward[i];
            }
            update[i] = current;
        }

        current = current->forward[0];

        if (current != nullptr && current->key == key) {
            for (int i = 0; i <= level; ++i) {
                if (update[i]->forward[i] != current) {
                    break;
                }
                update[i]->forward[i] = current->forward[i];
            }

            delete current;

            while (level > 0 && head->forward[level] == nullptr) {
                level--;
            }
        }
    }

    void display() {
        for (int i = level; i >= 0; --i) {
            SkipListNode<T>* current = head->forward[i];
            std::cout << "Level " << i << ": ";
            while (current != nullptr) {
                std::cout << current->key << " ";
                current = current->forward[i];
            }
            std::cout << std::endl;
        }
    }
};
```
- **私有成员变量**：
  - `int level`：记录当前跳表的最大层数。
  - `SkipListNode<T>* head`：指向跳表的头节点。
- **私有成员函数**：
  - `int randomLevel()`：随机生成一个层数，用于新插入节点的层数。通过不断抛硬币（`rand() % 2`）来决定是否增加层数，直到达到最大层数或者抛到反面。
- **公有成员函数**：
  - `SkipList()`：构造函数，初始化跳表的最大层数为 0，创建一个头节点，并设置随机数种子。
  - `~SkipList()`：析构函数，遍历跳表，释放所有节点所占用的内存。
  - `void insert(T key)`：插入一个新的键值对。首先找到每个层级上小于该键值的最大节点，记录在 `update` 数组中。如果该键值不存在，则随机生成一个新的层数，创建新节点，并更新相应层级的指针。
  - `bool search(T key)`：查找指定键值是否存在于跳表中。从最高层开始，尽可能向右移动，直到找到大于等于该键值的节点，然后向下移动一层继续查找，最后检查是否找到该键值。
  - `void remove(T key)`：删除指定键值的节点。首先找到每个层级上小于该键值的最大节点，记录在 `update` 数组中。如果该键值存在，则更新相应层级的指针，删除该节点，并更新跳表的最大层数。
  - `void display()`：打印跳表的每一层节点的键值。

**`main` 函数**  
```cpp
int main() {
    SkipList<int> skipList;

    skipList.insert(3);
    skipList.insert(6);
    skipList.insert(7);
    skipList.insert(9);
    skipList.insert(12);
    skipList.insert(19);
    skipList.insert(17);
    skipList.insert(26);
    skipList.insert(21);
    skipList.insert(25);

    std::cout << "Skip List after insertion:" << std::endl;
    skipList.display();

    std::cout << "Search for 19: " << (skipList.search(19) ? "Found" : "Not Found") << std::endl;

    skipList.remove(19);
    std::cout << "Skip List after removing 19:" << std::endl;
    skipList.display();

    return 0;
}
```
- 创建一个存储整数的跳表对象 `skipList`。
- 插入一系列键值。
- 打印插入后的跳表。
- 查找键值 19 是否存在，并输出结果。
- 删除键值 19。
- 打印删除后的跳表。

通过以上步骤，代码展示了跳表的基本操作，包括插入、查找和删除，并通过打印跳表的每一层节点来验证操作的正确性。


