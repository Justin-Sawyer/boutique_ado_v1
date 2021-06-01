## STRIPE

We will roughly follow [this article](https://stripe.com/docs/payments/integration-builder)

### Set up

1. Insert the script in the head element of base.html:

	`<script src="https://js.stripe.com/v3/"></script>`

### Create the card element

In the checkout app, there are two empty `<div>` elements that were created for Stripe payments. These will be filled with Stripe itself, but we need to get two values for these to work.

1. The stripe public key
2. The client secret

These values are available on the Stripe account, but need to be added to the code.

The public key is designed to be public, so can be included in the code, but of course the client secret must not be.

We could add a script to each page that includes the public key, but a better way is to write an external script, and then call the script from the external file as and when needed. 

However, we cannot render Django template variables in external scripts. We need to use a builtin template filter - `json_script` -  to do so.

1. In checkout.html add the postloadjs block and super block. Inside, use the template filter to create scripts containing the keys above

	```
	{% block postloadjs %}
	{{ block.super }}
	# Get the keys from Django and make them available in JS
    {{ stripe_public_key|json_script:"id_stripe_public_key"}}
    {{ client_secret|json_script:"id_client_secret"}}

	{% endblock %}
	```

2. In checkout/views.py, assign the variables their values in the context of the checkout function

	```
    def checkout(request):
        bag = request.session.get('bag', {})
        if not bag:
            messages.error(request, "There's nothing in your bag yet")
            return redirect(reverse('products'))

        order_form = OrderForm()
        template = 'checkout/checkout.html'
            context = {
            'order_form': order_form,

	         # Add the real public key, but only supply test value for client secret
            'stripe_public_key': 'pk_test_…',
            'client_secret': 'test client secret',
        }

        return render(request, template, context)
    ```

3. Run server and inspect the HTML in the inspector

	`python3 manage.py runserver`

4. At the bottom of the HTML page are two scripts that contain the values assinged to the variables in the context.

	```
	<script id="id_stripe_public_key" type="application/json">'pk_test_...'</script>
	<script id="id_client_secret" type="application/json">'test client secret'</script>

	```

	Effectively, the values for the keys has been taken from Django, rendered as JSON, and then been assigned to the scripts' ids.


5. Create external JavaScript file and get the values from the scripts
	1. In the checkout/static/checkout, create js folder
	2. In checkout/static/checkout/js create file, named approproately: stripe_elements.js

6. Get the values from the scripts
	
	```
	// use text() method to get values, and slice() to remove the quotation marks
	var stripePublicKey = $('#id_stripe_public_key').text().slice(1, -1);
	var clientSecret = $('#id_client_secret').text().slice(1, -1);
	```

7. Assign a variable as a Stripe object that calls the public key

	`var stripe = Stripe(stripePublicKey);`

8. Get elements from the stripe variabme (object)

	`var elements = stripe.elements();`

9. Apply styling, if needed. This can be done directly from the JS file, as the card element we are creating below can take a style agument. 

    ```
    var style = {
        base: {
            color: '#000',
            fontFamily: '"Helvetica Neue", Helvetica, sans-serif',
            fontSmoothing: 'antialiased',
            fontSize: '16px',
            '::placeholder': {
                color: '#aab7c4'
            }
        },
        invalid: {
            color: '#dc3545',
            iconColor: '#dc3545'
        }
    };
    ```

10. Create the (HTML) card from the element in order to display it on the checkout page and apply the style argument

	`var card = elements.create('card', {style:style});`

11. Mount the card element to the empty `<div>`

	`card.mount('#card-element')`

12. Align styling of the form with the card element.

	Previously, we created the form that users will fill in when submitting their address etc, ready for delivery and to create their customer account. In the form is stated

	`self.fields[field].widget.attrs['class'] = 'stripe-style-input'`

	The divs that contain the Stripe elements have been given classes by Stripe when they were mounted. The original div element was

	`<div class="mb-3" id="card-element"></div>`

	but StripeElement has been added as a class autoatically.

	This of course means we can combine the two, to make a better visual impression.

	From the Stripe documentation, the following CSS has been added to the checkout/static/checkout/css/checkout.css file:

    ```
    .StripeElement {						
        box-sizing: border-box;
        height: 40px;
        padding: 10px 12px;
        border: 1px solid transparent;
        border-radius: 0px;
        background-color: white;
        box-shadow: 0 1px 3px 0 #e6ebf1;
        -webkit-transition: box-shadow 150ms ease;
        transition: box-shadow 150ms ease;
    }

    .StripeElement--focus {
       box-shadow: 0 1px 3px 0 #cfd7df;
    }

    .StripeElement--webkit-autofill {
        background-color: #fefde5 !important;
    }

    ```

	To this CSS, we need only add the class set in the form, above:

	```
    .StripeElement,
    /* class taken from forms.py */
    .stripe-style-input  {						
        box-sizing: border-box;
        height: 40px;
        padding: 10px 12px;
        border: 1px solid transparent;
        border-radius: 0px;
        background-color: white;
        box-shadow: 0 1px 3px 0 #e6ebf1;
        -webkit-transition: box-shadow 150ms ease;
        transition: box-shadow 150ms ease;
    }

    .StripeElement--focus,
    /* class taken from forms.py */
    .stripe-style-input:focus,
    .stripe-style-input:active {
       box-shadow: 0 1px 3px 0 #cfd7df;
    }

    .StripeElement--webkit-autofill {
        background-color: #fefde5 !important;
    }

    /* class taken from forms.py */
    .stripe-style-input::placeholder {
        color: #aab7c4;
    }

    ```

#### Card element code
##### The template block in the HTML

```
{% block postloadjs %}
{{ block.super }}

    {{ stripe_public_key|json_script:"id_stripe_public_key"}}
    {{ client_secret|json_script:"id_client_secret"}}

{% endblock %}
```

##### Django

```
def checkout(request):
    bag = request.session.get('bag', {})
    if not bag:
        messages.error(request, "There's nothing in your bag yet")
        return redirect(reverse('products'))

    order_form = OrderForm()
    template = 'checkout/checkout.html'
    context = {
        'order_form': order_form,
        'stripe_public_key': 'pk_test_…',
        'client_secret': 'test client secret',
    }

    return render(request, template, context)
```

##### JS

```
var stripe_public_key = $('#id_stripe_public_key').text().slice(1, -1);
var client_secret = $(‘#id_client_secret').text().slice(1, -1);
var stripe = Stripe(stripe_public_key);
var elements = stripe.elements();

var style = {
    base: {
        color: '#000',
        fontFamily: '"Helvetica Neue", Helvetica, sans-serif',
        fontSmoothing: 'antialiased',
        fontSize: '16px',
        '::placeholder': {
            color: '#aab7c4'
        }
    },
    invalid: {
        color: '#dc3545',
        iconColor: '#dc3545'
    }
};

var card = elements.create('card', {style: style});
card.mount('#card-element');
```

##### CSS

```
.StripeElement,
.stripe-style-input {
    box-sizing: border-box;
    height: 40px;
    padding: 10px 12px;
    border: 1px solid transparent;
    border-radius: 0px;
    background-color: white;
    box-shadow: 0 1px 3px 0 #e6ebf1;
    -webkit-transition: box-shadow 150ms ease;
    transition: box-shadow 150ms ease;
}

.StripeElement--focus,
.stripe-style-input:focus,
.stripe-style-input:active {
    box-shadow: 0 1px 3px 0 #cfd7df;
}

.StripeElement--webkit-autofill {
    background-color: #fefde5 !important;
}

.stripe-style-input::placeholder {
    color: #aab7c4;
}

```

### PaymentIntent
#### How Stripe works
1. When user goes to checkout page, the checkout view reaches out to Stripe and creates a Payment Intent for the value of the shopping bag

2. When the Payment Intent is created, Stripe returns a client secret that identifies it.

3. This is returned to the website and is sent to the template as the client secret variable

4. In the JavaScript on the client side, we’ll call the confirm card payment method from stripe js using the client secret which will verify the card number.

#### Errors
1. To handle realtime validation errors on the card element, add to JS file 

	```
	// Handle realtime validation errors on the card element
	card.addEventListener('change', function (event) {
    	var errorDiv = document.getElementById('card-errors');
    	if (event.error) {
        	var html = `
            	<span class="icon" role="alert">
                	<i class="fas fa-times"></i>
            	</span>
            	// Render the error message returned from Stripe in the 
            	// second (empty) div of the checkout.html file
            	<span>${event.error.message}</span>
        	`;
        	$(errorDiv).html(html);
    	} else {
        	errorDiv.textContent = '';
    	}
	});
	```

#### Calculate current value of the bag
##### The view

1. In checkout/views.py import the bag_contents from bag/contexts.py

	`from bag.contexts import bag_contents`

2. Request the current_bag dictionary from the checkout view

	```
	def checkout(request):
    	bag = request.session.get('bag', {})
    	if not bag:
       	 	messages.error(request, "There's nothing in your bag yet")
        	return redirect(reverse('products'))

		# Request bag_contents and store in current_bag variable 
		# so as not to overwrite the bag contents
    	current_bag = bag_contents(request)
    	order_form = OrderForm()
    	template = 'checkout/checkout.html'
    	context = {
        	'order_form': order_form,
        	'stripe_public_key': 'pk_test_…’,
        	'client_secret': 'test client secret',
    	}

    	return render(request, template, context)
	```

3. Get the grand_total from the current_bag contents and prepare grand-toal for Stripe


	```
	def checkout(request):
    	bag = request.session.get('bag', {})
    	if not bag:
       	 	messages.error(request, "There's nothing in your bag yet")
        	return redirect(reverse('products'))

    	current_bag = bag_contents(request)
		# Get the grand_total value
		total = current_bag['grand_total']
		# Since Stripe requires an integer (not a float), we have to round
		# up the total and multiply by 100
		stripe_total = round(total * 100)
    	order_form = OrderForm()
    	template = 'checkout/checkout.html'
    	context = {
        	'order_form': order_form,
        	'stripe_public_key': 'pk_test_…’,
        	'client_secret': 'test client secret',
    	}

    	return render(request, template, context)
	```

4. Install Stripe

	`pip3 install stripe`

5. Import Stripe to the view

	`import stripe`

6. Import setting from django.conf

	`from django.conf import settings`

##### The settings

1. Add currency variable

	`STRIPE_CURRENCY = 'usd'

2. Add PUBLIC KEY and CLIENT SECRET variables. We'll want to get these from the environment giving them an empty default value.

	```
	STRIPE_PUBLIC_KEY = os.getenv('STRIPE_PUBLIC_KEY', '')
	STRIPE_SECRET_KEY = os.getenv('STRIPE_SECRET_KEY', '')
	```

	IMPORTANT:

	The reason we're getting these from the environment is because even though the public key is already in our github from the last commit, we really don't want the secret key in there because the secret_key can be used to do everything on stripe including creating charges, making payments, issuing refunds, and even updating our own account information.

	So it's really important to keep the secret_key safe and out of version control.

3. Set the variable in gitpod by exporting them via the CLI

	`export STRIPE_PUBLIC_KEY=pk_test_...`

	`export STRIPE_SECRET_KEY=sk_test_...`

4. Copy these values to Environment Variables in GitPod

##### Create the payment intent

1. Back in the checkout/views.py checkout() add the Stripe variables, the payment intent and alert message in case of forgetting to set public key, and set the client secret to the intent

	```
	from django.shortcuts import render, redirect, reverse
	from django.contrib import messages
	from django.conf import settings

	from .forms import OrderForm
	from bag.contexts import bag_contents

	import stripe

	# Create your views here.
	def checkout(request):
		# Get the Stripe vairables from settings.py
    	stripe_public_key = settings.STRIPE_PUBLIC_KEY
    	stripe_secret_key = settings.STRIPE_SECRET_KEY

    	bag = request.session.get('bag', {})
    	if not bag:
        		messages.error(request, "There's nothing in your bag yet")
        		return redirect(reverse('products'))

    	current_bag = bag_contents(request)
    	total = current_bag['grand_total']
    	stripe_total = round(total * 100)
		# Set the secret key on Stripe
    	stripe.api_key = stripe_secret_key
		# Create the payment intent using stripe.PaymentIntent.create()		# and give it the amount and the currency
    	intent = stripe.PaymentIntent.create(
        		amount=stripe_total,
        		currency=settings.STRIPE_CURRENCY,
    		)

    	order_form = OrderForm()

    	if not stripe_public_key:
        	messages.warning(request, 'Stripe public key is missing. \
            	Did you forget to set it in your environment?')

    	template = 'checkout/checkout.html'
    	context = {
        		'order_form': order_form,
                    # Update to use the public key from settings.py
        		'stripe_public_key': stripe_public_key,
                    # Set the client secret to the intent
        		'client_secret': intent.client_secret,
    		}

    	return render(request, template, context)
	```

##### Add a listener to the payment form’s submit event

From the Stripe documentation:

```
// Handle form submit
var form = document.getElementById('payment-form');

form.addEventListener('submit', function(ev) {
    # Prevents 'POST'
    ev.preventDefault();
    # Instead, execute the following
    stripe.confirmCardPayment(clientSecret, {
        payment_method: {
            card: card,
            billing_details: {
                name: 'Jenny Rosen'
            }
        }
    }).then(function(result) {
        if (result.error) {
            // Show error to your customers (eg insufficient funds)
            console.log(result.error.message);
        } else {
            // The payment has been processed!
            if (result.paymentIntent.status === 'succeeded') {
                // Show a success message to your customer
                // There's a risk of the customer closing the window before callback
                // execution. Set up a webhook or plugin to listen for the
                // payment_intent.succeeded event that handles any business critical
                // post payment actions
            }
        }
    });
});

```

We shall amend the above to the follwoing

```
// Handle form submit
var form = document.getElementById('payment-form');

form.addEventListener('submit', function(ev) {
    ev.preventDefault();
    # Disable both the card element and the submit button to prevent 
    # multiple CC submissions
    card.update({'disabled': true});
    $('#submit-button').attr('disabled', true);
    # Confirm the card payment, passing it the clientSecret, and the card 
    # created in the global environment
    stripe.confirmCardPayment(clientSecret, {
        payment_method: {
            card: card,
        }
    }).then(function(result) {
        if (result.error) {
            # Send the error message to the card-errors duv
            var errorDiv = document.getElementById('card-errors');
            var html = `
                <span class="icon" role="alert">
                <i class="fas fa-times"></i>
                </span>
                <span>${result.error.message}</span>`;
            $(errorDiv).html(html);
            # Allow re-submitting of CC details
            card.update({'disabled': false});
            $('#submit-button').attr('disabled', false);
        } else {
            # If no errors, and clientSecret is good, then status has succeeded
            if (result.paymentIntent.status === 'succeeded') {
                form.submit();
            }
        }
    });
});
```

Now, we can go to the site and try. Submit a payment and look for the payment_intent.succeeded message in Stripe's web dashboard.

Use the following number to test the CC details

	`4242 4242 4242 4242`

If 
1. The form is submitted and
2. The payment show as successful on Stripe, 

then all is ok.

#### git add, commit, push

### Create the order in the database

In checkout/views.py

1. If request.method == 'POST', create the order
2. Otherwise, since the PaymentIntent has been created by opening the checkout.html page, the rest of the code - which is a GET request - needs to be wrapped in an else block

	```
	from django.shortcuts import render, redirect, reverse
	from django.contrib import messages
	from django.conf import settings

	from .forms import OrderForm
	from bag.contexts import bag_contents

	import stripe

	# Create your views here.
	def checkout(request):
        stripe_public_key = settings.STRIPE_PUBLIC_KEY
        stripe_secret_key = settings.STRIPE_SECRET_KEY

       if request.method=='POST':
            # Code for adding order to db goes here

        else:
            bag = request.session.get('bag', {})
            if not bag:
        	    messages.error(request, "There's nothing in your bag yet")
        	    return redirect(reverse('products'))

            current_bag = bag_contents(request)
            total = current_bag['grand_total']
            stripe_total = round(total * 100)
            stripe.api_key = stripe_secret_key
            intent = stripe.PaymentIntent.create(
        	    amount=stripe_total,
        	    currency=settings.STRIPE_CURRENCY,
    	    )

            order_form = OrderForm()

        if not stripe_public_key:
        	messages.warning(request, 'Stripe public key is missing. \
            	Did you forget to set it in your environment?')

        template = 'checkout/checkout.html'
        context = {
        	'order_form': order_form,
        	'stripe_public_key': stripe_public_key,
        	'client_secret': intent.client_secret,
    	}

        return render(request, template, context)
	```

3. The if block

	```
	if request.method == 'POST':
        # Get the shopping back
        bag = request.session.get('bag', {})

        # Create dictionary that will post fields to the db
        form_data = {
            'full_name': request.POST['full_name'],
            'email': request.POST['email'],
            'phone_number': request.POST['phone_number'],
            'country': request.POST['country'],
            'postcode': request.POST['postcode'],
            'town_or_city': request.POST['town_or_city'],
            'street_address1': request.POST['street_address1'],
            'street_address2': request.POST['street_address2'],
            'county': request.POST['county'],
        }
        # Create instance of the order form that uses the form_data dictionary
        order_form = OrderForm(form_data)
        # If the order form is valed
        if order_form.is_valid():
            # Save the order_form as the order
            order = order_form.save()
            # Iterate through the order for each item and save each as a line item
            for item_id, item_data in bag.items():
                try:
                    product = Product.objects.get(id=item_id) # The Product model will be needed
                    # Those without sizes
                    if isinstance(item_data, int):
                        order_line_item = OrderLineItem(    # The OrderLineItem will be needed
                            order=order,
                            product=product,
                            quantity=item_data,
                        )
                        order_line_item.save()
                    # Those with sizes
                    else:
                        for size, quantity in item_data['items_by_size'].items():
                            order_line_item = OrderLineItem(
                                order=order,
                                product=product,
                                quantity=quantity,
                                product_size=size
                            )
                            order_line_item.save()
                # Else, if product does not exist
                except Product.DoesNotExist:
                    messages.error(request, (
                        "One of the products in your bag was not found in our database."
                        "Please call us for assistance!"
                    ))
                    order.delete()
                    return redirect(reverse('view_bag'))
            # If user wishes to save info, request the session cookie
            request.session['save_info'] = 'save-info' in request.POST
            # Send user to checkout_success page, with the order_number from the order
            return redirect(reverse('checkout_success', args=[order.order_number]))
        else:
            messages.error(request, 'There was an error in your form. \
                Please double check your information.')

    ```

4. Import the two models

	```
	from .models import OrderLineItem
	from products.models import Product
	```

### Signals

We now have the order form being sent to the database. But to test the payment flow, we need to make sure the signals are working.

1. In checkout/__init__.py, set the configClass

	`default_app_config = 'checkout.apps.CheckoutConfig'`

### 0 not None

Currently, upon deleting all the lines of an order, we reset the Order to None. This will cause an error when rendering the template, as the code will try to see if None is less that the FREE_DELIVERY_THRESHOLD.

We thus need to make a change to the update_total function in checkout/models.py, to reset to zero instead of None when deleting items.

All we need to do is add "or 0" to the end of the self.order_total variable

	`self.order_total = self.lineitems.aggregate(Sum('lineitem_total'))['lineitem_total__sum'] or 0`

### Checkout Success

1. With all the above done, we can now create the view for checkout_success in checkout/views.py

    ```
    # Inherits the order_number
    def checkout_success(request, order_number):
        """
        Handle successful checkouts
        """
        # Get the session cookie if user wishes to save their details
        save_info = request.session.get('save_info')
        # Get the order_number from the Order model and set it as order
        order = get_object_or_404(Order, order_number=order_number)    # Order model will need to be imported, as will get_object_or_404
        # Send success message
        messages.success(request, f'Order succesfully processed! \
            Your order number is {order_number}. A confirmation \
            email will be sent to {order.email}.')

        # Delete the bag from the session as no longer needed
        if 'bag' in request.session:
            del request.session['bag']

        template = 'checkout/checkout_success.html'
        context = {
            'order': order,
        }

        return render(request, template, context)
    ```

2. Import Order

	```
	from django.shortcuts import get_object_or_404
	from .models import Order
	```

3. Create the url in urls.py

	`path('checkout_success/<order_number>', views.checkout_success, name='checkout_success'),`

4. Add checkout_success.html page

```
{% extends "base.html" %}
{% load static %}

{% block extra_css %}
    <link rel="stylesheet" href="{% static 'checkout/css/checkout.css' %}">
{% endblock %}

{% block page_header %}
    <div class="container header-container">
        <div class="row">
            <div class="col">
                
            </div>
        </div>
    </div>
{% endblock %}

{% block content %}
    <div class="overlay"></div>
    <div class="container">
        <div class="row">
            <div class="col">
                <hr>
                <h2 class="logo-font mb-4">Thank You!</h2>
                <hr>
                <p class="text-black">Your order information is below. A confirmation email will be sent to <strong>{{ order.email }}</strong>.</p>
            </div>
        </div>

        <div class="row">
            <div class="col-12 col-lg-7">
                <div class="order-confirmation-wrapper p-2 border">

                    <div class="row">
                        <div class="col">
                            <small class="text-muted">Order Info:</small>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Order Number</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.order_number }}</p>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Order Date</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.date }}</p>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col">
                            <small class="text-muted">Order Details:</small>
                        </div>
                    </div>

                    {% for item in order.lineitems.all %}
                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="small mb-0 text-black font-weight-bold">{{ item.product.name }}{% if item.product_size %} - Size {{ item.product.size|upper }}{% endif %}</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class=" small mb-0">{{ item.quantity }} @ ${{ item.product.price }} each</p>
                        </div>
                    </div>
                    {% endfor %}

                    <div class="row">
                        <div class="col">
                            <small class="text-muted">Delivering To:</small>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Full Name</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.full_name }}</p>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            {% if order.street_address2 %}
                                <p class="mb-0 text-black font-weight-bold">Street Address 1</p>
                            {% else %}
                            <p class="mb-0 text-black font-weight-bold">Street Address</p>
                            {% endif %}
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.street_address1 }}</p>
                        </div>
                    </div>

                    {% if order.street_address2 %}
                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Street Address 2</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.street_address2 }}</p>
                        </div>
                    </div>
                    {% endif %}

                    {% if order.county %}
                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">County</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.county }}</p>
                        </div>
                    </div>
                    {% endif %}

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Town or City</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.town_or_city }}</p>
                        </div>
                    </div>

                    {% if order.postcode %}
                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Postal Code</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.postcode }}</p>
                        </div>
                    </div>
                    {% endif %}

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Country</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.country }}</p>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Phone Number:</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.phone_number }}</p>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col">
                            <small class="text-muted">Billing Info:</small>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Order Total</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.order_total }}</p>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Delivery</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.delivery_cost }}</p>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 col-md-4">
                            <p class="mb-0 text-black font-weight-bold">Grand Total</p>
                        </div>
                        <div class="col-12 col-md-8 text-md-right">
                            <p class="mb-0">{{ order.grand_total }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="row">
            <div class="col-12 col-lg-7 text-right">
                <a href="{% url 'products' %}?category=new_arrivals,deals,clearance" class="btn btn-black rounded-0 my-2">
                    <span class="icon mr-2">
                        <i class="fas fa-gifts"></i>
                    </span>
                    <span class="text-uppercase">Now check out the latest deals!</span>
                </a>
            </div>
        </div>
    </div>
{% endblock%}
```

### Test

Create an order. Test the payment was successful. Test that if admin update the order details, the payment amount detail also get updated.

### Add graphic to let user know details are being sent

Although we have disallowed double clicking on the checkout page when submitting an order, the user does not know that an order is being sent to the database or that the payment intent is being created. We need to alert the customer to this, and a good way to do is is to use an overlay spinner.

1. HTML

	We simply add the overlay div to the bottom of the html code, in the body

	```
	<div id="loading-overlay">
    	<h1 class="text-light logo-font loading-spinner">
        	<span class="icon">
            	<i class="fas fa-3x fa-sync-alt fa-spin"></i>
        	</span>
    	</h1>
	</div>
	```

2. CSS

	```
	#loading-overlay {
    	display: none;
    	position: fixed;
    	top: 0;
    	left: 0;
    	width: 100%;
    	height: 100%;
    	background: rgba(23, 162, 184, .85);
    	z-index: 9999;
	}

	.loading-spinner {
    	display: flex;
    	align-items: center;
    	justify-content: center;
    	margin: 0;
    	height: 100%;
	}
	```

3. JS

	In the JavaScript, we need to add the following in 2 places in the //Handle Form Submit block of code

	```
	$('#payment-form').fadeToggle(100);
	$(‘#loading-overlay’).fadeToggle(100);
	```


	```
    // Handle form submit
	var form = document.getElementById('payment-form');

	form.addEventListener('submit', function(ev) {
    	ev.preventDefault();
    	card.update({ 'disabled': true});
    	$('#submit-button').attr('disabled', true);
    	// Here...
    	$('#payment-form').fadeToggle(100);
    	$('#loading-overlay').fadeToggle(100);	
    	stripe.confirmCardPayment(clientSecret, {
        	payment_method: {
            	card: card,
        	}
    	}).then(function(result) {
        	if (result.error) {
            	var errorDiv = document.getElementById('card-errors');
            	var html = `
                	<span class="icon" role="alert">
                	<i class="fas fa-times"></i>
                	</span>
                	<span>${result.error.message}</span>`;
            	$(errorDiv).html(html);
	        	// ... and here
            	$('#payment-form').fadeToggle(100);
            	$('#loading-overlay').fadeToggle(100);
            	card.update({ 'disabled': false});
            	$('#submit-button').attr('disabled', false);
        	} else {
            	if (result.paymentIntent.status === 'succeeded') {
                	form.submit();
            	}
        	}
    	});
	});
    ```

4. To test, use the Stripe CC number that requires extra validation.

	`4000 0025 0000 3155`

5. git add, commit, push















