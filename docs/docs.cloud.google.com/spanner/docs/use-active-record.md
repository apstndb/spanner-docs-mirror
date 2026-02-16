Ruby [Active Record](https://guides.rubyonrails.org/active_record_basics.html) is an Object-Relational Mapping (ORM) library bundled with [Ruby on Rails](https://rubyonrails.org/) . Active Record provides an abstraction over the underlying database and includes capabilities such as automatically generating schema changes and managing schema version history.

The [Spanner support for Active Record](https://github.com/googleapis/ruby-spanner-activerecord) enables [Active Record](https://guides.rubyonrails.org/active_record_basics.html) users to use Spanner databases. With this support, Ruby applications can take advantage of Spanner's high availability and external consistency at scale through an ORM.

**PostgreSQL interface note:** The [PostgreSQL interface for Spanner](/spanner/docs/postgresql-interface) doesn't support Active Record.

## Setting up the Spanner support for Active Record

To setup the Spanner support for Active Record in your application, edit the `  Gemfile  ` of your Rails application and add the [activerecord-spanner-adapter](https://rubygems.org/gems/activerecord-spanner-adapter) gem.

``` text
gem 'activerecord-spanner-adapter'
```

Next, run bundle to install the gem.

``` text
bundle install
```

As authentication for the Spanner support for Active Record, the [service account JSON credentials](/docs/authentication/getting-started) file location should be provided in the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable. Otherwise, the Spanner support for Active Record can also use the default credentials set in the Google Cloud SDK `  gcloud  ` application.

## Using the Spanner support for Active Record

For more information about the available features, limitations of the Spanner support for Active Record, recommendations on how to use it, and for code samples, please consult the [reference documentation](https://github.com/googleapis/ruby-spanner-activerecord#readme) on GitHub.

## What's next

  - Checkout the [code examples](https://github.com/googleapis/ruby-spanner-activerecord#examples) on how to use the Spanner support for Active Record.
  - View the repository for the Spanner support for Active Record on [GitHub](https://github.com/googleapis/ruby-spanner-activerecord) .
  - File a [GitHub issue](https://github.com/googleapis/ruby-spanner-activerecord/issues) to report a bug or ask a question about the Spanner support for Active Record.
  - Learn more about [Active Record](https://guides.rubyonrails.org/active_record_basics.html) .
  - Learn more about [Ruby Gems](https://rubygems.org/) .
  - Learn about authorization and authentication credentials in [Getting started with authentication](/docs/authentication/getting-started) .
