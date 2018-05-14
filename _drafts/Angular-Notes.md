---
tags: [Angular, Notes, CheetSheet]
title: Angular Notes
---

## Overview ##

![Angular overview]({{ site.url }}{{ site.baseurl }}/assets/images/posts/Angular-Notes/ngXOverview.jpg){: .align-center}

The main building blocks of an Angular application

* Modules
* Components
* Directives
* Routing
* Services
* Pipes
* Templates (Interpolation and expressions)
* Data Binding
* Metadata (Decorators)
* Dependency Injection

## Components ##

A Component controls a patch of screen real estate that we could call a view.
Each Component is composed of three parts: Component Decorator, View (Template) and a Controller (The Class).

We define a Component's application logic - what it does to support the view - inside a class. The class interacts with the view through an API of properties and methods.

![Angular Component Structure]({{ site.url }}{{ site.baseurl }}/assets/images/posts/Angular-Notes/ngXComponent.jpg){: .align-center}

### Component Lifecycle ###

Directive and component instances have a lifecycle as Angular creates, updates, and destroys them. Developers can tap into key moments in that lifecycle by implementing one or more of the "Lifecycle Hook" interfaces, all of them available in the angular2/core library.

Here is the complete lifecycle hook interface inventory:

* OnInit
* OnDestroy
* DoCheck
* OnChanges
* AfterContentInit
* AfterContentChecked
* AfterViewInit
* AfterViewChecked

No directive or component will implement all of them and some of them only make sense for components.
Each interface has a single hook method whose name is the interface name prefixed with ng. For example, the OnInit interface has a hook method named ngOnInit.

Angular calls these hook methods in the following order:

* ngOnChanges - called when an input or output binding value changes
* ngOnInit - after the first ngOnChanges
* ngDoCheck - developer's custom change detection
* ngAfterContentInit - after component content initialized
* ngAfterContentChecked - after every check of component content
* ngAfterViewInit - after component's view(s) are initialized
* ngAfterViewChecked - after every check of a component's view(s)
* ngOnDestroy - just before the directive is destroyed.

Angular apps are built by composing multiple components essentially building a component tree.

![Angular Component Tree]({{ site.url }}{{ site.baseurl }}/assets/images/posts/Angular-Notes/ngxCompTree.jpg){: .align-center}

## Directives ##

Attach behaviour, extend, or transform a particular element or its children.

There are three kinds of directives in Angular:

* Components — directives with a template.

* Structural directives — change the DOM layout by adding and removing DOM elements.

* Attribute directives — change the appearance or behavior of an element, component, or another directive.

Components are the most common of the three directives. You saw a component for the first time in the QuickStart guide.

Structural Directives change the structure of the view. Two examples are NgFor and NgIf. Learn about them in the Structural Directives guide.

Attribute directives are used as attributes of elements. The built-in NgStyle directive in the Template Syntax guide, for example, can change several element styles at the same time.

* NgIf

* NgSwitch

```html
<div class="container" [ngSwitch]="myVar">
  <div *ngSwitchCase="'A'">Var is A</div>
  <div *ngSwitchCase="'B'">Var is B</div>
  <div *ngSwitchDefault>Var is something else</div>
</div>
```

* NgStyle

with this directive you can set a given DOM element CSS properties from Angular
expressions.The simplest way to use this directive is by doing `[style.<cssproperty>]="value"` or by setting fixed values is by using the NgStyle attribute and using key value pairs for each property you want to set.

```html
<div [style.background-color]="'yellow'">
  Uses fixed yellow background
</div>

<div [ngStyle]="{color: 'white', backgroundColor: 'blue'}">
  Uses fixed white text on blue background
</div>
```

* NgClass

```html
<div [ngClass]="{bordered: true}">This is always bordered</div>

<div class="base" [ngClass]="['blue', 'round']">
  This will always have a blue background and round corners.
</div>
```

* NgFor

The role of this directive is to repeat a given DOM element (or a collection of DOM elements) and
pass an element of the array on each iteration.

The syntax is `*ngFor="let item of items"`.

• The let item syntax specifies a (template) variable that’s receiving each element of the items;
• The items is the collection of items from your controller.

* NgNonBindable

We use ngNonBindable when we want tell Angular not to compile or bind a particular section of our page.

```html
<span class="bordered">{% raw %}{{ content }}{% endraw %}</span>
<span class="pre" ngNonBindable>This is what {% raw %}{{ content }}{% endraw %} rendered</span>
```

## Services ##

"Service" is a broad category encompassing any value, function or feature that our application needs. Almost anything can be a service.

 A service is typically a class with a narrow, well-defined purpose. 
 It should do something specific and do it well, such as Data layer, API requests and other logic that is not component related.

Examples include:

* logging service
* data service
* message bus
* tax calculator
* application configuration

There is nothing specifically Angular about services.
Angular itself has no definition of a service.
There is no ServiceBase class.
Yet services are fundamental to any Angular application.

## Routing & Navigation ##

Renders a bcomponent based on the URL state, drives navigation.

## Forms ##

* FormsModule – Template Driven Forms

* ReactiveFormsModule – Code Driven Forms

FormsModule gives us template driven directives such as:

* ngModel
* NgForm

ReactiveFormsModule gives us directives like:

* formControl
* ngFormGroup

### FormControl ###

To build up forms we create FormControls (and groups of FormControls) and then attach metadata and logic to them.

```typescript
 let nameControl = new FormControl("Nate");

 let name = nameControl.value; // -> Nate

 // now we can query this control for certain values:
 nameControl.errors // -> StringMap<string, any> of errors
 nameControl.dirty // -> false
 nameControl.valid // -> true
```

Template part:

```html
<input type="text" [formControl]="name" />
```

### FormGroup ###

Most forms have more than one field, so we need a way to manage multiple FormControls.
FormGroups solve this issue by providing a wrapper interface around a collection of FormControls.

```typescript
let personInfo = new FormGroup({
  firstName: new FormControl("Nate"),
  lastName: new FormControl("Murray"),
  zip: new FormControl("90210")
})
```

FormGroup and FormControl have a common ancestor AbstractControl.

```html
<form #f="ngForm" (ngSubmit)="onSubmit(f.value)" >
  <input type="text" id="skuInput" placeholder="SKU" name="sku" ngModel >
  <button type="submit">Submit</button>
</form>
```

If you import FormsModule, NgForm will get automatically attached to any `<form>` tags you have in your view. This is really useful but potentially confusing because it happens behind the scenes.
We bind to the ngSubmit action of our form by using the syntax: `(ngSubmit)="onSubmit(f.value)"`.

* (ngSubmit) - comes from NgForm
* onSubmit() - will be implemented in our component definition class (below)
* f.value - f is the FormGroup that we specified above. And .value will return the key/value pairs

### FormBuilder ###

Helper class that helps us build forms.
As you recall, forms are made up of FormControls and FormGroups and the FormBuilder helps us make them (you can think of it as a "factory" object).

```html
  <form [formGroup]="myForm">
```

Earlier we said that when using FormsModule that NgForm will be automatically applied to a `<form>` element? There is an exception: NgForm won’t be applied to a `<form>` that has formGroup.

When we want to bind an existing FormControl to an input we use formControl:

```html
  <input type="text" id="skuInput" placeholder="SKU" [formControl]="myForm.controls['sku']">
```

Remember:

* To create a new FormGroup and FormControls implicitly use:
  * ngForm
  * ngModel
* But to bind to an existing FormGroup and FormControls use:
  * formGroup
  * formControl

## Templates ##

We define a Component's view with its companion template.

A template is a form of HTML that tells Angular how to render the Component

The easiest way to display a component property is to bind the property name through interpolation.
With interpolation, you put the property name in the view template, enclosed in double curly braces: {{myHero}}.

notes :

* _The hash sign **#** in a template ... create local variable you can use in the template, this is called a template ref variable_

* _The **safe navigation operator ( ?. )** and null property paths_

  * The Angular safe navigation operator (?.) is a fluent and convenient way to guard against null and undefined values in property paths.

  * Here it is, protecting against a view render failure if the currentHero is null.

  ```html
    <p>The current hero's name is {% raw %}{{currentHero?.name}}{% endraw %}</p>
  ```

### Property binding ###

### Event binding ###

## Pipes ##

Sometimes the raw data is not what we want to display in the view. We often want to transform them, filter them, limit their number etc. for this we can use Pipes.

A pipe can be used either in HTML or in your applicative code.

How do we create a pipe?

First we need to create a new class.
It should implement the PipeTransform interface, which forces us to have a transform() method, the one doing the heavy lifting.
Next, we need to register the pipe in our app.
For this, there is a special decorator we can use: `@Pipe`

```typescript
import { PipeTransform, Pipe } from '@angular/core';

@Pipe({ name: 'toUpperCase' })
export class FromNowPipe implements PipeTransform {
  transform(value, args) {
    // do something here
    return value.toUpperCase();
  }
}
```

The chosen name will be the one allowing to use the pipe in the template.

To use the pipe in a template, the last thing you need to do is to add the pipe to the `declarations` of your `@NgModule`.

---
#### To Explore ####

* ngrx (Redux)
* Immutability