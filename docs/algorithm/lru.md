# LRU

逛牛客感觉所谓的手撕 LRU 变得很经典了，故而记录一下

以 [leetcode hot100 的 LRU](https://leetcode.cn/problems/lru-cache/description/?envType=study-plan-v2&envId=top-100-liked) 为例

不考虑性能的情况下，可以用下边这个解决

```cpp
class LRUCache {
    std::list<std::pair<int, int>> l{};
    std::unordered_map<int, decltype(l.begin())> m;
    size_t capacity{0};
public:
    LRUCache(size_t capacity) : capacity(capacity) {}

    int get(int key) {
        if (m.count(key)) {
            l.splice(l.begin(), l, m[key]);
            return m[key]->second;
        }
        return -1;
    }
    void put(int key, int value) {
        if (m.count(key))
            l.erase(m[key]);
        l.push_front({key, value});
        m[key] = l.begin();
        if (l.size() > capacity) {
            m.erase(l.back().first);
            l.pop_back();
        }
    }
};
```

代码很简单，基本都是用了 STL 提供好的函数，这里只有一个可能不太常见，那就是 `l.splice(l.begin(), l, m[key])`

在 [cppreference](https://zh.cppreference.com/w/cpp/container/list/splice) 中可以看到该函数的介绍。

简单来说，这个函数就是把一个 list 的元素转移到另一个 list 中。对于下边这个函数来说

```cpp
void splice( const_iterator pos, list& other, const_iterator it );
```

该函数会将 `other` 中，`it` 指向的元素转移到 this 指针的 list 中的 `pos` 指向的元素之前。
