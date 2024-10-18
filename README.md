# ExceptionsInAndroid
Crashes and exceptions in Android occur when the app encounters unexpected conditions that it cannot handle, resulting in the app terminating abruptly. Each crash type is tied to a specific programming error or mismanagement of app resources. Here’s an overview of common crash types and exceptions, why they happen, and how to prevent them.

### 1. **NullPointerException (NPE)**

#### **What Happens:**
A `NullPointerException` occurs when your code tries to access an object or call a method on an object that has not been initialized (i.e., it is `null`).

#### **Why It Happens:**
- Trying to invoke a method or access a field on an object that is `null`.
- Improper lifecycle management where views or resources are accessed before they are created or after they are destroyed.

#### **Example:**
```kotlin
val textView: TextView? = null
textView.text = "Hello, world!"  // Causes NullPointerException
```

#### **Prevention:**
- Always check for `null` before accessing an object:
  ```kotlin
  textView?.text = "Hello, world!"  // Safe access
  ```
- Use Kotlin’s `null-safety` features or make use of `lateinit` for initialization when certain that a value will be provided later:
  ```kotlin
  lateinit var textView: TextView
  ```

---

### 2. **IllegalStateException**

#### **What Happens:**
An `IllegalStateException` occurs when the app is in an inappropriate state for the requested operation. For example, performing an action before an `Activity` or `Fragment` is in a valid state.

#### **Why It Happens:**
- Calling methods on a `Fragment` or `Activity` before they are fully initialized or after they are destroyed.
- Changing the state of UI components (like `FragmentTransaction`) in an invalid lifecycle state.

#### **Example:**
- Accessing the `FragmentManager` after the `Activity` has been destroyed:
  ```kotlin
  fragmentManager.beginTransaction().commit()  // Causes IllegalStateException if the activity is destroyed
  ```

#### **Prevention:**
- **Use `isAdded` or `isVisible` for Fragments**: Always check whether a `Fragment` is attached before interacting with it:
  ```kotlin
  if (fragment.isAdded) {
      // Safe to interact with fragment
  }
  ```
- **Avoid UI operations after `onPause` or `onDestroy`:**
  Ensure you are not updating UI elements or running transactions once the `Fragment` or `Activity` is in the destroyed state.
  
  - **Proper Fragment Context**: Fragments should always be attached to a valid `Context` or `Activity`:
    ```kotlin
    override fun onAttach(context: Context) {
        super.onAttach(context)
        if (context is MainActivity) {
            // Safe to proceed
        } else {
            throw IllegalStateException("Context is not MainActivity")
        }
    }
    ```

---

### 3. **ArrayIndexOutOfBoundsException**

#### **What Happens:**
An `ArrayIndexOutOfBoundsException` occurs when your code tries to access an invalid index in an array or list. 

#### **Why It Happens:**
- Accessing an index that is less than 0 or greater than or equal to the size of the array/list.

#### **Example:**
```kotlin
val numbers = arrayOf(1, 2, 3)
val number = numbers[3]  // Causes ArrayIndexOutOfBoundsException
```

#### **Prevention:**
- Always check that the index is within bounds before accessing an array element:
  ```kotlin
  if (index >= 0 && index < numbers.size) {
      val number = numbers[index]
  }
  ```

---

### 4. **ClassCastException**

#### **What Happens:**
A `ClassCastException` occurs when your code attempts to cast an object to a class or interface that it is not an instance of.

#### **Why It Happens:**
- Incorrect casting of types, such as casting a `String` to an `Integer`, or casting a `Fragment` to the wrong type.

#### **Example:**
```kotlin
val obj: Any = "Hello"
val number = obj as Int  // Causes ClassCastException
```

#### **Prevention:**
- Use `is` and `as?` to safely check and cast types in Kotlin:
  ```kotlin
  if (obj is Int) {
      val number = obj
  }
  ```

---

### 5. **IllegalArgumentException**

#### **What Happens:**
An `IllegalArgumentException` is thrown when a method receives an argument that it doesn’t expect or that is not valid.

#### **Why It Happens:**
- Passing `null` or an invalid value to a method that doesn’t expect it.
- Invalid parameters during method calls, such as negative values in ranges that only accept positive numbers.

#### **Example:**
```kotlin
fun setAge(age: Int) {
    if (age < 0) {
        throw IllegalArgumentException("Age cannot be negative")
    }
}
```

#### **Prevention:**
- Validate input parameters before using them:
  ```kotlin
  fun setAge(age: Int) {
      require(age >= 0) { "Age cannot be negative" }
  }
  ```

---

### 6. **Resources.NotFoundException**

#### **What Happens:**
A `Resources.NotFoundException` occurs when your app attempts to access a resource (e.g., drawable, string, etc.) that does not exist.

#### **Why It Happens:**
- Trying to access a resource with an invalid resource ID.
- Resource files (e.g., strings, images) missing in certain configurations (like different locales or screen densities).

#### **Example:**
```kotlin
val drawable = getResources().getDrawable(R.drawable.nonexistent)  // Causes Resources.NotFoundException
```

#### **Prevention:**
- Ensure that resource IDs exist for the configuration being used.
- Use `getDrawable()` or other resource methods safely:
  ```kotlin
  val drawable = resources.getDrawable(R.drawable.icon, theme)
  ```

---

### 7. **ActivityNotFoundException**

#### **What Happens:**
An `ActivityNotFoundException` occurs when the app tries to launch an `Activity` that is not declared in the manifest or doesn’t exist.

#### **Why It Happens:**
- Trying to start an `Activity` with an incorrect `Intent` or when the target `Activity` is not registered in the `AndroidManifest.xml`.

#### **Example:**
```kotlin
val intent = Intent(this, NonExistentActivity::class.java)
startActivity(intent)  // Causes ActivityNotFoundException
```

#### **Prevention:**
- Double-check the target `Activity` is declared in the `AndroidManifest.xml`.
- Validate the `Intent` before starting the `Activity`:
  ```kotlin
  if (intent.resolveActivity(packageManager) != null) {
      startActivity(intent)
  }
  ```

---

### 8. **SecurityException**

#### **What Happens:**
A `SecurityException` occurs when your app doesn’t have the required permission to perform a certain action.

#### **Why It Happens:**
- Trying to access sensitive operations (e.g., accessing the camera, location, or file system) without requesting the necessary permissions in the manifest or during runtime.

#### **Example:**
```kotlin
val locationManager = getSystemService(Context.LOCATION_SERVICE) as LocationManager
locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0L, 0f, locationListener)  // Causes SecurityException if location permission is missing
```

#### **Prevention:**
- Declare necessary permissions in `AndroidManifest.xml`:
  ```xml
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  ```
- Request permissions at runtime (starting from Android 6.0):
  ```kotlin
  ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), REQUEST_CODE)
  ```

---

### 9. **NetworkOnMainThreadException**

#### **What Happens:**
A `NetworkOnMainThreadException` occurs when a network request is made on the main (UI) thread, which can freeze or crash the app.

#### **Why It Happens:**
- Performing long-running network operations (like downloading data or calling APIs) on the main thread, causing the app to freeze or become unresponsive.

#### **Example:**
```kotlin
val url = URL("https://example.com")
val response = url.openConnection().inputStream.bufferedReader().use { it.readText() }  // Causes NetworkOnMainThreadException
```

#### **Prevention:**
- Always perform network operations on background threads using `AsyncTask`, `Coroutines`, or `RxJava`:
  ```kotlin
  CoroutineScope(Dispatchers.IO).launch {
      val response = url.openConnection().inputStream.bufferedReader().use { it.readText() }
  }
  ```

---

### General Best Practices to Prevent Crashes

1. **Follow Lifecycle Methods Properly**:
   - Use `onStart`, `onResume`, `onPause`, `onDestroy` properly to manage resources, and avoid performing long-running tasks in methods like `onCreate`.

2. **Use Kotlin's Null Safety Features**:
   - Kotlin’s `null` safety with `?` and `!!` helps reduce the chances of `NullPointerException`.

3. **Avoid Memory Leaks**:
   - Use weak references, and avoid retaining `Activity` or `Context` references in static variables or long-running tasks.



4. **Handle Permissions Correctly**:
   - Always request the necessary permissions in both the manifest and at runtime.


Here’s a comprehensive list of common exceptions in Android, including a brief description of each and general prevention tips.

### 1. **NullPointerException**
- **Description**: Occurs when an app tries to access an object reference that is `null`.
- **Prevention**: Use null checks or Kotlin's null-safety features (`?.`, `?:`).

---

### 2. **ArrayIndexOutOfBoundsException**
- **Description**: Occurs when trying to access an array element with an invalid index.
- **Prevention**: Always ensure the index is within the array's bounds.

---

### 3. **IllegalArgumentException**
- **Description**: Occurs when an illegal argument is passed to a method.
- **Prevention**: Validate arguments before passing them to methods.

---

### 4. **IllegalStateException**
- **Description**: Thrown when a method is invoked at an inappropriate or invalid state.
- **Prevention**: Ensure that the component (e.g., `Fragment`, `Activity`) is in a valid state before performing actions.

---

### 5. **ClassCastException**
- **Description**: Occurs when trying to cast an object to a class it doesn't belong to.
- **Prevention**: Use `is` for type checks in Kotlin or `instanceof` in Java before casting.

---

### 6. **Resources.NotFoundException**
- **Description**: Thrown when a requested resource (e.g., drawable, string) is missing.
- **Prevention**: Double-check resource IDs and ensure they exist for all configurations.

---

### 7. **ActivityNotFoundException**
- **Description**: Occurs when an `Intent` targets an `Activity` that doesn’t exist.
- **Prevention**: Ensure the target `Activity` is declared in the manifest.

---

### 8. **SecurityException**
- **Description**: Thrown when the app tries to perform an action without the required permissions.
- **Prevention**: Declare necessary permissions in the manifest and request them at runtime.

---

### 9. **NetworkOnMainThreadException**
- **Description**: Occurs when network operations are done on the main thread.
- **Prevention**: Perform network operations on a background thread (`Coroutines`, `RxJava`, etc.).

---

### 10. **FileNotFoundException**
- **Description**: Thrown when a file specified in the code cannot be found.
- **Prevention**: Ensure the file exists at the given path before accessing it.

---

### 11. **IOException**
- **Description**: Occurs during I/O operations when reading or writing fails.
- **Prevention**: Use proper error handling (try-catch) for file operations.

---

### 12. **OutOfMemoryError**
- **Description**: Occurs when the app runs out of memory.
- **Prevention**: Optimize memory usage and ensure large resources (like bitmaps) are properly managed and recycled.

---

### 13. **SocketTimeoutException**
- **Description**: Thrown when a timeout happens during socket connections.
- **Prevention**: Set appropriate timeout values and handle timeout errors.

---

### 14. **MalformedURLException**
- **Description**: Thrown when a URL is not formatted correctly.
- **Prevention**: Validate the URL string format before using it.

---

### 15. **NoClassDefFoundError**
- **Description**: Occurs when the class required at runtime is not found.
- **Prevention**: Ensure that all classes are included in the project build.

---

### 16. **ClassNotFoundException**
- **Description**: Thrown when the JVM cannot find a class that it needs.
- **Prevention**: Ensure that the required class exists and is included in the app’s classpath.

---

### 17. **PackageManager.NameNotFoundException**
- **Description**: Occurs when a package or component is not found.
- **Prevention**: Ensure the package name or component exists before attempting to reference it.

---

### 18. **SecurityException**
- **Description**: Occurs when performing operations that require specific security permissions.
- **Prevention**: Check and request proper permissions in the manifest and runtime.

---

### 19. **SQLiteException**
- **Description**: Thrown when there is an error in SQL queries or the database.
- **Prevention**: Ensure the SQL syntax is correct, and the database is properly created.

---

### 20. **CursorIndexOutOfBoundsException**
- **Description**: Thrown when trying to access a cursor position that is out of bounds.
- **Prevention**: Ensure that cursor operations remain within the valid index range.

---

### 21. **SQLiteDiskIOException**
- **Description**: Occurs when there is an issue accessing the database file on disk.
- **Prevention**: Ensure that the storage medium is accessible and the database file is intact.

---

### 22. **SQLiteFullException**
- **Description**: Thrown when the SQLite database is full.
- **Prevention**: Clean up old data regularly and monitor database size.

---

### 23. **WindowManager.BadTokenException**
- **Description**: Occurs when trying to display a dialog or window using an invalid token.
- **Prevention**: Ensure the window is attached to a valid context and state before displaying.

---

### 24. **UnsupportedOperationException**
- **Description**: Thrown when an unsupported operation is invoked.
- **Prevention**: Avoid invoking methods or operations that are not supported by the current object or system version.

---

### 25. **InflateException**
- **Description**: Thrown when there is an error inflating a view layout.
- **Prevention**: Ensure that XML layout files are well-formed and all resources exist.

---

### 26. **Fragment.InstantiationException**
- **Description**: Thrown when a `Fragment` cannot be instantiated.
- **Prevention**: Ensure that the fragment class is properly declared and accessible.

---

### 27. **RuntimeException**
- **Description**: A superclass for exceptions that can occur at runtime.
- **Prevention**: Catch and handle runtime exceptions based on specific causes.

---

### 28. **NumberFormatException**
- **Description**: Thrown when trying to convert a string to a numeric type and the format is invalid.
- **Prevention**: Validate input strings before converting to numbers.

---

### 29. **IllegalMonitorStateException**
- **Description**: Occurs when a thread attempts to wait on a monitor that it does not own.
- **Prevention**: Ensure correct synchronization using `wait()` and `notify()` methods.

---

### 30. **InterruptedException**
- **Description**: Thrown when a thread is interrupted during sleep or waiting.
- **Prevention**: Handle thread interruptions properly and exit the thread safely.

---

### 31. **ConcurrentModificationException**
- **Description**: Occurs when a collection is modified while iterating through it.
- **Prevention**: Use iterator methods (`Iterator`) to safely modify collections during iteration.

---

### 32. **FileUriExposedException**
- **Description**: Thrown when a file `Uri` is exposed to another app.
- **Prevention**: Use `FileProvider` for securely sharing files between apps.

---

### 33. **JSONException**
- **Description**: Thrown when parsing JSON data fails.
- **Prevention**: Ensure the JSON data is correctly formatted and handle parsing exceptions.

---

### 34. **UriSyntaxException**
- **Description**: Occurs when a URI is not syntactically valid.
- **Prevention**: Ensure URIs are well-formed before using them.

---

### 35. **BadParcelableException**
- **Description**: Occurs when a parcelable object cannot be read properly from a parcel.
- **Prevention**: Ensure parcelable objects are correctly implemented.

---

### 36. **KeyStoreException**
- **Description**: Occurs when there is an issue with accessing the Android `Keystore`.
- **Prevention**: Handle encryption and security operations carefully, ensuring valid keys.

---

### 37. **SSLHandshakeException**
- **Description**: Thrown during SSL handshake issues.
- **Prevention**: Ensure proper SSL certificates and protocols are used.

---

### 38. **RemoteException**
- **Description**: Occurs when a remote service call fails.
- **Prevention**: Handle potential service communication failures and retry mechanisms.

---

### 39. **WindowManager.InvalidDisplayException**
- **Description**: Thrown when the display used is invalid.
- **Prevention**: Ensure the window is assigned to a valid display.

---

### 40. **CloneNotSupportedException**
- **Description**: Occurs when attempting to clone an object that does not implement `Cloneable`.
- **Prevention**: Ensure objects implement `Cloneable` before cloning.

---

### 41. **BinderTransactionFailure**
- **Description**: Occurs during an issue with IPC transactions.
- **Prevention**: Ensure that IPC transactions are managed properly within limits.

---

### 42. **TransactionTooLargeException**
- **Description**: Thrown when a binder transaction exceeds the allowed size.
- **Prevention**: Break down large data into smaller chunks during IPC transactions.

---

### 43. **ArrayStoreException**
- **Description**: Thrown when trying to store an object of the wrong type in an array.
- **Prevention**: Ensure the array can hold the object’s type.

---

### 44. **ArithmeticException**
- **Description**: Thrown during an illegal arithmetic operation, such as division by zero.
- **Prevention**:

 Validate input before performing arithmetic operations.

---

### 45. **ClassFormatError**
- **Description**: Thrown when the class file format is incorrect.
- **Prevention**: Ensure that the class files are valid and correctly compiled.

---

### 46. **NoSuchMethodException**
- **Description**: Occurs when a specified method does not exist.
- **Prevention**: Verify the method name, parameters, and access level before invoking it.

---

### 47. **NoSuchFieldException**
- **Description**: Thrown when a specified field does not exist.
- **Prevention**: Verify the field’s name and class before accessing it.

---

### 48. **NotSerializableException**
- **Description**: Occurs when an object is not serializable but serialization is attempted.
- **Prevention**: Implement the `Serializable` interface for the class.

---

### 49. **InstantiationException**
- **Description**: Thrown when an object cannot be instantiated.
- **Prevention**: Ensure constructors are accessible, and the class is not abstract.

---

### 50. **UnsupportedOperationException**
- **Description**: Thrown when an operation is not supported.
- **Prevention**: Avoid invoking unsupported operations based on the object’s capabilities.

Here is a more exhaustive list of **Android exceptions** that are common in app development. I’ll provide each exception with a one-line description and general guidance on how to prevent or handle it.

### 1. **NullPointerException**
- **Description**: Occurs when dereferencing a null object.
- **Prevention**: Always perform null checks or use Kotlin’s null safety features.

---

### 2. **ArrayIndexOutOfBoundsException**
- **Description**: Occurs when accessing an array index that is out of bounds.
- **Prevention**: Ensure index is within array bounds using checks.

---

### 3. **IllegalArgumentException**
- **Description**: Happens when a method receives an illegal or inappropriate argument.
- **Prevention**: Validate method parameters before use.

---

### 4. **IllegalStateException**
- **Description**: Raised when a method is called at an inappropriate time or state.
- **Prevention**: Ensure the correct state before calling methods, especially in `Fragment` and `Activity` lifecycles.

---

### 5. **IndexOutOfBoundsException**
- **Description**: Thrown when accessing an invalid index for collections like `List`.
- **Prevention**: Check index validity before accessing collections.

---

### 6. **Resources.NotFoundException**
- **Description**: Raised when a resource (e.g., drawable, string) cannot be found.
- **Prevention**: Ensure resources exist, especially in XML files.

---

### 7. **ActivityNotFoundException**
- **Description**: Raised when trying to launch an `Activity` with an invalid `Intent`.
- **Prevention**: Ensure the target activity is defined in the `AndroidManifest.xml`.

---

### 8. **SecurityException**
- **Description**: Raised when attempting a forbidden operation without proper permissions.
- **Prevention**: Check and request runtime permissions appropriately.

---

### 9. **NetworkOnMainThreadException**
- **Description**: Happens when performing network operations on the main thread.
- **Prevention**: Use background threads like `AsyncTask`, `Coroutines`, or `RxJava`.

---

### 10. **FileNotFoundException**
- **Description**: Raised when trying to access a file that does not exist.
- **Prevention**: Ensure the file path is valid and check file existence before access.

---

### 11. **IOException**
- **Description**: General exception for input-output failures.
- **Prevention**: Implement proper error handling in file/network operations.

---

### 12. **OutOfMemoryError**
- **Description**: Happens when the app runs out of memory.
- **Prevention**: Optimize memory usage by handling large objects (e.g., bitmaps) efficiently.

---

### 13. **SocketTimeoutException**
- **Description**: Raised when a socket operation times out.
- **Prevention**: Set appropriate timeout intervals and handle retries in network code.

---

### 14. **MalformedURLException**
- **Description**: Thrown when a URL string is not properly formatted.
- **Prevention**: Validate URLs before making network requests.

---

### 15. **NoClassDefFoundError**
- **Description**: Occurs when the JVM cannot find a class definition.
- **Prevention**: Ensure all required classes are compiled and bundled in the APK.

---

### 16. **ClassCastException**
- **Description**: Thrown when an object is cast to a type that it doesn’t belong to.
- **Prevention**: Use type checks before casting objects.

---

### 17. **IllegalMonitorStateException**
- **Description**: Happens when calling `wait()` or `notify()` on an object that isn’t synchronized.
- **Prevention**: Synchronize blocks properly before calling `wait()` or `notify()`.

---

### 18. **WindowManager.BadTokenException**
- **Description**: Happens when trying to display a dialog or window with an invalid window token.
- **Prevention**: Ensure you are attaching dialogs or windows to valid activities or contexts.

---

### 19. **TransactionTooLargeException**
- **Description**: Occurs when an IPC transaction exceeds the limit.
- **Prevention**: Break large data transactions into smaller chunks to avoid this issue.

---

### 20. **SQLiteException**
- **Description**: General SQLite error.
- **Prevention**: Ensure valid SQL queries and proper management of database connections.

---

### 21. **CursorIndexOutOfBoundsException**
- **Description**: Raised when accessing a cursor outside of its valid bounds.
- **Prevention**: Check cursor bounds before accessing data.

---

### 22. **WindowManager.InvalidDisplayException**
- **Description**: Happens when trying to operate on an invalid display.
- **Prevention**: Ensure that the window and display are valid before using them.

---

### 23. **OutOfMemoryError**
- **Description**: Occurs when the system is unable to allocate enough memory for an operation.
- **Prevention**: Optimize memory usage, recycle resources like bitmaps, and avoid memory leaks.

---

### 24. **InflateException**
- **Description**: Happens when there’s an error inflating a layout resource.
- **Prevention**: Ensure XML layouts are well-formed and resources exist.

---

### 25. **NoSuchMethodException**
- **Description**: Occurs when a requested method does not exist.
- **Prevention**: Ensure that methods are correctly defined and accessible.

---

### 26. **ArithmeticException**
- **Description**: Thrown when an illegal arithmetic operation, such as division by zero, occurs.
- **Prevention**: Validate input values before performing arithmetic operations.

---

### 27. **JSONException**
- **Description**: Occurs when there’s an error in parsing JSON data.
- **Prevention**: Ensure JSON data is well-formed and handle parsing errors.

---

### 28. **FileUriExposedException**
- **Description**: Happens when trying to expose a file `Uri` to another app without proper permissions.
- **Prevention**: Use `FileProvider` for securely sharing files.

---

### 29. **BadParcelableException**
- **Description**: Thrown when a `Parcelable` object cannot be unmarshalled.
- **Prevention**: Ensure all `Parcelable` objects are correctly implemented.

---

### 30. **DeadObjectException**
- **Description**: Occurs when a remote process dies during a transaction.
- **Prevention**: Handle such remote service errors gracefully with retries.

---

### 31. **KeyStoreException**
- **Description**: Raised when there’s an error related to the Android `Keystore`.
- **Prevention**: Ensure proper handling of keys and encryption.

---

### 32. **SSLException**
- **Description**: Happens during issues with SSL handshakes or invalid certificates.
- **Prevention**: Validate certificates and handle SSL errors properly.

---

### 33. **RemoteException**
- **Description**: General remote invocation error.
- **Prevention**: Handle `AIDL` (Android Interface Definition Language) service failures properly.

---

### 34. **SecurityException**
- **Description**: Raised when an action is attempted without the required security permissions.
- **Prevention**: Ensure proper permissions in the manifest and handle runtime permission requests.

---

### 35. **UnsupportedOperationException**
- **Description**: Raised when an unsupported operation is attempted.
- **Prevention**: Ensure the functionality is supported before invoking the method.

---

### 36. **PackageManager.NameNotFoundException**
- **Description**: Happens when a requested package or component cannot be found.
- **Prevention**: Ensure the package/component exists and is properly declared in the manifest.

---

### 37. **CloneNotSupportedException**
- **Description**: Occurs when trying to clone an object that doesn’t support cloning.
- **Prevention**: Ensure the class implements `Cloneable` interface.

---

### 38. **NotSerializableException**
- **Description**: Raised when an object that doesn’t implement `Serializable` is serialized.
- **Prevention**: Implement `Serializable` in objects meant for serialization.

---

### 39. **InstantiationException**
- **Description**: Thrown when an object cannot be instantiated.
- **Prevention**: Ensure that classes have proper constructors and are not abstract.

---

### 40. **NoSuchFieldException**
- **Description**: Occurs when trying to access a field that doesn’t exist.
- **Prevention**: Verify the field name and class before accessing it.

---

### 41. **VerifyError**
- **Description**: Thrown when bytecode verification fails during class loading.
- **Prevention**: Ensure code is properly compiled and meets JVM bytecode verification requirements.

---

### 42. **RuntimeException**
- **Description**: General superclass for exceptions that occur at runtime.
- **Prevention**: Catch specific exceptions to avoid catching `RuntimeException`.

---

### 43. **TypeCastException**
- **Description**: Thrown when an object is cast to an incompatible type.
- **Prevention**: Use proper type checks before casting.

---

### 44. **WindowLeakedException**
- **Description**: Happens when an activity tries to show a dialog after it has been destroyed.
- **Prevention**: Ensure the activity is not finishing before showing dialogs.

---

### 45. **ConnectionTimeoutException**
- **Description**: Occurs when a network connection times out.
- **Prevention**: Set appropriate timeout values and handle connection retries.

---

### 46. **SSLHandshakeException**
- **Description**: Occurs during SSL handshake issues.
- **Prevention**: Ensure the server has a

 valid SSL certificate and handle the exception.

---

### 47. **ViewNotFoundException**
- **Description**: Happens when a view is not found within the layout.
- **Prevention**: Ensure correct layout inflation and view initialization.

---

### 48. **DeadObjectException**
- **Description**: Raised when the communication link with a service is broken.
- **Prevention**: Handle service connection failures gracefully.

---

### 49. **NullPointerException**
- **Description**: Occurs when a null object is accessed.
- **Prevention**: Always check for null values before using objects.

---

### 50. **CursorIndexOutOfBoundsException**
- **Description**: Thrown when accessing a cursor outside of its valid range.
- **Prevention**: Ensure valid cursor access and index checks.



