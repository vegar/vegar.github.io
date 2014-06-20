---
title: 'Project Euler, Problem 18 &#038; 67'
author: Vegar
layout: post
permalink: /2010/project-euler-problem-18-67/
categories:
  - Development
tags:
  - Euler
---
<p>I have been struggling with a couple of Euler Problems the past days. Not so much finding the solution as finding a couple of bugs in my code…</p>

<p>By starting at the top of the triangle below and moving to adjacent numbers on the row below, the maximum total from top to bottom is 23.</p>

<p>3
7 4
2 4 6
8 5 9 3</p>

<p>Find the maximum total from top to bottom of the given triangle.The 18th problem involves finding the optimal route from top to bottom in a pyramid  of 15 rows. It is possible to solve this using a brute force algorithm, testing every possible solution. Problem no. 67 is the exact same problem, but this time with a pyramid of 100 rows. The problem text includes a little warning, stating that a brute force approach would take a couple of billion years to complete…</p>

<p>I remember back when I was a kid, I used to solve labyrinths like &#8216;help the little mouse find its cheese&#8217;, I always started with the cheese. It was always easier to find the way back to the mouse. I would guess that the same would go for this one. If we can solve the problem from the bottom instead of the top, it should be easier to choose the right path.</p>

<p>6
1 7
9 2 3
8 5 9 3</p>

<p>So, lets start at the bottom of the four-row pyramid. We have four numbers: 8 5 9 and 3. Even though 9 is the larger one, we have no idea if 9 is where we want to end up, since the numbers leading to the first number, 8, could be higher. Imagine the third row to be 9 2 and 3. The only way down to the 8 would be through the 9 in the third row, and down to the 9 we could have come through the 2 or the 3. 8 9 = 17 looks like a much better total then 2 9 = 11 or 3 9 = 12. Moving up one more row, to the second row, lets imagine the numbers 1 and 7. If we went for the  8 9 combination, we would only have one choice up to the second row, making a total of 8 9 1 = 18. If we went for the 9 3 combination, the 7  would be our only choice, giving a total of 19. From the 2 in the third row, we could reach either one, but would not get a higher total any way, so lets just drop that option all together.</p>

<p>Now, what seemed like a great path, starting with 8 9, turned out to be beaten by another option higher in the pyramid. How could we predict that? We couldn&#8217;t. So what would make going from bottom to top a better solution then from the top to bottom? Well, we managed to throw away a couple of options. We never payed either the 5 or the 3 in the bottom row any attention. From the 5, we could have gone up through 9 or 2 in the third row, but if we have come to the 9, why should we go down to the 5 when we know that the 8 would give a better total? Same goes for the 2. Why go down to 5 for a total of 7 when we can go down to 9 for a total of 11? And that is the clue to solve this problem. We need to know what would give the greater total, left or right.</p>

<p>OK. So lets start at the bottom again, this time, with some of the numbers from the given pyramid in problem 18:</p>

<figure>
<img src="http://blog.vi-kan.net/wp-content/uploads/2010/11/problem18_1.jpg" alt="" title="Last two rows" /></figure>



<p>If we start with the 63, we could either go down to 04 or 62. Obviously, knowing this is the last row, we would go for 62 and a total of 125. From 66 we can go to 62 or 98. Once again, we would take the right path for a total of 164. From 04, we would pick 98, from 68, 27 or 89, 23 and so on.</p>

<figure>
<img src="http://blog.vi-kan.net/wp-content/uploads/2010/11/problem18_21.jpg" alt="" title="Last two rows added together" /></figure>



<p>Now we can easily know where we want to go next from the 14th row. Lets add the 13th row:</p>

<figure>
<img src="http://blog.vi-kan.net/wp-content/uploads/2010/11/problem18_3.jpg" alt="" title="Three last rows" /></figure>



<p>Since we already know the best path from each number in the 14th row down to the 15th, we can treat the trip from the 13th row down to the 14th in the same way. From the first number, 91, we know we can choose between 63 and 66. We know that if we choose 63, we can reach a maximum of 91 125 by going right from 63. If we choose 66, we know that we can reach a total of 91 128 by going left. From the second number, 71, we can go left through 66, or right through 04. By choosing right, we can reach a maximum of 71 102, so clearly left would be a better choice, which gives us 71 164.</p>

<p><img src="http://blog.vi-kan.net/wp-content/uploads/2010/11/problem18_4.jpg" alt="" title="Three last rows added together" />Do you see the pattern? I think we are ready for some code! I started out with a NumberPiramid-class. It has a method for adding a row of numbers, and a method returning the calculated maximum total.</p>

```objective-c
@interface NumberPiramid : NSObject {
  NSMutableArray *rows;
}

-(void)addRow:(NSString *)stringWithNumbers;
-(int)numberOfRows;
-(int)maximumTotal;

@end
```

<p>The method addRow:stringWithNumbers takes all numbers for one row as a string separated by spaces. The string is split into an array of Number, a class able to hold both the number itself, and the totals for the left and right side.</p>

``òbjective-c
-(void)addRow:(NSString *)stringWithNumbers;
{
  NSArray *numbers = [stringWithNumbers componentsSeparatedByString:@&quot; &quot;];
  NSMutableArray *row = [[NSMutableArray alloc] initWithCapacity:[numbers count]];
  for (NSString *number in numbers) {
    Number *n = [[Number alloc] initWithNumber:number];
    [row addObject:n];
    [n release];
  }
  [rows addObject:row];
  [row release];
}
```

<p>So now, we have an array of arrays that we can use to find our total. The method maximumTotal will first traverse the arrays bottom up, adding the numbers one by one, always keeping the larger option. I have given the Number class a convenience method maxSum, that returns the larger number of leftSum and rightSum. When we have reached the top, we have our total.</p>

```objective-c
-(int)maximumTotal;
{
  NSArray *row = [rows lastObject];
  NSArray *prevRow;
  for (int r = [self numberOfRows] - 2; r &gt;= 0; r--) {
    prevRow = row;
    row = [rows objectAtIndex:r];
    for (int n = 0; n &lt; [row count]; n  ) {
      Number *number = [row objectAtIndex:n]&gt;
      Number *left = [prevRow objectAtIndex:n];
      Number *right = [prevRow objectAtIndex:n   1];
      number.leftSum = number.number   [left maxSum];
      number.rightSum = number.number   [right maxSum];
    }
  }
  Number *top = [row objectAtIndex:0];
  return [top maxSum];
}
```

<p>And thats all there is. Problem 67 is the exact same problem, but with a larger pyramid. The numbers are given in a textfile, so we need a new method that can read the textfile line by line. Actually, I found it easier to just read the whole file into a string, using NSString initWithContentsOfFile:filename. I then split this string into lines with the same method I use to split a single row.</p>

<p>As before, the code is available <a href="http://svn.vi-kan.net/euler">through svn</a>. There is a delphi implementation too.</p>