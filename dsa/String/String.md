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


## Valid Palindrome (LeetCode #125)

### Approach: **Two Pointer with Character Filtering**
- Use two pointers from start and end
- Skip non-alphanumeric characters
- Compare characters (case-insensitive)
- Move inward until pointers meet

### Complexity:
- **Time:** O(n) - Single pass through string
- **Space:** O(1) - Only two pointers used

### Key Points:
- ✅ Ignore non-alphanumeric characters
- ✅ Case-insensitive comparison
- ✅ Check `i<j` in inner loops to avoid out of bounds
- ✅ Use `Character.isLetterOrDigit()` and `Character.toLowerCase()`

### Code (Fixed):
```java
class Solution {
    public boolean isPalindrome(String s) {
        int i = 0;              // left pointer
        int j = s.length() - 1; // right pointer
        
        while(i < j) {
            // Skip non-alphanumeric from left
            while(i < j && !Character.isLetterOrDigit(s.charAt(i))) {
                i++;
            }
            
            // Skip non-alphanumeric from right
            while(i < j && !Character.isLetterOrDigit(s.charAt(j))) {
                j--;
            }
            
            // Compare characters (case-insensitive)
            if(Character.toLowerCase(s.charAt(i)) != Character.toLowerCase(s.charAt(j))) {
                return false;
            }
            
            i++;
            j--;
        }
        
        return true;
    }
}
```

**Example:**
- "A man, a plan, a canal: Panama" → `true`
- "race a car" → `false`

**Visual:**
```
"A man, a plan, a canal: Panama"
 ↑                             ↑
 Compare only alphanumeric, ignore spaces/punctuation
 A==a, m==m, a==a... → true
```

**Note:** The third condition should be `if` not `while` for character comparison!

## Longest Common Prefix (LeetCode #14)

### Approach: **Horizontal Scanning with Prefix Reduction**
- Start with first string as initial prefix
- For each string, reduce prefix until it matches the beginning
- Use `indexOf(prefix) == 0` to check if prefix matches start of string
- Keep removing last character until match found or prefix is empty

### Complexity:
- **Time:** O(S) where S = sum of all characters in all strings
- **Space:** O(1) - No extra space (substring reuses string pool)

### Key Points:
- ✅ `indexOf(prefix) == 0` ensures prefix is at the beginning
- ✅ Progressively shorten prefix using `substring(0, length-1)`
- ✅ Early termination when prefix becomes empty
- ✅ Works by finding common prefix between first string and all others

### Code:
```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs.length == 0)
            return "";
        
        String prefix = strs[0];  // start with first string
        
        for(int i = 1; i < strs.length; i++) {
            // Reduce prefix until it matches beginning of current string
            while(strs[i].indexOf(prefix) != 0) {
                prefix = prefix.substring(0, prefix.length() - 1);
                if(prefix.isEmpty())
                    return "";
            }
        }
        
        return prefix;
    }
}
```

**Example:** `["flower","flow","flight"]` → `"fl"`

**Step-by-step:**
```
prefix = "flower"
Compare with "flow":   "flower" → "flowe" → "flow" ✓
Compare with "flight": "flow" → "flo" → "fl" ✓
Result: "fl"
```

**Key Insight:** `indexOf(prefix) == 0` means prefix starts at index 0 (beginning of string)!


## Reverse Words in a String (LeetCode #151)

### Approach: **Split and Reverse**
- Trim leading/trailing spaces
- Split by regex `\\s+` (one or more spaces)
- Iterate words array backwards
- Build result with StringBuilder

### Complexity:
- **Time:** O(n) - Split and traverse once
- **Space:** O(n) - Array of words and StringBuilder

### Key Points:
- ✅ `trim()` removes leading/trailing spaces
- ✅ `\\s+` regex handles multiple spaces between words
- ✅ StringBuilder for efficient string concatenation
- ✅ Add space between words except after last word

### Code:
```java
class Solution {
    public String reverseWords(String s) {
        String[] st = s.trim().split("\\s+");  // split by spaces
        StringBuilder str = new StringBuilder();
        
        // Traverse backwards
        for(int i = st.length - 1; i >= 0; i--) {
            str.append(st[i]);
            if(i != 0) {
                str.append(" ");  // space between words
            }
        }
        
        return str.toString();
    }
}
```

**Example:**
- `"  the sky is  blue  "` → `"blue is sky the"`
- `"hello world"` → `"world hello"`

**Process:**
```
Input:  "  the sky is  blue  "
Trim:   "the sky is  blue"
Split:  ["the", "sky", "is", "blue"]
Reverse: "blue is sky the"
```

**Regex Note:**
- `\\s+` matches one or more whitespace characters
- Handles multiple spaces between words automatically