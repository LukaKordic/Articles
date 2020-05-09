# Taming File Storage on Android

A vast majority of apps are doing some form of data management. Whether it's just loading a profile image, sharing images or video/audio files through messaging or simply storing data. Working with files on Android can be daunting. Especially if you're new to Android, or just haven't worked with it in a while.

  
When it comes to saving data on Android, we can choose from a few options:  

* SharedPreferences - most commonly used for storing app's preferences, as key-value pairs.
* Databases (Room) — for storing structured data in a private database.
* File storage — for storing all kinds of media/documents to the file system.

If you are wondering which of these options is for you, there are four simple questions in the [official documentation](https://developer.android.com/training/data-storage#top_of_page) that you can go through to figure out what type of storage you need.  
In this article though, I am going to focus on storing data to a disk using some of the APIs provided by Android.  

Topics we are going to cover include:  

* Categories of physical storage locations
* Permissions and access to storage
* Scoped storage
* Working with media content
* Working with documents and files

## Internal storage vs external storage

Android system provides us with two types of physical storage locations: internal and external storage.  
The most significant difference between them is that the internal storage is smaller than external storage on most devices. Also, the external storage might not always be available, in contrast to internal, which is always available on all devices. The system creates an internal storage directory and names it with the app's package name.  

Keeping in mind that the internal storage is always available, this makes it a reliable place to store data that your app depends on. Furthermore, if you need to save sensitive data or data that only your app can have access to, go for the internal storage.  

Two things are important to note when working with internal storage:  

* A user cannot access these files through the file manager
* Files in this folder are deleted when an app is uninstalled.

To access and store data to *app's* files in the internal storage, you can use `File` API. I wrote a simple method just for an example:  

~~~kotlin
fun writeToFile(fileName: String, contentToWrite: String) {
    val file = File(context.filesDir, fileName)
    file.writeText(contentToWrite)
  }
~~~

Alternatively, you can call `openFileOutput()` method to obtain an instance of `FileOutputStream` that can write to a file in the `filesDir` directory.  

~~~kotlin
fun writeToFile(fileName: String, contentToWrite: String) {
    context.openFileOutput(fileName, Context.MODE_PRIVATE).use {
      it.write(contentToWrite.toByteArray())
    }
  }
~~~

You would read from a file in this directory in almost the same way. You can create a `File` and call `readText()` method or you can obtain an instance of `FileInputStream` by calling `openFileInput()`.  

Here's an example:  

~~~kotlin
fun readFromFile(fileName: String): String {
    context.openFileInput(fileName).bufferedReader().useLines {
      it.fold("") { some, text ->
        return "$some \n$text"
      }
    }
    return "The file is empty!"
  }
~~~

### Okay, but when should I use external storage?

Well, as I mentioned earlier, external storage is usually larger than internal storage. So, the first thing that comes to mind is to use it for storing larger files or apps. And exactly that is the most common usage of the external storage.  
Since internal storage has limited space for **app-specific** data, it's generally a good idea to use external storage for all of the non-sensitive data your app works with. Even if they are not so large.  

Generally, data you save to this storage is persistent across app uninstallation. There is a case though, where files are going to be removed when an app is deleted. If you store **app-specific** files to the external storage by using `getExternalFilesDir()`, you will lose the files when the app is uninstalled, so be aware of that. Additionally, data stored in this directory can be accessed by other applications if they have appropriate permission.  

#### Caveats  

When it comes to working with external storage, there are few things you should do before storing data there.  

Because there are cases where a user can remove a physical volume where the external storage resides, you should first verify that the storage is available before using it. You can check the volume's state by invoking `Environment.getExternalStorageState()` method. It will return a string representing the state. For example, if the method returns `MEDIA_MOUNTED` you can safely read and write **app-specific** files withing external storage.  

Another thing you need to keep in mind is that sometimes devices can have multiple physical volumes that could contain external storage. For example, a device can allocate a partition of its internal memory as external storage, but can also provide external storage on an SD card. In that case, you need to select which one to use for your **app-specific** files.  

There's a handy method in `ContextCompat` class, called `getExternalFilesDirs()`. It returns an array of `File`'s whose first element is considered the primary external storage volume. 

## Storage permissions  

Android defines two permissions related to storage: `READ_EXTERNAL_STORAGE` and `WRITE_EXTERNAL_STORAGE`. As you can see, permissions are only defined for accessing external storage. That means that every app, by default, has permissions to access its internal storage.

On the other hand, if your app has to access external storage, you are obliged to request permission for that.
That means if you're trying to access media on external storage by using **MediaStore** API you will need to request `READ_EXTERNAL_STORAGE` permission.  

However, few exceptions to this rule exist:  

* When you are accessing **app-specific** files on external storage you don't need to request any permission (on Android 4.4 and higher).
* With **scoped storage** introduced in Android 10, you no longer need to request permission when working with media files that are created by your app. 

You also don't need any permissions if you're trying to obtain any documents or other types of content when using **Storage Access Framework**. That's because a user is involved in the process of selecting the actual content to work with.  

When defining permissions, you can set a condition to only apply it for some versions. For example:  

~~~kotlin
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                 android:maxSdkVersion="28" />
~~~

In the example above I defined `WRITE_EXTERNAL_STORAGE` permission only for version 28 and below. This can come in handy in situations where apps running newer versions can work without permission, but apps on older versions will break without it.

## Introducing Scoped storage
Android is focusing on privacy more with every release. **Application Sandboxing** is a core part of Android's design, isolating apps from each other. Following that principle, the Android team introduced **Scoped storage** in Android 10. As the team said, "it's a way to ensure transparency, give users control and secure personal data." 

From the official documentation: 
> More recent versions of Android rely more on a file's purpose than its location for determining an app's ability to access that file. This purpose-based storage model improves user privacy because apps are given access only to the areas of the device's file system that they use.   

### Why Scoped storage?
Before Android 10, apps have private, **app-specific** storage. In addition to private storage, the system provides shared storage where all of the other files are stored. The problem is that apps can access all of these files when given storage permission. From a user's perspective, this doesn't make sense. 

Let's take an example with an app that needs to provide users with an ability to select a profile image. To allow the app to access images you need to ask for storage reading permission. Given the permission, your app can now access images but also all the other files within the shared storage. There is no need for your app to be able to see every file, only to allow the user to select an image. This is one of the main reasons scoped storage has been introduced. 

### The changes      
This means that apps that target Android 10 and higher are given scoped access to external storage, or **scoped storage**, by default. With this change, apps have unrestricted access to **app-specific** storage or media that the app itself has created. But, to read other applications' media files, you need to request reading permission.  
For accessing `MediaStore.Downloads` collection files that your app didn't create, you must use **Storage Access Framework**.

If your app is not ready to adopt scoped storage yet, you have two options to opt-out of using it: 

* You can set `targetSdkVersion` to target Android 9 or lower.
* If your app targets Android 10, you can set `requestLegacyExternalStorage` flag to true in `AndroidManifest.xml`. Keep in mind that this flag is removed in Android 11.

You can read more about Android 11 storage updates [here](https://developer.android.com/preview/privacy/storage).


## Working with media content
Up until this point I talked about storing non-media files on both internal and external storage. Let me now show you how to work with media files from **app-specific** and **shared** storage.

### App-specific media
If you have to store some media files, but the files are meant for use only inside of your app, it's probably best to store them to **app-specific** directories within external storage. 

You can obtain the pictures directory using the following code: 

~~~kotlin
fun getPhotoDirFromAppStorage(dirName: String): File? {
    val file = File(context.getExternalFilesDir(Environment.DIRECTORY_PICTURES), dirName)

    if (!file.mkdirs()) {
      Log.e("TEST", "Directory not created")
    }
    return file
  }
~~~

When you're accessing these directories, you must use constants provided by the API, like `DIRECTORY_PICTURES` in the example above. If you can't find a directory name defined in the API that suits your needs, you can pass `null` into `getExternalFilesDir()`. Passing `null` returns the root app-specific directory within external storage.

### Shared storage
Whenever you want your app's data to be accessible to other apps, or you want the data to persist even after the app has been uninstalled, you should store it to a **shared storage**.

Android provides two APIs for storing and accessing shareable data. **MediaStore API** is a recommended way to go when working with media files (pictures, audio, video). If, on the other hand, you need to work with documents and other files, you should use the platform's **Storage Access Framework**.

### MediaStore API
What's a MediaStore? According to documentation, **MediaStore** is an optimized index into media collections, that allows for easier retrieving and updating media files. Interaction with the media store is done through `ContentResolver` object. You can obtain its instance from your app's context.

**MediaStore** works by defining collections for every media type. The system automatically scans a storage volume and adds media files to appropriate collection. Collections are represented as tables that you can access by calling `MediaStore.<media-type>`. 

#### Querying collections
For example, to interact with the images table you would do something like the following: 

~~~kotlin
val projection = arrayOf(
		//you only want to retrieve _ID and DISPLAY_NAME columns
        MediaStore.Images.Media._ID,
        MediaStore.Images.Media.DISPLAY_NAME)
    context.contentResolver.query(
        uri, projection, null, null, null, null)?.use { cursor ->

      //cache column indices
      val idColumn = cursor.getColumnIndex(MediaStore.Images.Media._ID)
      val nameColumn = cursor.getColumnIndex(MediaStore.Images.Media.DISPLAY_NAME)
      
		//iterating over all of the found images 
      while (cursor.moveToNext()) {
        val imageId = cursor.getString(idColumn)
        val imageName = cursor.getString(nameColumn)
      }
    }
~~~

>Notice: When you are working with [`Cursor`](https://developer.android.com/reference/kotlin/android/database/Cursor?hl=en) objects, don't forget to close them. I've used Kotlin's `use()` function to automatically close it after the code inside the block has been executed.
>Also, make sure to call `query()` method on a worker thread. 

You can use the same code if you want to interact with video or audio files. All you have to do is change `MediaStore.Images` to `MediaStore.Video` or `MediaStore.Audio`, respectively.  

The Media store also includes a collection called `MediaStore.Files`. What you will find in there depends on the version of Android you have. To be precise, if your app uses **scoped storage** (available on Android 10 and higher) this collection will show only the photos, videos, and audio files that **your** app has created. When **scoped storage** is not being used, the collection shows all types of media files.

On newer versions of Android (only Android 10 and higher), `MediaStore` API also provides you with `MediaStore.Downloads` table, where you can access downloaded files.

#### Creating a new file
If you want to create a new file and store it to one of the collections, you can easily do so using the **MediaStore** API. Here's one example of creating a new image file: 

~~~kotlin
fun createNewImageFile(): Uri? {
    val resolver = context.contentResolver

    // On API <= 28, use VOLUME_EXTERNAL instead.
    val imageCollection = MediaStore.Images.Media.getContentUri(MediaStore.VOLUME_EXTERNAL_PRIMARY)

    val newImageDetails = ContentValues().apply {
      put(MediaStore.Images.Media.DISPLAY_NAME, "New image.jpg")
    }

    //return Uri of newly created file
    return resolver.insert(imageCollection, newImageDetails)
  }
~~~

First, you obtain an instance of `ContentResolver` class. After that, you get `Uri` of a collection where you want to store the new file. To put the new file within the collection you create `ContentValues` object and call `put()` method on it, passing key-value pairs as arguments. The last step is to call `insert()` method on the previously obtained `resolver` instance.  
`insert()` method returns newly created file's **Uri** which you can use to modify the file after creation. 

#### Deleting a file
I'm sure that you've noticed a pattern in the **MediaStore** API by now. It's a very similar procedure with update or deletion as well. Let's have a look!

To delete a file you would use code similar to this: 

~~~kotlin
fun deleteMediaFile(fileName: String) {
    val fileInfo = getFileInfoFromName(fileName) //this function returns Kotlin Pair<Long, Uri>
    val id = fileInfo.first
    val uri = fileInfo.second
    val selection = "${MediaStore.Images.Media._ID} = ?"
    val selectionArgs = arrayOf(id.toString())
    val resolver = context.contentResolver
    resolver.delete(uri, selection, selectionArgs)
  }
~~~

First, you need to obtain an **id** and a **uri** of the file you want to delete. This is what `getFileInfoFromName()` function does. Just a hint, this is my helper function, it's not part of the official API. When you have the needed info, you can retrieve an instance of `ContentResolver` as before and call its `delete()` method. This method accepts three arguments: 

* **Uri** of the file.
* **WHERE** clause without actual arguments, called `selection` — specify which rows are going to be deleted.
* **Array** of arguments for `selection` parameter — provide **id** for item you want to delete.

#### Updating a file
Updating a file is done the same way, except you need to call `contentResolver.update()` method instead. There's one thing to note here, however.  
If your app uses **scoped storage** you can't simply update or delete a file that your app **didn't** create. There are some additional steps you need to do before you are allowed to do those operations. Here's an [example](https://developer.android.com/training/data-storage/shared/media#update-other-apps-files) from the official documentation how this can be done.

### Storage Access Framework
In previous chapters, I've mostly talked about working with media files by using **MediaStore** API. In this chapter you will learn about **Storage Access Framework** and how to utilize it to browse and modify documents and other files across all document storage providers.

When you're working with files by using this framework, you don't need to request any system permissions because the user is involved in selecting the files or directories that your app can access. After the user has selected the file, your app gains read and write access to a URI representing the chosen file.

#### Opening a file
Opening a file using this framework is pretty straightforward. Here's one example to demonstrate it: 

~~~kotlin
private fun openFile() {
    val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
      addCategory(Intent.CATEGORY_OPENABLE)
      type = "*/*"
    }
    startActivityForResult(intent, OPEN_DOCUMENT)
  }
~~~

This code allows users to select any file from the system's file picker app. Let's break it down line by line.  
First, you create an intent with `ACTION_OPEN_DOCUMENT` action. Then, you set a MIME type that indicates which file types your app supports. In the code above I've used `*/*`, which means I want to show all files. If for example, you want to show only images, you use `images/*`, or for pdfs `application/pdf`. Besides the type, you should add a category for files. In this case, I chose `Intent.CATEGORY_OPENABLE`. This will show only files that can be opened with `ContentResolver.openFileDescriptor()` method.  
When you have prepared your intent, call `startActivityForResult()` and pass in the intent with a unique request code. 

Try to run the app and check what happens. You should see a screen similar to this one:

<img src="Open document.png" style="display:block; margin-left:auto; margin-right:auto; width:260px; height:500px"/>

#### Creating a new file
The process of creating a new file is very similar to the one for opening. Use `ACTION_CREATE_DOCUMENT` action to create a new intent.

~~~kotlin
private fun createFile(name: String) {
    val intent = Intent(Intent.ACTION_CREATE_DOCUMENT).apply {
      addCategory(Intent.CATEGORY_OPENABLE)
      type = "application/pdf"
      putExtra(Intent.EXTRA_TITLE, name)
    }
    startActivityForResult(intent, CREATE_DOCUMENT)
  }
~~~

As with opening, you need to add a category.  
In this case, it's `Intent.CATEGORY_OPENABLE`. Then set the MIME type for the file you want to create. If you would like to add a title for a file, you can do so by using `Intent.EXTRA_TITLE` intent extra. One thing to note here is that this action cannot overwrite an existing file. In case you put the same name, the system appends a number at the end of the file name. 

Try running the code. You should see something like this:

<img src="Create document.png" style="display:block; margin-left:auto; margin-right:auto; width:260px; height:500px"/>

#### Granting access to a directory
If for some reason, your app needs to get access to the contents of a directory, you can use `ACTION_OPEN_DOCUMENT_TREE` intent action. By using this, the user can grant your app access to the entire directory tree. Your app can then access any file in the directory and its subdirectories.  
It's worth noting that your app doesn't have access to other apps' files outside of the user-selected directory. 

#### Getting the result back
For each of these actions, you need to call `startActivityForResult()` passing in intent with the appropriate action. When the user has finished selecting the file or a directory, you will get the result in `onActivityResult()` callback. You get selected file's **Uri** within the intent's data property. You can then use this **Uri** to make modifications to the file.  

I will explain this in more detail later, but before making any modifications to the file, you should check the value of `DocumentsContract.Document.COLUMN_FLAGS`. It indicates which operations on the given file are supported by the provider.

Here's the code you can use to obtain the **Uri**:

~~~kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == OPEN_DOCUMENT && resultCode == Activity.RESULT_OK) {
      data?.data?.let { uri ->
        //use this uri to make supported modifications to the file
      }
    }
  }
~~~

#### Under the hood
**Storage Access Framework** works with content providers under the hood. Main parts of the framework are: 

* **Documents provider** — A provider that allows other apps to reveal the files they manage. This provider can be implemented by both local and cloud storage services. Android provides you with **Downloads**, **Images** and **Videos** built-in documents providers. To learn more about documents provider check out this [link](https://developer.android.com/reference/android/provider/DocumentsProvider). 
* **Client app** — This is an app that invokes some of the intent actions we've mentioned above, and receives selected files.  
* **Picker** — This is the UI that lets users select files from all providers that satisfy the search criteria defined in the client's app.

Every app that wants to contribute their files to the system picker UI has to implement its own documents provider.  
Now, to go back to the statement from the last chapter where I said that you need to check `DocumentsContract.Document.COLUMN_FLAGS` before trying to do any modifications to a file. When apps implement the provider, they can set different capabilities for each document with the aforementioned flags. For example, the provider can set `Document.FLAG_SUPPORTS_REMOVE` to indicate that you can delete this document.

### Conclusion
File storage in Android is a broad topic for sure, and can't really fit into a single post. So I'm going to stop here because this one is already a bit too long.  
I've put some additional resources below, so make sure to check them out for more details.

It can be really hard to start working with file storage on Android. There are quite a few things to consider when doing any kind of file management. I encourage you to take these examples and write them yourself (don't just copy-paste them). Also, run the app and see what's happening. Once you try it out, it will become much more clear how this stuff works.  

I hope that you learned something from this post and that you find it interesting. If you have any questions or suggestions for improvement, please leave a comment below. 

Thank you for reading! 

### Additional resources

* [Android 11 privacy changes](https://developer.android.com/preview/privacy)
* [GitHub repo with examples from Google](https://github.com/android/storage-samples)
* [Different operations on files using SAF](https://developer.android.com/training/data-storage/shared/documents-files#perform-operations)
