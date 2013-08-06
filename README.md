# nodejs-model

Okay, so you have a node app backed with some kind of NoSQL schema-less DB such as CouchDB and it all works pretty well,

But hey, even though schema-less is very cool and produces fast results, for small apps it makes sense, but while application code
grows more and more, you will end with low data integrity and things will start to become messy,


So this is what nodejs-model is for, its a super minimal, extensible model structure for node, it doesn't dictate any requirements
on the DB level, it's just a plain javascript object with some enhanced capabilities for validations and filtering.

Note: It is heavily inspired by Ruby AcitveObject Validations but is enhanced with more capabilities than validations such
as filtering and sanitization.


# Why use nodejs-model?

If one or more of the bullets below makes sense to you, then you should try nodejs-model.

* Model attributes: A lightweight javascript model with simple accessors.
* Attribute validations: define validation rules per defined attribute.
* Accessibility control: Sometimes your models may contain sensitive data (such as a 'password'/'token' attributes) and you want a simple way to filter such properties based on tags.
* Events: Events are fired when objects are being created or properties are modified.


#Installation

To install nodejs-model, use [npm](http://github.com/isaacs/npm):

```bash
$ npm install nodejs-model --save
```


# Basic Usage

This is how it works:

Create a _model_ definition with some validation rules

``` javascript
var model = require('nodejs-model');

//create a new model definition _User_ and define _name_/_password_ attributes
var User = model("User").attr('name', {
  validations: {
    presence: {
      message: 'Name is required!'
    }
  }
}).attr('password', {
  validations: {
    length: {
      minimum: 5,
      maximum: 20,
      messages: {
        tooShort: 'password is too short!',
        tooLong: 'password is too long!'
      }
    }
  },
  //this tags the accessibility as _private_
  accessibility: ['private']
});

var u1 = User.create();
//getters are generated automatically
u1.name('foo');
u1.password('password');

u1.name()
//prints _foo_

//Invoke validations and wait for the validations to fulfill
u1.validate(function() {
  if u1.isValid {
     //validated, perform business logic
  } else {
     //validation failed, dump validation errors to the console
     console.log(p1.errors)
  }
});

//get object as a plain object, ready for JSON
u1.toJSON();
//produces: { name: 'foo' }

//now also with attributes that their accessibility is 'private'
u1.toJSON('private')
//produces: { name: 'foo' } { password: 'password' }
```


Simple as that, your model is enhanced with a validate() method, simply invoke it to validate the model object
against the validation rules defined in the schema.


# Updating Model Instance

Assuming you have a simple model instance (`u1` as defined in the basic example above, you can update it with new data 
at some point after loading an object from DB / file / JSON / etc:


``` javascript
someObj = {
  name: 'bar',
  password: 'newpassword'
};

u1.update(someObj);

console.log(u1.name());
//prints bar
console.log(u1.password());
//NOTE: prints password
```

Pay attention that password wasn't updated, this is because when invoking `update(object)` only public attributes (any
attribute that its _accessibility_ metadata wasnt defined or defined as _['public']_ can be updated.

With this specific example, since _password_'s accessibility is _private_, you can update by suppling the _private_ accessibility
to the `update()` 2nd parameter as:

``` javascript
u1.update(someObj, 'private')
console.log(u1.name());
//prints bar
console.log(u1.password());
//NOTE: prints newpassword
```


# Validators

## Presence

The _Presence_ ensure that an attribute value is not null or empty string, example:

```javascript
var User = model("User").attr('name', {
  validations: {
    presence: true
  });
```

### Options

* `true` - value will be required, default message is set.
* `message` - string represents the error message if validator fails.

Example with custom message:

```javascript
validations: {
  presence: {
    message: 'Name is required!'
  }
}
```

## Length

Validates rules of the length of a property value.

### Options

* `is` keyword or `number` - An exact length
* `array` - Will expand to `minimum` and `maximum`. First element is the lower bound, second element is the upper bound.
* `allowBlank` - Validation is skipped if equal to `true` and value is empty
* `minimum` - Minimum length of the value allowed
* `maximum` - Maximum length of the value allowed

### Messages
  * `wrongLength` - any string represents the error message if `is`/`number` validation fails.
  * `tooShort` - any string represents the error message if `minimum` validation fails.
  * `tooLong` - any string represents the error message if `maximum` validation fails.

```javascript
// Examples of is, both are equal, exact 3 length match
length: 3
length: {is: 3}
//same as above, but empty string is allowed
length: { is: 3, allowBlank: true } 
//min legnth: 2, max length: 4
length: [2, 4]
//same as above with custom error messages
length: { minimum: 2, maximum: 4, messages { tooShort: 'min 3 length!', tooLong: 'max 5 length!' } }
```


# Accessibility

nodejs-model supports accessibility tagging per defined attribute, when new attribute is defined with no _accessibility_ 
array defined, it will be tagged as _public_ automatically.

Methods such as toJSON(accessibility_array) or `update(updatedObj, accessibility_array)` are accessbility aware when
updating or producing model instance output.


You can define accessibility tags per attribute by:

``` javascript
User = model("User").attr('name', {
  accessibility: ['ui', 'registered']
}).attr('password', {
  accessibility: ['private']
}).attr('age');

u1 = User.create();
u1.name('foo');
u1.password('secret');
u1.age(55);

console.log(u1.toJSON());
//prints { age: 55 }, this is because invoking toJSON(), it will only create an object with attributes defined
as public.

console.log(u1.toJSON(['ui', 'private']));
//prints { name: 'foo', password: 'secret' }

//* means any property with any accessibility
console.log(u1.toJSON('*'));
//prints { name: 'foo', password: 'secret', age: 55 }
```


Update mehtod `someInstance.update(newObj, accessibility)` is also _accessibility-aware_ as with `someInstance.toJSON(accessibility)`.

#License

See _LICENSE_ file.