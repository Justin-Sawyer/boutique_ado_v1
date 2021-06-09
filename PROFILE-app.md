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

<a href="">allauth rendering solution pdf</a>
