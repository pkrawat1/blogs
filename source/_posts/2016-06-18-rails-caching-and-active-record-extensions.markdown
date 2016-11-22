---
layout: post
title: "Rails Caching and Active Record Extensions"
date: 2016-06-18 21:34:14 +0530
comments: true
categories:
---

As a rails developer, I have gone through situations when we really want to get the most out of the limited resources.
So we opt for digging out the areas where we can improve. And caching is one of the option everybody thinks about, and implement.

#### Situation
Lately I have been looking at the solution to cache the Active Record Objects directly. Also if it could cache the methods frequently used methods too.
So I learn't that `Rails.cache` could actually store the Active records directly, that can be configured with redis easily.

###### The Problem
The code was too repetitive. Every Where I see was a similar pattern
like
``` ruby
# Write
Rails.cache.write(key, value)
# Read
Rails.cache.read(key)
# Clear
Rails.cache.del(key)
```

This was becoming a real issue. So I thought if we can have class method where we could just pass in the method names, and create the cache, that could be shared with all models. So first solution I thought of was creating a Concern that would be included in all models. So came up with this soution.

#### The Solution
```ruby
module CachedMethods
  extend ActiveSupport::Concern

  module ClassMethods
    def cached_methods(*methods)
      methods.each do |association|
        define_method("cached_#{association}") do |key = nil, cached = nil|
          key = "#{association}_for_#{self.class.to_s.underscore}_#{id}"
          cached = Rails.cache.read(key) rescue false
          return cached if cached
          load_assoc = send(association)
          Rails.cache.write(key, load_assoc, expires_in: 30.minutes)
          update_cache_keys(key)
          load_assoc
        end

        define_method("#{association}_is_cached?") do
          key = "#{association}_for_#{self.class.to_s.underscore}_#{id}"
          Rails.cache.read(key).present?
        end
      end
    end
  end

  private

  # keeps tracks of all cache keys, for the included class.
  define_method('update_cache_keys') do |key|
    multi_key = "#{self.class.to_s.underscore}_#{id}_cache_keys"
    keys = Rails.cache.read(multi_key) || []
    keys << key
    Rails.cache.write(multi_key, keys.uniq)
  end

  # can be called to clear all cached data for included class
  def delete_cache
    cached = Rails.cache.read("#{self.class.to_s.underscore}_#{id}_cache_keys")
    return unless cached
    cached.each { |key| Rails.cache.delete(key) }
  end
end
```

But I did not want this to manually including every where in the application. I don't want to remember which concern to use. So thought of creating an extension for the Active record.
#### Improvement
So created an ActiveRecordExtension.

``` ruby
# Inculde custom enxtensions here
module ActiveRecordExtension
  extend ActiveSupport::Concern
  included do
    include CachedMethods
  end
end

# include the extension
ActiveRecord::Base.send(:include, ActiveRecordExtension)
```

>Create an initializer called extensions, that will load it during application initialization.

>`require "active_record_extension"`

#### How to Use
Now anybody can use this extension in the system easily.

``` ruby
# ========================================
#                  USAGE
# ========================================
# // Ruby.
Class Fruit
#   ...
#   # this code generates some instance methods
#   # cached_apples and cached_mangoes
#   # which when called the first time, will cache the ActiveRecord Object,
#   # and after every call will get the object back from the cache only.
#   # You can also cache the method results too.
#   # Also it creates cache check instance methods
#   # as in this case apples_is_cached? And mangos_is_cached?
cached_methods :apples, :mangos
#   # As the name suggests, this method will clear all the cache related
#   # to the class.
after_save :delete_cache
#   ...
end
```
>Now that looks easier to use.

#### What is provides
```ruby
fruit = Fruit.first
apples = fruit.cached_apples # cache apples
mangos = fruit.cached_mangoes # cache mangoes

# Now if you this code is run again
apples = fruit.cached_apples # from cache
mangoes = fruit.cached_mangoes # from cache

# the cache will auto expire after 30 minutes
# or if the Fruit object is updated.
# we have added delete_cache callback
after_save :delete_cache
```
> How cool is that.

##### Try it yourself Now.
