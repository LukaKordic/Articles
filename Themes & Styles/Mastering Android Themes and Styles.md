# Mastering Android Themes and Styles


- themes and styles are technically the same thing. Both declared using the \<style\> tag  
- difference is how they are used  
- key—value pairs

## Attributes

- keys
- defines format types
	- resource types
	- special types
- \<attr\>
- custom attributes
- may be conflicts with naming - global attrs

## Themes

- tied to context
- get/set (apply on top of the theme that's already set in the activity, applies to hierarchy)
- manifest

### What's inside of themes?

- default widget styles
- color values
- text appearance
- window config values
- drawables

## Styles

- example of using a style
- order of application

## Inheritance of styles

- dot notation
- parent attribute (has priority)

## Referencing values from the theme

- ?android:attr/colorPrimary
- ? - theme lookupw
- android: - namespace
- attr/ - looking for an attribute (can be omitted)
- separating declaration from providing a value

## Theme overlays

- applying theme overlay through code

## Color State Lists

- single item csls (to avoid multiple instances of the same color with different alpha values)

## Drawable and View tinting