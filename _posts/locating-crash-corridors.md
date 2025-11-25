---
layout: post
title: 'Locating pedestrian crash corridors with DBSCAN and PCA'
categories: [mapping]
---

I used some code to take the pedestrian crash points and highlight the corridor segments. Instead of drawing an arbitrary hexagon grid to find crash clusters, I used the data points themselves to indicate the linear pattern from which we can identify problem corridors where multiple pedestrians are hurt over time. That looks like these:

![Pedestrian crash corridors map](/assets/ped-crash-corridors.png)

To do this, I used the processed data I mapped initially. That data contains crashes without any coordinates or geography. Those had to be removed before the code would run. So Caveat \#1: this analysis only includes crashes for which a location was provided in the crash data.

Once removed, each crash point is converted into a pair of coordinates using a projected coordinate system (so distances are in meters, not degrees, which is what lat/lon are).

We then run a clustering algorithm called DBSCAN, which stands for Density-Based Spatial Clustering of Applications with Noise.

The algorith looks for places where points are close together and ignores the others as noise.

**In this case, we explicitly told DBSCAN that crashes have to be within 75 meters of each other (eps \= 75\) and that at least 10 crash points must be present (min_samples \= 10\) to count as a cluster.**

These settings are arbitrary, chosen by me to see what kind of hot spots jumped out. If they serve the purpose we need, we’ll keep them. If not, they can be easily modified and rerun to produce a different result — either greater or fewer corridor hotspots.

For each cluster that DBSCAN finds, we then use a statistical tool called PCA (Principal Component Analysis) to figure out the main direction in which those crashes line up.

You can think of PCA as asking: “If I had to connect these points with a line, what direction would it point?”

The result PCA produces gives us a center and a direction vector for each cluster. We then project every crash in that cluster onto that direction, find how far they extend at each end, and use the extreme points to draw a single line segment through the cluster.

The end result is a new layer made of line features, each representing a corridor with a statistically significant grouping of pedestrian crashes.

Each line has attributes like the cluster ID and the number of crashes that contributed to it. When this polyline layer is loaded into mapping software like QGIS and laid atop the original crash points, the highest-density, street-shaped problem areas become visible at a glance.

The caveats are straightforward but important:

1. crashes without valid geometry never appear in any corridor; and
2. the patterns you see are conditional based on the chosen parameters of 75 meters for neighborhood radius and a minimum of 10 points per corridor.
