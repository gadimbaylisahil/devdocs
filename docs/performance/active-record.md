## Using group_by instead of distinct to remove duplicates

```ruby
User.group(:currency).select(:currency).pluck(:currency)
```

Performs better than:

```ruby
User.select(:currency).distinct.pluck(:currency)
```