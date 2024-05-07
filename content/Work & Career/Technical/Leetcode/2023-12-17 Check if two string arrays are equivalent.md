**Name**: Check If Two String Arrays Are Equivalent
**Date:** 2023-12-17
**Type:** generators, string, array?
**Difficulty:** Easy
**Link:** https://leetcode.com/problems/check-if-two-string-arrays-are-equivalent/description/



## submitted solution

### easy way
```python
class Solution:
    def arrayStringsAreEqual(self, word1: List[str], word2: List[str]) -> bool:
        return "".join(word1) == "".join(word2)
```

### 4 ptrs
```Python
class Solution:
    def arrayStringsAreEqual(self, word1: List[str], word2: List[str]) -> bool:
        first_length = 0
        second_length = 0 

        first_word_ptr = 0
        second_word_ptr = 0

        fchar_ptr = 0
        schar_ptr = 0

        while first_word_ptr < len(word1) and second_word_ptr < len(word2):
            first = word1[first_word_ptr]
            second = word2[second_word_ptr]

            if first[fchar_ptr] != second[schar_ptr]:
                return False
            else:
                if fchar_ptr >= len(first)-1:
                    first_word_ptr += 1
                    fchar_ptr = 0
                else:
                    fchar_ptr += 1
                
                if schar_ptr >= len(second)-1:
                    second_word_ptr += 1
                    schar_ptr = 0
                else:
                    schar_ptr += 1
        
        return fchar_ptr == 0 and schar_ptr == 0 and first_word_ptr == len(word1) and second_word_ptr == len(word2)

```

### Generators
```Python
class Solution:
    def arrayStringsAreEqual(self, word1: List[str], word2: List[str]) -> bool:
        first = self.generator(word1)
        second = self.generator(word2)

        x = next(first)
        y = next(second)

        while x is not None and y is not None:
            if x != y:
                return False
            
            x = next(first)
            y = next(second)
        
        return x == y

    def generator(self, word):
        for segment in word:
            for char in segment:
                yield char
        yield None

```
**Peeked at solution?:** Yes

## submitted solution, explained

- 4 pointers: traverse each character within the list and compare
	- at the end, both the char pointers should be reset & the word ptrs should be at its last index (this will indicate we are all the way at the end, at the same time. This accounts for words that are uneven)
	- O(n) compute, O(1) memory
- generators: same, but better
	- way easier to read
	- O(n) compute, O(1) memory?
- naive: O(n) compute, O(n) memory

#leetcode #algorithms #revisit