Exercises
Insert into the database a set of documents that represent a shopping list.

a) Each item should be represented as a single document:

{"product":"bread", "price": 2.2, "store":"Biedronka", "when":"weekly"}     
{"product":"tomato", "price": 2, "store":"Biedronka", "when":"weekly"}
{"product":"tomato", "price": 3, "store":"Lidl", "when":"weekly"}
{"product":"tomato", "price": 4, "store":"Auchan", "when":"weekly"}
{"product":"moon rocket", "price": 119, "store":"Auchan", "description":"Black and White Falcon Heavy"}
{"product":"lollipop", "store":"Lidl"}
b) add 

prince:1.1 
to the document which already contains: 

{"product":"lollipop", "store":"Lidl"}
Write a view that sorts all items according to the store's name.

Write a view that counts number of items to buy in each store.

Write a view which calculates an average price of each product (mind reduce/rereduce).

What is the URL which shows all items to buy at Biedronka? Hint: use the views you already have.

What is the URL which shows how many items is to buy at Biedronka? Hint: use the views you already have.