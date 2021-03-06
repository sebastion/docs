October's database models are named Active Record which is based on the [Eloquent ORM provided by Laravel](http://laravel.com/docs/eloquent).

#### Relations

The following relations are available, along with their optional and required arguments:

| Name           | Description                                    | Optional                                  | Required  |
|:-------------- |:-----------------------------------------------|:----------------------------------------- |:--------- |
| $hasOne        | Has a single related model that belongs to it  | foreignKey                                |           |
| $hasMany       | Has many related models that belong to         | foreignKey                                |           |
| $belongsTo     | Owned by another related model (slave)         | foreignKey                                |           |
| $belongsToMany | Owned by multiple related models               | table, foreignKey, primaryKey, pivotData  |           |
| $morphTo       | Polymorphic version of belongs to              | name, type, id                            |           |
| $morphOne      | Polymorphic version of has one                 | type, id                                  | name      |
| $morphMany     | Polymorphic version of has many                | type, id                                  | name      |

An example of defining a relationship:

```php
class BlogPost extends Model
{
  public $belongsTo = [
    'user' => ['User', 'foreignKey' => 'user_id']
  ];

  public $belongsToMany = [
    'categories' => ['Category', 'table' => 'october_blog_posts_categories']
  ];

  public $morphMany = [
    'featured_images' => ['Modules\System\Models\File', 'name' => 'attachment']
  ];
}
```

#### Events

The following events are available:

| Name           | Description                                                |
|:-------------- |:-----------------------------------------------------------|
| beforeCreate   | Before the model is saved, when first created              |
| afterCreate    | After the model is saved, when first created               |
| beforeSave     | Before the model is saved, either created or updated       |
| afterSave      | After the model is saved, either created or updated        |
| beforeValidate | Before the supplied model data is validated                |
| afterValidate  | After the supplied model data has been validated           |
| beforeUpdate   | Before an existing model is saved                          |
| afterUpdate    | After an existing model is saved                           |
| beforeDelete   | Before an existing model is deleted                        |
| afterDelete    | After an existing model is deleted                         |

An example of using an event:

```php
public function beforeCreate()
{
  // Generate a URL slug for this model
  $this->slug = slug($this->name);
}
```

##### Joined Eager Load

Similar to the standard [Eager Loading](http://laravel.com/docs/eloquent#eager-loading), you eager load and join a relation to the main query.

```php
Post::joinWith('Category')->select("concat(posts.name, ' - ', category.name)")->get();
```

This will also eager load 

##### Further reading on Active Record (Eloquent)

* [Eloquent ORM - Laravel documentation](http://laravel.com/docs/eloquent)
* [Active record pattern - Wikipedia](http://en.wikipedia.org/wiki/Active_record_pattern)

---

## Model Validation

October models use Laravel's built-in [Validator class](http://laravel.com/docs/validation). 
Defining validation rules are defined in the model class as a variable named `$rules`:

```php
class User extends \October\Rain\Database\Model
{
  public $rules = [
    'name'                  => 'required|between:4,16',
    'email'                 => 'required|email',
    'password'              => 'required|alpha_num|between:4,8|confirmed',
    'password_confirmation' => 'required|alpha_num|between:4,8',
  ];
}
```

> **Note**: you're free to use the [array syntax](http://laravel.com/docs/validation#basic-usage) for validation rules as well.

Models validate themselves automatically when the `save()` method is called.

```php
$user = new User;
$user->name = 'John doe';
$user->email = 'john@doe.com';
$user->password = 'test';

$success = $user->save(); // Returns false if model is invalid
```

> **Note:** You can also validate a model at any time using the `validate()` method.

#### Retrieving Validation Errors

When a model fails to validate, a `Illuminate\Support\MessageBag` object is attached to the object which contains validation failure messages.

Retrieve the validation errors message collection instance with `errors()` method or `validationErrors` property.

Retrieve all validation errors with `errors()->all()`. Retrieve errors for a *specific* attribute using `validationErrors->get('attribute')`.

> **Note:** The Model leverages Laravel's MessagesBag object which has a [simple and elegant method](http://laravel.com/docs/validation#working-with-error-messages) of formatting errors.

#### Overriding Validation

There are two ways to override Ardent's validation:

##### 1. Forced Save
`forceSave()` validates the model but saves regardless of whether or not there are validation errors.

##### 2. Override Rules and Messages
The `validate($rules, $customMessages)` take two parameters:

- `$rules` is an array of Validator rules of the same form as `Model::$rules` property.
- The same is true of the `$customMessages` parameter (same as `Model::$customMessages` property)

An array that is **not empty** will override the rules or custom error messages specified by the class for that instance of the method only.

> **Note:** the default value for `$rules` and `$customMessages` is empty `array()`; thus, if you pass an `array()` nothing will be overridden.

#### Custom Error Messages

Just like the Laravel Validator, Ardent lets you set custom error messages using the [same syntax](http://laravel.com/docs/validation#custom-error-messages).

```php
class User extends \October\Rain\Database\Model
{
  public static $customMessages = [
    'required' => 'The :attribute field is required.',
    ...
  ];
}
```

#### Custom Validation Rules

You can create custom validation rules the [same way](http://laravel.com/docs/validation#custom-validation-rules) you would for the Laravel Validator.
