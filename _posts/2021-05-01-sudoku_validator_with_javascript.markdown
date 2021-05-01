---
layout: post
title:      "Sudoku Validator with JavaScript"
date:       2021-05-01 16:13:17 +0000
permalink:  sudoku_validator_with_javascript
---


Ok, this week I did a simple validator for Sudoku solutions. As you probably know, a valid Sudoku solution will have the numbers 1-9 on every row and column, and each 3 x 3 box will also have numbers 1-9. Checking is fairly simple enough, but the challenge is making the code concise, clean, and easy to go through. My solution isn't the most elegant, but it'll do for now.

First off, I've set up a valid solution and invalid one to use for testing
```
const CORRECT_SUDOKU = [
    [8,3,5,4,1,6,9,2,7],
    [2,9,6,8,5,7,4,3,1],
    [4,1,7,2,9,3,6,5,8],
    [5,6,9,1,3,4,7,8,2],
    [1,2,3,6,7,8,5,4,9],
    [7,4,8,5,2,9,1,6,3],
    [6,5,2,7,8,1,3,9,4],
    [9,8,1,3,4,5,2,7,6],
    [3,7,4,9,6,2,8,1,5]
]

const INCORRECT_SUDOKU = [
    [8,3,5,4,1,6,9,2,7],
    [2,9,6,8,5,7,4,3,1],
    [4,1,7,2,9,3,6,5,8],
    [5,6,9,1,3,4,7,8,2],
    [1,2,3,6,7,8,5,4,9],
    [7,4,8,5,2,9,1,6,3],
    [6,5,2,7,8,1,3,9,4],
    [9,8,1,3,4,5,2,7,6],
    [3,7,4,9,6,2,8,1,1]
]
```
Next, I'll address the rows first since that will be the most simple - going through each sub array within the matrix array to see if it contains 1-9. However, since JS can't compare the values of objects, but rather only that the object identifiers are the same, then we need to create our own array checker.
```
function sequenceChecker (sequence) {
    const validSeq = [1, 2, 3, 4, 5, 6, 7, 8, 9]
    return sequence.every( (num, i) => num === validSeq[i] )
}
```
Now that we have a sequence checker function up, lets make a function that goes through the whole matrix to check each row. This simply entails sorting each row and then using our sequence checker for each row array.
```
function checkRows(matrix){
    let arr = matrix.map(seq => seq.sort())
    return arr.every(seqArr => sequenceChecker(seqArr))
}
```
Next, we'll do the columns. For this, I simply made another matrix where each row is the original matrix's column and then using the `checkRows()` function that I've already made.
```
function checkColumns(matrix){
    let columnMatrix = [[], [], [], [], [], [], [], [], []]
    matrix.forEach( array => array.forEach((num, i) => columnMatrix[i].push(num)))
    return checkRows(columnMatrix)
}
```
And finally, we need make a `checkBoxes()` function to handle all 9 boxes within the matrix. This one is certainly not as elegant as I'd like it to be, but at the end of the day it's functional, and I didn't want to spend too much time figuring out all the loops necessary for the most elegant solution. So, what I did was make a function that takes the given matrix, the row index, and the column index of a box and then prints out that box as a row. Then, I created a `boxMatrix` and called that function for each box, which creates the box matrix, and then call my `checkRows` function on that. Heres what that looks like.
```
function makeBox(matrix, rowOne, colOne){
    let arr = [
        matrix[rowOne][colOne],
        matrix[rowOne][colOne + 1],
        matrix[rowOne][colOne + 2],
        matrix[rowOne + 1][colOne],
        matrix[rowOne + 1][colOne + 1],
        matrix[rowOne + 1][colOne + 2],
        matrix[rowOne + 2][colOne],
        matrix[rowOne + 2][colOne + 1],
        matrix[rowOne + 2][colOne + 2]
    ]
    return arr
}

function checkBoxes(matrix){
    let boxesMatrix = [
        makeBox(matrix, 0, 0),
        makeBox(matrix, 0, 3),
        makeBox(matrix, 0, 6),
        makeBox(matrix, 3, 0),
        makeBox(matrix, 3, 3),
        makeBox(matrix, 3, 6),
        makeBox(matrix, 6, 0),
        makeBox(matrix, 6, 3),
        makeBox(matrix, 6, 6)
    ]
    return checkRows(boxesMatrix)
}
```
Again, not the most elegant, but it works. Now, we've just got to put it all together. I hit one more snag along the way, which was that the original matrix was getting manipulated as I passed it through the functions. So, to solve that I begin by making three copies of the matrix to pass through each function. Then, I put each of those checkers into an array and see if each function return true. If so, then the result is a winner!
```
function validSudoku (matrix) {
    const copy1 = [...matrix.map(arr => [...arr])]
    const copy2 = [...matrix.map(arr => [...arr])]
    const copy3 = [...matrix.map(arr => [...arr])]
    const checker = [
        checkRows(copy1),
        checkColumns(copy2),
        checkBoxes(copy3)
    ]
    if (checker.every(seq => seq)){
        return "Winner"
    } else {
        return "Please try again"
    }
}

console.log(validSudoku(CORRECT_SUDOKU))   // => "Winner"
console.log(validSudoku(INCORRECT_SUDOKU))   // => "Please try again"
```
And there we go! A functional sudoku answer validator. Again, not the most elegant. Clearly, there are ways to maximize loops and/or iterators to improve the creation of the box matrix, and I also could have included a way to return false as soon as an invalid sequence is hit, but this is functional and I need to move on to the next thing.
