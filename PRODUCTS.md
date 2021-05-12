# PRODUCTS app

## Adding the app

1. Add images to media file
2. Create products app: 

	`python3 manage.py startapp products`

3. Add products app to INSTALLED_APPS in settings.py
4. Create fixtures folder inside products app: 

	`mkdir products/fixtures`

	(Fixtures are used to load data very quickly into a django database so we don't have to do it all manually in the admin)

5. Create JSON file(s) (if not already created) and add them to fixtures folder: 

	`<name_of_file>.json` 

	(categories.json, products.json)

	Note the model it uses: 
	
	products.product, 

	products.category

	“products” is taken from the name of our app, while “.product” is the file name.

	Together, these make up the model.

	```
	[
  	  {
    	"pk": 1,
    	"model": "products.product",
    	"fields": {
      		"sku": "",
      		"name": "Arizona Original Bootcut Jeans",
      		"description": "",
      		"price": 53.99,
      		"category": 6,
      		"rating": 4.6,
      		"image_url": "http://s7d9.scene7.com/is/image/JCPenney/DP0709201205510679M.tif?hei=380&amp;wid=380&op_usm=.4,.8,0,0&resmode=sharp2&op_usm=1.5,.8,0,0&resmode=sharp",
      		"image": "DP0709201205510679M.jpg"
      	}
      }, 
	  {“etc”: “etc”}
	]
	```

    6. Create models for the fixtures in products/models.py
        1. The models inherit from models.Model
        2. The variables are the fields from the JSON files
        3. If the models reference a different file, these are ForeignKeys
        4. Products refer to categories, but categories don’t have to reference products


		```
		class Category(models.Model):
    		name = models.CharField(max_length=254)
    		friendly_name = models.CharField(max_length=254, null=True, blank=True)

		class Product(models.Model):
    		category = models.ForeignKey('Category', null=True, blank=True, 		on_delete=models.SET_NULL)
		```


      	5. CharField = “short string: eg. name”
        6. TextField = “long string: eg. description”
        7. DecimalField = Int or Float
        8. URLField = “url string”
        9. ImageField = “image file name string”
        10. null=True: optional
        11. blank=True: optional
        12. on_delete=models.SET_NULL: sets product as null rather than deleting product entirely

		```
		class Product(models.Model):
    		category = models.ForeignKey('Category', null=True, blank=True, 		on_delete=models.SET_NULL)
    		sku = models.CharField(max_length=254, null=True, blank=True)
    		name = models.CharField(max_length=254)
    		description = models.TextField()
    		price = models.DecimalField(max_digits=6, decimal_places=2)
    		rating = models.DecimalField(max_digits=6, decimal_places=2, null=True, 		blank=True)
    		image_url = models.URLField(max_length=1024, null=True, blank=True)
    		image = models.ImageField(null=True, blank=True)
		```

        13. Create string method
		
		```
		def __str__(self):
        	return self.name
		```

        14. Create model method if necessary

		```
		def get_friendly_name(self):
        	return self.friendly_name
		```

7. dry run makemigrations: 

	`python3 manage.py makemigrations --dry-run`

8. If all OK, make migrations: 

	`python3 manage.py makemigrations`

9. If not, install suggestions, then dry run makemigrations again, etc.
10. Migrate with plan flag to ensure models are all ok: 

	`python3 manage.py migrate --plan`

11. If all OK, migrate: 

	`python3 manage.py migrate`

12. Register the product model in admin.py
    1. Import the model:

		`from .models import Product`

    2. Register the model:

		`admin.site.register(Product)`

    3. If more than one:

		```
		from .models import Product, Category

		admin.site.register(Product)
		admin.site.register(Category)
		```

13. Load the data to the DB: 

	`python3 manage.py loaddata`

    If more than one, stipulate which: 

		`python3 manage.py loaddata categories`
		
		`python3 manage.py loaddata products`

14. Confirm all OK on /admin page: 

	`python3 manage.py runserver`

15. git add, commit, push