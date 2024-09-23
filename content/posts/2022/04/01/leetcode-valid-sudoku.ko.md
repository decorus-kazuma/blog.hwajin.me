+++
title = "Leetcode - Valid Sudoku"
date = "2022-04-01T15:58:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Determine if a 9 x 9 Sudoku board is valid. Only the filled cells need to be validated according to the following rules:"
keywords = ["algorithm","leetcode"]
+++

Determine if a 9 x 9 Sudoku board is valid. Only the filled cells need to be validated according to the following rules:

Each row must contain the digits 1-9 without repetition.
Each column must contain the digits 1-9 without repetition.
Each of the nine 3 x 3 sub-boxes of the grid must contain the digits 1-9 without repetition.
Note:

A Sudoku board (partially filled) could be valid but is not necessarily solvable.
Only the filled cells need to be validated according to the mentioned rules.

![Leetcode](https://user-images.githubusercontent.com/8151366/161214870-77d62353-e26a-4dab-b2f2-2eb8cebe97f4.png)

## Solution,

다른 방법도 몇 있는 것 같던데 나는 비트 연산으로 풀었다. 추가 공간을 크게 먹지 않는다.

행, 열, 박스를 체크해야 한다. 시프트 연산으로 각 비트에 플래그를 걸고 현재 행/열/박스 와 AND 연산시 비트가 존재하는가 존재하지 않는가로 검사한다. 왜냐면, 이미 해당 비트(숫자)가 있다면 `true` 일 테니.

```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        int[] row = new int[9];
        int[] col = new int[9];
        int[] box = new int[9];
        
        for (int i = 0; i < 9; ++i) {
            for (int j = 0; j < 9; ++j) {
                if (board[i][j] == '.') continue;
                int boxpos = ((i / 3) * 3) + (j / 3);
                int t = 1 << ((int) board[i][j]);
                if ((row[i] & t) > 0 || (col[j] & t) > 0 || (box[boxpos] & t) > 0) {
                    return false;
                }
                row[i] |= t;
                col[j] |= t;
                box[boxpos] |= t;
            }
        }
        
        return true;
    }
}
```