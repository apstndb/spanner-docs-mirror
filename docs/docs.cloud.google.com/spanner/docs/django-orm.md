The Django ORM is a powerful standalone component of the [Django web framework](https://www.djangoproject.com/) that maps Python objects to relational data. It provides a Pythonic interface to the underlying database, and includes tools for automatically generating schema changes and managing schema version history.

**Note:** The PostgreSQL interface for Spanner doesn't support the Django ORM.

The [django-google-spanner](https://pypi.org/project/django-google-spanner/) package is a third-party database backend for using Spanner with the Django ORM, powered by the [Spanner Python client library](https://github.com/googleapis/python-spanner) .

With this integration, Django applications can take advantage of Spanner's high availability and consistency at scale.

## Install, configure, and use

Refer to the [django-google-spanner documentation](https://pypi.org/project/django-google-spanner/) for instructions on installing and configuring your environment.

## What's next

  - Read our [blog post](https://cloud.google.com/blog/topics/developers-practitioners/django-orm-support-cloud-spanner-now-generally-available) for a walkthrough and insight into how the code is designed.
  - See [code examples](https://github.com/googleapis/python-spanner-django/tree/master/examples) using Django with Spanner.
  - Learn more about the [Django project](https://www.djangoproject.com/) .
  - Learn more about [DB API](https://www.python.org/dev/peps/pep-0249/) .
  - [File a GitHub issue](https://github.com/googleapis/python-spanner-django/issues) to report a bug or ask a question about using the Django ORM with Spanner.
