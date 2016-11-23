# :flags: Flagship :ship:

Ship/unship features using flags defined with declarative DSL.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'flagship'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install flagship

## Usage

### Define and use a flagset

```rb
Flagship.define :app do
  enable  :stable_feature
  enable  :experimental_feature, if: ->(context) { context.current_user.staff? }
  disable :deprecate_feature
end

Flagship.select_flagset(:app)
```

### Branch with a flag

```rb
if Flagship.enabled?(:some_feature)
  # Implement the feature here
end
```

### Set context variables

Both of below can be called as `context.foo` from `:if` block.

```rb
# Set a value
Flagship.set_context :foo, 'FOO'

# Set a lambda
Flagship.set_context :foo, ->(context) { 'FOO' }
```

Or you can set a method too.

```rb
Flagship.set_context :current_user, method(:current_user)
```

### Extend flagset

```rb
Flagship.define :common do
  enable :stable_feature
end

Flagship.define :development, extend: :common do
  enable :experimental_feature
end

Flagship.define :production, extend: :common do
  disable :experimental_feature
end

if Rails.env.production?
  Flagship.select_flagset(:production)
else
  Flagship.select_flagset(:development)
end
```

### Override flag with ENV

You can override flags with ENV named `FLAGSHIP_***`.

Assuming that there is a flag `:foo`, you can override it with ENV `FLAGSHIP_FOO=1`.

### Fetch all features

```rb
Flagship.features
# => Array of Flagship::Feature

Flagship.features.map(&:key)
# => Array key of enabled features
```

### Categorize features with tags

```rb
Flagship.define :blog do
  enable :post
  enable :comment, communication: true
  enable :trackback, communication: true
end

Flagship.select_flagset(:blog)

# Fetch keys of enabled communication features
Flagship.features.select{ |feature| feature.tags[:communication] && feature.enabled? }.map(&:key)
# => [:comment, :trackback]
```

### `with_tags`

Using `with_tags`, you can set same tags to multiple features at once.

```rb
Flagship.define :blog do
  enable :post

  with_tags(communication: true) do
    enable :comment
    enable :trackback
  end
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/yuya-takeyama/flagship.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
