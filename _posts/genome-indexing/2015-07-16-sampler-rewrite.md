---
layout: page-fullwidth
subheadline: Test driving the rewrite of a C program into Java
title:  Sampler Rewrite
teaser: "The first computationally interesting piece of the Map/Reduce genome indexer is the 'sampler'. The original sampler was written in C, with typically short variable names, large methods and deeply nested loops. I wanted to rewrite to be sure I understood it well and could make any desired tweaks with confidence. I also wanted a design that improved the code clarity so that I can more easily explain it to others."
breadcrumb: true
header: no
tags:
    - genome-indexing java c hadoop map-reduce
categories:
    - genome-indexing
---
##The Sampler##
The first computationally interesting piece of the Map/Reduce genome indexer is the "sampler". The original sampler was written in C, with typically short variable names, large methods and deeply nested loops. I wanted to rewrite to be sure I understood it well and could make any desired tweaks with confidence. I also wanted a design that improved the code clarity so that I can more easily explain it to others.

The sampler takes a genome and picks a set of short sample sequences from it. The samples are in sorted order. The goal of the sampler is to provide input to the Map/Reduce partitioning step. During the partitioning step, a suffix (a substring of the genome that ends at the last character of the genome) will be compared with the samples so that the suffix can be assigned to a reducer node. That way, we have divided up the sorting work of the reducers such that the results of each reducer can simply be concatenated together to form our suffix index.

In order to load balance the reducer tasks, it is important to evenly distribute the suffixes to the reducers. Therefore, the value of the samples chosen must reasonably well represent the distribution of suffixes in the genome. To accomplish this, the sampler algorithm picks many samples from the genome, sorts them in alphabetic order and chooses and evenly distanced subset of samples.

An additional challenge is that some genomes, such as the human genome, contain very long lengths of repeated characters. This can happen near the centromere and telomere, but can also be long periods of unknown content represented as the character 'N'. These long repeat regions can skew the distributions and also make sorting more expensive. Hence, we limit the impact of large repeats by making sure that each sample includes at least two unique characters and using run-length encoding to compress the sample size.

##Algorithm##
The psuedo code for the Sampler is:
<ol>
  <li>
    Step through the genome, skipping 1000 characters at each step.<br>At each step:
	<ol>
	  <li>
	    Collect a sample in a list. Samples will contain at least 15 characters, at least 2 of which will be unique.
	  </li>
	  <li>
	    Run-length encode the sample.
	  </li>
	</ol>
  </li>
  <li>
    Sort the list of samples.
  </li>
  <li>
    Step through the sorted list of samples. Step size = list size / number of partitions.<br>At each step:
	<ol>
	  <li>
	    Output the sample. This is one of the partition boundaries.
	  </li>
	</ol>
  </li>
</ol>

##Process for Rewriting##
Test driven design (TDD) was used to rewrite the Sampler code. I began at the outside and worked my way in. That way, I had to break the algorithm into it's major parts first, with each ending up as a new class:
<ul>
  <li>Sampler: the main class that drives the algorithm, really delegating all of the heavy lifting</li>
  <li>SequenceReader: simply reads the genome sequence into a Sequence class.</li>
  <li>Sequence: simply contains the sequence string</li>
  <li>SubSequenceCollector: steps through the genome Sequence and collects the initial sample sequences according to the rules above.</li>
  <li>PeriodicSampler: sorts the list of sample sub-sequences and chooses an even distribution of samples.</li>
  <li>ListWriter: writes out the final list of samples. A pretty trivial class.</li>
</ul>

Deciding on how to break things into classes while writing the Sampler was a bit of a challenge. It forced me to really write out the algorithm so that I could understand the important steps. TDD practice was valuable here because it encouraged separating the complex portions of the algorithm as well as fully separating the input and output steps from the rest of the code.

There were sections of both the SubSequenceCollector and the PeriodicSampler that occasionally offered a challenge as well. It is my experience that these types of iterating/collecting/sampling algorithms are susceptible to off-by-one errors; where computing an index incorrectly or messing up a < vs. <= can produce results that look correct but are actually wrong. Test driving the code was particularly important here as it ensured that at any point, the scenarios I had already encoded still worked while I accounted for new scenarios. TDD also helped make sure that the code was very well covered by tests.

##Observations##
###Performance###
The Java version executed somewhat slower than the C version. Most of the difference is actually in the start up of the JVM and was expected. The Java performance was entirely acceptable however as the Sampler is executed only once before the Map/Reduce job begins and still completed in under 1 second for the ecoli genome.

###Development Process###
One thing I would do differently when performing such a rewrite would be to first write tests for the old version of the code. This would have helped me understand a few of the nuances with stepping logic that the original authors used. For example, in the PeriodicSampler, I was trying to get the same results as the original source. However, the original code uses a slightly different method for stepping through the sorted list. I would get results that were 'mostly' correct, choosing a very similar distribution of samples, but being off by one. This was a result of rounding effects and I had to do some playing with different rounding techniques to get the exact same results.

Overall, test driving the rewrite was very effective in understanding the algorithm being employed and getting the code correct. In addition to writing automated test cases, I compared the final results of my program with the original. This level of acceptance testing on top of the unit testing is of course required when developing any program.

###Final Code Structure###
The final code is, at least to me, more readable and editable. It can be changed with confidence due to the test cases. The break down into individual classes aids in understanding the algorithm. The PeriodicSampler actually is something generic enough that it could be used in other algorithms outside of genome indexing. That wasn't one of my goals, but TDD made that abstraction pretty clear and is one of the nice results of TDD that I've observed in my practice.

There are a few trivial classes, namely SequenceReader and ListWriter. My guess is that I could have eliminated these using Java 8 streams. My VirtualBox installation of Hadoop had Java 7 though, so I didn't go there on this rewrite. Perhaps a future refactoring?