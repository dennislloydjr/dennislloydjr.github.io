---
layout: page-fullwidth
show_meta: false
title: "Hadoop Genome Indexing Project"
subheadline: "Using Hadoop to Quickly Create an Index for a Genome"
header:
   image_fullwidth: "header_dna_john_goode.jpg"
permalink: "/genome-indexing/"
---
##Background##
I started this project to gain a better grasp on Map/Reduce on Hadoop. The source code and research for this project is based on the work of researchers Rohith Menon, Goutham Bhat and Michael Schatz ([Rapid Parallel Genome Indexing with MapReduce](http://schatzlab.cshl.edu/publications/2011-GenomeIndexingMapReduce.pdf)). They developed an algorithm for creating a suffix-index and Burrow-Wheeler Transform for a whole genome using Map/Reduce on Hadoop. Generating a pre-computed index of a genome is an important step to optimize sequence alignment, which is an essential technique in computation biology. Techniques to distribute the indexing workload across multiple nodes existed before their paper. What they introduced however, is a technique to distribute the workload using commodity hardware on Amazon's EC2 platform and Hadoop's Map/Reduce.

##Project Goals##
My goals for this project are:
<ol>
  <li>Understand how the algorithm distributes the work among the Map and Reduce tasks and explain it more plainly.</li>
  <li>Understand the benefit behind creating a suffix index and Burrows-Wheeler Transform (BWT)</li>
  <li>Re-express the C and C++ portions of the code in Java and analyse any significant performance impact this has on the technique</li>
  <li>Implement unit tests for the existing code to support refactoring and extension</li>
  <li>Reproduce the results for indexing the genomes published in their paper</li>
  <li>Utilize the Gradle build tool to manage dependencies and make testing and building more fluid</li>
</ol>

##About the Original Source##
The original source code is available under MIT license [here](http://code.google.com/p/genome-indexing]). Future posts will cover each portion of the algorithm in more depth. Here I am just providing an overview of the source code in the original project. It contains the following components:
<ul>
  <li><b>Sampler</b>: This C module creates a reasonably well distributed partitioning of the suffixes that will be generated.</li>
  <li><b>run.sh</b>: This shell scripts generates ranges of integers used in the <b>map</b> phase, calls the Sampler to create the partitions used to distribute the work among the <b>reducers</b>, loads the DNA sequence into HDFS, and then executes the Map/Reduce job.</li>
  <li><b>Buck Sort Mapper</b>: This is a Java implementation of the <b>map</b> phase. Each call to <b>map</b> is given a range of integers. The <b>map</b> method creates a key for each integer in the range and assigns it a null value. When all mappers are taken as a whole, a key-value pair is generated for each integer from 1 to G, where G is the size of the genome.</li>
  <li><b>Bucket Sort Total Order Partitioner</b>: This Java class determines how the Map/Reduce algorithm will assign the map results to reducers. It distributes subsequences of the genome to the reducers based on the partitions created by the <b>Sampler</b></li>
  <li><b>PSort</b>: This is the Reducer. It is a small amount of Java code that wraps a sorting algorithm implemented in C++ and called through JNI. The <b>reduce</b> method simply collects the indices produced by the <b>map</b> phase in a buffer. The work of sorting the suffixes is actually done in the <b>cleanup</b> method of the reducers.</li>
</ul>

##Posts in this Series##
<ul>
    {% for post in site.categories.genome-indexing %}
    <li><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>

###About the Image###
The header image on this page is ["DNA"](https://www.flickr.com/photos/johnnieb/17200471) by [John Goode](https://www.flickr.com/photos/johnnieb), licensed under [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/). 