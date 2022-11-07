### Catalog.API
**Configuring MongoDB**
- Pull Docker images
- Following commands will be useful

```bash
#  Now we can open interactive terminal for mongo

docker exec -it shopping-mongo /bin/bash

# Next step is type mongo in the new interactive
root@18260a70c28d:/# mongo

# After that, we are able to run mongo commands. 
Let me try with 

 - create database
 - create collection
 - add items into collection
 - list collection


ls
mongo
show dbs
use CatalogDb  --> for create db on mongo
db.createCollection('Products')  --> for create people collection

db.Products.insertMany([{ 'Name':'Asus Laptop','Category':'Computers', 'Summary':'Summary', 'Description':'Description', 'ImageFile':'ImageFile', 'Price':54.93 }, { 'Name':'HP Laptop','Category':'Computers', 'Summary':'Summary', 'Description':'Description', 'ImageFile':'ImageFile', 'Price':88.93 } ])

db.Products.insertMany(
			[
			    {
			        "Name": "Asus Laptop",
			        "Category": "Computers",
			        "Summary": "Summary",
			        "Description": "Description",
			        "ImageFile": "ImageFile",
			        "Price": 54.93
			    },
			    {
			        "Name": "HP Laptop",
			        "Category": "Computers",
			        "Summary": "Summary",
			        "Description": "Description",
			        "ImageFile": "ImageFile",
			        "Price": 88.93d
			    }
			])

db.Products.find({}).pretty()
db.Products.remove({})

show databases
show collections
db.Products.find({}).pretty()
```

**To update package in Nuget Package**
```bash
 Update-Package -ProjectName Catalog.API
```