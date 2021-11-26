# Hilt Dependency Injection: Getting Started
###### Manage your dependencies with ease by utilising Hilt library  

Hilt is a dependency injection library built on top of the Dagger library. Dagger is arguably the most popular and most complex tool for dependency injection on Android. Hilt brings us major improvements in terms of overal usage complexity and less boilerplate code. This makes Hilt an appealing tool to use in modern Android development.  

###Gradle dependencies

To start, you need to set up your gradle files. Add `hilt-android-gradle-plugin` to the project-level **build.gradle** file.  

~~~kotlin
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.38.1'
    }
}
~~~  

Next, apply the plugin and add the dependencies to the module-level **build.gradle** file.

~~~kotlin
plugins {
  id 'kotlin-kapt'
  id 'dagger.hilt.android.plugin'
}

android {
    ...
}

dependencies {
    implementation "com.google.dagger:hilt-android:2.38.1"
    kapt "com.google.dagger:hilt-compiler:2.38.1"
}
~~~

Final step in the gradle setup is to enable Java 8 support.  

~~~kotlin
android {
    ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
~~~

With the gradle ceremonies out of the way, you can start using Hilt in your app.

###Creating Hilt Application

First thing you need to do when integrating Hilt is to create a custom class which extends `Application`. Annotate the class with `@HiltAndroidApp` annotation. 

~~~kotlin
@HiltAndroidApp
class App : Application()
~~~

This enables Hilt code generation and creates a dependency container on the application level. 
Build your app and search for **Hilt_App.java**. It should look something like this: 

<img src=hilt_app_generated.png/>