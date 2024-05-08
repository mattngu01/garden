**Name**: Element appearing more than 25% in a sorted array
**Date:** 2023-11-29
**Type:** 
**Difficulty:** Easy
**Link:** https://leetcode.com/problems/element-appearing-more-than-25-in-sorted-array/


## submitted solution
```python
class Solution:
    def findSpecialInteger(self, arr: List[int]) -> int:
        nums = defaultdict(int)

        highest = arr[0]

        for x in arr:
            nums[x] += 1
            highest = max(highest, x, key=lambda y: nums[y])
        
        return                      
```

**Peeked at solution?:** No

## submitted solution, explained

## Use O(1) space
```Python
class Solution:
    def findSpecialInteger(self, arr: List[int]) -> int:
        count = 1
        previous = arr[0]
        num = arr[0]
        quarter = len(arr) / 4

        for index in range(1, len(arr)):
            num = arr[index]

            if num == previous:
                count += 1
            
            if num != previous:
                count = 1
            
            if count > quarter:
                return num
            
            previous = num

        return num
```

This solution utilizes the fact that only _1_ number will take more than a quarter of the arr and that is sorted. As soon we count that one of the numbers is >25%, then we return it immediately.
## Faster than O(n)
- since the number takes up more than a quarter, it is guaranteed to be at the one of the quarter intervals (`1/4`, `2/4`, `3/4`). At each quarter interval, we can check for the upper and lower bound of that particular element with binary search.
- note: this binary search doesn't have a case of `target == middle_element`, this gets merged with whatever side of the boundary (i.e. `>= or <=`)

```Python

```

## Takeways
- typically sorting the array is a clue
- when doing leetcode, think about better solutions that have better runtime or space

 
#leetcode #algorithms #revisit