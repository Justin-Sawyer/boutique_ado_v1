# Boutique Ado e-commerce store

## Built using Django

## HOW THIS PROJECT WAS SET UP:

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
