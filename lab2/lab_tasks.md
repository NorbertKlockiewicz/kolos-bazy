Exercises
1. B-tree indexes
Create a B-tree index for the surname column in the employees table.

In this exercise, you should only use comparison operators (e.g. <, >=). Write a query that returns all employees whose last names (surnames) begin with the letter “A”. Inspect its query plan. Is the index used? If there are two steps to the plan, why is that?

Now write a query that returns the rest of the employees, i.e. those with last names beginning with “B”, “C”, and so on. Is the index used this time? Why?

Finally, try to encourage the database to use the index for the latter query as well, by altering the appropriate planner setting. Re-run the previous query.

Now, write a query that finds all employees with the first name “Piotr” and inspect its plan. Is there anything peculiar about the costs in this one?

2. Indexes and pattern matching
Again, write a query to find all employees with surnames beginning with “A” (case-sensitive), but this time use pattern matching. Check the plan.

Change the operator to the case-insensitive variant and review the plan (i.e., include both those beginning with “A” and with “a”). Is the index used now?

Finally, write a query find all employees whose last names end with “ski” (case-sensitive). Review the plan.

We will return to these two cases later.

Note: In PostgreSQL, varchar_pattern_ops (or text_pattern_ops) makes B-tree indexes usable for prefix searches like WHERE name LIKE 'abc%', which default indexes normally can’t optimize under non-C locales. It provides byte-wise ordering so the planner can use the index for pattern matching. When the database uses the simple C locale, this operator class isn’t needed because its default collation already orders strings by raw bytes (as in ASCII), making prefix comparisons indexable. See the PostgreSQL documentation on pattern matching operator classes. If needed, re-create the index with the appropriate operator class.

3. Multi-column indexes
Drop the index created in Exercise 1.

Create an index that includes both the last name and the first name (in this order).

Write a query that looks for Melania Pindor. Inspect the plan and note the cost.

Now inspect the plans for queries that look for:

all people with the last name “Pindor”,
all people with the first name “Melania”.
Explain the differences in the costs of the plans for these three queries.

4. Indexes and sorting
Write a query that returns all employees sorted by:

their last name (ascending),
their salary (descending).
Explain its execution plan.

Now reverse the sorting order (ascending/descending) and explain the plan. Did it change?

Inspect the plan with different values of the planner configuration setting you changed in Exercise 1.

5. Partial indexes
Drop the index used in the previous exercise. Create an index on the surname column, but only include the employees with salaries lower than 10,000.

Inspect the behaviour of the planner for queries with different selection predicates for the salary column – stricted and less strict than the one used for index creation.

6. Indexes on expressions
Remember the two cases we were not able to cover with plain pattern matching in Exercise 2? Try to solve those by creating indexes not directly on columns, but on appropriate expressions, and modifying the queries accordingly – keeping their semantics.

As usual, inspect and interpret the plans.

7. GiST indexes
In this exercise, we will demonstrate the use of GiST indexes. For that, we will use PostgreSQL’s built-in geometric types and the appropriate functions and operators. Indexing strategies are provided out-of-the-box with GiST for these.

Add a column called location to the employees table. It should store the locations of points on a 2D X-Y plane.

Fill the column with random points with coordinate values ranging between -100 and 100 – which will distribute them at random within a square shape.

Write a query that find all employees with locations in the top-right quarter (e.g., with positive values for both coordinates).

Create an index that is able to support that query and inspect its execution plan.

Now write a query that finds all employees that are not further than 20 units from (0, 0) and inspect the plan. For this part, try both an approach using the circle geometric primitive and one that uses the distance operator (<->). Try various things to make the database use the index.

Note: GiST will usually show a Bitmap Index Scan with a Recheck Cond because geometric shapes use bounding boxes to narrow candidates, then recheck the exact geometry. Using plain location[0] > 0 AND location[1] > 0 is logically fine but won’t leverage GiST; wrapping the predicate in a box @> point (or circle @> point) lets the index do its magic.

