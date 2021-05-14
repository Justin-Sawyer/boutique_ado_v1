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

### Admin
1. Tidying:
    1. Add Meta class and stipulate plural name if needed to model itself in products/models.py:

		```
		class Category(models.Model):
    			class Meta:
        			verbose_name_plural = 'Categories'
		```

    2. In products/admin.py, tell DB which field keys we want admin to see :
        1. Create classes:

			```
			# Register your models here.
			class ProductAdmin(admin.ModelAdmin):
			class CategoryAdmin(admin.ModelAdmin):
			```

        2. Use list_dislpay tuple to list fields:

			```
			class CategoryAdmin(admin.ModelAdmin):
    			list_display = (
       				'friendly_name',
        			'name',
    			)
			```

    3. Change ordering of display on backend if necessary. This is a tuple!:

		`ordering = ('sku',)`

    4. Import classes from .models if not done already
    5. Register classes:

		```
		admin.site.register(Product, ProductAdmin)
		admin.site.register(Category, CategoryAdmin)
		```

### Views:

1. In products/views.py we can import our project level home/views.py file and change the necessary fields. So for example:

	```
	def all_products(request):
    		""" A view to show all products, including sorting and search queries """
    		return render(request, 'products/products.html')
	```

2. Import the Product model:

	`from .models import Product`

3. Import all products to the function view:

	```
	def all_products(request):
    	""" A view to show all products, including sorting and search queries """

    	products = Product.objects.all()

    	return render(request, 'products/products.html')
	```

4. Add context to the return render and add the context dictionary to the function view:

	```
	def all_products(request):
    	""" A view to show all products, including sorting and search queries """

    	products = Product.objects.all()

    	context = {
       		'products': products,
   		}

   		return render(request, 'products/products.html', context)

	```		

### URLs

1. Copy home/urls.py as base for products.urls.py
2. Change the urlpatterns:

	```
	urlpatterns = [
    	path('', views.all_products, name='products'),
	]
	```

3. Include these urls in the urlpatterns of the project level urls.py file and give them a top-level url name:

	```
	urlpatterns = [
    	path('admin/', admin.site.urls),
    	path('accounts/', include('allauth.urls')),
    	path('', include('home.urls')),
    	path('products/‘, include('products.urls')),
	] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
	```

### Template

1. Create the appropriate directories. Remember we're creating that inner products directory to make sure that django knows which app these templates belong to: 

	`mkdir -p products/templates/products`

2. Create a products.html fileinside that directory and copy the content of the home template in as a shell.
3. Adjust HTML as needed.
4. Add template literal to make sure that the relevant things are being read from the DB: 
	
	`{{ products }}`

5. If DB details show up, we can then go about building the page using more template literals. In this example we’ve used:
    1. Loops: 
	
		`{% for product in products %}`

    2. If statements: 

		`{% if product.image %}`

    3. If [forloop.counter](https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#for): 

		`{% if forloop.counter|divisibleby:2 %}`

6. Git add, commit, push

