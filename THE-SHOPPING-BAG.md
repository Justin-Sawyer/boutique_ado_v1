# THE SHOPPING BAG
## General Set Up

When dealing with money, [Decimal](https://docs.python.org/3/library/decimal.html) is needed. See “The TRACKING logic, 8” below

1. Create bag app: 

	`python3 manage.py startapp bag`

2. Create view in bag/views.py
    1. Copy simplest view
    2. Change name
    3. Change rendered view
    4. Change docstring
3. Create templates folder (bag/templates)
4. Create app folder (templates/bag)
5. Create template file (bag.html)
6. Copy HTML from index.html and adjust as necessary
7. Create urls.py file (bag/urls.py)
8. Copy home/urls.py file into bag/urls.py and adjust the urlpatterns as necessary: 

	`path('', views.view_bag, name='view_bag')`

9. Include the bag URLs in the project level URLs file’s urlpatterns
10. Add the HTML link where necessary
11. Check link works

## The HTML Logic

1. If/else template for if shopping bag has items or is empty: 

	`{% if bag_items %}`

2. If empty, direct back to Products page: 

	`{% else %}`

## The TRACKING logic

1. Create contexts.py file in bag app
    1. This file is being created so that we can create a function that serves as a context processor. Since we need the contents of the bag to be available across the whole site, we need to do this.
    2. This file will handle the bag_items referenced above.
2. In contexts.py, create a view that renders the context dictionary, not a page:

	```
	def bag_contents(request):
    	context = {}
    	return context
	```

3. This function is a context processor. Its purpose is to make this dictionary available to all templates across the entire application. In order to make this context processor available to the entire application we need to add it to the list of ‘context processors’ in the TEMPLATES variable in settings.py: 
	
	`'bag.contexts.bag_contents',`

    1. This simple change means that anytime we need to access the bag contents in any template across the entire site they'll be available to us without having to return them from a whole bunch of different views across different apps.

4. Still in settings.py, add variables if needed for delivery and free delivery, and set their default values:

	```
	FREE_DELIVERY_THRESHOLD = 50
	STANDARD_DELIVERY_PRECENTAGE = 10
	```

5. The bag_items need to be placed inside a [list] in the bag_contents() function, contexts.py: 

	`bag_items = []`

6. Create “total” and “order_count” variables and set their values to 0

7. Write logic for delivery charges, if applicable:

	```
	if total < settings.FREE_DELIVERY_THRESHOLD:
        delivery = total * Decimal(settings.STANDARD_DELIVERY_PRECENTAGE)
	```

8. Import Decimal and settings:

	```
	from decimal import Decimal
	from django.conf import settings
	```

9. Add delta to let shopper know how near they are to free delivery: 

	`free_delivery_delta = settings.FREE_DELIVERY_THRESHOLD - total`

10. Else, if shopper goes over free delivery threshold and delta, set delivery to zero:

	```
	else:
    	delivery = 0
    	free_delivery_delta = 0
	```

11. Calculate the grand total: 

	`grand_total = delivery + total`

12. Add all the above variables to the context:

	```
	context = {
        'bag_items': bag_items,
        'total': total,
        'product_count': product_count,
        'delivery': delivery,
        'free_delivery_delta': free_delivery_delta,
        'free_delivery_threshold': settings.FREE_DELIVERY_THRESHOLD,
        'grand_total': grand_total,
    }
	```

13. Check it all renders

14. git add, commit, push

## Adding Products to the Shopping Bag
### Adding quantity, item_id and the Product

1. Add form to relevant page
2. Add `{% csrf_token %}` to the form as we are POST submitting it
3. Write view in bag app bag/views.py
    1. The function needs to take in request and item_id:

		`def add_to_bag(request, item_id):`

    2. Add docstring
    3. Get quantity and set it as quantity variable. Convert quantity to int: 

		`quantity = int(request.POST.get('quantity'))`

    4. Get url for redirect once quantity has been added to bag: 

		`redirect_url = request.POST.get('redirect_url')`

		1. This redirect come from the hidden input in the HTML form: 

			`<input type="hidden" name="redirect_url" value="{{ request.path }}">`

    5. Get or Set session cookie: 
		
		`bag = request.session.get('bag', {})`

        1. get ‘bag’ if it exists, set ‘bag’ as empty {dictionary} if it doesn’t

    6. Create bag key and set the item_id (the key) to be equal to the quantity (the value): 

		`bag[item_id] = quantity`

        1. This will return a dictionary of {item_id: quantity}

    7. If the item is already listed in the bag - in other words if there's already a key in the bag dictionary matching this product id, then increment its quantity accordingly:

		```
		if item_id in list(bag.keys()):
    		bag[item_id] += quantity
		else:
    		bag[item_id] = quantity
		```

    8. Add the bag variable into the session, which itself is just a python dictionary:
		`request.session['bag'] = bag`

    9. Import redirect: 

		`from django.shortcuts import render, redirect`

    10. Redirect back to the redirect_url variable: 

		`return redirect(redirect_url)`

4. Add path in bag/urls.py: 

	`path('add/<item_id>/', views.add_to_bag, name='add_to_bag'),`

5. Add url path to the action in the HTML form: 

	`action="{% url 'add_to_bag' product.id %}"`

6. Test by printing the shopping bag from the session in the add to bag view: 

	`print(request.session['bag'])`

7. git add, commit, push

8. Access the shopping bag in the session in contexts.py
    1. As step 3, 5 above, but in contexts.py bag_contents()

9. Populate the values in the (at the moment empty) [bag_items] with the bag.items from the session cookie:
    1. Iterate through item and quantity: 

		`for item_id, quantity in bag.items():`

    2. Get the Product object or 404: 

		`product = get_object_or_404(Product, pk=item_id)`

    3. Add append quantity*price to total: 

		`total += quantity * product.price`

    4. Update the product_count: 

		`product_count += quantity`

    5. Append dictionary of item_id, quantity and the product object to bag_items. Getting the product object makes its other fields (image, etc) available:

		```
		bag_items.append({
        	'item_id': item_id,
        	'quantity': quantity,
        	'product': product
    	})
		```

10. Import get_object_or_404: 

	`from django.shortcuts import get_object_or_404`

11. Import the Product model: 
	
	`from products.models import Product`

12. Render the bag items on the screen:

	```
	{% if bag_items %}
    	<div class="table-responsive rounded">
        	{{ bag_items }}
    	</div>
	{% else %}
	```

13. If ok, git add, commit

14. Assuming that there is information being passed into the {{ bag_items }} template, we can now write the HTML to extract the details by looping through what is in the bag:

	```
	{% for item in bag_items %}
    <tr>
        <td class="p-3 w-25">
            <img class="img-fluid rounded" src="{{ item.product.image.url }}" alt="Item image">
        </td>
	```

    1. Note that in the src, we’re calling the item iteration, then the product, then the entry required (url, price…)

15. git add, commit, push

## Adding extra fields to DB entries:

Since some products might have different data fields - like sizes for clothing, lack of sizes for non clothing items or weights for food items - we need to add the field in the DB, the Product model and of course, in the HTML. The easiest way to do this is in the shell.

1. Add field to Product class: 

	`has_sizes = models.BooleanField(default=False, null=True, blank=True)`

2. Run migrations as we’re changing the structure of the DB
    1. Dry run to check first: 

		`python3 manage.py makemigrations --dry-run`

    2. Make migrations: 

		`python3 manage.py makemigrations`

    3. Flag migrate changes: 

		`python3 manage.py migrate --flag`

    4. Migrate: 

		`python3 manage.py migrate`

3. Open shell. From the shell, we can easily set which do and which do not have the field we’ve added
    1. Open shell command: 

		`python3 manage.py shell`

    2. Import the Product model: 

		`from (app).models import Product`

    3. Create a variable to hold a list of categories we want to ignore: 
	
		`name_of_variable = [‘category_name’, ‘category_name’]`

    4. Use the exclude() method to get all other categories: 

		`others = Product.objects.exclude(category__name__in=name_of_variable)`

    5. Get the count(): 

		`others.count()`

    6. Loop through and add the new field:

		```
		for item in others:
			item.has_sizes = True
			item.save()
		```

    7. Filter() to check: 

		`Product.objects.filter(has_sizes=True)`

    8. Get the count again to check all applied: 

		`Product.objects.filter(has_sizes=True).count()`

    9. Exit the shell: 

		`exit()`

4. Add the HTML and template logic to the HTML
5. Use the with block, in order to reuse the loop if the product has sizes:

	```
	{% with product.has_sizes as s %}
	{% if s %}
	{% endif %}
	{% endwith %}
	```

## Adding these extra fields to the view and the context processor
### Updating the view

1. Set size initially to None: 

	`size = None`

2. If product_size is in request.POST, set size to equal request.POST(‘size’):

    ```
	if 'product_size' in request.POST:
        size = request.POST[‘product_size']
	```

3. If size statement: 

	```
	# if size in post request
	if size:
		# If the item is already in the bag
		if item_id in list(bag.keys()):
			# Check if another item of the same id and same size already exists
			if size in bag[item_id]['items_by_size'].keys():
				# if yes, increment the quantity for that size
				bag[item_id]['items_by_size'][size] += quantity
			# If item of the same id and same size does not exist in the bag
			else:
				# Add the quantity for that size
                bag[item_id]['items_by_size'][size] = quantity
		# If the item is not already in the bag
		else:
			# Add the item to the bag
			bag[item_id] = {'items_by_size': {size: quantity}}
	```

    If the item is not already in the bag, add it as a dictionary with a key of ‘items_by_size’ since there may be multiple items with this item_id, but in different sizes. This allows structuring the bag such that we can have a single item id for each item, but still track multiple sizes.

4. Else (if no size in post request) statement is the original code wrapped in the else block:

	```
	else:
        if item_id in list(bag.keys()):
            bag[item_id] += quantity
        else:
            bag[item_id] = quantity
	```

### Updating contexts.py

1. Since there are now two different types of data that might be in our bag items - with sizes and without sizes - the loop variable - quantity - needs to be changed to something more generic, like item_data:
    1. In the case of an item with no sizes, the item_data will just be the quantity.
    2. But in the case of an item that has sizes the item_data will be a dictionary of all the items_by_size.
    3. The key in the template remains as ‘quantity’ since we do still want that to actually be called quantity.

		```
		for item_id, item_data in bag.items():
    		product = get_object_or_404(Product, pk=item_id)
    		total += item_data * product.price
    		product_count += item_data
    		bag_items.append({
        		'item_id': item_id,
        		'quantity': item_data,
        		'product': product
    			})
		```

2. The entire amended contexts.py code block below:
	1. `if isinstance(item_data, int)` block deals with items without sizes
	2. `else` block deals with items with sizes

		```
		def bag_contents(request):

			bag_items = []
			total = 0
			product_count = 0
			bag = request.session.get('bag', {})

			for item_id, item_data in bag.items():
			# Since quantity will always be an Integer:
				if isinstance(item_data, int):
					product = get_object_or_404(Product, pk=item_id)
					total += item_data * product.price
					product_count += item_data
					bag_items.append({
						'item_id': item_id,
						'quantity': item_data,
						'product': product
					})
				else:
					product = get_object_or_404(Product, pk=item_id)
					for size, quantity in item_data['items_by_size'].items():
						total += quantity * product.price
						product_count += quantity
						bag_items.append({
							'item_id': item_id,
							'quantity': item_data,
							'product': product,
							'size': size
						})

			if total < settings.FREE_DELIVERY_THRESHOLD:
				delivery = total * Decimal(settings.STANDARD_DELIVERY_PRECENTAGE)
				free_delivery_delta = settings.FREE_DELIVERY_THRESHOLD - total
			else:
				delivery = 0
				free_delivery_delta = 0

			grand_total = delivery + total

			context = {
				'bag_items': bag_items,
				'total': total,
				'product_count': product_count,
				'delivery': delivery,
				'free_delivery_delta': free_delivery_delta,
				'free_delivery_threshold': settings.FREE_DELIVERY_THRESHOLD,
				'grand_total': grand_total,
			}

			return context
		```

Visual representation of amended code:

```
{
  "9": 1,		# Integer, thus object without size
  "21": 1,		# Integer, thus object without size
  "37": 8,		# Integer, thus object without size
  "73": {		# Dictionary, thus object with size
    "items_by_size": {
      "m": 1
    }
  },
  "139": {		# Dictionary, thus object with size
    "items_by_size": {
      "m": 1,
      "xl": 1
    }
  }
}
```