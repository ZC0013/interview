

# vector

**为什么要采用倍增的方法进行动态分配？**

采用倍增的原因：系统为某一程序分配空间时，所需的时间预申请的空间大小无关，主要与申请的次数有关。
即：申请1000的空间和 申请 1 的空间几乎相同。但是申请 1000 * 1 和 1 * 1000 效率相差近千倍   

```c++
/*
vector, 变长数组，倍增的思想(倍增扩容)

	size()  返回元素个数
    empty()  返回是否为空
    ** size 和 empty 所有的容器都有, 并且时间复杂度为O(1), 有一个专门的变量存储 size 不需要遍历 **
    clear()  清空
    front() / back()
    push_back() 在最后插入一个数
    pop_back()  弹出最后一个数
    迭代器：
    begin()		vector 的第 0 个数
    end()		vector 的最后一个数的后面
    支持随机寻址
    支持比较运算，按字典序  ( 原因是重载了位运算符 )
*/
#include <cstdio>
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

int main()
{
    /*
    vector 的初始化
    */
    vector<int> e;  // 1. 默认初始化
    vector<int> a(10);  // 2. 带长度的初始化
    vector<int> c(10, 3);  // 3. 全部初始化为3
    vector<int> d[10];  // 定义了一个vector数组, 10个vector
    vector<vector<int>> vis(m, vector<int>(n, 0));
    /*
    vector 的函数 API
    */
    cout << c.size() << endl;   
    cout << c.empty() << endl;
    
    for(int i = 0; i < 10; i ++) e.push_back(i);
    
    // 遍历1 vector 支持随即查找
    for(int i = 0; i < e.size(); i ++ ) cout << e[i] << ' ';
    cout << endl;
    
    // 遍历2 迭代器方式 e.begin() == &a[0], e.end() == &a[a.size()]; 
    // 迭代器指向的可以理解为指针 *解引用
    for(vector<int>::iterator i = e.begin(); i != e.end(); i ++) cout << *i << ' ';
    cout << endl;
    
    // 遍历2 可以用 auto 简写
    for(auto i = e.begin(); i != e.end(); i ++) cout << *i << ' ';
    cout << endl;
    
    // 遍历3 范围遍历
    for(auto x : e) cout << x << ' ';
    cout << endl;
    
    //支持比较运算，按字典序  ( 原因是重载了位运算符 )
    vector<int> f(4, 3), g(3, 4);
    if(f < g) puts("f < g");
    
    return 0;
}
```

# pair

二元组中的两个类型可以任意

pair<int, int>

- first, 第一个元素
- second, 第二个元素
- 支持比较运算，以first为第一关键字，以second为第二关键字（字典序）

```c++
#include <cstdio>
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

int main()
{
    pair<int, string> p;
    
    // 1. pair 的初始化方式
    p = make_pair(10, "yxc");
    p = {20, "abc"};
    
    // 2. pair存储三元素
    pair<int, pair<int, int>> pp;
    
    return 0;
}
```

# string

- size()                 返回字符串长度
- length()            返回字符串长度
- empty()            判断字符串是否为空
- clear()               清空字符串
- substr(起始下标，(子串长度))  返回子串

第一个参数：起始下标( 从0开始 )，第二个参数：子串长度(超过字母长度时，会输出到最后一个字母为止)

- c_str()  返回字符串所在字符数组的起始地址

```c++
#include <cstdio>
#include <iostream>
#include <cstring>

using namespace std;

int main()
{
    string a = "yxc";
    a += "def";
    a += 'c';
    // 第一个参数：起始下标( 从0开始 )，第二个参数：子串长度(超过字母长度时，会输出到最后一个字母为止)
    cout << a.substr(1, 2) << endl;
    cout << a.substr(1) << endl;	// 返回从 1 开始的所有子串
    // 因为 printf 无法直接输出 string 类型
    printf("%s\n", a.c_str());
    
    return 0;
}
```

# queue

先进先出队列

注意：没有 clear() 函数

- size()
- empty()
- push()            向队尾插入一个元素
- front()            返回队头元素
- back()            返回队尾元素
- pop()              弹出队头元素

```c++
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <queue>

using namespace std;

int main()
{
    queue<int> q;
    // 没有 clear 如何清空。重新构造一个空的 queue
    q = queue<int>();
    
    return 0;
}
```

# priority_queue

优先队列，默认是大根堆

- size()

- empty()

- push()               插入一个元素

- top()                 返回堆顶元素

- pop()                弹出堆顶元素 

```c++
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <queue>
#include <vector>

using namespace std;

int main()
{
    // 定义堆 默认大根堆
    priority_queue<int> heap;
    // 定义成小根堆的方式：
    priority_queue<int> heap<int, vector<int>, greater<int>> q;
    // 定义成小根堆的方式2：
    // -x 按从大到小的顺序排序 == x 按从小到大的顺序排序
    heap.push(-x);
    
    return 0;
}
```

# stack

先入后出，栈

- size()
- empty()
- push()                   向栈顶插入一个元素
- top()                      返回栈顶元素
- pop()                     弹出栈顶元素

# deque

 双端队列, 支持随机访问，但是效率比别的 STL 容器要慢很多

- size()
- empty()
- clear()
- front()                    返回第一个元素
- back()                    返回最后一个元素
- push_back()         向队尾插入一个元素
- pop_back()           从队尾弹出一个元素
- push_front()        向队头插入一个元素
- pop_front()          从队头弹出一个元素
- begin()                  支持迭代器 返回第一个元素的地址
- end()                     支持迭代器 返回最后一个元素后一位的地址
- []                           支持随机访问

# set & multiset

set 中不可以插入重复元素，multiset 可以插入重复元素，底层基于平衡二叉树（红黑树），动态维护有序序列，所以所以的操作时间复杂度都为O(logN)

- size()
- empty()
- clear()

- insert()          插入一个数
- find()             查找一个数
- count()          返回某一个数的个数
- erase()
          (1) 输入是一个数 x，删除所有等于 x 的节点   时间复杂度   O(k + logn)  k是x的个数
          (2) 输入一个迭代器，删除这个迭代器 
-  lower_bound()/upper_bound()
          lower_bound(x)  返回大于等于x的最小的数的迭代器
          upper_bound(x)  返回大于x的最小的数的迭代器
- begin()/end()          支持迭代器
- ++ ,  --                      返回前驱和后继，时间复杂度 O(logn)



```
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <set>

using namespace std;

int main()
{
    set<int> s;
    multiset<int> ms;
    
    
    return 0;
}
```

# map & multimap

基于平衡二叉树（红黑树），动态维护有序序列

- size()
- empty()
- clear()

- insert()             插入的数是一个pair
- erase()             输入的参数是pair或者迭代器
- find()
- []  注意multimap不支持此操作。 时间复杂度是 O(logn)
- lower_bound()/upper_bound()
- begin()/end()          支持迭代器
- ++ ,  --                      返回前驱和后继，时间复杂度 O(logn)

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <map>

using namespace std;

int main()
{
    
    map<string, int> a;
    
    a["yxc"] = 1;
    
    cout << a["yxc"] << endl;
    
    
    return 0;
}
```

# unordered_set/multiset/map/multimap

unordered_set, unordered_map, unordered_multiset, unordered_multimap。与上面的各自对应的set map一样

 **底层实现：哈希表**
和上面类似，**增删改查的时间复杂度是 O(1)**

不支持 lower_bound()/upper_bound()， 迭代器的++，--，因为内部是无序的
```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <unordered_map>

using namespace std;

int main()
{
    
    map<string, int> a;
    
    a["yxc"] = 1;
    
    cout << a["yxc"] << endl;
    
    
    return 0;
}
```

# bitset

圧位

- count()          返回有多少个1
- any()              判断是否至少有一个1
- none()           判断是否全为0
- set()               把所有位置成1
- set(k, v)         将第k位变成v
- reset()           把所有位变成0
- flip()              等价于~
- flip(k)            把第k位取反
- 支持 
  1. 位运算 ~, &, |, ^
  2. 位移  >>, <<
  3. ==,  !=
  4. []

使用场景： 

c++中的bool[] 底层是存储的整数0,1。所以 bool[1024]  大小为 1024Byte = 1KB，如果采用 bitset 则只需要 128 Byte(1 Byte 可以存储 8 位)。

10000 * 10000 的 bool[] ，转用 bitset，可以剩 8 倍空间

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <bitset>

using namespace std;

int main()
{
    // 定义 bitset<个数>
    bitset<10000> s;
    /*
    支持
    1. 位运算 ~, &, |, ^
    2. 位移  >>, <<
    3. ==, ！=
    4. []
    */
    
    return 0;
}
```



# 权威语法查询

c++ reference

http://www.cplusplus.com/reference/
