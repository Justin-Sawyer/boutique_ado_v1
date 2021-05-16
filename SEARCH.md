# SEARCH
## General Search

1. Add the relevant url path to the search form(s): 

	`{% url 'products' %}`

2. Add the search functionality to the relevant function 
    1. In that function, check if the search request exists: 
	
		`if request.GET”:`

    2. The name attributeof our query, and what is rendered in the url address is what we are checking for. Our input had a name of “q”, so:

		```
		if request.GET:
			if ‘q’ in request.GET:
		```

3. Set the name equal to a variable called query:

	```
	if ‘q’ in request.GET:
		query = request.GET[‘q’]
	```
		


4. Request error message if query is blank (ie, someone clicks “search” without entering anything to search for), then return redirect back to page:

	```
	if ‘q’ in request.GET:
		query = request.GET[‘q’]
		if not query:
			messages.error(request, “Error message”)
			return redirect(reverse (‘products’))
	```

5. Import redirect and reverse from django.shortcuts: 

	`from django.shortcuts import reverse, redirect`

6. Import messages from django.contrib: 

	`from django.contrib import messages`

7. Import Q from django.db.models:
    1. Filtering objects in Django returns AND, not OR. In other words, if we set search parameters to search a name and a description, the search term would have to be in both the name and the description. Importing Q and tweaking the code means we can search for multiple criteria and return all elements, even if they are only in one of the two parameters (name, description).  
    2. Set the search parameters: 

		`queries = Q(name__icontains=query) | Q(description__icontains=query)`

        1. variable named “queries” as we search more than one parameter
        2. | = “or”
        3. i = search is case insensitive
8. Return the searched products:
    1. Originally we return all Product objects, but now, we are returning the filtered results, so we need to reassign the products variable to be the filtered queries: 

		`products = products.filter(queries)`

    2.  Add the query to the context as “search_term” (for example):

		```
		context = {
        	'products': products,
        	'search_term': query,
		}
		```

9. Set query outside of this code block to None, so that we are not returning a no existant search when we load the page: 

	`query = None`

10. Git add, commit, push

```
def all_products(request):
    """ A view to show all products, including sorting and search queries """

    products = Product.objects.all()
    query = None

    if request.GET:
        if 'q' in request.GET:
            query = request.GET['q']
            if not query:
                messages.error(request, "You didn't enter any search criteria!")
                return redirect(reverse('products'))

            queries = Q(name__icontains=query) | Q(description__icontains=query)
            products = products.filter(queries)

    context = {
        'products': products,
        'search_term': query,
    }

    return render(request, 'products/products.html', context)
```

## Search by Categories (Navigation)

1. Add the template literals to the href=“” links: 

	`href="{% url ‘products’ %}?category=name_of_category,”`

2. Capture the categories parameter in the relevant function (all_products()) in views.py
    1. Start with categories being None: 

		`categories = None`

    2. Check if category exists in request.GET: 

		`if 'category' in request.GET:`

    3. Split if necessary and assign to variable: 

		`categories = request.GET['category'].split(',')`

    4.  Filter the current query set of all products down to only products whose category name is in the list: 

		`products = products.filter(category__name__in=categories)`

    5. Import the Category model: 

		`from .models import Product, Category`

    6. Filter all categories down to the ones whose name is in the list from the URL: 

		`categories = products.filter(name__in=categories)`

    7. Add the list of category objects to context: 

		`'current_categories': categories,`

### A Note on "`__`" syntax:

This double underscore syntax is common when making queries in django.
Using it here means we're looking for the name field of the category model and we're able to do this because category and product are related with a `foreignKey`.

- ### [README](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/README.md)
- ### [PRODUCTS app](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/PRODUCTS.md)
- ### [SORTING](https://github.com/Justin-Sawyer/boutique_ado_v1/blob/master/SORTING.md)