## PRODUCT ADMIN

### The Product Form

#### products/forms.py

```
# Import forms from django
from django import forms
# Import the Product and Category models from product.models.py
from .models import Product, Category


# The ProductFom extends the built in ModelForm
class ProductForm(forms.ModelForm):

    class Meta():
        model = Product
        # Include all fields from the Product model
        fields = '__all__'

    def __init__(self, *args, **kwargs):
        # Override the standard __init__ method
        super().__init__(*args, **kwargs)
        # Get all the categories from the Category model
        categories = Category.objects.all()
        # Create a list of tuples of the friendly names associated with their category ids
        friendly_name = [(c.id, c.get_friendly_name()) for c in categories]

        # Update the category field on the form to use friendly names for choices instead of using the id
        self.fields['category'].choices = friendly_name
        # Set the styling to match the rest of the store
        for field_name, field in self.fields.items():
            field.widget.attrs['class'] = 'border-black rounded-0'
```

### Adding products

##### products/view.py

```
# Import the ProductForm
from .forms import ProductForm



def add_product(request):
    """ Add a product to the store """

    # Render an empty instance of the form
    form = ProductForm
    template = 'products/add_product.html'
    context = {
        'form': form
    }

    return render(request, template, context)
```

##### products/urls.py

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.all_products, name='products'),
    # Set product_id view as integer
    path('<int:product_id>/', views.product_detail, name='product_detail'),
    # Add path for add_product view
    path('add/', views.add_product, name='add_product'),
]
```

##### products/templates/products/add_product.html

Add necessary HTML.

The form element:

```
<div class="row">
    <div class="col-12 col-md-6">
        <!-- enctype is needed as we are potentially adding image files -->
        <form method="POST" action="{% url 'add_product' %}" class="form mb-2" enctype="multipart/form-data">
            {% csrf_token %}
            {{ form | crispy }}
            <div class="text-right">
                <a class="btn btn-outline-black rounded-0" href="{% url 'products' %}">Cancel</a>
                <button class="btn btn-black rounded-0" type="submit">Add Product</button>
            </div>
        </form>
    </div>
</div>
```

#### The POST handler for the form

##### products/views.py

```
def add_product(request):
    """ Add a product to the store """
    if request.method == "POST":
        # request POST and FILES (for the image)
        form = ProductForm(request.POST, request.FILES)
        if form.is_valid():
            # Save the form as product
            product = form.save()
            messages.success(request, 'Successfully added product!')
            # Go to saved product's details page
            return redirect(reverse('product_detail', args=[product.id])))
        else:
            messages.error(request, 'Failed to add product. Please ensure the form is valid.')
    else:
        form = ProductForm

    template = 'products/add_product.html'
    context = {
        'form': form
    }

    return render(request, template, context)
```

#### Associate the image file with the image attribute in the HTML

##### templates/includes/toasts/toast_sucess.html
##### bag/templates/bag/bag.html

```
{% if item.product.image %}
    <img class="w-100" src="{{ item.product.image.url }}" alt="{{ item.product.name }}">
{% else %}
    <img class="w-100" src="{{ MEDIA_URL }}noimage.png" alt="{{ item.product.name }}">
{% endif %}
```

#### Add link to Product Management page

##### templates/base.html
##### templates/includes/mobile-top-header.html

`<a href="{% url 'add_product' %}" class="dropdown-item">Product Management</a>`

#### Test

Test by adding product without image, and product with image.

### Updating Products

##### products/templates/products/edit_product.html

Copy the add_products.html file and update the headers etc as necessary.

Send the form to a new URL called `edit_product` and include the product.id with it

`<form method="POST" action="{% url 'edit_product' product.id %}" class="form mb-2" enctype="multipart/form-data">`

#### The view
##### products/views.py

```
def edit_product(request, product_id):
    """ Edit a product in the store """
    product = get_object_or_404(Product, pk=product_id)
    form = ProductForm(instance=product)
    messages.info(request, f'You are editing {product.name}')

    template = 'products/edit_product.html'
    context = {
        'form': form,
        'product': product,
    }

    return render(request, template, context)
```
##### products.urls.py

`path('edit/<int:product_id>/', views.edit_product, name='edit_product'),`

#### The POST handler for the form
##### products/views.py

```
def edit_product(request, product_id):
    """ Edit a product in the store """
    product = get_object_or_404(Product, pk=product_id)
    
    # POST
    if request.method == 'POST':
        form = ProductForm(request.POST, request.FILES, instance=product)
        if form.is_valid():
            form.save()
            messages.success(request, 'Successfully updated product!')
            return redirect(reverse('product_detail', args=[product.id]))
        else:
            messages.error(request, 'Failed to update product. Please ensure the form is valid.')
    else:
        form = ProductForm(instance=product)
        messages.info(request, f'You are editing {product.name}')

    template = 'products/edit_product.html'
    context = {
        'form': form,
        'product': product,
    }

    return render(request, template, context)
```

#### Test

Test by going to a product by manually typing its address in the address bar. Edit the product by adding an invalid price. Then a valid price. Revert to original afterwards!

### Deleting Products

No template required.

##### products/urls.py

`path('delete/<int:product_id>/', views.delete_product, name='delete_product'),`

##### products/views.py

```
def delete_product(request, product_id):
    """ Delete a product from the store """
    product = get_object_or_404(Product, pk=product_id)
    product.delete()
    messages.success(request, 'Product deleted!')
    return redirect(reverse('products'))
```
#### Test

Test by adding a product, then going to that product's product_details page, and insert `/delete/<product_id>` in the adress string.

### Edit & Delete links

Where needed in the HTML

```
{% if request.user.is_superuser %}
    <small class="ml-3">
        <a href=“{% url 'edit_product' product.id %}”>Edit</a> |
        <a class="text-danger" href=“{% url 'delete_product' product.id %}”>Delete</a>
    </small>
{% endif %}
```

### Securing Products

Ideally, only superusers should be able to add, update and delete products from the store.

This can be achieved by importing login_required to products/views.py from django.contrib.auth.decorators.

`from django.contrib.auth.decorators import login_required`

The decorator needs to be added to the relevant views (add_product(), edit_product() and delete_product())

`@login_required`

Also, a message telling the user that adding, editing and deleting products can only be done by superusers can be added to each view

```
@login_required
def add_product(request):
    """ Add a product to the store """
    if not request.user.is_superuser:
        messages.error(request, 'Sorry, only store owners can do that')
        return redirect(reverse('home'))

    if request.method == "POST":
        (...)
```

*This is also a good moment to import login_required to the profile view (profiles/views.py) and add the same decorator to the profile() view*

##### git add, commit, and push

### The image field

Currently, the add and edit html pages render the bog standard file chooser elements, which do not fit in with the UI.

This file chooser can be styled, with a little effort.

Django works with widgets, and the file chooser widget is called the [ClearableFileInput widget](https://github.com/django/django/blob/main/django/forms/templates/django/forms/widgets/clearable_file_input.html)

The ClearableFileInput is an Object, a class, and as such it can be edited. Here is the original taken from the [django docs](https://github.com/django/django/blob/main/django/forms/widgets.py-

```
class ClearableFileInput(FileInput):

    clear_checkbox_label = _('Clear')
    initial_text = _('Currently')
    input_text = _('Change')
    template_name = 'django/forms/widgets/clearable_file_input.html'
```

By creating a custom class, we can thus edit the values each variable is assigned.

##### Create products/widgets.py file

```
# Import the original ClearableFileInput object so we can edit it
from django.forms.widgets import ClearableFileInput
# Not actually necessary, but import in order to keep custom class as close as possible to the original object
from django.utils.translation import gettext_lazy as _


# Custom class inherits from original
class CustomClearableFileInput(ClearableFileInput):
    # Changed values
    clear_checkbox_label = _('Remove')
    initial_text = _('Current Image')
    input_text = _('')
    template_name = 'products/custom_widget_templates/custom_clearable_file_input.html'
```

##### Create products/custom_widget_templates/custom_clearable_file_input.html

Copy the original widget from the [widget link](https://github.com/django/django/blob/main/django/forms/templates/django/forms/widgets/clearable_file_input.html)

```
{% if widget.is_initial %}
    {{ widget.initial_text }}: 
    <a href="{{ widget.value.url }}">{{ widget.value }}</a>
    {% if not widget.required %}
        <input type="checkbox" name="{{ widget.checkbox_name }}" id="{{ widget.checkbox_id }}"{% if widget.attrs.disabled %} disabled{% endif %}>
        <label for="{{ widget.checkbox_id }}">{{ widget.clear_checkbox_label }}</label>
    {% endif %}
    <br>
    {{ widget.input_text }}:
{% endif %}
<input type="{{ widget.type }}" name="{{ widget.name }}"{% include "django/forms/widgets/attrs.html" %}>
```

Edit the widget

```
{% if widget.is_initial %}
    <!-- Wrap the initial text in a paragraph -->
    <p>{{ widget.initial_text }}:</p>
    <!-- Replace filename value with small preview of the image -->
    <a href="{{ widget.value.url }}">
        <img width="96" height="96" class="rounded shadow-sm" src="{{ widget.value.url }}">
    </a>
    <!-- Wrap checkbox in a div, add bootstrap classes,a little top margin and
    give the input and the label the required bootstrap classes to style them -->
    {% if not widget.required %}
        <div class="custom-control custom-checkbox mt-2">
            <input class="custom-control-input" type="checkbox" name="{{ widget.checkbox_name }}" id="{{ widget.checkbox_id }}">
            <label class="custom-control-label text-danger" for="{{ widget.checkbox_id }}">{{ widget.clear_checkbox_label }}</label>
        </div>
    {% endif %}
    <br>
    <!-- Remove the colon from the widget.input_text -->
    {{ widget.input_text }}
{% endif %}
<!-- Wrap the actual file input itself in a span which looks like a button and 
add a paragraph below it to hold the file name once one is selected -->
<span class="btn btn-black rounded-0 btn-file">
    Select Image <input id="new-image" type="{{ widget.type }}" name="{{ widget.name }}"{% include "django/forms/widgets/attrs.html" %}>
</span>
<strong><p class="text-danger" id="filename"></p></strong>
```

#### Import the CustomClearableFileInput into the ProductForm

##### products/forms.py

```
from django import forms
# Import the custom widget
from .widgets import CustomClearableFileInput
from .models import Product, Category


class ProductForm(forms.ModelForm):

    class Meta():
        model = Product
        fields = '__all__'

    # Replace image field with the custom widget
    image = forms.ImageField(label='Image', required=False, widget=CustomClearableFileInput)

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        categories = Category.objects.all()
        friendly_name = [(c.id, c.get_friendly_name()) for c in categories]

        self.fields['category'].choices = friendly_name
        for field_name, field in self.fields.items():
            field.widget.attrs['class'] = 'border-black rounded-0'
```

#### Styling the widget

Since the widget is being used in both the add and edit product html pages, add the CSS to the base css file

##### static/css/base.css

```
/* Product form */
.btn-file {
    position: relative;
    overflow: hidden;
}

.btn-file input[type='file'] {
    position: absolute;
    top: 0;
    right: 0;
    min-width: 100%;
    min-height: 100%;
    opacity: 0;
    cursor: pointer;
}

.custom-checkbox .custom-control-label::before {
    border-radius: 0;
    border-color: #dc3545;
}

.custom-checkbox .custom-control-input:checked~.custom-control-label::before {
    background-color: #dc3545;
    border-color: #dc3545;
    border-radius: 0;
}
```

#### The HTML

On rendering the page, the crispy forms "Image" label is being rendered. Remove this with a for loop and if statement, on all pages concerned (add_product and edit_product)

```
{% csrf_token %}
{% for field in form %}
    {% if field.name != 'image' %}
        {{ field | as_crispy_field }}
    {% else %}
        {{ field }}
    {% endif %}
{% endfor %}
```

#### JavaScript

Notify the superuser of the change to the image in realtime on all pages concerned

```
{% block postloadjs %}
    {{ block.super }}
    <script type="text/javascript">
        // Listen for the change event on the new-image input from the widget
        $('#new-image').change(function() {
            var file = $('#new-image')[0].files[0];
            $('#filename').text(`Image will be set to: ${file.name}`);
        });
    </script>
{% endblock %}
```

### Test

The site should now be completely functional. Test everything, then

git add, commit, push
