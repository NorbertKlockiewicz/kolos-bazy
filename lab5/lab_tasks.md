Enter data
There is a  map of the AGH Campus. Create a graph using Cypher and load it into your Neo4J instance in stages.

Create the following vertices which represent buildings: S-1, S-2, D-1, U-2, A-4, A-3, C-7, C-6, C-5, C-4, C-3, C-2, U-1, H-A2, H-A1, A-2, A-1, C-1,A-0, B-1, B-2, B-3, B-4, H-B1B2, for each building use a property to indicate its name, indicate that these are buildings (use BLDG label), and that they are of given kind: research and teaching (RAT label), service (SRV label), or under construction (UCO label).
and edges between proper vertices indicating how you can get from building to building without going outside, indicate floors at which the buildings are connected. E.g. C-3 is adjacent to C-2 and there is a foot path connecting them on each floor; mind that not all floors are connected e.g.  C-1 and A-1 is connected via 1st floor only, if you are not sure which buildings are connected at which floors make it up :), indicate a floor for each such a connection.
For existing nodes and edges from previous steps add faculty head quarter locations (labeled with numbers on the map), use a separate node for each headquarter labeled HQ with attribute of your choosing to indicate what headquarter it is. You must use MATCH and MERGE or CREATE queries. 
Add nodes indicating your lecture and lab classrooms (one lab and one lecture hall are enough).  The classrooms must be labeled CLS and store information about the room number and floor. You must use MATCH and MERGE or CREATE queries.
Analytics
Run the following queries:

are there any buildings that are not connected to other buildings,
how many service facilities there are,
how many connections to other buildings each building has,
what buildings A-1 is connected with and at which floors,
find all paths from C-4 to C-3 (include connections on different floors),
what is a shortest path (minimal number of buildings to go through) from C-3 to A-4?
what are the buildings from which you can walk to other three buildings that are directly adjacent to it, without going outside?
Modifications
Write the following queries, that:

change HQ to HEADQUARTER on all vertices with HQ label,
add 1000 to each classroom number,
remove all vertices representing classrooms,
