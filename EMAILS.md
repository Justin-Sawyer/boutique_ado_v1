## EMAILS

### Gmail

Open an account and go to settings, then import and then import other settings.

Enable 2 step verification in order to create a specific password for the django app.

Go to App Passwords, select "mail" and "django" for other.

### Heroku

Copy the 16 letter password and enter this as a value in Heroku's config vars for a variable named `EMAIL_HOST_PASS`

Add `EMAIL_HOST_USER`and enter email address

### settings.py

```
if 'DEVELOPMENT' in os.environ:
    EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
    DEFAULT_FROM_EMAIL = 'boutiqueado@example.com'
else:
    EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
    EMAIL_USE_TLS = True
    EMAIL_PORT = 587
    EMAIL_HOST = 'smtp.gmail.com'
    EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
    EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASS')
    DEFAULT_FROM_EMAIL = os.environ.get('EMAIL_HOST_USER')

```

### Commit

Commit, then set up an account on the live site to test.

