# kioapp

# A brief explanation of the procedure taken to ﬁnd the postcodes.
(Beginning of the notebook; It is best to follow explanation in parallel with viewing the notebook and code comments.)
Dataset checking for cleaning.
Note: some checks which didn't yield any abnormal results were omitted in the final version of the notebook.
Were removed some outliers in geographical sense, and one very expensive deal, since the task is clusterization into bigger groups.

As it is also noted in the task, there is a correlation between living_area_sqm and price_sqm. In particular, the bigger the area, the cheaper price_sqm. Dependency is not inverse (which would mean that prices of all apartments are similar disregarding the area). Hence, logarithmic model was chosen for modelling this dependency:

price_sqm = a + b*log(living_area_sqm)

This model determines a logarithmic function described by parameters (a,b). Any curve yields a price_sqm for any realistic living_area_sqm. Apartments which belong to the same curve are viewed as belonging to a similar price class (or category), hence this model helps to compare prise-wise property of apartments of any area to each other.
To compare data item curves to each other, we need curves to not intersect. This is most intuitively and simply done by fixing parameter a and comparing parameter b of logarithmic curve scaling (this assumes living area is always bigger than 1 sqm). For this, average curve parameters over the dataset are needed.
This is done by linear regression with pre-computed logarithms of living areas:

P = [ln(S) 1] * [a, b]
[a, b] = ([ln(S) 1]' * [ln(S) 1])^-1 * [ln(S) 1]' * P

After this, b" is computed for every data item (p", s") using fixed a:
b" = (p"-a) / ln(s")

Note: This model is chosen for its simplicity among models which answers well enough to needs of the task. There could be better models or this model could be tuned in more complex ways; this wasn't explored further.

At this point data can be already divided into price-wise categories if geolocation is not important. See histogram of curve coefficient distribution for example. This is also quite easy to request certain 'price class' (from b_low to b_high) items from database. (b_low < (p"-a) / ln(s") < b_high)

However as far as I understood, this task wants to use geographical data for clusterization too. Hence we need to optimize over 2 tasks: geometrical clusterization and by price. Since similar price classes are distributed in many geolocations, tasks could be viewed as independent, at least there will be different locations with similar price classes.
For geometrical clusterization, k-means type algorithms can be used. Price coefficient is added as third dimension.
Important: Weights for geographical distance and for 'price curve' distance from a group centre determine significance and balance of the tasks. When increasing weight of prise-wise grouping, some group members can be located in separate locations in area of another (neighbouring) clusters because of price property.

Starting parameters for classes were sampled from the dataset points to avoid classes not getting any members from the beginning of the algorithm. This is a simple solution and it is good enough for the current task. Otherwise starting with non-empty groups can be done in other ways; this wasn't explored further.

Were performed tests for various group numbers (k up to 15) and a number of tests per each k.
Further, are visualized best cost-wise results for every number of groups. Colours are assigned in order of increasing of the group price coefficient.

#  A JSON ﬁle containing the postcodes, each of which is a list of locations.
In the last part, results for best experiment for k = 15 are saved into JSON file.
# An SQL query that returns the latest transaction at each location.
For the same experiment is constructed MySQL command which returns the latest transaction from a desired group x.
Membership in group x is determined analogically to softmax: cost for group x must be smaller than for all other groups.

It is possible to merge a few groups together which have close price property, decreasing total number of groups. However this would make impossible to use the same MySQL request. Alternatively requests can be done first for a few close groups and then results joined.

Minus of k-means is relative difficulty of setting the balance between 2 optimization problems (finding optimal parameters). Secondly, optimization costs increasing in terms of distance from a class centre is not necessarily the best model for the given task. However this approach is simple enough and computing distances is feasible for SQL request.

For comparison we can get a visually better price heat map by computing for each datapoint an average of neighbour price coefficients and visualizing this. However such zones depending on the actual underlying dataset are not as easy to parametrize for further SQL requests.

Certain values could be normalized and denormalized for computation but this wasn't done for the sake of better visibility of all stages and due to methods used not requiring this for better performance.
