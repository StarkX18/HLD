Nearest neighbour systems

Uber - locations of people! Drivers etc
Zomato - location of delivery partners etc
Google maps - user, place etc

Nearest may mean different things depending on problem statement

These kind of queries would be difficult to answer using conventional db - why?

Sql?

circle radius based approach :
------------------------------------------------------
we store longitude and lattitude -> find distance from our location to each -> find top K results and display

problem: in each case - will HAVE to iterate over ALL data points and run the dist query

as no way to know the nearest ones, no matter what index we choose!


Pin Code based approach -

pros: 
- easy to query 

cons
- nearest might not show up if not in the same pin-code 
=> but easy to solve : we can store neighbouring pincodes and search all

- what if a pincode and its neighbours area is large? 
- what is too many restaurants?

--------------------------------------------------------
rectangle based query

- easier coordinate system 
- indexing can be done on both lattitude and longitude
- very fast queries? NO

- only SINGLE index can be used: one will have binary scan, other, linear.

why? DB can only use one index at a time - verify!!!!!

K-d tree : hack DS that allows you to use multiple indices at a time

=> builds a k-dimensional index at once
- not efficient in worst case but average case is good

- if k~N : curse of dimensionality

A  2D K-d tree is a quad tree

Lets consider a grid based approach

we can set the height and width of each grid to a fixed dimsensions
- each cell can be numbered according to a particular layout scheme

-> each item can be marked as lying in a particular cell

for nearest queries : search this cell and sorrounding cells

problem:

wastefulness for low density areas / zero density areas like oceans etc

too high dimensions for high density areas

magical solution?

a dynamic grid that adjusts its dimensions based on the density of the area

-> lets put a constraint: each cell must not have more than 5 restaurants

->  its like the SWC pro question! recursively split the cells like 

((1001)(1110)00)

-> start with the entire world into 4 quads - do this recursively based on constraint defined above

-> similar to R trees but without overlaps

How to do this for a moving target etc - insertions deletions etc  recomputations etc for next class