# hash冲突与退化
std::unordered_map 里面的哈希表，它所提供的的方法基本都是用hashtable封装实现的.
从源码来看，它的定义是这样的
```c++
    //gnu
    template<typename _Key,
             typename _Tp,
             typename _Hash = hash<_Key>,
             typename _Pred = equal_to<_key>,
             typename _Alloc = allocator<std::pair<const _Key, _Tp>>>
    class unordered_map
    {
        typedef _umap_hashtable<_Key, _Tp, _Hash, _Alloc> _Hashtable;
        _Hashtable _M_h;
    };
```
### hash冲突
在hashtable中，数组的每个元素称作桶(bucket)。
hashtable采用一个hash函数来计算元素的索引值，来满足O(1)的搜索时间复杂度。
其过程为：
- 计算元素的哈希值。对于单个键值对{key, value}，计算key对应的哈希值`hashcode = hash_func(key)`
- 计算元素在数组中的索引。由于hashcode不一定处于[0, bucket_count]范围内，因此需要将hashcode映射到该范围: `index = hashcode % bucket_count`
为了解决hash冲突，hashtable在每个桶里bucket[index]不在直接存储待插入节点的值，而是存储一个哨兵节点，使其指向一个链表，由这个链表来存储每次插入节点的值：
- 桶的索引值index依然是bucket_index函数的计算方式，即通过待插入节点的键来获取
- 待插入节点的值在哨兵指向的链表头部插入，由于是头部插入，整个插入过程是O(1)时间复杂度。

### hash退化
为了解决hash退化，引入了两个概念：
- 负载因子(load_factor)，是hashtable的元素个数与hashtable的桶数之间比值；
- 最大负载因子，是负载因子的上限(max_load_factor)
当hashtable中的元素个数与桶数比值load_factor >= max_load_factor时，hashtable就自动发生rehash行为，来降低
load_factor:
- 扩容。分配一块更大的内存，来容纳更多的桶
- 重新插入。按照上述插入步骤将原来桶中的buck_size个节点重新插入到新的桶中。

Rehash后，桶数增加了而元素个数不变，再次满足load_factor <= max_load_factor条件。
