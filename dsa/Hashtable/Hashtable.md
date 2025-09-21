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


## Maximum Number of Balloons (LeetCode #1189)

### Approach: **Character Frequency Counting**
- Count frequency of all characters in text
- "balloon" requires: b(1), a(1), l(2), o(2), n(1)
- Find limiting factor: minimum possible formations
- For 'l' and 'o': divide count by 2 (they appear twice in "balloon")

### Complexity:
- **Time:** O(n) - Single pass through string
- **Space:** O(1) - Fixed size array (26 characters)

### Key Points:
- ✅ "balloon" needs: 1b, 1a, 2l, 2o, 1n
- ✅ Divide count of 'l' and 'o' by 2
- ✅ Minimum count determines max balloons
- ✅ Use array indexing: `c - 'a'` for lowercase letters

### Code:
```java
public class Solution {
    public int maxNumberOfBalloons(String text) {
        // Frequency array for all lowercase characters
        int[] count = new int[26];
        for (char c : text.toCharArray()) {
            count[c - 'a']++;
        }
        
        // Calculate the minimum number of "balloon" we can form
        int minBalloons = Integer.MAX_VALUE;
        
        // Check against required characters
        minBalloons = Math.min(minBalloons, count['b' - 'a']);     // need 1 'b'
        minBalloons = Math.min(minBalloons, count['a' - 'a']);     // need 1 'a'
        minBalloons = Math.min(minBalloons, count['l' - 'a'] / 2); // need 2 'l'
        minBalloons = Math.min(minBalloons, count['o' - 'a'] / 2); // need 2 'o'
        minBalloons = Math.min(minBalloons, count['n' - 'a']);     // need 1 'n'
        
        return minBalloons;
    }
}
```

**Example:**
- `"loonbalxballpoon"` → `2`
- b=2, a=2, l=4, o=4, n=2
- Min(2, 2, 4/2, 4/2, 2) = 2

**Visual:**
```
"balloon" requires:
b: 1 × n times
a: 1 × n times  
l: 2 × n times
o: 2 × n times
n: 1 × n times

Find max n where all requirements are met!
```


## Number of Good Pairs (LeetCode #1512)

### Approach: **HashMap with Running Count**
- For each number, count how many times we've seen it before
- If seen k times before, we can form k new pairs with current occurrence
- Add k to result, then increment frequency
- Formula: For n occurrences of a number → n*(n-1)/2 total pairs

### Complexity:
- **Time:** O(n) - Single pass through array
- **Space:** O(n) - HashMap for frequencies

### Key Points:
- ✅ Good pair: `nums[i] == nums[j]` and `i < j`
- ✅ When we see a number, it forms pairs with ALL previous occurrences
- ✅ Running count avoids need for combination formula
- ✅ Increment frequency AFTER adding to count

### Code:
```java
class Solution {
    public int numIdenticalPairs(int[] nums) {
        Map<Integer, Integer> mp = new HashMap<>();
        int count = 0;
        
        for(int i : nums) {
            if(mp.containsKey(i)) {
                count += mp.get(i);        // add previous occurrences
                mp.put(i, mp.get(i) + 1);  // increment frequency
            }
            else {
                mp.put(i, 1);              // first occurrence
            }
        }
        
        return count;
    }
}
```

**Example:** `[1,2,3,1,1,3]` → `4`

**Step-by-step:**
```
[1]: mp={1:1}, count=0
[2]: mp={1:1, 2:1}, count=0
[3]: mp={1:1, 2:1, 3:1}, count=0
[1]: found 1, count+=1, mp={1:2, 2:1, 3:1}, count=1
[1]: found 2, count+=2, mp={1:3, 2:1, 3:1}, count=3
[3]: found 1, count+=1, mp={1:3, 2:1, 3:2}, count=4
```

**Key Insight:** Each new occurrence pairs with ALL previous occurrences of the same number!