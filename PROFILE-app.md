## PROFILE app

### Create the app

1. Create profile app

	`python3 manage.py startapp profiles`

2. Add profile app to list of INSTALLED_APPS in `settings.py`

#### Purpose

1. Provide a user with a place to save default delivery information.
2. Provide user with a record of their order history.

To do this we need

1. A UserProfile model which is attached to the logged-in user.
2. To attach the user's profile to all their orders.

### The UserProfile model

1. Import the [User model](https://docs.djangoproject.com/en/3.2/topics/auth/default/#user-objects) into profiles/models.py from django.contrib.auth.models

	`from django.contrib.auth.models import User`

2. Import CountryField

	`from django_countries.fields import CountryField`

3. Create the model

	```
	from django.db import models
	from django.contrib.auth.models import User

	from django_countries.fields import CountryField


	# Create your models here.
	class UserProfile(models.Model):
        """
        A user profile model for maintaining default
        delivery information and order history
        """
        user = models.OneToOneField(User, on_delete=models.CASCADE)
        default_phone_number = models.CharField(max_length=20, null=True, blank=True)
        default_street_address1 = models.CharField(max_length=80, null=True, blank=True)
        default_street_address2 = models.CharField(max_length=80, null=True, blank=True)
        default_town_or_city = models.CharField(max_length=40, null=True, blank=True)
        default_county = models.CharField(max_length=80, null=True, blank=True)
        default_postcode = models.CharField(max_length=20, null=True, blank=True)
        default_country = CountryField(blank_label='Country', null=True, blank=True)

        def __str__(self):
            return self.user.username

	```

	1. OneToOneField =  each user can only have one profile and each profile can only be attached to one user

	2. The other fields come from the Order model, but have been renamed `default_..." to reflect they are the default

	3. Since these fields are optional in the profile, they are all `null=True, blank=True`

	4. __str__ method returns user's username

#### Create or update the user profile

The model for the UserProfile has been created. Now, write the function that CREATES or SAVES the user's UserProfile

This function is outside of the model, but in the same file.

1. We are receiving the user from the Usermodel, after the user has been saved. Thus,the post_save and receiver signals both need to be imported

	```
	from django.db.models.signals import post_save
	from django.dispatch import receiver
	```
2. The function

	```
	@receiver(post_save, sender=User)
	def create_or_update_user_profile(sender, instance, created, **kwargs):
        """
        Create or update the user profile
        """
        if created:
            UserProfile.objects.create(user=instance)
        # Existing users: just save the profile
        instance.userprofile.save()
	```

#### Attach the UserProfile to the Order model

1. In checkout/models.py, import UserProfile from profiles.models

	`from profiles.models import UserProfile`

2. Add the UserProfile to the Order model

	```
	class Order(models.Model):
        order_number = models.CharField(max_length=32, null=False, editable=False)
        user_profile = models.ForeignKey(UserProfile, on_delete=models.SET_NULL,
                                         null=True, blank=True,
                                         related_name='orders')
        full_name = ...
    ```

	1. ForeignKey to UserProfile
	2. SET_NULL on delete allows admin to keep record of Order if User is deleted
	3. related_name allows access to user's orders by calling something like `user.user_profile.orders`

#### Migrate

```
python3 manage.py makemigrations --dry-run
python3 manage.py makemigrations
python3 manage.py migrate --plan
python3 manage.py migrate
```

### Set up the basic URLs views and templates for the profiles app

#### profiles/views.py

```
# Create your views here.
def profile(request):
    """ Display the user's profile """
    
    template = 'profiles/profile.html'
    context = {}

    return render(request, template, context)
```

#### urls.py

Create the profiles.urls.py file and add the path

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.profile, name='profile'),
]
```

#### Add the url paths to the project level urls.py file's urlpatterns

`path('profile/', include('profiles.urls')),`

#### profiles.html template

Write/copy/adjust as necessary

#### link profiles.html to its css file

```
{% block extra_css %}
    <link rel="stylesheet" href="{% static 'profiles/css/profile.css' %}">
{% endblock %}
```

#### css file

`profiles/static/profiles/css/profile.css`

#### Test

1. Test page renders

	`python3 manage.py runserver`

2. git add, commit, push

### Formatting the allauth templates

<a href="https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/assets/pdf/allauth-rendering.pdf">allauth rendering solution pdf</a>

#### New users and registered users side note

With the code as is, I can peruse the site. But if I log out, I cannnot then log in again, as although I am a user, there is no profile attached to my user account.

We can override this and create a profile by commenting out some of the code of the create_or_update_user_profile function:

```
@receiver(post_save, sender=User)
def create_or_update_user_profile(sender, instance, created, **kwargs):
    """
    Create or update the user profile
    """
    # if created:
    UserProfile.objects.create(user=instance)
    # Existing users: just save the profile
    # instance.userprofile.save()
```

Now we can go back and create a profile, and then uncomment the above code.

Since emails are being logged in the terminal, we can also copy the email link and verify the profile.

#### Verify everything works

1. Update the base.html profile link

	`<a href="{% url 'profile' %}" class="dropdown-item">My Profile</a>`

2. Import the UserProfile model to profiles/views.py

	`from .models import UserProfile`

3. Import get_object_or_404

	`from django.shortcuts import get_object_or_404`

4. Get the user's profile and return it to the template via the context

	```
	def profile(request):
        """ Display the user's profile """
        profile = get_object_or_404(UserProfile, user=request.user)

        template = 'profiles/profile.html'
        context = {
            'profile': profile,
        }

        return render(request, template, context)
	```

5. Render `{{ profile }}` in the template

The user name should be rendered to the live page.

6. git add, commit, push

### The UserProfileForm

The UserProfile needs to create the profile, thus a form is needed. It will be modelled more or less on the OrderForm as the profile model is so similar to the order model.

##### profiles/forms.py

```
from django import forms
# Import the model
from .models import UserProfile


class UserProfileForm(forms.ModelForm):
    class Meta:
        # Use UserProfile model
        model = UserProfile
        # Use all fields from the model except 'user', as user should not change
        exclude = ('user',)

    # Override default __init__ method in order to customize form
    def __init__(self, *args, **kwargs):
        """
        Add placeholders and classes, remove auto-generated
        labels and set autofocus on first field
        """
        super().__init__(*args, **kwargs)
        placeholders = {
            'default_phone_number': 'Phone Number',
            'default_postcode': 'Postal Code',
            'default_town_or_city': 'Town or City',
            'default_street_address1': 'Street Address 1',
            'default_street_address2': 'Street Address 2',
            'default_county': 'County, State or Locality',
        }

        # Set mouse focus to Phone Number field
        self.fields['default_phone_number'].widget.attrs['autofocus'] = True
        for field in self.fields:
            # Attach * to all required fields except the Country field
            if field != 'default_country':
                if self.fields[field].required:
                    placeholder = f'{placeholders[field]} *'
                else:
                    placeholder = placeholders[field]
                self.fields[field].widget.attrs['placeholder'] = placeholder
            # Attach classes to fields
            self.fields[field].widget.attrs['class'] = 'border-black rounded-0 profile-form-input'
            # Remove form's default field labels as placeholders have been set above
            self.fields[field].label = False
```

#### profiles/views.py
 
```
# Import the UserProfile user object or 404
from django.shortcuts import render, get_object_or_404

# Import UserProfile model
from .models import UserProfile
# Import form
from .forms import UserProfileForm


# Create your views here.
def profile(request):
    """ Display the user's profile """
    # request the user from UserProfile and assign it as profile
    profile = get_object_or_404(UserProfile, user=request.user)

    # Populate the form with the user's profile info
    form = UserProfileForm(instance=profile)

    template = 'profiles/profile.html'
    context = {
        # Return form to the template
        'form': form,
    }

    return render(request, template, context)
```

#### profiles/templates/profile.html

```
<!-- Set the PoST method action -->
<form class="mt-3" action="{% url 'profile' %}" method="POST" id="profile-update-form">
    <!-- Add csrf_token and the form -->
    {% csrf_token %}
    {{ form|crispy }}
    <button class="btn btn-black rounded-0 text-uppercase float-right">Update Information</button>
</form>
```

#### JavaScript for the dropdown box text color

##### In profiles/static/profiles/js/countryfield.js

```
let countrySelected = $('#id_default_country').val();
if(!countrySelected) {
    $('#id_default_country').css('color', '#aab7c4');
};
$('#id_default_country').change(function() {
    countrySelected = $(this).val();
    if(!countrySelected) {
        $(this).css('color', '#aab7c4');
    } else {
        $(this).css('color', '#000');
    }
});
```

##### In profile.html

```
{% block postloadjs %}
    {{ block.super }}
    <script type="text/javascript" src="{% static 'profiles/js/countryfield.js' %}"></script>
{% endblock %}
```

#### POST

Since we are posting the user's profile to the db, we need to save it, and also let the user know that the profile has been saved or updated.

##### profiles/views.py

```
from django.shortcuts import render, get_object_or_404
# Import messages
from django.contrib import messages

from .models import UserProfile
from .forms import UserProfileForm


# Create your views here.
def profile(request):
    """ Display the user's profile """
    profile = get_object_or_404(UserProfile, user=request.user)

    # If POST
    if request.method == 'POST':
        # Create a new instance of the UserProfileForm using the 
        # post data and tell it the instance we're updating is the profile we've 
        # just retrieved above.
        form = UserProfileForm(request.POST, instance=profile)
        if form.is_valid():
            form.save()
            messages.success(request, 'Profile updated successfully')

    # Populate the form with the user's profile info
    else:
        form = UserProfileForm(instance=profile)

    template = 'profiles/profile.html'
    context = {
        # Return form to the template
        'form': form,
    }

    return render(request, template, context)
```

Now, when a user saves the profile, he gets the popup window saying the profile has been saved. However, it also displays the bag. 

#### Remove the bag from the popup message when updating profile
##### profiles/views.py

```
def profile(request):
    """ Display the user's profile """
    profile = get_object_or_404(UserProfile, user=request.user)

    if request.method == 'POST':
        form = UserProfileForm(request.POST, instance=profile)
        if form.is_valid():
            form.save()
            messages.success(request, 'Profile updated successfully')

    # Populate the form with the user's profile info
    else:
        form = UserProfileForm(instance=profile)
    orders = profile.orders.all()

    template = 'profiles/profile.html'
    context = {
        'form': form,
        'orders': orders,
        # Add this
        'on_profile_page': True,
    }

    return render(request, template, context)
```
##### templates/includes/toasts/toast_success.html

Currently, the html reads "display the bag `{{ if grand_total }}`

We can amend this to `{{ if grand_total and not on_profile_page }}`


### git add, commit, push

## Order History

Display the user's order history on the user's profile page.

### HTML
#### profiles/templates/profile.html

Outside of the form

```
<div class="col-12 col-lg-6">
    <p class="text-muted">Order History</p>
    <div class="order-history table-responsive">
        <table class="table table-sm table-borderless">
            <thead>
                <tr>
                    <th>Order Number</th>
                    <th>Date</th>
                    <th>Items</th>
                    <th>Order Total</th>
                </tr>
            </thead>
            <tbody>
                {% for order in orders %}
                    <tr>
                        <td>
                            <!-- Link to specific order in order history, see below -->
                            <a href="{% url 'order_history' order.order_number %}" title="{{ order.order_number }}">
                                {{ order.order_number|truncatechars:6}}
                            </a>
                        </td>
                        <td>{{ order.date }}</td>
                        <td>
                            <ul class="list-unstyled">
                                {% for item in order.lineitems.all %}
                                    <li class="small">
                                        {% if item.product.has_sizes %}
                                            Size {{ item.product.size|upper }}
                                        {% endif %}
                                        {{ item.product.name }} x{{ item.quantity }}
                                    </li>
                                {% endfor%}
                            </ul>
                        </td>
                        <td>${{ order.grand_total }}</td>
                    </tr>
                {% endfor %}
            </tbody>
        </table>
    </div>
</div>
```

### CSS

It would be more appealing to make the Order History column the same height as the User's Profile form

#### profiles/static/profiles/css/profile.css

```
.order-history {
    max-height: 416px; /* height of profile form and submit button */
    overflow-y: auto;
}
```

### Link to specific order in order history

#### profiles/views.py

```
from django.shortcuts import render, get_object_or_404
from django.contrib import messages

from .models import UserProfile
from .forms import UserProfileForm

# Import the Order model from checkout app
from checkout.models import Order


def profile(request):
    """ Display the user's profile """
    profile = get_object_or_404(UserProfile, user=request.user)

    if request.method == 'POST':
        form = UserProfileForm(request.POST, instance=profile)
        if form.is_valid():
            form.save()
            messages.success(request, 'Profile updated successfully')

    form = UserProfileForm(instance=profile)
    orders = profile.orders.all()

    template = 'profiles/profile.html'
    context = {
        'form': form,
        'orders': orders,
        'on_profile_page': True,
    }

    return render(request, template, context)


# order_history takes the order_number as parameter
def order_history(request, order_number):
    # Get specific order
    order = get_object_or_404(Order, order_number=order_number)

    # Alert user that this is a past order
    messages.info(request, (
        f'This is a past confirmation for order number {order_number}.'
        'A confirmation email was sent on the order date.'
    ))
    # Use checkout/checkout_success.html template
    template = 'checkout/checkout_success.html'
    context = {
        'order': order,
        # from_profile: True tells the context that we are coming from the 
        # profile view and not the checkout view. See below
        'from_profile': True,
    }

    return render(request, template, context)
```

#### Add the path to profiles/urls.py

`path('order_history/<order_number>', views.order_history, name='order_history'),`

#### checkout/checkout_success.html

Thanks to the `from_profile` in the context above, we can display a different button to the page rather than the "Latest Deals" button

```
<div class="row">
    <div class="col-12 col-lg-7 text-right">
        {% if from_profile %}
            <a href="{% url 'profile' %}" class="btn btn-black rounded-0 my-2">
                <span class="icon mr-2">
                    <i class="fas fa-angle-left"></i>
                </span>
                <span class="text-uppercase">Back to Profile</span>
            </a>
        {% else %}
            <a href="{% url 'products' %}?category=new_arrivals,deals,clearance" class="btn btn-black rounded-0 my-2">
                <span class="icon mr-2">
                    <i class="fas fa-gifts"></i>
                </span>
                <span class="text-uppercase">Now check out the latest deals!</span>
            </a>
        {% endif %}
    </div>
</div>
```

Now, when we get to checkout_succes.html:
1. If we come from checkout.html, "Latest deal" button
2. If we come from profile.html, "Back to Profile" button

### Assign order to a specific user profile

#### checkout/admin.py

Add `'user_profile'` to the fields dictionary of OrderAdmin class

#### Associate order with the user's profile when it's created during the checkout process

#### checkout/views.py/checkout_success()

```
from django.shortcuts import render, redirect, reverse, get_object_or_404, HttpResponse
from django.views.decorators.http import require_POST
from django.contrib import messages
from django.conf import settings

from .forms import OrderForm
from .models import Order, OrderLineItem
from products.models import Product
# Import the UserProfileForm
from profiles.forms import UserProfileForm
# Import the UserProfile model
from profiles.models import UserProfile
from bag.contexts import bag_contents

import stripe
import json


# Create your views here.
@require_POST
def cache_checkout_data(request):
    #cache_checkout_data view
        return HttpResponse(content=e, status=400)


def checkout(request):
    # Checkout view
    return render(request, template, context)


def checkout_success(request, order_number):
    """
    Handle successful checkouts
    """
    """ Previous code """
    save_info = request.session.get('save_info')
    order = get_object_or_404(Order, order_number=order_number)

    """ New code """
    # Check if the user is authenticated because if so they'll have a 
    # profile that was created when they created their account
    if request.user.is_authenticated:
        # Get the user's profile
        profile = UserProfile.objects.get(user=request.user)
        # Attach the user's profile to the order
        order.user_profile = profile
        order.save()

    # If save_info was checked
    if save_info:
        # Dictionary's keys match the fields on the UserProfile model, while
        # Dictionary's keys' values come from the Order model
        profile_data = {
            'default_phone_number': order.phone_number,
            'default_country': order.country,
            'default_postcode': order.postcode,
            'default_town_or_city': order.town_or_city,
            'default_street_address1': order.street_address1,
            'default_street_address2': order.street_address2,
            'default_county': order.county,
        }
        # Ceate instance of the user profile form, using the profile data
        user_profile_form = UserProfileForm(profile_data, instance=profile)
        if user_profile_form.is_valid():
            user_profile_form.save()

    """ Previous code """
    messages.success(request, f'Order succesfully processed! \
        Your order number is {order_number}. A confirmation \
        email will be sent to {order.email}.')

    if 'bag' in request.session:
        del request.session['bag']

    template = 'checkout/checkout_success.html'
    context = {
        'order': order,
    }

    return render(request, template, context)

```

Test by making sure user profile is not completed, place an order and checking the save info checkbox.

### Prefill the checkout form

Since users now have profiles we can use their default delivery information to pre-fill the form on the checkout page just before we create the order form in the checkout view. We use the `initial()` mthod for this
 
#### checkout/views.py/checkout()

```
def checkout(request):
    stripe_public_key = settings.STRIPE_PUBLIC_KEY
    stripe_secret_key = settings.STRIPE_SECRET_KEY

    if request.method == 'POST':
        bag = request.session.get('bag', {})

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
        order_form = OrderForm(form_data)
        if order_form.is_valid():
            order = order_form.save(commit=False)
            # Get client secret
            pid = request.POST.get('client_secret').split('_secret')[0]
            order.stripe_pid = pid
            order.original_bag = json.dumps(bag)
            order.save()
            for item_id, item_data in bag.items():
                try:
                    product = Product.objects.get(id=item_id)
                    if isinstance(item_data, int):
                        order_line_item = OrderLineItem(
                            order=order,
                            product=product,
                            quantity=item_data,
                        )
                        order_line_item.save()
                    else:
                        for size, quantity in item_data['items_by_size'].items():
                            order_line_item = OrderLineItem(
                                order=order,
                                product=product,
                                quantity=quantity,
                                product_size=size
                            )
                            order_line_item.save()
                except Product.DoesNotExist:
                    messages.error(request, (
                        "One of the products in your bag was not found in our database."
                        "Please call us for assistance!"
                    ))
                    order.delete()
                    return redirect(reverse('view_bag'))
            request.session['save_info'] = 'save-info' in request.POST
            return redirect(reverse('checkout_success', args=[order.order_number]))
        else:
            messages.error(request, 'There was an error in your form. \
                Please double check your information.')

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

        """ New code """
        # Attempt to prefill the form with any info the user maintains in their profile
        if request.user.is_authenticated:
            try:
                profile = UserProfile.objects.get(user=request.user)
                order_form = OrderForm(initial={
                    'full_name': profile.user.get_full_name(),
                    'email': profile.user.email,
                    'phone_number': profile.default_phone_number,
                    'country': profile.default_country,
                    'postcode': profile.default_postcode,
                    'town_or_city': profile.default_town_or_city,
                    'street_address1': profile.default_street_address1,
                    'street_address2': profile.default_street_address2,
                    'county': profile.default_county,
                })
            except UserProfile.DoesNotExist:
                order_form = OrderForm()
        else:
            order_form = OrderForm()
    
    """ Previous code """
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

### Adding the profile to the webhook

If the checkout view fails we can depend on the webhook_handler to add the user profile.

#### checkout/webhook_handler.py

```
# Import UserProfile model
from profiles.models import UserProfile

def handle_payment_intent_succeeded(self, event):
    # Handle the payment_intent.succeeded webhook even
    intent = event.data.object
    pid = intent.id
    bag = intent.metadata.bag
    save_info = intent.metadata.save_info

    billing_details = intent.charges.data[0].billing_details
    shipping_details = intent.shipping
    grand_total = round(intent.charges.data[0].amount/100, 2)

    # Clean data in the shipping fields
    for field, value in shipping_details.address.items():
        if value == "":
            shipping_details.address[field] = None

    # New code
    # Update profile information if save_info was checked
    profile = None
    username = intent.metadata.username
    if username != 'AnonymousUser':
        profile = UserProfile.objects.get(user__username=username)
        if save_info:
            profile.default_phone_number__iexact = shipping_details.phone
            profile.default_country__iexact = shipping_details.address.country
            profile.default_postcode__iexact = shipping_details.address.postal_code
            profile.default_town_or_city__iexact = shipping_details.address.city
            profile.default_street_address1__iexact = shipping_details.address.line1
            profile.default_street_address2__iexact = shipping_details.address.line2
            profile.default_county__iexact = shipping_details.address.state
            profile.save()

    # Previous code
    order_exists = False
    attempt = 1
    while attempt <= 5:
        try:
            order = Order.objects.get(
                full_name__iexact=shipping_details.name,
                email__iexact=billing_details.email,
                phone_number__iexact=shipping_details.phone,
                country__iexact=shipping_details.address.country,
                postcode__iexact=shipping_details.address.postal_code,
                town_or_city__iexact=shipping_details.address.city,
                street_address1__iexact=shipping_details.address.line1,
                street_address2__iexact=shipping_details.address.line2,
                county__iexact=shipping_details.address.state,
                grand_total=grand_total,
                original_bag=bag,
                stripe_pid=pid
            )
            order_exists = True
            break
        except Order.DoesNotExist:
            attempt += 1
            time.sleep(1)
    if order_exists:
        self._send_confirmation_email(order)
        return HttpResponse(
            content=f'Webhook received: {event["type"]} | SUCCESS: Verified order already in database',
            status=200)
    else:
        order = None    
        try:
            order = Order.objects.create(
                    full_name=shipping_details.name,
                    # Add user_profile to Order object
                    user_profile=profile,
                    email=billing_details.email,
                    phone_number=shipping_details.phone,
                    country=shipping_details.address.country,
                    postcode=shipping_details.address.postal_code,
                    town_or_city=shipping_details.address.city,
                    street_address1=shipping_details.address.line1,
                    street_address2=shipping_details.address.line2,
                    county=shipping_details.address.state,
                    original_bag=bag,
                    stripe_pid=pid
                )
            for item_id, item_data in json.loads(bag).items():
                product = Product.objects.get(id=item_id)
                if isinstance(item_data, int):
                    order_line_item = OrderLineItem(
                        order=order,
                        product=product,
                        quantity=item_data,
                    )
                    order_line_item.save()
                else:
                    for size, quantity in item_data['items_by_size'].items():
                        order_line_item = OrderLineItem(
                            order=order,
                            product=product,
                            quantity=quantity,
                            product_size=size
                        )
                        order_line_item.save()
        except Exception as e:
            if order:
                order.delete()
                return HttpResponse(
                    content=f'Webhook received: {event["type"]} | ERROR: {e}',
                    status=500)
    self._send_confirmation_email(order)
    return HttpResponse(
        # Response to Stripe
        content=f'Webhook received: {event["type"]} | SUCCESS: Created order in webhook',
        status=200)
```

Test it is working by breaking the JavaScript once again, so that the form is never submitted. CHeck the webhook response from Stripe, and the admin for the order and the profile.

If all is working, enable the form.sumbit in the JavaScript again

### Confirmation emails

The best place to do this from is the webhook_handler, since at that point we know the payment has definitely been made since the only thing that can trigger it is a webhook from stripe.

#### The emails

The mails will be in the form of two .txt files

1. checkout/templates/checkout/confirmtaion_emails/confirmation_email_body.txt

	
	```
	Hello {{ order.full_name}}!

	This is a confirmation of your order at Boutique Ado. Your order information is below:

	Order Number: {{ order.order_number }}
	Order Date: {{ order.date }}

	Order Total: ${{ order.order_total }}
	Delivery: ${{ order.delivery_cost }}
	Grand Total: ${{ order.grand_total }}

	Your order will be shipped to {{ order.street_address1 }} in {{ order.town_or_city }}, {{ order.country }}.

	We've got your phone number on file as {{ order.phone_number }}.

	If you have any questions, feel free to contact us at {{ contact_email }}.

	Thank you for your order!

	Sincerely,

	Boutique Ado
	```
		
2. checkout/templates/checkout/confirmtaion_emails/confirmation_email_subject.txt

	```
	Boutique Ado Confirmation for Order Number {{ order.order_number }}
	```

#### The webhook_handler

##### checkout/webhook_handler.py

```
from django.http import HttpResponse
# Import send_mail
from django.core.mail import send_mail
# Import render_to_string
from django.template.loader import render_to_string
from django.conf import settings

from .models import Order, OrderLineItem
from products.models import Product
from profiles.models import UserProfile

import json
import time


class StripeWH_Handler:
    """ Handle Stripe webhooks """

    def __init__(self, request):
        self.request = request

    # Private method as only being used within this class
    # Takes order as parameter
    def _send_confirmation_email(self, order):
        """ Send the user a confirmation email """
        # Get email from the order
        cust_email = order.email
        subject = render_to_string(
            'checkout/confirmation_emails/confirmation_email_subject.txt',
            {'order': order})
        body = render_to_string(
            'checkout/confirmation_emails/confirmation_email_body.txt',
            {'order': order, 'contact_email': settings.DEFAULT_FROM_EMAIL})
        
        send_mail(
            subject,
            body,
            settings.DEFAULT_FROM_EMAIL,
            [cust_email],
        )
```

##### checkout/webhook_handler/handle_payment_intent_succeeded()

Call _send_confirmation_email at the relevant places in the function (ie. just before returning the response to stripe)

`self._send_confirmation_email(order)`

##### settings.py

Add DEFAULT_FROM_EMAIL variable

`DEFAULT_FROM_EMAIL = 'boutiqueado@example.com'`

##### Test

Although an email will not currently be sent, we can test by making a purchase. The email will be rendered in the terminal

##### git add, commit, push


