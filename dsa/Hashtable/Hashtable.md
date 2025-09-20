## Design HashMap (LeetCode #706)

### Approach: **Separate Chaining with Dummy Head**
- Array of linked lists (size 10000) for buckets
- Each bucket has dummy head node (-1, -1) for easier operations
- Hash function: `hashCode(key) % SIZE`
- `find()` returns node BEFORE target (or last node if not found)

### Complexity:
- **Time:** O(n/k) average, O(n) worst case - where n=keys, k=buckets
- **Space:** O(n+k) - n keys + k buckets

### Key Points:
- ✅ Dummy head simplifies insert/delete operations
- ✅ `find()` returns previous node for easy linking/unlinking
- ✅ Handles collisions via separate chaining
- ✅ Fixed bucket size (can be optimized with dynamic resizing)

### Code:
```java
class MyHashMap {
    private class Node {
        int key, value;
        Node next;
        
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private final int SIZE = 10000;
    private Node[] nodes;
    
    public MyHashMap() {
        nodes = new Node[SIZE];
    }
    
    private int getIndex(int key) {
        return Integer.hashCode(key) % SIZE;
    }
    
    // Returns node BEFORE target (or last node)
    private Node find(Node head, int key) {
        Node prev = null;
        while (head != null && head.key != key) {
            prev = head;
            head = head.next;
        }
        return prev;
    }
    
    public void put(int key, int value) {
        int index = getIndex(key);
        if (nodes[index] == null) {
            nodes[index] = new Node(-1, -1);  // dummy head
        }
        Node prev = find(nodes[index], key);
        if (prev.next == null) {
            prev.next = new Node(key, value);  // add new
        } else {
            prev.next.value = value;  // update existing
        }
    }
    
    public int get(int key) {
        int index = getIndex(key);
        if (nodes[index] == null)
            return -1;
        Node prev = find(nodes[index], key);
        if (prev.next == null)
            return -1;
        return prev.next.value;
    }
    
    public void remove(int key) {
        int index = getIndex(key);
        if (nodes[index] == null)
            return;
        Node prev = find(nodes[index], key);
        if (prev.next == null)
            return;
        prev.next = prev.next.next;  // unlink node
    }
}
```

**Visual Structure:**
```
Bucket[0] → [-1,-1] → [100,5] → [1000,7] → null
Bucket[1] → [-1,-1] → [1,3] → null  
Bucket[2] → null
...
```

**Key Insight:** Dummy head node eliminates edge cases for insert/delete at head!