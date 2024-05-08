**Name**:  3Sum
**Date:** 2023-05-29
**Type:** Array
**Difficulty:** Medium
**Link:** https://leetcode.com/problems/3sum/description/



## submitted solution
```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums_dict = {} #number -> index
        answer_set = set()
        for a in range(len(nums)):
            nums_dict[nums[a]] = a
        
        numbers = set()
        for x in range(len(nums)):
            for y in range(x, len(nums)):
                third_number = 0 - (nums[x] + nums[y])
                if x != y and third_number in nums_dict and x != nums_dict[third_number] and y != nums_dict[third_number]:
                        answer_set.add(tuple(sorted([nums[x], nums[y], third_number])))

        return answer_set

```

**Peeked at solution?:** No, but this is not the best answer.

## submitted / best solution, explained

This is essentially a rehash of the 2sum, but now with three. You can figure out the remainder by subtracting `third_number = 0 - (first + second)`. In 2sum, you figure it out by doing `second_number = 0 - first_number`. The submitted solution is O(n^2), but it seems like the best answer is O(n^2) anyways.


### Optimal Solution
1. sort the array
2. iterate through the array. This is the middle number
3. have left & right of the middle number.  `L = middle_index + 1`, `R = end_index`
4. if L + R < -middle
5. honestly this is kinda confusing how is this really readable lol? Am i just dumb
#leetcode #algorithms 