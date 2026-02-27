```java
**
 *
 * 如果一个数据最近被访问过，那么它将来被访问的概率也更高；
 * 如果一个数据长时间没有被使用，它将来被使用的概率较低。
 * 当缓存空间满了，需要腾出位置存放新数据时，LRU 模式会选择淘汰那些最久没有被访问的数据
 *@author peilizhi
 *@date 2026/2/27 13:28
 **/
public class LRUCache extends LinkedHashMap<Integer, Integer> {

    private int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    public int get(int key) {
        return super.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        super.put(key, value);
    }

    @Override
    protected boolean removeEldestEntry(java.util.Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}

```

