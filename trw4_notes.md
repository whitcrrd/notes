### Associate Class Methods ###

`has_many :published_posts, -> { where(published: true) }, class_name: 'Post'`

`has_many :birthday_events, -> (user) { where starts_on: user.birthday }, class_name: 'Event'`

Using a hasone together with a hasmany relationship

```
has_many :comments
has_one :last_comment, -> { order 'posted_on' }, class_name: 'Comment'

has_one :primary_address, -> { where(primary: true)}, through: :addressables, source: :addressable
```

Including nested association in normal query:

```
class Person < ActiveRecord::Base
  has_many :addresses, -> { include(:city) }
end

```


Active Record

- `default_scope` also gets applied to your models when building or creating them, which can be a gotchya if you are not careful:
 - `default_scope { where(status: 'open') }`
	 	- `door = Door.new`
	 	- `door.status #=> 'open'`


### Active Record Callbacks ###

##### All Records #####
- before_validation
- after_validation
- before_save
- after_save
- around_save

##### New Records #####
- before_create
- after_create
- around_create

##### Persisted Records #####
- before_update
- after_update
- around_update


The access level for callback methods should always be **protected** or **private**, but never public



##### Cleaning Up User Inputs using `before_validation` on create: #####

```
class PhoneNumber < ActiveRecord::Base
  before_validation on: :create do 
    self.number = number.gsub(/[^0-9]/, '') # keep only numbers
  end
end
```


### Polymorphism ###

Assuming we have and `Comment` classes, and we want to be able to add comments to multiple types of objects, say photos and videos.

In your migration, your comments table should look something like this: 
```
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.integer :commentable
      t.string :commentable_type
    end
  end
end

```
And models:
```
class Comment < ActiveRecord::Base
 belongs_to :commentable, polymorphic: true
end

class Photo < ActiveRecord::Base
  has_many :comments, as: :commentable
end

class Video < ActiveRecord::Base
  has_many :comments, as: :commentable
end
```

### enum in Rails 4 ###

Rails 4 provides the `enum` macro style class method, that allows you to pass it an attribute name and an array of status values that the attribute can be set to -- allowing for a simple state machine.

Rails will map the status values to integers, so you need to use set the column type as an integer when migrating: 

```
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.integer :status, default: 0
    end    
  end
end
```

and with our `Post` model looking like this: 

```
class Post < ActiveRecord::Base
  enum status: %i(draft reviewed published)
end
```

Rails provides both instance (predicate & bang) methods for each of the statuses you define, as well as class methods for scoping.  Given the previous example:

```
post = Post.new
post.status # => 'draft'
post.draft? # => true
post.reviewed? # => false
post.published!
post.status # => 'published'
post.published? # => true

published_posts = Post.published
```








