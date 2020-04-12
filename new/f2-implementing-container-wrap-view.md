# `f2` Implementing Container-Wrap-View

You can put a **view** (for example, input field) into any **container** (page, form) and if needed, it will automatically adjust using a **wrap** - wrapper element. 

{{ toc }}

## Guidelines

### Container Properties

In the parent view, define one or more **container properties** where view can be added to. if there is a single container property, name it `views`.

For each container property, define calculated property with the same child views ordered by their `sort_order` property.  

### Child View Properties

There are 2 container-related properties you can assign in child views:

* `sort_order` - affect the order in which child views are rendered
* `wrap_modifier` - add parent view-specific CSS modifiers 

Add more properties container-specific properties to the `View` class if needed. by conventions, prefix such properties with `wrap_`.

### HTML Rendering

In the parent view template, render views from the calculated property and if needed, add a wrapper element around each child view. 

### Resize Behavior

Make sure that child views look good with any width of the container. If possible, use CSS media queries and CSS grid, otherwise implement JS view-model.      

## Examples

See "`f1` Laying Out Fields In Forms".

## Why This Way

It promotes easier dropping a view anywhere and displaying it correctly provided that it doesn't alter its 

* `margin` (0, but container may modify it) 
* `width` (100%, but container may modify it)  

Previously, it was the responsibility of a child view to add CSS classes (for instance, `.page-section`) to its main HTML element. This way, without a wrap, was quite limited (for instance, `padding` of `.page-section` and the child itself conflicted).   

