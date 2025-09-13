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


## Remove Duplicates from Sorted Array (LeetCode #26)

### Approach: **Two Pointer Technique**
- Use pointer `temp` to track position for next unique element
- Compare each element with previous one (array is sorted)
- Place unique elements at the front, return count of unique elements

### Complexity:
- **Time:** O(n) - Single pass through array
- **Space:** O(1) - In-place modification

### Key Points:
- ✅ Works only on **sorted array** (duplicates are adjacent)
- ✅ Returns the count of unique elements
- ✅ First `temp` elements of modified array contain unique values

### Code (Fixed):
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int temp = 1;  // pointer for next unique element position
        
        // Start from index 1, compare with previous element
        for(int i = 1; i < nums.length; i++) {
            if(nums[i] != nums[i-1]) {
                nums[temp] = nums[i];  // place unique element
                temp++;    
            }
        }
        
        return temp;  // return count of unique elements
    }
}
```

**Example:** `[1,1,2,2,3]` → `[1,2,3,_,_]` returns `3`

**Note:** The array after first `temp` positions can have any values (LeetCode doesn't check them)


## Best Time to Buy and Sell Stock (LeetCode #121)

### Approach: **Single Pass - Track Min Price & Max Profit**
- Track minimum price seen so far (best day to buy)
- Calculate profit if selling at current price
- Keep maximum profit encountered
- Update minimum price as we go

### Complexity:
- **Time:** O(n) - Single pass through array
- **Space:** O(1) - Only two variables used

### Key Points:
- ✅ Can only buy once and sell once
- ✅ Must buy before selling (no shorting)
- ✅ Greedy approach: always consider cheapest buy price so far

### Code:
```java
class Solution {
    public int maxProfit(int[] prices) {
        int min = prices[0];     // minimum price seen so far
        int profit = 0;          // maximum profit possible
        
        for (int i = 0; i < prices.length; i++) {
            // Calculate profit if we sell today
            profit = Math.max(profit, prices[i] - min);
            // Update minimum buy price
            min = Math.min(min, prices[i]);
        }
        
        return profit;
    }
}
```

**Example:** `[7,1,5,3,6,4]` → `5` (buy at 1, sell at 6)

**Visual:**
```
Buy at min=1, Sell at 6 → Profit = 5
```


## Rotate Array (LeetCode #189)

### Approach: **Reversal Algorithm (3-Step Reverse)**
1. Reverse entire array
2. Reverse first k elements
3. Reverse remaining n-k elements

### Complexity:
- **Time:** O(n) - Three passes, each O(n)
- **Space:** O(1) - In-place rotation

### Key Points:
- ✅ Handle `k > array.length` using modulo: `k = k % n`
- ✅ Right rotation by k positions
- ✅ Trick: Multiple reversals achieve rotation without extra space

### Code:
```java
class Solution {
    public void rotate(int[] nums, int k) {
        k = k % nums.length;  // handle k > length
        if(nums.length <= 1)
            return;

        reverse(nums, 0, nums.length - 1);  // reverse entire array
        reverse(nums, 0, k - 1);            // reverse first k elements
        reverse(nums, k, nums.length - 1);  // reverse remaining elements
    }
    
    public void reverse(int[] arr, int i, int j) {
        while(i < j) {
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
            i++;
            j--;
        }
    }
}
```

**Example:** `[1,2,3,4,5,6,7]`, k=3 → `[5,6,7,1,2,3,4]`

**Step-by-step:**
```
Original:     [1,2,3,4,5,6,7]
Step 1:       [7,6,5,4,3,2,1]  (reverse all)
Step 2:       [5,6,7,4,3,2,1]  (reverse first 3)
Step 3:       [5,6,7,1,2,3,4]  (reverse last 4)
```


## Product of Array Except Self (LeetCode #238)

### Approach: **Prefix and Suffix Products**
- Build prefix array: product of all elements before index i
- Build suffix array: product of all elements after index i
- Result[i] = prefix[i] × suffix[i]

### Complexity:
- **Time:** O(n) - Three passes through array
- **Space:** O(n) - Two auxiliary arrays (can be optimized to O(1))

### Key Points:
- ✅ Cannot use division (handle zeros case)
- ✅ `pre[i]` = product of all elements from 0 to i-1
- ✅ `post[i]` = product of all elements from i+1 to n-1
- ✅ Can optimize space by using output array and single variable

### Code:
```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] pre = new int[n];   // prefix products
        int[] post = new int[n];  // suffix products
        
        pre[0] = 1;      // no elements before index 0
        post[n-1] = 1;   // no elements after last index
        
        // Build prefix products
        for(int i = 1; i < n; i++) {
            pre[i] = pre[i-1] * nums[i-1];
        }
        
        // Build suffix products
        for(int i = n-2; i >= 0; i--) {
            post[i] = post[i+1] * nums[i+1];
        }
        
        // Calculate result
        for(int i = 0; i < n; i++) {
            nums[i] = pre[i] * post[i];
        }
        
        return nums;
    }
}
```

**Example:** `[1,2,3,4]` → `[24,12,8,6]`

**Visualization:**
```
nums:  [1,  2,  3,  4]
pre:   [1,  1,  2,  6]  (products before i)
post:  [24, 12, 4,  1]  (products after i)
result:[24, 12, 8,  6]  (pre[i] × post[i])
```

## Best Time to Buy and Sell Stock II (LeetCode #122)

### Approach: **Greedy - Capture Every Upward Movement**
- Buy and sell whenever there's profit (next day price is higher)
- Sum all positive differences between consecutive days
- Equivalent to holding stock through all upward trends

### Complexity:
- **Time:** O(n) - Single pass through array
- **Space:** O(1) - Only using one variable

### Key Points:
- ✅ **Multiple transactions allowed** (unlike Stock I)
- ✅ Can't hold multiple stocks (must sell before buying again)
- ✅ Greedy: capture every price increase
- ✅ Peak-Valley approach: buy at every valley, sell at every peak

### Code:
```java
class Solution {
    public int maxProfit(int[] arr) {
        int maxp = 0;
        
        // Add profit whenever price increases
        for(int i = 1; i < arr.length; i++) {
            if(arr[i] > arr[i-1])
                maxp += arr[i] - arr[i-1];
        }
        
        return maxp;
    }
}
```

**Example:** `[7,1,5,3,6,4]` → `7`
- Buy at 1, sell at 5 (profit = 4)
- Buy at 3, sell at 6 (profit = 3)
- Total = 7

**Visual:**
```
Prices: [7, 1, 5, 3, 6, 4]
         ↓  ↑  ↓  ↑  ↓
        Buy→Sell Buy→Sell
        Profit: 4 + 3 = 7
```

**Key Insight:** Every upward slope contributes to total profit!