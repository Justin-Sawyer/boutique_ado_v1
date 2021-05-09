# Boutique Ado e-commerce store

# Built using Django

- [SET UP](#set-up)
- [USER STORIES](#user-stories)
- [ALLAUTH](#allauth)
  * [ADDING ALLAUTH](#adding-allauth)
  * [TESTING ALLAUTH](#testing-allauth)
- [TEMPLATES](#templates)
  * [HOW TO ADD THE BASE TEMPLATE](#how-to-add-the-base-template)
  * [TEMPLATE TAGS AND FILTERS](#template-tags-and-filters)
- [MANAGING STATIC FILES](#managing-static-files)
  * [CSS](#css)
  * [MEDIA](#media)
  * [IMPORTING](#importing)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

# SET UP

  1. INSTALL DJANGO:

		`pip3 install django`

  2. CREATE PROJECT ITSELF:

		`django-admin startproject boutique_ado .`

  3. CREATE .GITIGNORE FILE:

		`touch .gitignore`

4. ADD FILES TO .GITIGNORE:

	```
	*.sqlite3
	*.pyc
	__pycache__/
	```

5. RUN SERVER:

	`python3 manage.py runserver`

6. MIGRATE FILES:

	`python3 manage.py migrate`

7. CREATE SUPERUSER:

	`python3 manage.py createsuperuser`

8. COMMIT TO GIT:

	```
	git add . 
	git commit - m 'initial commit'
	git push
	```
# USER STORIES

| ID  | As A/An    | I want to...                                                      | So I can...                                                                           |
| --- | ---------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
|     |            | **_Viewing and Navigation_**                                      |                                                                                       |
| 1   | Shopper    | View list of products                                             | Find something to purchase                                                            |
| 2   | Shopper    | View details of product                                           | See Price, Description, Image, and Sizes i/a                                          |
| 3   | Shopper    | See list of deals, clearance items, etc                           | Take advantage of deals and save money                                                |
| 4   | Shopper    | See my cart's total at any time                                   | Avoid spending too much                                                               |
|     |            | **_Registration and User Accounts_**                              |                                                                                       |
| 5   | Reg User   | Register for an account                                           | Save my delivery details and order history                                            |
| 6   | Reg User   | Quickly login/out                                                 | Access my account                                                                     |
| 7   | Reg User   | Request a password reset                                          | receive and email to reset my password in case I forget it                            |
| 8   | Reg User   | Receive an email confirming my registration                       | Verify my account was registered successfully                                         |
| 9   | Reg User   | Access my user profile                                            | View my order history, manage my personal details                                     |
|     |            | **_Sorting and Searching_**                                       |                                                                                       |
| 10  | Shopper    | Sort the list of available products                               | See the products in a list sorted by price, rating, quantity available etc            |
| 11  | Shopper    | Sort a category of products                                       | See the products in a category sorted by name, price, rating, etc                     |
| 12  | Shopper    | Sort multiple categories simultaneously                           | Find the best rated or best priced across broad categories such as 'books' or 'honey' |
| 13  | Shopper    | Search for product                                                | Find a specific item I wish to purchase                                               |
| 14  | Shopper    | View a list of search results                                     | See if the product I want is available to purchase                                    |
|     |            | **_Purchasing and Checkout_**                                     |                                                                                       |
| 15  | Shopper    | Easily select the size and quantity whilst purchasing an item     | Ensure I don't accidentally select the wrong product, quantity, or size               |
| 16  | Shopper    | View items in my basket                                           | See what items are in my basket at a glance to ensure the items are correct           |
| 17  | Shopper    | Adjust the quantity of individual items in my bag                 | Easily adjust the amount of an item I intended to purchase (including removing)       |
| 18  | Shopper    | Easily enter my payment information                               | Checkout quickly, without hassle                                                      |
| 19  | Shopper    | Feel my payment and personal information is secure                | Provide the needed payment and personal information, and feel it is handled safely    |
| 20  | Shopper    | View confirmation of order before completing purchase             | Verify I haven't made any mistakes                                                    |
| 21  | Shopper    | Receive confirmation email after checking out                     | To keep my own record of the purchase                                                 |
|     |            | **_Admin and Store Management_**                                  |                                                                                       |
| 22  | Site Owner | Add a product                                                     | Add new products to my store                                                          |
| 23  | Site Owner | Edit/update a product                                             | Change the price, description, images etc of a product                                |
| 24  | Site Owner | Delete a product                                                  | Remove items that aren't for sale anymore                                             |

# ALLAUTH
## ADDING ALLAUTH
Allauth allows us to log in, log out, register etc...

HOW TO ADD ALLAUTH

1. INSTALL ALLAUTH:

	`pip3 install django-allauth==0.41.0`

2. ADD RELEVANT FILES FROM ALLAUTH DOCUMENTATION INTO settings.py:

	In TEMPLATES of the settings.py file, comment added to ensure no deletion of this line of code:

	`'django.template.context_processors.request', # Required by allauth`

	Allows allauth and django itself for that matter to access the HTTP request object in our templates. Used frequently by allauth for "request.user", "request.user.email" etc.

	—————————————

	Add beneath TEMPLATES:

	```
	AUTHENTICATION_BACKENDS = [
    # Needed to login by username in Django admin, regardless of `allauth`
    'django.contrib.auth.backends.ModelBackend',

    # `allauth` specific authentication methods, such as login by e-mail
    'allauth.account.auth_backends.AuthenticationBackend',
	]
	```

	ModelBackend: Allows superusers to log in via the back end.

	AuthenticationBackend: Allows users to log into our store via their email address.

	—————————————

	Add to INSTALLED_APPS:

	```
	'django.contrib.sites',
	'allauth',
	'allauth.account',
	'allauth.socialaccount',
	```
	allauth: the app.

	allauth.account: Allows all the basic user account stuff like logging in and out, user registration and password resets.

	allauth.socialaccount: Handles logging in via social media providers like Facebook and Google.

	django.contrib.sites: Used by the social account app to create the proper callback URLs when connecting via social media accounts, in conjunction with SITE_ID, below.

	—————————————

	Type beneath AUTHENTICATION_BACKENDS:

	`SITE_ID = 1`

	—————————————

3. UPDATE urls.py
	1. ADD PATH TO URLS.PY urlpatterns:

		```
		urlpatterns = [
    	path('admin/', admin.site.urls),
    	path('accounts', include('allauth.urls')),
		]
		```

	2. IMPORT include FROM django.urls:

		`from django.urls import path, include`

		Gives app all the urls for logging in and out, password resets etc.

4. MIGRATE:

	`python3 manage.py migrate`

5. RUN SERVER:

	`python3 manage.py runserver`

6. LOG IN AND CHANGE SITE'S NAMES:

	boutiqueado.example.com

	Boutique Ado

## TESTING ALLAUTH

1. ADD EMAIL_BACKEND variable TO settings.py:

	`EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'`

2. ADD EMAIL ACCOUNT CREATION CODE:

	```
	ACCOUNT_AUTHENTICATION_METHOD = 'username_email'
	ACCOUNT_EMAIL_REQUIRED = True
	ACCOUNT_EMAIL_VERIFICATION = 'mandatory'
	ACCOUNT_SIGNUP_EMAIL_ENTER_TWICE = True
	ACCOUNT_USERNAME_MIN_LENGTH = 4
	LOGIN_URL = '/accounts/login/'
	LOGIN_REDIRECT_URL = '/'
	```

3. CHANGE LOGIN_REDIRECT_URL:

	`LOGIN_REDIRECT_URL = ‘/success’`

4. RUN SERVER AND GOTO /accounts/login
    1. Try to log in
    2. Taken to “Confirm Email” page
    3. Log out and goto /admin
    4. “Verify” and “Primary” superuser’s email account
    5. Log out
    6. Log In at /account/login
    7. 404 page - taken to /success which is our “false” success page

5. CHANGE LOGIN_REDIRECT_URL BACK:

	`LOGIN_REDIRECT_URL = '/'`

6. FREEZE TO REQUIREMENTS.TXT:

	`pip3 freeze > requirements.txt`

7. COMMIT TO GIT:

	```
	git add . 
	git commit - m 'initial commit'
	git push
	```

# TEMPLATES

## HOW TO ADD THE BASE TEMPLATE

1. COPY ALLAUTH TEMPLATES TO OUR OWN TEMPLATES DIRECTORY:

	`cp -r ../.pip-modules/lib/python3.8/site-packages/allauth/templates/* ./templates/allauth`

2. IN THE PROJECT LEVEL DIRECTORY, CREATE BASE TEMPLATE:
    1. Create file called
		base.html
    2. Go to [getbootstrap.com](https://getbootstrap.com/docs/4.4/getting-started/introduction/#starter-template) and get boilerplate html
    3. Amend [boilerplate](https://www.icloud.com/notes/0HlikolTmtWcQmqtKzMDKsOuA)
    4. Above the !DOCTYPE, add {% load static %} for loading the css and js
    5. Add extra meta, css, js blocks ({% block extra_css %}) for page level changes
    6. Add title block {% block extra_title %} for page level title
    7. Add `<header>` for navigation etc
    8. Add messages block {% if messages %} and its `<div class=“message-container”>`
    9. Add page_header block {% block page_header %}
    10. Add content block {% block content %}
    11. Add postloadjs block for post JS {% block postloadjs %}

3. CREATE HOME APP:
    1. To create home:
	
		`python3 manage.py startapp home`

    2. Make home directory: 

		`mkdir -p home/templates/home`

    3. Create index.html file in the INNER home directory
    4. Extend “base.html”: {% extends “base.html” %}
    5. Load static files: {% load static %}
    6. Create content block and content:

		```
		{% block content %}
			<h1 class="display-4 text-success">It works!</h1>
		{% endblock%}
		```

    7. Add the view in the HOME directory’s views.py file:
        1. As the html file we have created is called “index.html”, we create a view called “index”: 

			`def index():`

        2. index must inherit request: 

			`def index(request):`

        3. Return render the file: 

			`return render(request, 'home/index.html')`

        4. Create new urls.py file in the HOME app and copy from PROJECT LEVEL urls.py file
        5. Remove unwanted imported items (such as “include”) from django.urls import
        6. Remove `“”” docstring “””`
        7. Add PATH to urlspattern: 

			`path('', views.index, name='home'),`

            1. ‘linkname/‘ or ‘’ if home page (root url)
            2. what it should render (views.index for example)
            3. name=‘name of page to be rendered in the browser tab’
        8. Import views from the current directory: 

			`from . import views`

    8. Include home urls in project level urls.py file:
        1. Add PATH to urlspattern: 

			`path('', include('home.urls')),`

    9. Add HOME to INSTALLED_APPS in settings.py
    10. Import os to settings.py: 
		
		`import os`

    11. Add ROOT templates directory and ALLAUTH templates directory to TEMPLATES = [ DIR=[ ] ]:

		```
		'DIRS': [
    		os.path.join(BASE_DIR, 'templates'),
    		os.path.join(BASE_DIR, 'templates', 'allauth'),
		],
		```

4. PRAY WHILE YOU START THE DEVELOPMENT SERVER:

	`python3 manage.py runserver`

5. ADD, COMMIT and PUSH

## TEMPLATE TAGS AND FILTERS

[Built-in template tags and filters](https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#built-in-template-tags-and-filters)

# MANAGING STATIC FILES
[Documentation](https://docs.djangoproject.com/en/3.2/howto/static-files/)

## CSS

1. CREATE STATIC DIRECTRORY AND ADD base.css FILE

	This is where the base CSS will live

2. ADD STATICFILES_DIR TO settings.py UNDER THE STATIC_URL SETTING:

	1. Join the path using os, making sure that it is a tuple:
		
		`STATICFILES_DIR = ((os.path.join(BASE_DIR, 'static),)`

## MEDIA

1. CREATE MEDIA DIRECTORY

	This is where images will live

2. Add MEDIA_URL TO settings.py:
	
	`MEDIA_URL = '/media/'`

3. Add MEDIA_ROOT path:

	`MEDIA_ROOT = (os.pth.join(BASE_DIR, 'media')`

## IMPORTING

2. IMPORT STATIC TO HOME DIRECTORY urls_py:

	`from django.conf.urls.static import static`

3. ADD PATH TO HOME DIRECTORY'S urls.py's urlpatterns:
	
	```
	urlpatterns = [
    	path('admin/', admin.site.urls),
    	path('accounts/', include('allauth.urls')),
    	path('', include('home.urls')),
	] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
	```
