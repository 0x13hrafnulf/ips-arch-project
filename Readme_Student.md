## Part #1, Brightness and Contrast Adjustments with x87 Instructions
In this part, there were no major problems. Two information sources were used as the documentation in this part to clarify questions and problems.
Yet there were some minor ones, like the adjustments for the negative brightness (passing negative brightness). There reason is in the list of commands that were used for comparisons to clamp the pixel values (cmovb and cmova), which are used for unsigned values. Thus, when passing negative brightness, signed flag were ignored in this case. Therefore, I changed those commands to (cmovg and cmovl) to solve this issue.

* Assembly Language for x86 Processors, 7th Edition by Kip R. Irvine.

* [SIMPLY FPU, by Raymond Filiatreault.](http://www.website.masmforum.com/tutorials/fptute/index.html)

## Part #2, Brightness and Contrast Adjustments with SIMD AVX-512 Instructions
There were some confussion at the beginning with passing array into the vector registers. Since my first tries were unsuccesful, where my attempts were based on loading values to xmmA, then passing those xmmA value to ymmA, and then to zmmA. The results that I received were patterns of lines. By the way, there weren't good information sources to clarify this part. Yet, after posting of this part of code. I understood how to solve the problem, to start doing part #4.

Just as like as in the first case, I had to modify this code to manage negative brightness using "k1" mask, which was succesful and used in the part #4.

## Part #3, The Sepia Filter
Just as like as with the Part #1, no major problems occured. Clarification of minor problems were solved by using documentation listed below.

* Assembly Language for x86 Processors, 7th Edition by Kip R. Irvine.

*[SIMPLY FPU, by Raymond Filiatreault.](http://www.website.masmforum.com/tutorials/fptute/index.html)

## Part 4, The Median Filter
Based on the part #2, I was able to manage to implement the bitonic sort suggested in the paper for 8 values. Yet, there were some confussions in the beggining which sort to use, since there were two types of sort suggested: bitonic sort, quick sort (partition). Had to check another paper by the same author to clarify this moment, which provided several sorting algorithms, and suggested to use bitonic sort among them by comparing them with the "std::sort". 
In this part, I didn't make one to one conversion of the code, since I wanted to achieve a speed-up:
1) Gave up the conversions to double precision (from unsigned in to double, then from double to int64). Instead I used avx commands specially designed for integers. By the way, paper suggests to use integers, since comparisons for them are faster.
2) Made a track of vector values, to reduce the unnecessary movements of values from one register to another. Instead of moving values back to input vector, I periodically moved result values from zmm0 to zmm6, and finally back to the window array. 

As a result, my implementation was about 2 times faster(37-40sec) than "c optimized"(67-70sec). After reading paper again, I found that suggested bitonic sort achieved better performance than STL when sorting more than 5 values. Thus, based on the graph provided in the paper, my results might be close to 8 values sorting. Experimented by changing window size from 9 to 16, qsort for the small image was slower by 1 sec. Therefore, it might be that changing the size of the window can increase the performance of the bitonic sort, yet we have to use another code:
1) Merging of 2 vector registers
2) Using another similar implementation of the code, but for the 16 values (lines 2633-2716, sort-512.hpp from 2nd link below)

*[Fast Sorting Algorithms using AVX-512 on Intel Knights Landing.](https://hal.inria.fr/hal-01512970v1/document)

*[avx-512-sort, by Berenger Bramas.](https://gitlab.mpcdf.mpg.de/bbramas/avx-512-sort)
