## Is Subsequence (LeetCode #392)

### Approach: **Two Pointer Technique**
- Use two pointers to traverse both strings
- Match characters of `s` in order within `t`
- If all characters of `s` are found in order, it's a subsequence

### Complexity:
- **Time:** O(n) where n = length of string `t`
- **Space:** O(1) - Only two pointers used

### Key Points:
- ✅ Subsequence maintains relative order (not necessarily consecutive)
- ✅ Move `j` always, move `i` only on match
- ✅ Success when `i` reaches end of string `s`
- ✅ Greedy: take first matching character

### Code:
```java
class Solution {
    public boolean isSubsequence(String s, String t) {
        int i = 0;  // pointer for string s
        int j = 0;  // pointer for string t
        
        while(i < s.length() && j < t.length()) {
            if(s.charAt(i) == t.charAt(j)) {
                i++;  // match found, move s pointer
            }
            j++;      // always move t pointer
        }
        
        if(i == s.length())
            return true;  // all characters of s found
            
        return false;
    }
}
```

**Example:**
- s = "abc", t = "ahbgdc" → `true`
- s = "axc", t = "ahbgdc" → `false`

**Visual:**
```
s: a b c
   ↓ ↓ ↓
t: a h b g d c
   ↑   ↑     ↑
   Found all characters in order → true
```

**Key Insight:** We don't need consecutive characters, just maintain order!