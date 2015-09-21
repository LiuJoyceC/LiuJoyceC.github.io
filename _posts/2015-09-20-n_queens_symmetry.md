---
layout: post
comments: true
publish: false
title: Using Symmetry to Optimize an N-Queens Solution
---

### Introduction to N-Queens and Bitwise Operation in Javascript

Several weeks ago, I was introduced to the N-Queens problem, and I got to solve it using [bitwise operation](https://en.wikipedia.org/wiki/Bitwise_operation) in Javascript.

The [N-Queens problem](https://en.wikipedia.org/wiki/Eight_queens_puzzle) is a puzzle in which you are given an N-by-N chessboard, and you must place exactly N queens on it in such a way that none of the queens can attack each other in one move (remember that the queen can attack any piece that is in the same row, column, or diagonal). For a large N, just finding one solution can be quite daunting for a human but is a fairly quick task for a computer. The real challenge is counting all of the unique solutions that exist for a particular N. Unlike finding the number of ways to place N rooks on an N-by-N chessboard, which is just simply N! (N factorial), there isn't any known mathematical formula to quickly calculate the number of N-Queens solutions. So the only way to count the solutions is to actually find every solution. This exhausts the computer's resources before the program finishes for very large N. A standard 8x8 chessboard has 92 distinct solutions, and when N=17, there are 95,815,104 distinct solutions! According to [this blogpost by Paul Sokolik](http://paulsokolik.com/n-queens-algorithm/) as of June 2015, the largest N to date for which all the distinct solutions have been enumerated is N=26. The computation took about 9 months using 11 super-computers! This is why this puzzle has become so famous in the computer science community, because every optimization in the computer and algorithm counts.

<div class="post-image"><img src="http://schinckel.net/images/2008/06/8queens.jpg"></div>

For those that have not seen the N-Queens problem solved using [bitwise operation](https://en.wikipedia.org/wiki/Bitwise_operation), here's a brief summary of how the chessboard is represented. Bitwise operators are very fast because they directly operate on individual bits, and a squence of bits can visually represent a row on a chessboard, making bitwise operation ideal for the N-Queens problem. Let's use a 4x4 chessboard as an example. Each row in the chessboard is represented by a single binary number, which is just a sequence of bits. If a queen is placed in the leftmost square of a row, the row is represented by the number 8, which in binary is 1000. If a queen is in the 3rd square from the left, then the row is represented by 2, or 0010. The numbers other than 1, 2, 4, and 8 can be used to represent occupation in more than one square, such as 9 -> 1001, or 15 -> 1111. This is useful for marking squares which would cause conflict (squares which are in the same column or diagonal as another queen in a different row). For example, the number 5 (0101) would indicate that the only open squares left that won't conflict with other queens already on the board are the 1st and 3rd squares from the left, so those two squares will be the only ones where we try to place the next queen. When you get to a row where every square is in conflict (1111), then you know you've gone down the wrong path which will not lead to a valid solution, so you then backtrack up a row to place the previous queen somewhere else. If the previous queen can't be placed anywhere else, then you must keep backtracking up further to change other queens' positions until you find a solution where all the queens are happy :)

For reference, [here is a list of the bitwise operators in Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators).

### A Great Bitwise Solution in Javascript...

After I had written my bitwise solution (which was indeed significantly faster than the non-bitwise solution I had written earlier), I later found [this blogpost by Greg Trowbridge](http://gregtrowbridge.com/a-bitwise-solution-to-the-n-queens-problem-in-javascript/) which presents a Javascript version of the algorithm found in [this paper by Martin Richards](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.51.7113&rep=rep1&type=pdf) (N-Queens is discussed on pages 2-4 of the paper). I really recommend reading [Greg Trowbridge's post](http://gregtrowbridge.com/a-bitwise-solution-to-the-n-queens-problem-in-javascript/), because it does a great job of explaining how the bitwise solution works.

For reference, this is the solution presented in the blogpost:

<div class="message"><pre><code>
countNQueensSolutions = function(n) {
  //Keeps track of the # of valid solutions
  var count = 0;

  //Helps identify valid solutions
  var done = Math.pow(2,n) - 1;

  //Checks all possible board configurations
  var innerRecurse = function(ld, col, rd) {

    //All columns are occupied,
    //so the solution must be complete
    if (col === done) {
      count++;
      return;
    }

    //Gets a bit sequence with "1"s
    //whereever there is an open "slot"
    var poss = ~(ld | rd | col);

    //Loops as long as there is a valid
    //place to put another queen.
    while ( poss & done ) {
      var bit = poss & -poss;
      poss -= bit;
      innerRecurse((ld|bit)>>1, col|bit, (rd|bit)<<1);
    }
  };

  innerRecurse(0,0,0);

  return count;
};
</code></pre></div>

In comparing my original solution to this one, I found a really cool optimization here that my solution didn't have, in the line that says "<i>var bit = poss & -poss</i>". In the binary representation of the variable <i>poss</i>, the zeroes denote spaces in conflict, and the ones are the remaining possibilities. With a simple binary operation, <i>bit</i> gives us the position of the first square from the right that is available, without having to iterate over each square to check if it's open. For example, if <i>poss</i> is 01010000, then <i>bit</i> is 00010000, and there was no need to waste time and computing resources on iterating over the first four spots to check if they are open (or rather, iterating over the powers of two, which represent the spots). This saves a lot of time when you get to the lower rows where most of the squares have some conflict with queens in the rows above.

However, I noticed that this solution did not take advantage of one optimization: symmetry. So I decided to rewrite it to include this optimization.

### ...And How I Improved Upon It Using Symmetry

When we have a valid N-Queens solution, the mirror image of it will obviously still be a valid solution. What's more, this actually counts as a distinct solution from the first one, even though all we did was flip the board over! As long as we can be certain that no solution is identical to its mirror image and no mirror image is identical to any other solution we've found, then we can just find half of the solutions and multiply the count by 2. When N is even, we can just filter out one half of the first row, knowing that the solutions we miss out on will have their mirror images found when we explore the other half of the row. The case when N is odd is a tiny bit trickier, since we can't divide the odd number of squares in the first row by 2. For all solutions where the queen in the first row is not in the middle square, we can still find half of those solutions and multiply by 2. But it turns out we can do the same thing when the first row has it's middle square occupied. When there is a queen in middle square of the first row, then there can't be a queen in the middle square of the second row because then it would be in the same column as the first queen. Now there are an even number of squares in the second row that are still available! We can just exclude half of the squares in the second row so that we find exactly half of the solutions in which the first queen is in the middle. Add that to half of the solutions where the first queen is not in the middle, and we get exactly half of all solutions, which we then multiply by 2. Voila!

We will exclude the right half of the first row. For odd N, this means up to, but not including, the middle square. That means that we will miss solutions such as the this one:

<div class="post-image"><img src="{{ site.baseurl }}public/images/5-Queen-excluded-solution.png"></div>

But that's ok, because we will find its mirror image, and multiply the count by 2:

<div class="post-image"><img src="{{ site.baseurl }}public/images/5-Queen-excluded-Mirror.png"></div>



