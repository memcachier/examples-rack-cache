# MemCachier Rack::Cache Example

This is an example Rails app that uses
[Rack::Cache](http://rtomayko.github.io/rack-cache/) with
[MemCachier](http://www.memcachier.com) for caching assets. This
example is written with Rails 4.0, but also runs fine with Rails 3.2.

Please follow this
[tutorial](https://devcenter.heroku.com/articles/rack-cache-memcached-rails31)
for getting started with Rack::Cache.

You can view a working version of this app
[here](http://memcachier-examples-rack.herokuapp.com) that uses
[MemCachier on Heroku](https://addons.heroku.com/memcachier). Running
this app on your local machine in development will work as well,
although then you won't be using MemCachier -- you'll be using a local
dummy cache. MemCachier is currently only available with various cloud
providers.

## Running

To run this rails app, first install the needed gems.

```shell
  gem install bundler
  bundle install
```

Then create a database and run your migrations.

```shell
  bundle exec rake db:create
  bundle exec rake db:migrate
````

Then create your production database and precompile your assets.

```shell
  bundle exec rake db:create RAILS_ENV=production
  bundle exec rake assets:precompile
```

Now start memcached locally.

```shell
  memcached
```

Next, start your server. You'll want to run in production so that
caching is enabled.

```shell
  rails s -e production
```

Visit http://localhost:3000 and view your logs, you should see some cache entries

```shell
    cache: [GET /assets/application-193197163ac0c1601c69cbdaf22f6ce6.css] miss, store
    cache: [GET /assets/application-bf044948e75c7535c35baf4b42604116.js] miss, store
    cache: [GET /assets/rails-782b548cc1ba7f898cdad2d9eb8420d2.png] miss, store
```

Refresh the page and you should see that those entries can be pulled fresh from cache

```shell
    cache: [GET /assets/application-193197163ac0c1601c69cbdaf22f6ce6.css] fresh
    cache: [GET /assets/application-bf044948e75c7535c35baf4b42604116.js] fresh
    cache: [GET /assets/rails-782b548cc1ba7f898cdad2d9eb8420d2.png] fresh
```

When you modify a file such as `app/assets/stylsheets/application.css` and run
`rake assets:precompile` and then start and stop your server, you should see
that there is now a new digest for the file and it must be stored in cache
again.

## Setup MemCachier

Setting up MemCachier to work in Rails is very easy. You need to make
changes to Gemfile, production.rb, and any app code that you want
cached. These changes are covered in detail below.

### Gemfile

MemCachier has been tested with the [dalli memcache
client](https://github.com/mperham/dalli). Add the following Gem to
your Gemfile:

~~~~ .ruby
gem 'memcachier'
gem 'dalli'
~~~~

Then run `bundle install` as usual.

Note that the `memcachier` gem simply sets the appropriate environment
variables for Dalli. You can also do this manually in your
production.rb file if you prefer:

~~~~ .ruby
ENV["MEMCACHE_SERVERS"] = ENV["MEMCACHIER_SERVERS"]
ENV["MEMCACHE_USERNAME"] = ENV["MEMCACHIER_USERNAME"]
ENV["MEMCACHE_PASSWORD"] = ENV["MEMCACHIER_PASSWORD"]
~~~~

Alternatively, you can pass these options to config.cache_store (also
in production.rb):

~~~~ .ruby
config.cache_store = :dalli_store, ENV["MEMCACHIER_SERVERS"].split(','),
                    {:username => ENV["MEMCACHIER_USERNAME"],
                     :password => ENV["MEMCACHIER_PASSWORD"]}
~~~~

### production.rb

Ensure that the following configuration option is set in production.rb:

    ~~~~ .ruby
    # Configure rails caching (action, fragment)
    config.cache_store = :dalli_store
    
    # Configure Rack::Cache (rack middleware, whole page / static assets)
    client = Dalli::Client.new(ENV["MEMCACHIER_SERVERS"],
                               :value_max_bytes => 10485760)
    config.action_dispatch.rack_cache = {
      :metastore    => client,
      :entitystore  => client
    }
    config.static_cache_control = "public, max-age=2592000"
    ~~~~

## Rack::Cache

For detailed documentation see the [Rack::Cache
homepage](http://rtomayko.github.io/rack-cache/).

### Rack::Cache Metastore

The metastore holds metadata about the objects in cache. Metastore entries are
small and accessed frequently. It makes alot of sense to put this data in a
fast light datastore such as Memcache

### Rack::Cache Entitystore

The entity store is where the objects get cached. Entity store objects are
typically large and accessed infrequently. For that reason it might make sense
to store them on disk rather than to take up a large amount of room in
Memcache.

## Get involved!

We are happy to receive bug reports, fixes, documentation enhancements,
and other improvements.

Please report bugs via the
[github issue tracker](http://github.com/memcachier/examples-rack-cache/issues).

Master [git repository](http://github.com/memcachier/examples-rack-cache):

* `git clone git://github.com/memcachier/examples-rack-cache.git`

## Licensing

This library is BSD-licensed.

licensed under MIT License Copyright (c) 2012 Schneems. See LICENSE.txt for
further details.

