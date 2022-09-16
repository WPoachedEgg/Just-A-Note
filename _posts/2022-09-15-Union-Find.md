---
layout: post
title:  "Union Find"
date:   2022-09-15 12:00:00
categories: Algorithm
tags: [UnionFind, QuickFind, QuickUnion]
---

**Cost model.**

Number of array accesses (for read or write).

| Algorithm  | Initialize  | Union  | Find |
| ----- | ----- | ----- | ----- |
| quick-find | N | N | 1 |
| quick-union | N | N | N |


M union-find operations on a set of N objects.

| algorithm | worst-case time |
| ---- | ---- |
| quick-find | M N |
| quick-union | M N |
| weighted QU | N + M log N |
| QU + path compression | N + M log N |
| weighted QU + path compression | N + M lg* N | <!-- more -->


## Union find
**Dynamic connectivity.** 

The input is a sequence of pairs of integers, where each integer represents an object of some type and we are to interpret the pair p q as meaning p is connected to q. We assume that "is connected to" is an equivalence relation:
* symmetric: If p is connected to q, then q is connected to p.
* transitive: If p is connected to q and q is connected to r, then p is connected to r.
* reflexive: p is connected to p.

An equivalence relation partitions the objects into equivalence classes or connected components.

**Connected components.** Maximal set of objects that are mutually
connected.

**Union-Find API.** The following API encapsulates the basic operations that we need.
![UFApi](/assets/uf-api.png)

## Quick Find 

[Eager Approach]

**Data structure.**

* Integer array id[] of length N.
* Interpretation: p and q are connected iff they have the same id.

**Find.** Check if p and q have the same id.

**Union.** To merge components containing p and q, change all entries whose id equals id[p] to id[q].

![UFApi](/assets/quick-find-overview.png)

[QuickFindUF.java](https://algs4.cs.princeton.edu/15uf/QuickFindUF.java.html)


## Quick Union

[Lazy Approach]

**Data structure.**
* Integer array id[] of length N.
* Interpretation: id[i] is parent of i.
* Root of i is id[id[id[...id[i]...]]].

**Find.**  Check if p and q have the same root.

**Union.** To merge components containing p and q,
set the id of p's root to the id of q's root.

```java
public class QuickUnionUF
{
  private int[] id;
  public QuickUnionUF(int N)
  {
    id = new int[N];
    for (int i = 0; i < N; i++) id[i] = i;
  }
  private int root(int i)
  {
    while (i != id[i]) i = id[i];
    return i;
  }
  public boolean connected(int p, int q)
  {
    return root(p) == root(q);
  }
  public void union(int p, int q)
  {
    int i = root(p);
    int j = root(q);
    id[i] = j;
  }
}
```  

## Improvements

**Quick-find defect.**
* Union too expensive (N array accesses).
* Trees are flat, but too expensive to keep them flat.

**Quick-union defect.**
* Trees can get tall.
* Find too expensive (could be N array accesses).

### Weighted quick-union.

* Modify quick-union to avoid tall trees.
* Keep track of size of each tree (number of objects).
* Balance by linking root of smaller tree to root of larger tree.

**Data structure.**
Same as quick-union, but maintain extra array sz[i] to count number of objects in the tree rooted at i.
**Find.** Identical to quick-union.

```java
  return root(p) == root(q)
```
**Union.** Modify quick-union to:
* Link root of smaller tree to root of larger tree.
* Update the sz[] array

```java
  int i = root(p);
  int j = root(q);
  if (i == j) return;
  if (sz[i] < sz[j]) { id[i] = j; sz[j] += sz[i]; }
  else { id[j] = i; sz[i] += sz[j]; } 
```

**Running time.**

* Find: takes time proportional to depth of p and q.
* Union: takes constant time, given roots.
* Proposition. Depth of any node x is at most lg N.
* Pf. When does depth of x increase?

  Increases by 1 when tree T1 containing x is merged into another tree T2.
  * The size of the tree containing x at least doubles since | T 2 | ≥ | T 1 |.
  * Size of tree containing x can double at most lg N times. Why?

### Weighted quick-union with path compression.

 There are a number of easy ways to improve the weighted quick-union algorithm further. Ideally, we would like every node to link directly to the root of its tree, but we do not want to pay the price of changing a large number of links. We can approach the ideal simply by making all the nodes that we do examine directly link to the root.

**Quick union with path compression.**
Just after computing the root of p, set the id of each examined node to point to that root

**Two-pass implementation:** add second loop to root() to set the id[] of each examined node to the root.

**Simpler one-pass variant:** Make every other node in path point to its grandparent (thereby halving path length).

```java
  private int root(int i)
  {
    while (i != id[i])
    {
      id[i] = id[id[i]]; // this line
      i = id[i];
    }
    return i;
  }
```

![pathcompression](/assets/union-pathcompression.png)



