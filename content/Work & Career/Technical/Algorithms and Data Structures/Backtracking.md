
# Backtracking vs. Dynamic Programming
- the answer doesn't seem to be very clear
- dynamic programming seems to refer to the optimizations of a recursive function, such as memoization, etc
- backtracking creates / searches through all possible solutions of a particular problem set, without any optimization




https://www.reddit.com/r/leetcode/comments/ntuycc/how_do_i_decide_between_dynamic_programming_vs/
my usual steps for any problem involving choice:

1. start with backtracking (topdown)
2. add memoization table if possible (topdown)
3. convert the memoization table into dp table if possible (bottomup)
4. go from dp table to dp array (or several) if possible (bottomup)
5. keep an eye out for greedy solutions along the way as they are almost always better