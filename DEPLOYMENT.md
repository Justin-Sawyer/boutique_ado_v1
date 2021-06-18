## DEPLOYMENT

### The Heroku App

#### online

Create new app on [heroku](heroku.com)

Provision new postgres database from the resources tab

#### Terminal

Install dj_database_url and psychopg2 in the CLI
	
```
pip3 install dj_database_url
pip3 install psycopg2-binary
```

Freeze requirements

`pip3 freeze > requirements.txt`

#### settings.py

Import dj_database_url

```
from pathlib import Path
import os
import dj_database_url
```

Comment out the default configuration of DATABASES

```
# DATABASES = {
#    'default': {
#        'ENGINE': 'django.db.backends.sqlite3',
#        'NAME': BASE_DIR / 'db.sqlite3',
#    }
# }
```

Replace the default database with a call to dj_database_url.parse() and give it the DATABASE_URL from heroku's config vars

```
DATABASES = {
    'default': dj_database_url.parse('DATABASE_URL value')
}
```

#### Terminal

Show and run migrations

```
python3 manage.py showmigrations
python3 manage.py migrate
```
### Importing the database product data

#### Terminal

Load the data

Since the products depend on the categories existing, the CATEGORIES MUST BE LOADED FIRST

```
python3 manage.py loaddata categories
python3 manage.py loaddata products
```

### Create superuser

#### Terminal

`python3 manage.py createsuperuser`

### Ensure database url does not exist in version control

1. Remove the Heroku database configuration completely
2. Uncomment the commented out DATABASES

### Commit and push to Git

## Deploying to Heroku

#### settings.py

Set DATABASES to use postgres when site is live on heroku, but the default sqlite in development mode

```
if 'DATABASE_URL' in os.environ:
    DATABASES = {
        'default': dj_database_url.parse(os.environ.get('DATABASE_URL'))
    }
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
        }
    }
```

#### Terminal

Install gunicorn as webserver and freeze requirements

```
pip3 install gunicorm
pip3 freeze > requirements.txt
```

### The Procfile

Create Procfile at system level and use it to tell Heroku to create a web dyno which will run gunicorn and serve the django app

#### Procfile

`web: gunicorn boutique_ado.wsgi:application`

#### Terminal

Log in to Heroku and enter login details

`heroku login -i`

Temporarily disable COLLECTSTATIC

`heroku config:set DISABLE_COLLECTSTATIC=1`

If more than one app exists in heroku account, tell heroku WHICH app COLLECTSTATIC should be disabled on

`heroku config:set DISABLE_COLLECTSTATIC=1 --app <name_of_app>`

#### settings.py

Add hostname of Heroku app and 'localhost' to ALLOWED_HOSTS. Adding 'localhost' allows for GitPod development

`ALLOWED_HOSTS = ['<name_of_app>.herokuapp.com', 'localhost']`

#### Terminal

First commit and push to GitHub

```
git add .
git commit -m 'deploy to GitHub'
git push
```

Now deploy to heroku

`git push heroku master`

If deployment to heroku does not work, initialize the heroku git remote and push again

```
heroku git:remote -a <name_of_app>
git push heroku master
```

### Set up automatic deploys to heroku and GitHub

#### heroku.com

Deploy tab, then "Connect to GitHub". Search for repo, click "Connect" then "Enable Automatic Deploys"

#### settings.py

Remove SECRET_KEY

#### online

Get new SECRET_KEY from a [django secret key generator](https://miniwebtool.com/django-secret-key-generator/)

Add new SECRET_KEY to the config vars in Heroku

Repeat for GitPod with a different SECRET_KEY

#### settings.py

Add SECRET_KEY variable, gettings its value from the environment

`SECRET_KEY = os.environ.get('SECRET_KEY', '')`

Set debug to be true only if there's a variable called development in the environment
`DEBUG = 'DEVELOPMENT' in os.environ`

#### Commit and push

## Amazon Web Services (AWS)
### The Account

Above, we disallowed the collection of static files (such as css and media folders), so the website is currently "bare bones": there are no images and there is no styling.

We will use AWS for the hosting of these files.

#### online

Create and AWS account via [aws.amazon.com](http://aws.amazon.com)

Once created, navigate to the AWS management console under “my account”

Search for S3 and then create a new bucket - name it to match the heroku app's name

Uncheck block all public access and acknowledge that the bucket will be public

Click "create bucket"

##### Bucket properties

Turn on static website hosting to allow a new endpoint that can be used for access from the internet

Enter default values for index and error documents

```
index.html
error.html

```

##### Bucket permissions

Paste in a coors configuration to set up the required access between the Heroku app and this s3 bucket

```
[
  {
      "AllowedHeaders": [
          "Authorization"
      ],
      "AllowedMethods": [
          "GET"
      ],
      "AllowedOrigins": [
          "*"
      ],
      "ExposeHeaders": []
  }
]
```

Create a Security Policy by selecting "Policy Genreator". A new window opens, from which
1. Choose S3 Bucket Policy from the Type of Policy dropdown
2. Add `* `to Principal to allow all principals
3. Copy the Bucket ARN value from the first tab and paste it into the Amazon Resource Name (ARN) infobox
4. Click "Add Statement"
5. Click "Genarate Policy" and copy the policy into the bucket policy editor of the first tab.
6. Add `/*`to the end of the Resource key's value, but within the string.

	`"Resource": "arn:etc/*"`

7. Save

##### Bucket Access Control

Set ACL to "Everyone" and "list" and confirm understanding

### AWS Groups, Policies and Users

#### IAM (Identity and Access Management)

The process here is:
1. Create a group for the user to live in
2. Create an access policy giving the group access to the s3 bucket3. Assign the user to the group so it can use the policy to access all the files.

##### User Groups

Create a new group called `manage-<name_of_app>`

Since the policy has not yet been created, click through to the end then "Create Group"

##### Policies

Create a new policy

Select "import managed policy" from the JSON tab

Choose and import the "AmazonS3FullAccess" policy

Once imported, change the "Resource" value to a list of own values

```
"Resouce": [
	"<the Bucket ARN>",
	"<the Bucket ARN>/*"
	]
```
Review policy

Name policy `<name_of_app>-policy`

Give policy a description `Access to S3 bucket for <name_of_app> bucket`

Create policy

##### Groups

Select group

Attach policy

### Create User for group

##### Users

Add user and create user named `<name_of_site>-staticfiles-user`

Give user programmatic access

Add user to group by checking the checkbox of the group

Create user

DOWNLOAD THE CSV FILE and keep it safe

## Connect django to S3

#### Terminal

Install boto3 and django-strorages, then freeze requirements

```
pip3 install boto3
pip3 install django-storages
pip3 freeze > requirements.txt
```

#### settings.py

Add storages to list of INSTALLED_APPS

`'storages',`

Tell django which S3 bucket to communicate with. We only want to do this with heroku, so an if statement is needed: `if 'USE_AWS' in os.environ:`. In other words, in development, using GitPod, we will not add `'USE_AWS'`to the list of variables, but we will to heroku config vars

```
if 'USE_AWS' in os.environ:
    # Bucket config
    AWS_STORAGE_BUCKET_NAME = '<name_of_app>'
    AWS_S3_REGION_NAME = 'eu-west-3'
    AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
```

MAKE SURE NOT TO ADD THE VALUES OF THE AWS_ACCESS_KEY_ID AND AWS_SECRET_ACCESS_KEY TO VERSION CONTROL

#### Heroku

Add AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY variables to config vars and insert their values taken from the downloaded .csv file, above

Remove the DISABLE_COLLECTSTATIC variable

#### settings.py

Tell django where the production static files come from

```
if 'USE_AWS' in os.environ:
    # Bucket config
    AWS_STORAGE_BUCKET_NAME = 'boutique-ado-justin'
    AWS_S3_REGION_NAME = 'eu-west-3'
    AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
    # Production static files location formatted as f-string to make url
    AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'
```

### Custom storage

Tell django to use s3 in production to store static files whenever collectstatic is run and that uploaded product images go there too.

Create custom_storages.py file

#### custom_storages.py

Import settings and boto3

```
from django.conf import settings
from storages.backends.s3boto3 import S3Boto3Storage
```

Create custom classes that inherit from django-storages for static and media files

```
class StaticStorage(S3Boto3Storage):
    location = settings.STATICFILES_LOCATION
class MediaStorage(S3Boto3Storage):
    location = settings.MEDIAFILES_LOCATION
```

#### settings.py

Use the custome storage class and that the location it should save static files is a folder called static, and media files should be stored in a folder called media

```
if 'USE_AWS' in os.environ:
    # Bucket config
    AWS_STORAGE_BUCKET_NAME = 'boutique-ado-justin'
    AWS_S3_REGION_NAME = 'eu-west-3'
    AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
    AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'

    # Static and media files storage
    STATICFILES_STORAGE = 'custom_storages.StaticStorage'
    STATICFILES_LOCATION = 'static'
    DEFAULT_FILE_STORAGE = 'custom_storages.MediaStorage'
    MEDIAFILES_LOCATION = 'media'
```

Override  and explicitly set the URLs for static and media files using the custom domain and the new locations

```
if 'USE_AWS' in os.environ:
    # Bucket config
    AWS_STORAGE_BUCKET_NAME = 'boutique-ado-justin'
    AWS_S3_REGION_NAME = 'eu-west-3'
    AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
    AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'

    # Static and media files storage
    STATICFILES_STORAGE = 'custom_storages.StaticStorage'
    STATICFILES_LOCATION = 'static'
    DEFAULT_FILE_STORAGE = 'custom_storages.MediaStorage'
    MEDIAFILES_LOCATION = 'media'

    # Override static and media URLS in production
    STATIC_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/{STATICFILES_LOCATION}'
    MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/{MEDIAFILES_LOCATION}'
```

This means that whenever the project is deployed to Heroku, Heroku will run `python3 manage.py collectstatic` during the build process and will search through all of the apps and project folders looking for static files.
It will use the S3 custom domain setting here in conjunction with the custom storage classes that tell it the location at that URL where we'd like to save things.
So in effect when the USE_AWS setting is true, whenever collectstatic is run, static files will be collected into a static folder in the S3 bucket.

To trigger this,

###### git add, commit and push

Once pushed, refresh the static/ page of AWS.

## Caching

Tell the browser to cache static files for a long time since they don't change very often, thus improving performance
```
if 'USE_AWS' in os.environ:
    # Cache control
    AWS_S3_OBJECT_PARAMETERS = {
        'Expires': 'Thu, 31 Dec 2099 20:00:00 GMT',
        'CacheControl': 'max-age=94608000',
    }
    
    # Bucket config
```

git add, commit and push


## Media Files

### Amazon S3 bucket

Add a folder called "media" and upload the images physically via the internet

Click "next" then grant public read access under Permissions

## Superuser

Log in to live site and then go to admin. Check the boxes in the admin. Update the host names, too.

## Stripe

From stripe.com, get the API keys and add them as variables to the config variables in Heroku

Then add new endpoint for the webhooks `https://<name_of_site>/checkout/wh/`, select "receive all webhooks"

Add the signing secret as a variable to heroku's config vars

Send test webhook