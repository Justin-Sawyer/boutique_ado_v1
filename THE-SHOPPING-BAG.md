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