# Android Themes vs Styles Part 1: Introduction to themes & styles

Android provides us with a powerful system of themes and styles, so we can make apps look exactly the way we want. This system can be hard to grasp for beginners. It's also often misused, even by more experienced Android developers. 

My goal with the *Mastering Android Themes and Styles* series is to demystify themes and styles, and to show you how to use them optimally. When used properly, they allow you to express design ideas inside your app without too much trouble. 

This, first-in-a-series article is intended for beginners in Android. That being said, I'm going to try to keep it short and clear.

## Styles

First, let's take a look at styles. What they are, how you can declare them, where, and when you should use them.

I believe it's best to learn by example, so let's start with one. It will help you answer the questions from above.

![Example button](example_button.png)

We have a simple button which is defined in the xml like this: 

~~~xml
<Button
    android:id="@+id/exampleButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|center"
    android:layout_margin="@dimen/defaultMargin"
    android:textAllCaps="false"
    android:textAppearance="?textAppearanceButton"
    android:text="Example" />
~~~

Now, let's say you want multiple buttons in your app to have the same margins, textAppearance and lower-case text. You can repeat this declaration every time, or you can extract those repeating attributes into a style and reuse that style on every button. Let's see how you can achieve this. 

### Declaring styles

~~~xml
<style name="ExampleButtonStyle" parent="Widget.MaterialComponents.Button">
    <item name="textAllCaps">false</item>
    <item name="android:layout_margin">@dimen/defaultMargin</item>
    <item name="android:textAppearance">@style/TextAppearance.MaterialComponents.Button</item>
    <item name="android:layout_gravity">bottom|center</item>
  </style>
~~~

Every style declaration begins with a `<style>` tag. Inside of it, you specify a **name** for the style and its **parent**. When you specify a parent for a style, it inherits **all** of the attributes from the parent. Inside of the style tag you can then define **view** attributes by using `<item>` tags. In each `<item>` you need to specify a name for a **view** attribute and its value.

I put **view** attributes in bold here because it's important to note that you should only use attributes which can be applied to views when defining styles. You'll see later what is the difference between theme attributes and view attributes, and why it's important to know when to use each one of them. 

### Using styles

Now you can simply apply this newly created style to every `Button` that needs to have these attributes. 

~~~xml
<Button
    android:id="@+id/exampleButton"
    style="@style/ExampleButtonStyle"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Example" />
~~~

To apply style to a view, use the `style` attribute and pass it a reference to your newly created style. It's important to note that this will only apply the style to this particular view. If  the view has children, they won't have this style applied.

## Themes

Themes are also defined using the same `<style>` tag as styles. I believe this is the main reason people often misuse themes and styles. Even though they are defined using the same syntax, there are few things in which they differ. The most significant difference is that, when defining themes you must use **theme** attributes, whereas in styles, you use **view** attributes. 

Ok, so you might be thinking now, *How should I know what are view attributes and what are theme attributes?*. Well, it's actually pretty simple. You can't apply theme attributes to views in xml.  
Another major difference is in the way themes and styles are applied. As mentioned before, when you apply style to a view, only that view gets the attributes from that specific style. But, when applying a theme, attributes get applied to the view and all of its children. 

Let's take a look how you can apply themes to your app. When you create a new project you get a theme applied by default, and it looks something like this: 

~~~xml
<!-- Base application theme. -->
  <style name="Theme.AppTheme" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
    <!-- Primary brand color. -->
    <item name="colorPrimary">@color/purple_500</item>
    <item name="colorPrimaryVariant">@color/purple_700</item>
    <item name="colorOnPrimary">@color/white</item>
    <!-- Secondary brand color. -->
    <item name="colorSecondary">@color/teal_200</item>
    <item name="colorSecondaryVariant">@color/teal_700</item>
    <item name="colorOnSecondary">@color/black</item>
    <!-- Status bar color. -->
    <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
    <!-- Customize your theme here. -->
  </style>
~~~