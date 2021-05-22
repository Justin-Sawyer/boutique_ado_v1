## TOASTS & MESSAGES

Toasts are little pop up elements found in Bootstrap.

We can combine them with the django messages framework to communicate with our users as they use our store.

### Toasts
1. Create a toasts subfolder in the templates/includes folder
2. Create HTML files and name them accordingly: 

	`toast_success.html`, `toast_warning.html` etc

3. Copy in the HTML from bootstrap, and amend as needed. The data-auto-hide and data-dismiss attributes are required to prevent auto-hiding the notification after a few seconds and instead give the user the ability to dismiss it on their own.

4. Add the message template variable to the body of the toast: 

	`{{ message }}`

5. With the toasts created, add to the HTML in the base.html `{% if messages %}` block:

	1. Add loop to iterate through messages: 
		
		`{% for message in messages %}`

	2. Use include template variable to access the toast HTML files: 

		`{% include 'includes/toasts/toast_success.html' %}`

### Views
1. In the relevant views.py file, import messages from django.contrib: 

	`from django.contrib import messages`

2. Import the Product model: 

	`from products.model import Product`

3. Access the product in the function: 

	`product = Product.objects.get(pk=item_id)`

4. Add the relevant messages function in the relevant place: 

	`messages.success(request, f'Added {product.name} to your bag!')`

### The JS
1. Add the JS to the postloadjs block of base.html:

	```
	{% block postloadjs %}
    	<script type="text/javascript">
        	$('.toast').toast('show');
    	</script>
	{% endblock %}
	```

	Putting this in the base HTML template will ensure that every page that loads will immediately call the show option on all toasts that have been rendered in the messages container. This also explains why we've been including `{{ block.super }}` in our templates when overriding the postloadjs block. By doing so, we ensure that any JavaScript we've written in the templates that extend this one won't overwrite this call to show all the toasts.

### Settings
1. Update settings.py to tell it to store messages in the session: 

	`MESSAGE_STORAGE = 'django.contrib.messages.storage.session.SessionStorage'`

### Styling
1. In the CSS, give the toasts a high z-index to make sure they are above everything else on the page (= above, not ^ higher than)

Test the functionality

### Rendering different messages
1. Django messages have levels which are classifiers like debug info error and so on for different message types. In the Django Docs you'll see they can be represented with an integer as well, so we can choose which of our includes we want to render based on the message level.
	
	Given this, by using a `{% with message.level %}` statement, we can check the level of the message and render the appropriate toast in the base.html file:

    ```
    {% if messages %}
        <div class="message-container">
            {% for message in messages %}
                {% with message.level as level %}
                    {% if level == 40 %}
                        {% include 'includes/toasts/toast_error.html' %}
                    {% elif level == 30 %}
                        {% include 'includes/toasts/toast_warning.html' %}
                    {% elif level == 25 %}
                        {% include 'includes/toasts/toast_success.html' %}
                    {% else %}
                        {% include 'includes/toasts/toast_info.html' %}
                    {% endif %}
                {% endwith %}
            {% endfor %}
        </div>
    {% endif %}

    ```
2. Import `get_object_or_404` from `django.shortcuts` (since we know that the functionality above works, we can access the product object using get_object_or_404 instead of using the above product = Product.objects.get(). If no product matches, the user will be shown the 404 page): 

	`from django.shortcuts import render, redirect, reverse, HttpResponse, get_object_or_404`

3. Update the views accordingly: 

	`product = get_object_or_404(Product, pk=item_id)`

	Below are the views with their messages:

```
def add_to_bag(request, item_id):
    """ Add a quantity of the specified product to the shopping bag """

    product = get_object_or_404(Product, pk=item_id)
    quantity = int(request.POST.get('quantity'))
    redirect_url = request.POST.get('redirect_url')
    size = None
    if 'product_size' in request.POST:
        size = request.POST['product_size']
    bag = request.session.get('bag', {})

    if size:
        if item_id in list(bag.keys()):
            if size in bag[item_id]['items_by_size'].keys():
                bag[item_id]['items_by_size'][size] += quantity
                messages.success(request, f'Updated size {size.upper()} {product.name} quantity to {bag[item_id]["items_by_size"][size]}')
            else:
                bag[item_id]['items_by_size'][size] = quantity
                messages.success(request, f'Added size {size.upper()} {product.name} to your bag')
        else:
            bag[item_id] = {'items_by_size': {size: quantity}}
            messages.success(request, f'Added size {size.upper()} {product.name} to your bag')
    else:
        if item_id in list(bag.keys()):
            bag[item_id] += quantity
            messages.success(request, f'Updated {product.name} quantity to {bag[item_id]}')
        else:
            bag[item_id] = quantity
            messages.success(request, f'Added {product.name} to your bag')

    request.session['bag'] = bag

    return redirect(redirect_url)


def adjust_bag(request, item_id):
    """ Adjust the quantity of the specified product to the specified amount """

    product = get_object_or_404(Product, pk=item_id)
    quantity = int(request.POST.get('quantity'))
    size = None
    if 'product_size' in request.POST:
        size = request.POST['product_size']
    bag = request.session.get('bag', {})

    if size:
        if quantity > 0:
            bag[item_id]['items_by_size'][size] = quantity
            messages.success(request, f'Updated size {size.upper()} {product.name} quantity to {bag[item_id]["items_by_size"][size]}')
        else:
            del bag[item_id]['items_by_size'][size]
            if not bag[item_id]['items_by_size']:
                bag.pop(item_id)
            messages.success(request, f'Removed size {size.upper()} {product.name} from your bag')
    else:
        if quantity > 0:
            bag[item_id] = quantity
            messages.success(request, f'Updated {product.name} quantity to {bag[item_id]}')
        else:
            bag.pop(item_id)
            messages.success(request, f'Removed {product.name} from your bag')

    request.session['bag'] = bag

    return redirect(reverse('view_bag'))


def remove_from_bag(request, item_id):
    """Remove the item from the shopping bag"""

    try:
        product = get_object_or_404(Product, pk=item_id)
        size = None
        if 'product_size' in request.POST:
            size = request.POST['product_size']
        bag = request.session.get('bag', {})

        if size:
            del bag[item_id]['items_by_size'][size]
            if not bag[item_id]['items_by_size']:
                bag.pop(item_id)
            messages.success(request, f'Removed size {size.upper()} {product.name} from your bag')
        else:
            bag.pop(item_id)
            messages.success(request, f'Removed {product.name} from your bag')

        request.session['bag'] = bag
        return HttpResponse(status=200)

    except Exception as e:
        messages.error(request, f'Error removing item: {e}')
        return HttpResponse(status=500)
```

5. Note that we also have an error message in the all_products() view of the products app. This will also be working now if the user submits a search with no search criteria.

### Taking toasts to the next level
Since the toasts are simple HTML code, they can be styled into a “mini shopping bag”. Thus whenever the user adds a product, or removes a product, the user can be presented with this mini shopping bag popup.

For this project, we only need to decorate the success toast:

```
<div class="toast custom-toast rounded-0 border-top-0" data-autohide="false">
    <div class="arrow-up arrow-success"></div>
    <div class="w-100 toast-capper bg-success"></div>
    <div class="toast-header bg-white text-dark">
        <strong class="mr-auto">Success!</strong>
        <button type="button" class="ml-2 mb-1 close text-dark" data-dismiss="toast" aria-label="Close">
            <span aria-hidden="true">&times;</span>
        </button>
    </div>
    <div class="toast-body bg-white">
        <div class="row">
            <div class="col">
                {{ message }}
                <hr class="mt-1 mb-3">
            </div>
        </div>
        {% if grand_total %}
            <p class="logo-font bg-white text-black py-1">Your bag: ({{ product_count }})</p>
            <div class="bag-notification-wrapper">
                {% for item in bag_items %}
                    <div class="row">
                        <div class="col-3 my-1">
                            <img class="w-100" src="{{ item.product.image_url }}" alt="Product image">
                        </div>
                        <div class="col-9">
                            <p class="my-0"><strong>{{ item.product.name }}</strong></p>
                            <p class="my-0 small">Size: {% if item.product.has_sizes %}{{ item.size|upper }}{% else %}N/A{% endif %}</p>
                            <p class="my-0 small text-muted">Qty: {{ item.quantity }}</p>
                        </div>
                    </div>
                {% endfor %}
            </div>
            <div class="row">
                <div class="col">
                    <strong><p class="mt-3 mb-1 text-black">
                        Total{% if free_delivery_delta > 0 %} (Exc. delivery){% endif %}: 
                        <span class="float-right">${{ total|floatformat:2 }}</span>
                    </p></strong>
                    {% if free_delivery_delta > 0 %}
                        <p class="mb-0 p-2 bg-warning shadow-sm text-black text-center">
                            Spend <strong>${{ free_delivery_delta }}</strong> more to get free next day delivery!
                        </p>
                    {% endif %}
                    <a href="{% url 'view_bag' %}" class="btn btn-black btn-block rounded-0">
                        <span class="text-uppercase">Go To Secure Checkout</span>
                        <span class="icon">
                            <i class="fas fa-lock"></i>
                        </span>
                    </a>
                </div>
            </div>
        {% endif %}   
    </div>
</div>

```

- ### [README](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/README.md)
- ### [PRODUCTS app](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/PRODUCTS.md)
- ### [SEARCH](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/SEARCH.md)
- ### [SORTING](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/SORTING.md)
- ### [THE SHOPPING BAG](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/THE-SHOPPING-BAG.md)