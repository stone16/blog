---
title: System Design Patterns - Bloom Filters
date: 2021-08-09 19:33:45
categories: SystemDesign
tags:
top:
---
# System Design Patterns - Bloom Filters

Created: August 8, 2021 10:06 PM
Status: Finished
Tags: System Design
Type: Tech Resource

# 1. Background

- Suppose we have a large set of structured data(identified by record IDs) stored in a set of data files, and we want to know which file might contain our required data
    - we don't want to read each file, as it's slow and we have to read a lot of data from the disk
- Solution 1: Build an index on each data file and store it in a separate index file, to map each record ID to its offset in the data file; each index file will be sorted on the record ID. Then we could do a binary search in index file
- Solution 2: We could use Bloom Filters

# 2. How does Bloom Filter work?

- The Bloom filter data structure tells whether an element **may be in a set, or definitely is not**
    - which means the only possible errors are false positives
- How it looks
    - An empty bloom filter is a bit array of m bits, all set to 0
    - There are also k different hash functions, each of which maps a set element to one of the m bit positions
- Workflow
    - To add an element, feed it to the hash functions to get k bit positions, and set the bits at these positions to 1
    - To test if an element is in the set, feed it to the hash functions to get k bit positions
        - if any of the bits at these positions is 0, the element is definitely not in the set
        - if all are 1, then the element may be in the set

# 3. When will we use Bloom Filter?

- In BigTable, any read operation has to read from all SSTables that make up a tablet
    - if these SSTables are not in memory, the read operation may end up doing many disk accesses
    - BigTable uses bloom filters to reduce the number of disk accesses
    - Store of BloomFilter could drastically reduces the number of disk seeks, thereby improving read performance

# 4. How to use Bloom Filter in Java?

- We could use BloomFilter class from the Guava library to achieve this

```jsx
BloomFilter<Integer> filter = BloomFilter.create(

	Funnels.integerFunnel(), 
	500, 
	0.01
);
```

- How to implement from scratch   [https://www.inlighting.org/archives/java-implement-bloom-filter/](https://www.inlighting.org/archives/java-implement-bloom-filter/)