## Move Zeroes (LeetCode #283)

### Approach: **Two Pointer Technique**
- Use a pointer `d` to track the position where next non-zero element should be placed
- First pass: Move all non-zero elements to the front, maintaining their relative order
- Second pass: Fill remaining positions with zeros

### Complexity:
- **Time:** O(n) - Single pass through array
- **Space:** O(1) - In-place modification

### Key Points:
- ✅ Maintains relative order of non-zero elements
- ✅ In-place algorithm (no extra array needed)
- ✅ `d` acts as destination index for non-zero values

### Code:
```java
class Solution {
    public void moveZeroes(int[] arr) {
        int d = 0;  // destination pointer for non-zero elements
        
        // Move all non-zero elements to front
        for(int i = 0; i < arr.length; i++) {
            if(arr[i] != 0)
                arr[d++] = arr[i];
        }
        
        // Fill remaining positions with zeros
        while(d != arr.length)
            arr[d++] = 0;
    }
}
```

**Example:** `[0,1,0,3,12]` → `[1,3,12,0,0]`

