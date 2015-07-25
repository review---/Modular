# Modular

> A set of SCSS mixins for architecting intelligent, modular & scalable CSS.

## Overview

Modular aims to take modular CSS architecting to the next level. Similar in principle to the popular BEM convention, Modular is based off the idea of having **modules**, **components** and **modifiers**. 

Have you ever found yourself using BEM and ending up with HTML like this?

```html
<div class="button  button--large  button--success">Large Success Button</div>
```

What if you could just do this:

```html
<div class="button-large-success">Large Success Button</div>
```

The benefits of using this HTML over conventional BEM syntax are self apparant. However, you may be looking at that and be thinking of several reasons why it wouldn't work; what if I want to only use the "button" class on its own? What if I only want a large button, or only want a success button? Well, with Modular, all this is possible.

```css
@include component(button) {
	/* core button styles */
	...
	@include modifier(large) {
		/* large button styles */
		...
	}
	@include modifier(success) {
		/* success button styles */
		...
	}
}
```

The above code would allow for the use of all of the following:

```html
<div class="button">Button</div>
<div class="button-large">Large Button</div>
<div class="button-success">Success Button</div>
<div class="button-large-success">Large Success Button</div>
<div class="button-success-large">Success Large Button</div>
```

And, crazily enough, you could also use the original BEM syntax of:

```html
<div class="button  button--large  button--success">Large Success Button</div>
```

### But how?

I'm glad you asked. The answer is simple - [wildcard selectors](#). Under the hood, Modular has created a wildcard selector for the component and each modifier

*But aren't wildcard selectors bad for performance?*

Well, no. Perhaps this was true many years ago, but today, [any performance impact is negilable](http://www.telerik.com/blogs/css_tip_star_selector_not_that_bad) (and this article is 3 years old). [Further reading](http://benfrain.com/css-performance-revisited-selectors-bloat-expensive-styles/). 

*Why bother using a mixin for this? Why not just write the wildcard selector?*

For starters, writing `[class*="component-"]` over and over again can become tedious. Secondly, for our core styles we also need them to be applied to the naked `.component` class, meaning we would now have to write:

```css
.component,
[class*="component-"] {
	...
}
```

Which is exactly what the  "component" mixin does. The reason `[class*="component"]` on its own isn't used is because this can cause undesired effects elsewhere in your styles. A very simple example would be if you wanted to use a `.buttons` class in the presence on a **button** component - `[class*="button"]` would target this class and apply the core button styles to it. Using `[class*="button-"]` is a fairly safe selector in a project we have control over, in terms of potential conflicts.

### Configuring a Module

Modular allows you to create confirguble components with customizable parameters. To configure a new module, create a mixin named after your module (ensure the name is unique):

```css
@mixin header($config: ()) {
	...	
}
```

The `$config` variable is required to accept the custom options when including the mixin; the default options are defined inside the mixin:

```css
@mixin header($config: ()) {

		$header: config((
			
			/* Options */
			dark : false,
			top  : 50px
			
		), $config) !global;
		
		...
		
}
```

In the example above, we have two different types of options; a bool and a number. Typically, the "setting" mixin used in the example below would be used for options which are bools (although strictly speaking, it's used for options which are able to have a value of "false" - read further on for examples). We now have the basis for our example module. Next, the actual component itself:

```css
@mixin header($config: ()) {

		$header: config((
			
			/* Options */
			dark : false,
			top  : 50px
			
		), $config) !global;
		

		@include component(header) {
			
			// Core Styles
			margin-top: map-get($header, top);
			
			// Settings
			@include setting(dark) {
				background-color: black;
			}
			
		}
		
}
```

To call an option, the "map-get" feature of Sass is used. To create an optional setting, the "setting" mixin is used. We can now create our header with the following HTML:

```html
<div class="header">
	...
</div>
```

We can now easily create a dark header by setting the "dark" option to "true". Alternatively, we can add a modifier of "dark" to our header, regardless of the settings value:

```html
<div class="header-dark">
	...
</div>
```

## Complete Documentation

### Mixins

#### Component

The `component` mixin is what generates the selectors for your component/module. The mixin accepts 2 parameters:

* **$component** - the name of your component (required)
* **$type** - this defines how the mixin generates the selectors for your component (optional)

**$type** can be one of three values: `flex` (default), `chain` and `static`. By default, `flex` is enabled for all componenets. To globally change the default type, change the `$type` variable at the top of **modular.scss**.

##### Flex

This is the default value for a component; it creates wildcards for both `.component` and `[class*="component-"]`, allowing you to use both the naked component as well as modifiers. Whilst this is the most flexible option, it does mean the generated CSS is slightly greater, which is what the other 2 options are for.

```css
@include component(header, flex) {
	...
}
```

Or if using the default `$type` value, you do not need to pass a second parmeter here:

```css
@include component(header) {
	...
}
```

##### Chain

The chain option should be used if you are looking to optimise your CSS output, and you know your component will not exist as a naked selector without modifiers. Ie - this option outputs only `[class*="component-"]`, thefore you cannot use `.component` to achieve any styles.

```css
@include component(header, chain) {
	...
}
```

##### Static

The static option creates only the naked selector for your component; ie - `.selector`, meaning no modifiers can be used. This option is only available for consistency; it probably makes more sense to just write `.component` instead of using the mixin in this case - I'll let you think about that one.

```css
@include component(header, static) {
	...
}
```

#### Nested Component

Nested components are either components which already exist which you wish to overwrite due to their context, or sub-componenets which relate to your main component. This mixin accepts 3 parameters:

* **$nested-component** - the name of your component to be nested (required)
* **$type** - as above, this can be either `flex` (default), `chain` or `static` (optional)
* **$root** - defines whether the component should be generated outside the parent component - false by default (optional)

```css

@include component(logo) {
	font-size: 1em;	
}

@include component(header) {

	@include nested-component(logo) {
		font-size: 2em;	
	}
	
	@include nested-component(wrap-header, $root: true) {
		/* wrap-header sub-component styles */	
	}
	
}
```

```html
<div class="wrap-header">
	<div class="header">
		<div class="logo">...</div>
	</div>
</div>
```

#### Modifier

The `modifier` mixin generates the selector for any modifier of your component, for example a **small** or **large** modifier. This mixin accepts only 1 paramter:

* **$modifier** - the name of your modifier (required)


```css
@include component(button) {
	
	@include modifier(small) {
		font-size: 0.75em;
	}
	
	@include modifier(large) {
		font-size: 1.5em;
	}
	
}
```

#### Nested Modifier

The `nested-modifier` mixin is used to nest modifiers within one another, meaning that both modifiers must be passed to the element for the styles to take effect. Again, this mixin accepts only 1 parameter:

* **$modifier** - the name of your modifier (required)

_**`print`** used below isn't a real/valid property and is used for demonstrational purposes only_

```css
@include component(button) {
	
	print: "null";

	@include modifier(white) {
		print: "foo";
	}
	
	@include modifier(border) {
		print: "bar";
		@include nested-modifier(white) {
			print: "baz";
		}
	}
	
}
```

This means that in your HTML the element would require both the **border** and **white** modifiers for the styles to take place:

```html
<div class="button">null</div>
<div class="button-white">foo</div>
<div class="button-border">bar</div>
<div class="button-border-white">baz</div>
<div class="button-white-border">baz</div>
```

> If you try to nest the regular modifier mixin, it will output the CSS as if it weren't nested. It is essential to use the `nested-modifier` mixin for any nested modifiers. Other than that, modifiers can be infinitely nested.

#### Extend Modifiers

This mixin allows you to extend multiple modifiers into a new, seperate modifer, essentially combining several modifiers into one.

```css
@include component(button) {

	@include modifier(round)   {...}
	@include modifier(large)   {...}
	@include modifier(success) {...}

	@include modifier(primary) {
		@include extend(round, large, success);
	}	

}
```

```html
<div class="button-primary">...</div>
```

### Module Configuration

As outlined in the [overview](#) section, Modular allows you to configure your components with customizable options.

```css
@mixin header($config: ()) {

		$header: config((
			
			/* Options */
			dark : false,
			top  : 50px
			
		), $config) !global;
		

		@include component(header) {
			
			// Core Styles
			margin-top: map-get($header, top);
			
			// Settings
			@include setting(dark) {
				background-color: black;
			}
			
		}
		
}
```

For all intents and purposes, there are 2 types of options; bools and non-bools. A bool option is one whose value determines whether or not some code should be applied. A non-bool option is one whose value is used as a value for a CSS property. In the above example we have one of each.

#### Bool Options

If your option is a bool, you can use the `setting` mixin. The styles added within this mixin will automatically be applied to the component is the option is set to **true**. Alternatively, since by default adding a setting also creates a modifier for the setting, you can apply the styles by adding the modifier to your HTML tag, regardless of the settings value:

```html
<div class="header-dark">
	...
</div>
```

If you are watching your CSS output, you may wish to remove these modifiers (and related wildcard selectors) from the generated styles and only use them conditionally. To do so, you can pass the `exteding-settings` options to your module's config, and set it to **false**:

```css
@mixin header($config: ()) {

		$header: config((
			
			/* Options */
			extend-settings: false,
			dark : false,
			top  : 50px
			
		), $config) !global;
		
		...
		
}
```

To disable the extension of settings globally by default, set the `$extend-settings` variable in **modular.scss** to **false**. This is defined above the settings mixin.

#### Non-Bool Options

If your option is a CSS property, to call the option in your component the **map-get** function is used, like so:

```css
margin-top: map-get($header, top);
```

which will generate:

```css
margin-top: 50px;
```

#### Hybrid Options

In some cases, you may require a hybrid of the above 2 options. You may have a set of styles you wish to use conditionally, and you may wish for these styles to vary depending on the value passed. Let's look at the following example:

```css
@mixin header($config: ()) {

		$header: config((
			
			/* Options */
			side: false; // left or right
			
		), $config) !global;
		
		...
		
		@include setting(side) {
			/* Side-Header Styles */
			...
			#{map-get($header, side)}: 0;
		}
}
```