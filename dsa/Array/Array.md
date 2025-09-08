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


## Majority Element (LeetCode #169)

### Approach: **Boyer-Moore Voting Algorithm**
- Find a candidate that could be the majority element by canceling out different elements
- When count reaches 0, switch to new candidate
- Verify if candidate appears more than n/2 times (optional if majority is guaranteed)

### Complexity:
- **Time:** O(n) - Two passes through array
- **Space:** O(1) - Only using count and candidate variables

### Key Points:
- ✅ Works because majority element appears > n/2 times
- ✅ Majority element will survive the cancellation process
- ✅ Second loop is verification (can be skipped if majority is guaranteed to exist)

### Code:
```java
class Solution {
    public int majorityElement(int[] nums) {
        int count = 1;
        int candidate = 0;  // index of candidate
        
        // Phase 1: Find candidate
        for(int i = 0; i < nums.length; i++) {
            if(nums[candidate] == nums[i])
                count++;
            else
                count--;
            
            if(count == 0) {
                candidate = i;  // switch candidate
                count = 1;
            }
        }
        
        // Phase 2: Verify candidate (optional if majority guaranteed)
        count = 0;
        for(int i = 0; i < nums.length; i++) {
            if(nums[candidate] == nums[i])
                count++;
            if(count > nums.length/2)
                return nums[candidate];
        }
        return -1;
    }
}
```

**Example:** `[2,2,1,1,1,2,2]` → `2` (appears 4 times > 7/2)