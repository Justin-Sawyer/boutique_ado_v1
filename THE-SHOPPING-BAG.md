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

## Buttons for UI

Let's give the quantity selector on the product detail template a facelift.

```
<div class="input-group">
    <input class="form-control qty_input" type="number" name="quantity" value="1" min="1" max="99" data-item_id="{{ product.id }}" id="id_qty_{{ product.id }}">
</div>
```

Now if you were wondering why this input element was inside an input group with only a single input, this is why:

I'm going to attach some plus and minus buttons to this input to make it easier to use on mobile and also to align it more closely with our current black and white theme.
To do this I can just use the built-in input-group-append
`<div class="input-group-append"></div>`
and input-group-prepend 
`<div class="input-group-prepend"></div>`
classes from bootstrap and toss a couple of buttons in them with the appropriate font awesome icons.

```
<div class="input-group">
    <div class="input-group-prepend">
        <button class="decrement-qty btn btn-black rounded-0" 
            data-item_id="{{ product.id }}" id="decrement-qty_{{ product.id }}">
            <span class="icon">
                <i class="fas fa-minus"></i>
            </span>
        </button>
    </div>
    <input class="form-control qty_input" type="number" name="quantity" value="1" min="1" max="99" data-item_id="{{ product.id }}" id="id_qty_{{ product.id }}">
    <div class="input-group-append">
        <button class="increment-qty btn btn-black rounded-0" 
            data-item_id="{{ product.id }}" id="increment-qty_{{ product.id }}">
            <span class="icon">
                <i class="fas fa-minus"></i>
            </span>
        </button>
    </div>
</div>
```

Note the extra attributes on these buttons

`data-item_id="{{ product.id }}"`

and the id attribute itself

`id="increment-qty_{{ product.id }}"`

These will be used when we write the JavaScript which will handle updating the input box itself, since these buttons won't do anything by default.
1. Create an includes directory in the products template folder.
2. And then the HTML file: 

	`quantity_input_script.html`

3. The script to increment the quantity:

	```
	$('.increment-qty').click(function(e) {
		// Prevent default button action
    	e.preventDefault();
		// Climb up DOM tree to .input-group then down to first instance of .qty_input
    	var closestInput = $(this).closest('.input-group').find('.qty_input')[0];
		// Get value of closestInput as Integer
    	var currentValue = parseInt($(closestInput).val());
		// Add 1 to currentValue and set as closestInput
    	$(closestInput).val(currentValue + 1);
	})
	```
4. The script to decrement the quantity:

	```
	$('.decrement-qty').click(function(e) {
		// Prevent default button action
    	e.preventDefault();
		// Climb up DOM tree to .input-group then down to first instance of .qty_input
    	var closestInput = $(this).closest('.input-group').find('.qty_input')[0];
		// Get value of closestInput as Integer
    	var currentValue = parseInt($(closestInput).val());
		// Subtract 1 from currentValue and set as closestInput
    	$(closestInput).val(currentValue _ 1);
	})
	```

5. To test, insert the templates into the HTML:

	```
	{% block postloadjs %}
	{{ block.super }}
	{% include 'products/includes/quantity_input_script.html’ %}
	{% endblock %}
	```

6. Disable buttons past their min and max ranges

	1. HTML max min ranges:

		`<input class="form-control qty_input" type="number" name="quantity" value="1" min="1" max="99" data-item_id="{{ product.id }}" id="id_qty_{{ product.id }}">`


	2. The script:

		```
		// The itemId being passed as an argument is coming from the id attribute above. 
		// It is the id of the quantity of {{ product.id }}
		function handleEnableDisable(itemId) {
			// Get itemId as Integoer and assign it to currentValue
    		var currentValue = parseInt($(`#id_qty_${itemId}`).val());
			// if currentValue less than 2 assigned to minusDisabled
    		var minusDisabled = currentValue < 2;
			// if currentValue more than 99 assigned to plusDisabled
    		var plusDisabled = currentValue > 98;
			// Set buttons as disabled if values out of above range using .prop()
    		$(`#decrement-qty_${itemId}`).prop('disabled', minusDisabled);
    		$(`#increment-qty_${itemId}`).prop('disabled', plusDisabled);
		}
		```

		i. This function needs to be called every time a button is clicked, and thus needs to be appended to the increment/decrement functions above:

			```
			$('.decrement-qty').click(function(e) {
        		e.preventDefault();
        		var closestInput = $(this).closest('.input-group').find('.qty_input')[0];
        		var currentValue = parseInt($(closestInput).val());
        		$(closestInput).val(currentValue - 1);
				// Get the item_id from this data-item_id="{{ product.id }}"
        		var itemId = $(this).data('item_id');
				// Call handleEnableDisable function and argument
        		handleEnableDisable(itemId);
    		})

			$('.increment-qty').click(function(e) {
        		e.preventDefault();
        		var closestInput = $(this).closest('.input-group').find('.qty_input')[0];
        		var currentValue = parseInt($(closestInput).val());
        		$(closestInput).val(currentValue + 1);
				// Get the item_id from this data-item_id="{{ product.id }}"
        		var itemId = $(this).data('item_id');
				// Call handleEnableDisable function and argument
        		handleEnableDisable(itemId);
    		})
			```

7. As the default setting for the built-in buttons is 1, and as the scripts above are only run upon clicking the buttons, it is still possible to click the buttons and assign a value outside of the range - ie, from 1 to zero - upon page load. Thus, the minus button needs to de disabled by default on page load, and only come alive if a user first clicks plus. 

	```
	var allQtyInputs = $('.qty_input');
	for(var i = 0; i < allQtyInputs.length; i++) {
    	var itemId = $(allQtyInputs[i]).data('item_id');
    	handleEnableDisable(itemId);
	}
	```

	Since 1 is less than 2, but not greater than 98, the minus button will be inactive, but the plus button active when the page loads.

8. Associate the built-in up and down arrows of the input box with the + and - buttons:

	```
	$('.qty_input').change(function() {
    	var itemId = $(this).data('item_id');
    	handleEnableDisable(itemId);
	})
	```

The entire JS:
```
<script type="text/javascript">

    // Disable +/- buttons outside 1-99 range
    function handleEnableDisable(itemId) {
        var currentValue = parseInt($(`#id_qty_${itemId}`).val());
        var minusDisabled = currentValue < 2;
        var plusDisabled = currentValue > 98;
        $(`#decrement-qty_${itemId}`).prop('disabled', minusDisabled);
        $(`#increment-qty_${itemId}`).prop('disabled', plusDisabled);
    }

    // Ensure proper enabling/disabling of all inputs on page load
    var allQtyInputs = $('.qty_input');
    for(var i = 0; i < allQtyInputs.length; i++) {
        var itemId = $(allQtyInputs[i]).data('item_id');
        handleEnableDisable(itemId);
    }

    // Check enable/disable every time the input is changed
    $('.qty_input').change(function() {
        var itemId = $(this).data('item_id');
        handleEnableDisable(itemId);
    })

    // Increment quantity
    $('.increment-qty').click(function(e) {
        e.preventDefault();
        var closestInput = $(this).closest('.input-group').find('.qty_input')[0];
        var currentValue = parseInt($(closestInput).val());
        $(closestInput).val(currentValue + 1);
        var itemId = $(this).data('item_id');
        handleEnableDisable(itemId);
    })
    // Decrement quantity
    $('.decrement-qty').click(function(e) {
        e.preventDefault();
        var closestInput = $(this).closest('.input-group').find('.qty_input')[0];
        var currentValue = parseInt($(closestInput).val());
        $(closestInput).val(currentValue - 1);
        var itemId = $(this).data('item_id');
        handleEnableDisable(itemId);
    })
</script>
```

## Updating quantity from Shopping Bag
### HTML
The bag at the moment just display the template quantity value. We can change this and use virtually the same form as we do on the product page.

1. Remove {{ item.quantity }}
2. In its place, use form:

	```
	<form class="form update-form" method="POST" action="">
        {% csrf_token %}
        <div class="form-group">
            <div class="input-group">
                <div class="input-group-prepend">
			    <button class="decrement-qty btn btn-sm btn-black rounded-0" data-item_id="{{ item.item_id }}" id="decrement-qty_{{ item.item_id }}">
				    <span">
					    <i class="fas fa-minus fa-sm"></i>
				    </span>
			    </button>
                </div>
                <input class="form-control form-control-sm qty_input" type="number" name="quantity" value="{{ item.quantity }}" min="1" max="99" data-item_id="{{ item.item_id }}" id="id_qty_{{ item.item_id }}">
                <div class="input-group-append">
			    <button class="increment-qty btn btn-sm btn-black rounded-0" data-item_id="{{ item.item_id }}" id="increment-qty_{{ item.item_id }}">
				    <span">
					    <i class="fas fa-plus fa-sm"></i>
				    </span>
			    </button>
		    </div>
		    {% if item.product.has_sizes %}
			    <input type="hidden" name="product_size" value="{{ item.size }}">
		    {% endif %}
		    </div>
        </div>
	</form>
	```


      1. Note that we are using `item.item_id` here instead of `product.id` as we do on the product page. This is because of the for loop `{% for item in bag_items %}`
      2. Note also that the value in the `<input>` box has been amended, to reflect the quantity in the bag:

			`value="{{ item.quantity }}"`

3. Because there is no size selector box on this page, we’ll need to submit the size of the item the user wants to update or remove in a hidden input field in the form, if the product does in fact have sizes.

	```
	{% if item.product.has_sizes %}
		<input type="hidden" name="product_size" value="{{ item.size }}">
	{% endif %}
	```

4. Check how page renders.
    1. We have an error coming from the contexts.py file, as the page inspector is showing the value attribute as `value=“{item.by_size: {‘m’: 1}}`
	2. The inner loop of the contexts.py file needs to be `'quantity': quantity,` rather than `'quantity': item_data,`

5. Use the script from the include folder to make the buttons work:

	```
	{% block postloadjs %}
	{{ block.super }}
	{% include 'products/includes/quantity_input_script.html' %}
	{% endblock %}
	```

6. Submit the form
    1. We do not really want a submit button in this form, as it will cause reloading of the page.
    2. Instead, we can use `<a>` without href=“” attributes and submit using JavaScript:
        1. The a buttons:
			```
			<a class="update-link text-info"><small>Update</small></a>
			<a class="remove-item text-danger"><small>Remove</small></a>
			```
		2. The remove button needs to be pushed to the right, and have some attributes attached to it:

			`<a class="remove-item text-danger float-right" id="remove_{{ item.item_id }}" data-size="{{ item.size }}"><small>Remove</small></a>`

			i. The id attribute discerns the item itself

			ii. The data-size attribute discerns the size of the article the user wishes to remove

			iii. They will work together to only remove the size in question if the same article is being purchased but in two different sizes

### JS

1. .update-link

    ```
	// Update qty on click
    $('.update-link').click(function(e) {
        var form = $(this).prev('.update-form');
        form.submit();
    })
	```


2. .remove-item

	```
	// Remove item and reload on click
	$('.remove-item').click(function(e) {
        var csrfToken = "{{ csrf_token }}";
        var itemId = $(this).attr('id').split('remove_')[1];
        var size = $(this).data('size');
        var url = `/bag/remove/${itemId}`;
        var data = {'scrfmiddlewaretoken': csrfToken, 'size': size};

        $.post(url, data)
        .done(function() {
            location.reload();
        })
    })
	```

### CSS
1. Attach cursor pointer to the two anchor elements:

	```
	.update-link,
	.remove-item {
    	cursor: pointer;
	}
	```

