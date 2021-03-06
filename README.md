backbone-forms
==============

A form framework for Backbone.JS applications.

The following default editors are included:

- Text
- Number
- Password
- TextArea
- Select
- Object
- NestedModel

In addition there is a separate file with editors that depend on jQuery UI:

- Date
- DateTime
- List (Editable and sortable. Can use any of the other editors for each item)


Installation
============

Requires BackboneJS and jQuery.

Include backbone-forms.js and backbone-forms.css:

    <link href="backbone-forms/backbone-forms.css" rel="stylesheet" type="text/css"/> 
    <script src="backbone-forms/src/backbone-forms.js"></script>

Optionally, you can include the extra editors, for example those that require jQuery UI:

    <script src="backbone-forms/src/jquery-ui-editors.js"></script>
    
    
Usage
=====

![Example form](http://i56.tinypic.com/a3zfyt.png)

You can create something like the form above with the following steps:

Define a 'schema' attribute on your Backbone models. The schema keys should match the attributes that get set on the model. Note that `type` defaults to `Text`.

    var User = Backbone.Model.extend({
        schema: {
            start:      { type: 'DateTime' },
            contact:    { type: 'Object', subSchema: {
                            name: {},
                            phone: {}
                        }}
            address:    { type: 'NestedModel', model: Address },
            notes:      { type: 'List' }
        }
    });

Create the form in your Views:
    
    var formView = Backbone.View.extend({
        render: function() {
            var form = new Backbone.Form({
                model: users.get(userId)
            }).render();
            
            $(this.el).append(form.el);
            
            return this;
        }
    });


Once the user is done with the form, call commit() to apply the updated values to the model. If there are validation errors they will be returned:

    var errors = form.commit();

To update a field after the form has been rendered, use `setValue`:

    model.bind('change:name', function(model, name) {
        form.fields.name.setValue(name);
    });


Usage without models
--------------------

You can create a form without tying it to a model. For example, to create a form for a simple object of data:

    var form = new Backbone.Form({
        data: { id: 123, name: 'Rod Kimble', password: 'cool beans' }, //Data to populate the form with
        schema: {
            id:         { type: 'Number' },
            name:       {},
            password:   { type: 'Password' }
        }
    }).render();

Then instead of form.commit(), do:
    
    var data = form.getValue(); //Returns object with new form values


Schema definition
=================

Main attributes
---------------

For each field definition in the schema you can use the following optional attributes:

**`type`**

- The editor to use in the field
- Can be a string for any editor that has been added to Backbone.Form.editors, such as the built-in editors. E.g.: `{ type: 'TextArea' }`
- Or can be a constructor function, e.g. for a custom editor: `{ type: MyEditor }`
- If not defined, defaults to 'Text'

**`title`**

- Defines the text that appears in a form field's &lt;label&gt;
- If not defined, defaults to a formatted version of the camelCased field key. E.g. `firstName` becomes `First Name`. This behaviour can be changed by assigning your own function to Backbone.Form.helpers.keyToTitle.


Editor-specific attributes
--------------------------

If the schema `type` is one of the following, some extra schema attributes are required:

Select
------

Creates and populates a &lt;select&gt; element.

**`options`**

- Options to populate the &lt;select&gt;
- Can be either:
    - String of HTML &lt;option&gt;`s
    - Array of strings/numbers
    - Array of objects in the form `{ val: 123, label: 'Text' }`
    - A Backbone collection
    - A function that calls back with one of the above 

Examples:
    
    var schema = {
        country: { 'Select', options: new CountryCollection() }
    };
    
    var schema = {
        users: { 'Select', options: function(callback) {
            users = db.getUsers();
            
            callback(users);
        }}
    }

**Backbone collection notes**

If using a Backbone collection as the `option` attribute, models in the collection must implement a `toString()` method. This populates the label of the &lt;option&gt;. The ID of the model populates the `value` attribute.

If there are no models in the collection, it will be `fetch()`ed.


Object
------

The Object editor creates an embedded child form representing a Javascript object.

**`subSchema`**

- A schema object which defines the field schema for each attribute in the object

Examples:

    var schema = {
        address: { type: 'Object', subSchema: {
            street: {},
            zip: { type: 'Number' },
            country: { 'Select', options: countries }
        }}
    };


NestedModel
-----------

Used to embed models within models.  Similar to the Object editor, but adds validation of the child form (if it is defined on the model), and keeps your schema cleaner.

**`model`**

- A reference to the constructor function for your nested model
- The referenced model must have it's own `schema` attribute

Examples:

    var schema = {
        address: { type: 'NestedModel', model: Address }
    };
    

List
----

Creates a sortable and editable list of items, which can be any of the above schema types, e.g. Object, Number, Text etc. Currently requires jQuery UI for creating dialogs etc.

**`listType`**

- Defines the editor that will be used for each item in the list.
- Similar in use to the main 'type' schema attribute.
- Defaults to 'Text'

**`itemToString`**

- Optional, but recommended when using listType 'Object'
- A function that returns a string representing how the object should be displayed in a list item.
- When listType is 'NestedModel', the model's `toString()` method will be used, unless a specific `itemToString()` function is defined on the schema.

**`sortable`**

- Optional. Set to false to disable drag and drop sorting


Examples:
    
    var schema = {
        users: { type: 'List', listType: 'Object', itemToString: function(user) {
                return user.firstName + ' ' + user.lastName;
            }
        }
    };


Date
----

Creates a jQuery UI datepicker

DateTime
--------

Creates a jQuery UI datepicker and time select field.

**`minsInterval`**

- Optional. Controls the numbers in the minutes dropdown. Defaults to 15, so it is populated with 0, 15, 30, and 45 minutes;



Form options
============

**`model`**

The model to tie the form to. Calling `form.commit()` will update the model with new values.

**`data`**

If not using the `model` option, pass a native object through the `data` option. Then use `form.getValue()` to get the new values.

**`schema`**

The schema to use to create the form. Pass it in if you don't want to store the schema on the model, or to override the model schema.

**`fields`**

An array of field names (keys). Only the fields defined here will be added to the form. You can also use this to re-order the fields.

**`idPrefix`**

A string that will be prefixed to the form DOM element IDs. Useful if you will have multiple forms on the same page. E.g. `idPrefix: 'user-'` will result in IDs like 'user-name', 'user-email', etc.



Editors without forms
=====================

You can add editors by themselves, without being part of a form. For example: 

    var select = new Backbone.Form.editors.Select({
        model: user,
        key: 'country',
        options: getCountries()
    }).render();
    
    //When done, apply selection to model:
    select.commit();


Custom Editors
==============

Custom editors can be written. They must extend from Backbone.Form.editors.Base.
    
    var CustomEditor = Backbone.Form.editors.Base.extend({
        
        tagName: 'input',
        
        initialize: function(options) {
            //Call parent constructor
            Backbone.Form.editors.Base.prototype.initialize.call(this, options);
            
            //Custom setup code.
            if (this.schema.customParam) this.doSomething();
        },
        
        render: function() {
            this.setValue(this.value);
            
            return this;
        },
        
        getValue: function() {
            return $(this.el).val();
        },
        
        setValue: function(value) {
            $(this.el).val(this.value);
        }
        
    });

**Notes:**

- The editor must implement a getValue() and setValue().
- The original value is available through this.value.
- The field schema can be accessed via this.schema. This allows you to pass in custom parameters.


Defaults & Validation
=====================

Formal uses the built in Backbone validation and defaults as defined on the model.

For validation, it will attempt to update the model and if there are validation failures, it will report them back.

See the Backbone documentation for more details.


Known issues
============

- List editor with listType NestedModel doesn't run validation
- There may be CSS issues across browsers.  You can customise your CSS by editing the backbone-forms.css file.

Contributors
============

- Charles Davison - [powmedia](http://github.com/powmedia)
