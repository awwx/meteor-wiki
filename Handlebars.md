
For the official docs on Handlebars syntax, see http://handlebarsjs.com/.  Meteor supports Handlebars syntax with some extensions and clarifications.

## Expressions

Here's a template containing the expression `{{foo}}`:

```
<template name="myTemplate">
  {{foo}}
</template>
```

We invoke the template on an object `data`:

```
var html = Template.myTemplate(data);
```

Then `{{foo}}` will insert the value of `data.foo`, suitably HTML-escaped.  We say that `data` is the current context object inside the template.

### Helpers

Helpers take precendence over properties, and if there is a helper named `foo` active in the template, it will shadow the property `foo`.

Helpers are attached to templates by assigning them as follows:

```
Template.myTemplate.foo = function() {
  return "blah"; // (calculate value here)
};
```

The value of `this` inside the helper is the current context where the helper was called.

This helper is only visible from `myTemplate`, and not other templates invoked inside it.  The only way to make a helper visible to multiple templates is to assign the same function to each one, or to declare a global helper.

Global helpers are lower precedence than template-bound helpers, and are declared as follows:

```
Handlebars.registerHelper("foo", function() {
  return "blah"; // (calculate value here)
});
```

## Expression Arguments

Handlebars expressions can have arguments separated by spaces, which will be passed to the helper (or method) given by the initial identifier.  Handlebars also support keyword arguments (name=value) which are passed to the helper in a hash.

XXX: How are they received as function arguments in the helper implementation?  What about in a method?

Arguments can be helpers or methods themselves, or string literals.  XXX: Anything else?

## Reserved Helper Names

Unfortunately, since `Template.myTemplate` is a function object as well as a place to bind helpers, some helper names are illegal.  For example, the name `name` is problematic because in many browsers, functions have a built-in `name` property that you can't change (it's whatever name the function was given in the source code).  When you try to assign `Template.myTemplate.name` to a function, nothing happens!

Helper names to avoid include built-in function properties like `name`, `length`, `arity`, `arguments`, and `caller`, as well as methods like `call` and `apply`.  Note that it is only helpers, not properties of the context object, that have this problem.

## Expressions with Dots

Handlebars allows expressions of the form `{{foo.bar}}`, which in the basic case means `data.foo.bar`.  With function/value coercion (see next section), it can also mean `data.foo.bar()`, `data.foo().bar`, or `data.foo().bar()`.  If there exists a helper `foo`, the helper is called instead to evalute the first segment of the expression.  Multi-segment expressions can take arguments, as in `{{foo.bar baz}}`, where `baz` will be passed as an argument to `bar` if `bar` is a function.

The expression `{{this}}` evaluates to the current data context.  Paths starting with `this` always refer to properties of the current data context and not to helpers.

You can access properties of *parent* data contexts by beginning an expression with `../`, as in `{{../foo}}` or `{{../../foo.bar}}`.  Expressions having a `..` never invoke template-bound or global helpers.

## Function/Value Coercion

Template-bound helpers can be constant values instead of functions:

```
Template.myTemplate.colors = ['red', 'green', 'blue'];
```

Also, properties of the context object can be functions, in which case they are called for their values:

```
data.foo = function() { return "blah"; };
```

In a multi-segment expression, each segment is called if it is a function.  Take `{{foo.bar.baz}}` as an example.  The identifier `foo` refers either to a helper function, a constant template property, or to a property or getter method of the current data context.  If it's a helper or a getter, the function is called with no arguments and the current data context as `this`.  We expect the result to be an object with a `bar` property.  If the `bar` property is a function (a getter method), it is called with no arguments and *the object it came from* as `this`.  This is a Meteor extension to Handlebars to support getters.

In a case like `{{../foo}}` where `foo` is a function, the value of `this` inside the function is the parent context (`..`) and not the current context, as you would expect.

## Nonexistent Identifiers

Handlebars paths that name nonexistent properties silently evaluate to the empty string, even if they are nested.  `{{abc.def.ghi}}` evaluates to `""` and doesn't fail even if there is no property or helper called `abc`.

