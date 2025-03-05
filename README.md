# clustering-closest-pairs
python jupyter book for describing the usage of the one and only clustering algorithm - closest-pairs



# What is clustering about?

In general, clustering is about knowing what individual points in a coordinate-system belong together.
So it's about grouping of individual points with an aay tobitrary count of attributes and any possible metric. Also the triangle equation doesn't need to be fulfilled.



## What are we looking for in clustering?

We are looking for a split-up of groups.
That is what we are really interested in.
Not how it can be combined or how it is connected, but a split-up for groups.
Including also, that individual points can resist as single group.

What we would have, if we would look for individuals and how to connect?
We would concentrate on combing only, and missing the split-up.
Many algorithm for clustering are going exactly this way. They start with individual groups and combine up till all are connected.
Although, I was long time researching for, I didn't find any algorithm which concentrate on split-up mainly.

There are hierarchical clustering with bottom-up or top-down approaches.
But to be honest, it's almost the same, you just use different measurement. In bottom-up you use a normal connecting measurement, in top-down you use a differencing measurement, or just the same, nur different points.
But they all are not combining both, bottom-up and differencing, calling: connecting with care on split-up moment.
So, to be honest, for what we choose top-down, when having equal results?
Just use bottom-up only.
These two techniques are called: agglomerative (bottom-up) and divisive hierarchical clustering. 
[https://www.geeksforgeeks.org/hierarchical-clustering-in-data-mining/]
They are coming from the area of statistics, where also their names from.

*So, we better forget the top-down.*


## What are we next interested in?

Let's see other way around:
What we are not interested in is, that all individuals can be in own groups, or all can be in one big group, because this is obvious and known before. So, again:

the reason for clustering is, to find a good split-up!



## What kind of methods currently exist?


Let's start with a short list of existing methods.
- Vector quantification
	- like: k-nearest neighbour
- hierarchical clustering


### vector-quantification / k-nearest neighbour

k-nearest neighbour is choosing k random points as start and attach the closest neighbours, till no other point is left.
That cannot be our aim, cause randomly it can choos two points in the same set, and the main-question "when to split" is not fulfilled in this clustering algorithm

So here is the focus on "where/when to split" and after studiing different clustering-algorithms, the best looks to be hierachical clustering, for me.


### Hierarchical clustering - existing algorithms

The hierarchical clustering can have different measurements:
- least distance between clusters (min)
	- The easiest, which result also in good approximation  
	- --> similar to the shown here, but calculates all distances between points again, so more time consuming
- Minimize max distance between clusters (max)  
	- looks for minimize the maximal distance between the points of two clusters
	- -->  same time-consuming and only good on divisive clustering
- Unweighted average linkage clustering (or UPGMA)
	- Is the average of all distances between points from cluster A and cluster B
	- --> creates mainly round clusters, at least no oval
- Weighted average linkage clustering (or WPGMA)
	- Which is like median linkage, but also changing distance matrix for points
	- Which is like taking the middle of two clusters by their center calculated before, and update the distances of all points to this new center.
	- --> That means, that the groups will be mainly round, never oval. Not interesting.
- Centroids
	- Which is minimizing the distance of two centers before, and choosing closest point to average-center as new centroid
	- --> causes shifting on the data, instead of interpreting, which might result in wrong cluster-distinction
- Median (WPGMC)
	- Like centroids, but not choosing the centroid point, but calculating new middle of one cluster by the middles before
	- --> better than centroids cause, shifting issue. Will be similar to this approach, except the delta-values are here typically implemented as commulative-dists
- Ward 
	- Minimizing distance between average points of two clusters and weighting according to variance-by-covariance 
	- Meaning weighting by prefer similar variance in the two clusters
	- --> That means, that the groups will be mainly round, never oval. Quite good, but not interesting
- And many more, which can be seen in:  https://en.m.wikipedia.org/wiki/Hierarchical_clustering 



# Focus on: When to split?

Despite all the different methods of unification of point to clusters, there is still the question: When to split?
This is typically estimated in the linkage-algorithms meanwhile clustering.
This splitting-criteria is often used as the height distance in the dendrogram.
The dendrogram is a graphical representation of the clusters, shown as an computer-science typical tree, with the elements on the bottom.
The dendrogram needs also a height, in the linkage-algorithm, to draw the point of the new cluster in that height.
This height is typically representated by the so called delta.
For the most measurements, delta is just the measurement-result between the unified clusters, cause the value increases continuously.
But some measurements need a separate delta-calculation. 


## What is the requested delta?


When we are true, we are looking for the maximal distance between the clusters.
Because this distance is max, when we just using all points as individual clusters, we should better looking for averaged distances:

*the average maximal distance between clusters*

Generally for any delta-calculation, we know, that, when the measurement is not increasing by its self, the delta needs to be accommulated values, cause else, the drawing point can be suddently below other groups, which were grouped before higher.




# The algorithm - closest-pairs measurement


Time to get into the details of the algorithm.
When we are only interested in keeping cluster together, but finding the biggest distance in the average,
the trick is not to struggle with the decision which measurement of split-up is best, but choose the next nearest pairs.
More over, split-up measurements have principally the same issue like unifying measurements, they need to decide which of the current points inside the clusters should be reused or skipped for the calculation.
So, you can use all points of each cluster, as the UPGMA is doing, or you can adjust the existing distance of all points, as the WPGMA is doing, or you recalculate middle points, as WPGMC is doing.
Or we are not following this approach of handling all points in unification of clusters.
In the end, the distance between the points are the basic-measurement on which the decision should be taken.
When recalculating all points again, it is not guarantied, that the measurements have no influence of the distincion between the groups.
Thats why we choose to look on the
sorted list of pairwise distances between points = pairDists



## Unification and simultane split-up

In the unification we take the list of pairwise distances between points and define:

a new cluster is combined by the two clusters, which comming together within the next item of pairDists.

So we iterate through the pairDists and unify to a new cluster, when those two points from the next item in pariDists is in separate clusters.
Of course, we assume the initial situation is that all points have its own cluster.
To implement this, a union-find data-structure is extremly suitable, because it can easily add a point to a set and find the root of a point in a set. By looking for the root, we have directly the name of the set. When both names differ, we can unify those sets to the new set/cluster.

The next cool thing is, that the unification step before, was also the biggest distance before.
Meaning the delta is directly calculated out of the lastDistOfClusters.
Unfortunatelly, this doesn't take all existing split-up-distance between the clusters in the current iteration.
So, we need to go in an second loop through all left pairDists.
We add all those pairDists together, which are:
- not belonging to the same cluster. (mandatory)
- not both points are used already in split-up-distance calculation (delta-calc). (mandatory)
	- because, then both points does not represent a new distance between clusters.
- the two clusters, they belongs to, are not already in dist calculation used. (mandatory)
	- because, then the dist between both clusters is already calculated.
- or/and are the smallest dist to each clusters is summed-up. (optional - to avoid crossing)

In the end, the mandatory hints provide already a delta-measurement, which can be used to be maximized.
Meaning, when the maximum of these delta-dist is reached in one iteration, this is the final clustering.



# Conclusion

It often happen, that the unification-measurments can influence the resulting search for the split-up.
So we recommen to use a sorted pairDists list to iterate over for unification and for specifying the split-up moment.
The concentration on the pairwise points-dist only guaranties that the split-up is definitely happen, when the next unification would be biggest step. 
Additionally the concentration on pairwise points-dist is fairer then any other measurement, which includes many other points repeatedly.
More over, we think, that the pairwise points-dist is the natural type of building/combining groups together.
There always exists one person/item, which works as bridge.

If you are looking for a good hierarchical clustering algorithm, more over a grouping algorithm in general, then please consider this closest-pairs algorithm!

In all my researches since more than ten years, I did not find a better one, cause all other have direct flaws.

Be happy to try with this implementation in python in this repo!


*Kind regards, Basti.YourDeveloper*






