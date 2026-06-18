# DAY 2 — ANDROID FUNDAMENTALS (DEEP DIVE)
### Activities · Fragments · Intents · Services · Broadcast Receivers · Manifest · Permissions · Parcelable

> Every line of code is commented. Format: Problem → Definition → How It Works → Code → Real Usage → Interview Q&A → Common Mistakes

---

## HOW ANDROID APPS ACTUALLY WORK (Read This First)

Before touching any class, understand the big picture.

```
Your App
    │
    ├── AndroidManifest.xml   ← App's ID card. Declares every component to Android OS.
    │
    ├── Activities            ← Screens the user sees and interacts with.
    │
    ├── Fragments             ← Reusable UI pieces that live inside Activities.
    │
    ├── Services              ← Background work with no UI (music, downloads).
    │
    ├── Broadcast Receivers   ← Listen for system-wide events (battery low, wifi on).
    │
    └── Content Providers     ← Share data between apps (not in today's scope).
```

Android is component-based. The OS manages your components' lifecycle.
You don't control when your Activity is destroyed — Android does.
That's why lifecycle methods exist.

---

## PART 1: ANDROIDMANIFEST.XML (The App's Identity Card)

### Problem It Solves
Android OS needs to know: what screens does this app have, what permissions does it need, which screen opens first, what services run. Without Manifest, Android can't launch your app.

### Definition
`AndroidManifest.xml` is an XML file at the root of every Android project. It declares all app components and metadata to the Android OS **before the app runs**.

### Complete Annotated Manifest

```xml
<?xml version="1.0" encoding="utf-8"?>

<!-- package = unique ID for your app on the Play Store and device -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">

    <!-- ═══ PERMISSIONS ═══ -->
    <!-- Declare every dangerous or special permission your app needs -->
    <!-- Without this line, camera access will silently fail -->
    <uses-permission android:name="android.permission.CAMERA" />

    <!-- Needed for any HTTP/HTTPS network call -->
    <uses-permission android:name="android.permission.INTERNET" />

    <!-- Coarse = cell tower location. Fine = GPS (more precise) -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

    <!-- Allows running a foreground service (like music player) -->
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <!-- ═══ APPLICATION ═══ -->
    <!-- android:name = custom Application class (optional, for app-wide init) -->
    <!-- android:icon = launcher icon shown on home screen -->
    <!-- android:label = app name shown under icon and in recents -->
    <!-- android:theme = default visual theme for all screens -->
    <application
        android:name=".MyApplication"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.MyApp">

        <!-- ═══ ACTIVITY ═══ -->
        <!-- Every Activity MUST be declared here or Android will crash on launch -->
        <!-- android:name = full class name (.MainActivity = com.example.myapp.MainActivity) -->
        <!-- android:exported = true means other apps/OS can launch this Activity -->
        <activity
            android:name=".MainActivity"
            android:exported="true">

            <!-- intent-filter tells Android: this is the LAUNCHER activity -->
            <!-- MAIN = entry point of app -->
            <!-- LAUNCHER = show this icon in the app drawer -->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Second activity — no intent-filter means only your app can open it -->
        <activity
            android:name=".DetailActivity"
            android:exported="false" />

        <!-- ═══ SERVICE ═══ -->
        <!-- Declare background services here -->
        <service
            android:name=".MusicService"
            android:exported="false" />

        <!-- ═══ BROADCAST RECEIVER ═══ -->
        <!-- Statically registered receiver — works even if app is not open -->
        <receiver
            android:name=".BootReceiver"
            android:exported="true">
            <intent-filter>
                <!-- Listen for device boot completion -->
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>

    </application>

</manifest>
```

### Interview Q&A

**Q: What happens if you forget to declare an Activity in the Manifest?**
A: App crashes with `ActivityNotFoundException` at runtime.

**Q: What is `android:exported`?**
A: Controls whether other apps can launch this component. `true` = any app can. `false` = only your app can.

---

## PART 2: ACTIVITY (DEEP DIVE)

### Problem It Solves
Users need to see and interact with your app. An Activity provides one **focused screen** — like a login screen, a home screen, or a detail screen.

### Definition
An Activity is an Android component that represents a **single screen with a user interface**. It has a strict lifecycle managed by the Android OS. When the user presses Home, the OS can pause or even destroy your Activity to free memory.

### Activity Lifecycle — The Most Important Diagram in Android

```
App launched by user
        ↓
   ┌─ onCreate()   ← Called ONCE. Initialize UI, set layout, get references.
   │
   ├─ onStart()    ← Activity becomes visible to user.
   │
   ├─ onResume()   ← Activity is in foreground, user can interact. START YOUR WORK HERE.
   │
   │   [User is using app]
   │
   ├─ onPause()    ← Activity partially hidden (dialog opened, or going to another app).
   │                  SAVE UNSAVED DATA HERE. Must be fast (< 500ms).
   │
   ├─ onStop()     ← Activity fully hidden. Release heavy resources here.
   │
   ├─ onRestart()  ← Called when coming back from Stopped state.
   │
   └─ onDestroy()  ← Activity is being destroyed. Clean up everything.
```

**Key rule:** The OS can kill your process any time after `onStop()`. Never rely on `onDestroy()` being called.

### Complete Activity With Every Line Commented

```java
package com.example.myapp;                          // package name — must match Manifest

import android.os.Bundle;                           // Bundle = key-value store for saving state
import android.util.Log;                            // Log = print messages to Logcat for debugging
import android.view.View;                           // View = base class for all UI elements
import android.widget.Button;                       // Button widget class
import android.widget.TextView;                     // TextView widget class
import androidx.appcompat.app.AppCompatActivity;    // base class that adds modern features to Activity

// extends AppCompatActivity = inherit lifecycle management, toolbar support, theme support
public class MainActivity extends AppCompatActivity {

    // TAG used in Log statements — helps filter Logcat by class name
    private static final String TAG = "MainActivity";

    // Declare UI references as fields so all methods can access them
    private TextView textView;                      // reference to the text display
    private Button button;                          // reference to the button
    private int clickCount = 0;                     // state variable — counts button clicks

    // ─── LIFECYCLE METHOD 1 ───────────────────────────────────────────────
    // Called ONCE when Activity is first created
    // savedInstanceState = Bundle with data from last time (if Activity was rotated/restored)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);         // MUST call super first — sets up base Activity

        // Tell Android which XML layout file to inflate (draw) as the screen
        setContentView(R.layout.activity_main);     // R = auto-generated resource index; layout folder

        Log.d(TAG, "onCreate called");              // d = debug level; prints to Logcat

        // ── Find UI elements by their XML id ──────────────────────────────
        // findViewById searches the inflated layout for a View with this id
        // R.id.textView = ID declared in XML as android:id="@+id/textView"
        textView = (TextView) findViewById(R.id.textView);
        button   = (Button)   findViewById(R.id.button);

        // ── Restore state if Activity was recreated (e.g. screen rotation) ─
        if (savedInstanceState != null) {           // null = first launch, not null = restored
            // Retrieve previously saved click count using the same key used in onSaveInstanceState
            clickCount = savedInstanceState.getInt("clickCount", 0);
            // Update UI to show restored count
            textView.setText("Clicks: " + clickCount);
        }

        // ── Set up click listener on button ──────────────────────────────
        // setOnClickListener takes a View.OnClickListener (interface with one method: onClick)
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {           // v = the View that was clicked (the button)
                clickCount++;                       // increment our counter
                // setText updates the displayed text dynamically
                textView.setText("Clicks: " + clickCount);
                Log.d(TAG, "Button clicked, count = " + clickCount);
            }
        });

        // ── Lambda shorthand for the same thing ──────────────────────────
        // button.setOnClickListener(v -> textView.setText("Clicks: " + (++clickCount)));
    }

    // ─── LIFECYCLE METHOD 2 ───────────────────────────────────────────────
    // Called when Activity becomes visible but NOT interactive yet
    @Override
    protected void onStart() {
        super.onStart();                            // always call super in lifecycle methods
        Log.d(TAG, "onStart called");
    }

    // ─── LIFECYCLE METHOD 3 ───────────────────────────────────────────────
    // Called when Activity is in foreground and user can interact
    // Resume cameras, sensors, animations here
    @Override
    protected void onResume() {
        super.onResume();
        Log.d(TAG, "onResume called — app is active");
    }

    // ─── LIFECYCLE METHOD 4 ───────────────────────────────────────────────
    // Called when Activity loses focus — another Activity or dialog appears on top
    // SAVE DATA HERE — this is the last guaranteed callback
    @Override
    protected void onPause() {
        super.onPause();
        Log.d(TAG, "onPause called — saving data");
        // Example: pause video, save draft, stop GPS
    }

    // ─── LIFECYCLE METHOD 5 ───────────────────────────────────────────────
    // Called when Activity is completely hidden from user
    @Override
    protected void onStop() {
        super.onStop();
        Log.d(TAG, "onStop called — releasing resources");
        // Example: stop network polling, release heavy objects
    }

    // ─── LIFECYCLE METHOD 6 ───────────────────────────────────────────────
    // Called to save UI state before Activity is destroyed
    // This is called BEFORE onDestroy — use it to save transient UI state
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        // Save clickCount with the key "clickCount" — retrieved in onCreate
        outState.putInt("clickCount", clickCount);
        Log.d(TAG, "onSaveInstanceState — saved clickCount=" + clickCount);
    }

    // ─── LIFECYCLE METHOD 7 ───────────────────────────────────────────────
    // Called last before Activity is destroyed
    // Release ALL resources here (DB connections, threads, listeners)
    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy called");
    }
}
```

### Corresponding XML Layout

```xml
<!-- res/layout/activity_main.xml -->

<!-- ConstraintLayout = powerful layout where elements are positioned relative to each other -->
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"        <!-- fill parent's width (the screen) -->
    android:layout_height="match_parent">      <!-- fill parent's height (the screen) -->

    <!-- TextView — displays text -->
    <TextView
        android:id="@+id/textView"             <!-- unique id — used in findViewById() -->
        android:layout_width="wrap_content"    <!-- width = just enough to fit the text -->
        android:layout_height="wrap_content"   <!-- height = just enough to fit the text -->
        android:text="Clicks: 0"              <!-- initial displayed text -->
        android:textSize="24sp"               <!-- sp = scale-independent pixels, respects font size setting -->
        app:layout_constraintTop_toTopOf="parent"      <!-- align top edge to parent's top -->
        app:layout_constraintStart_toStartOf="parent"  <!-- align left edge to parent's left -->
        app:layout_constraintEnd_toEndOf="parent"      <!-- center horizontally -->
        android:layout_marginTop="100dp" />            <!-- margin from top in dp (density-independent) -->

    <!-- Button — tappable button -->
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click Me"               <!-- text shown on button -->
        app:layout_constraintTop_toBottomOf="@id/textView"   <!-- place below textView -->
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="24dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

---

## PART 3: INTENTS (DEEP DIVE)

### Problem It Solves
Activities are separate classes — they can't directly call each other. Intents are **messages** that tell the Android OS to navigate to another Activity, or trigger a system action like opening a camera or dialing a number.

### Definition
An Intent is an abstract description of an operation to be performed. It's a messaging object used to request an action from another component (Activity, Service, BroadcastReceiver).

### Two Types of Intents

```
Explicit Intent:    You specify EXACTLY which class to open.
                    Use within your own app.

Implicit Intent:    You specify WHAT to do, not WHO does it.
                    Android finds the right app to handle it.
```

### Explicit Intent — Navigate Between Screens

```java
package com.example.myapp;

import android.content.Intent;     // Intent class for sending messages between components
import android.os.Bundle;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button openDetailBtn = findViewById(R.id.btnOpenDetail);

        openDetailBtn.setOnClickListener(v -> {

            // Create an explicit Intent — specifying exact destination class
            // this = current Context (Activity is a Context)
            // DetailActivity.class = the screen we want to open
            Intent intent = new Intent(this, DetailActivity.class);

            // putExtra = attach data to the intent (key-value pairs)
            // "USER_NAME" is the key — DetailActivity will use this same key to read it
            intent.putExtra("USER_NAME", "John Doe");

            // putExtra with int
            intent.putExtra("USER_AGE", 25);

            // putExtra with boolean
            intent.putExtra("IS_PREMIUM", true);

            // startActivity = send the intent — Android OS creates DetailActivity
            startActivity(intent);

            // Optional: add transition animation
            // overridePendingTransition(R.anim.slide_in, R.anim.slide_out);
        });
    }
}
```

### Receiving Data in the Destination Activity

```java
package com.example.myapp;

import android.content.Intent;
import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;

public class DetailActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_detail);

        TextView nameText = findViewById(R.id.tvName);
        TextView ageText  = findViewById(R.id.tvAge);

        // getIntent() returns the Intent that was used to start this Activity
        Intent intent = getIntent();

        // If no Activity started this one, intent is still non-null but has no extras
        if (intent != null) {

            // getStringExtra = read string value by key
            // "Unknown" = default value if "USER_NAME" key doesn't exist
            String name = intent.getStringExtra("USER_NAME");

            // getIntExtra = read int value; second param is the default if key missing
            int age     = intent.getIntExtra("USER_AGE", 0);

            // getBooleanExtra = read boolean value
            boolean premium = intent.getBooleanExtra("IS_PREMIUM", false);

            // Update UI with received data
            nameText.setText("Name: " + name);
            ageText.setText("Age: " + age);
        }
    }
}
```

### Implicit Intent — Use System Apps

```java
import android.content.Intent;
import android.net.Uri;     // Uri = Uniform Resource Identifier — represents a URL or address

// ── Open a website in browser ─────────────────────────────────────────────
Intent browserIntent = new Intent(
    Intent.ACTION_VIEW,              // ACTION_VIEW = show this data to the user
    Uri.parse("https://google.com") // Uri.parse = converts string to Uri object
);
startActivity(browserIntent);       // Android picks the default browser

// ── Make a phone call ─────────────────────────────────────────────────────
Intent callIntent = new Intent(
    Intent.ACTION_DIAL,              // ACTION_DIAL = open dialer with number pre-filled
    Uri.parse("tel:+91-9876543210") // tel: scheme tells Android this is a phone number
);
startActivity(callIntent);

// ── Share text ────────────────────────────────────────────────────────────
Intent shareIntent = new Intent(Intent.ACTION_SEND);  // ACTION_SEND = share content
shareIntent.setType("text/plain");                    // MIME type of data being shared
shareIntent.putExtra(Intent.EXTRA_TEXT, "Check this out!"); // the text to share
// createChooser = shows a picker so user can choose WhatsApp, Gmail, etc.
startActivity(Intent.createChooser(shareIntent, "Share via"));

// ── Open camera ────────────────────────────────────────────────────────────
Intent cameraIntent = new Intent("android.media.action.IMAGE_CAPTURE");
startActivity(cameraIntent);

// ── Open Settings ──────────────────────────────────────────────────────────
Intent settingsIntent = new Intent(android.provider.Settings.ACTION_SETTINGS);
startActivity(settingsIntent);
```

### startActivityForResult Pattern (Getting Result Back)

```java
// Modern way using ActivityResultLauncher (replaces deprecated startActivityForResult)
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import android.app.Activity;

public class MainActivity extends AppCompatActivity {

    // Declare launcher — registers a callback for the result
    // Must be declared as a field, registered during onCreate
    private ActivityResultLauncher<Intent> detailLauncher;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // registerForActivityResult = set up what to do when result comes back
        // StartActivityForResult = contract that handles Intent → result
        detailLauncher = registerForActivityResult(
            new ActivityResultContracts.StartActivityForResult(),
            result -> {                                         // lambda: called when DetailActivity finishes
                // result.getResultCode() = RESULT_OK or RESULT_CANCELED
                if (result.getResultCode() == Activity.RESULT_OK) {
                    // result.getData() = the Intent returned by DetailActivity
                    Intent data = result.getData();
                    if (data != null) {
                        String returnedValue = data.getStringExtra("RESULT_DATA");
                        // use returnedValue
                    }
                }
            }
        );

        Button openBtn = findViewById(R.id.btnOpen);
        openBtn.setOnClickListener(v -> {
            Intent intent = new Intent(this, DetailActivity.class);
            detailLauncher.launch(intent);   // launch — replaces startActivityForResult()
        });
    }
}
```

```java
// In DetailActivity — sending result back
public class DetailActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_detail);

        Button doneBtn = findViewById(R.id.btnDone);
        doneBtn.setOnClickListener(v -> {

            Intent resultIntent = new Intent();           // create empty intent for result
            resultIntent.putExtra("RESULT_DATA", "User selected item X");

            // setResult = declare the outcome and attach data
            // RESULT_OK = standard code for success
            setResult(Activity.RESULT_OK, resultIntent);

            finish();   // close this Activity — control returns to MainActivity
        });
    }
}
```

### Interview Q&A

**Q: Explicit vs Implicit Intent?**
A: Explicit specifies the exact component (class). Implicit specifies an action and lets Android find the right app to handle it.

**Q: What is `Intent.ACTION_VIEW`?**
A: A standard intent action that tells Android to display data to the user — browser for URLs, gallery for images, etc.

**Q: Can Intent carry large data like bitmaps?**
A: No. Intent extras have a size limit (~1MB). Use file paths, URIs, or shared ViewModel for large data.

---

## PART 4: FRAGMENT (DEEP DIVE)

### Problem It Solves
An Activity is one whole screen. But tablets and modern phones need **reusable UI sections** — like a list on the left and detail on the right, or tabs at the bottom. Fragments solve this.

### Definition
A Fragment is a **reusable portion of a UI** that lives inside an Activity. It has its own lifecycle, but that lifecycle is tied to the host Activity's lifecycle. One Activity can host multiple Fragments simultaneously.

### Fragment Lifecycle vs Activity Lifecycle

```
Activity:                    Fragment:
onCreate()                   onAttach()     ← Fragment attached to Activity
onStart()                    onCreate()     ← Fragment initialized (no view yet)
onResume()                   onCreateView() ← Fragment creates its UI
                             onViewCreated()← View is ready, bind UI elements here
                             onStart()
                             onResume()
                             onPause()
                             onStop()
                             onDestroyView()← Fragment's view destroyed (Fragment may live on)
onStop()                     onDestroy()
onDestroy()                  onDetach()     ← Fragment detached from Activity

IMPORTANT: onDestroyView ≠ onDestroy
  Fragment can exist in back stack with no view — view is destroyed but fragment is not.
```

### Complete Fragment With Every Line Commented

```java
package com.example.myapp;

import android.os.Bundle;
import android.view.LayoutInflater;   // LayoutInflater = converts XML layout to View objects
import android.view.View;
import android.view.ViewGroup;        // ViewGroup = container that holds other Views
import android.widget.Button;
import android.widget.TextView;
import androidx.fragment.app.Fragment;  // Fragment base class (use androidx, not old android.app)

public class HomeFragment extends Fragment {

    // ── CONSTANTS ─────────────────────────────────────────────────────────
    // Key for the argument bundle — this is how you pass data TO the fragment
    private static final String ARG_USER_NAME = "user_name";

    // ── FIELDS ────────────────────────────────────────────────────────────
    private String userName;       // data received from Activity
    private TextView greetingText; // UI reference — initialized in onViewCreated
    private Button actionButton;

    // ── FACTORY METHOD (preferred pattern) ────────────────────────────────
    // Instead of calling constructor directly, use this static factory method
    // Reason: If Android recreates the fragment, it calls the no-arg constructor
    //         Arguments in Bundle survive recreation — constructor parameters don't
    public static HomeFragment newInstance(String userName) {
        HomeFragment fragment = new HomeFragment();   // create fragment instance

        Bundle args = new Bundle();                   // Bundle = key-value store
        args.putString(ARG_USER_NAME, userName);      // store data with a key

        fragment.setArguments(args);                  // attach args to the fragment
        return fragment;                              // return ready-to-use fragment
    }

    // ── LIFECYCLE: onCreate ────────────────────────────────────────────────
    // Called when Fragment is first created — no view available yet
    // Good place to read arguments, initialize non-UI data
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // getArguments() = retrieve the Bundle we set in newInstance()
        if (getArguments() != null) {
            // Read the value using the same key we used to store it
            userName = getArguments().getString(ARG_USER_NAME, "Guest");
        }
    }

    // ── LIFECYCLE: onCreateView ────────────────────────────────────────────
    // Called when Fragment needs to create its View hierarchy
    // MUST return the root View of the fragment's layout
    // inflater  = used to convert XML layout file to View objects
    // container = the parent ViewGroup (from the Activity's layout)
    // savedInstanceState = saved state from before
    @Override
    public View onCreateView(LayoutInflater inflater,
                             ViewGroup container,
                             Bundle savedInstanceState) {

        // inflate = read XML file and create View objects from it
        // R.layout.fragment_home = your XML file at res/layout/fragment_home.xml
        // container = parent — needed for correct layout params
        // false = don't attach to parent yet (FragmentManager does that)
        View rootView = inflater.inflate(R.layout.fragment_home, container, false);

        return rootView;    // return the root view of this fragment
    }

    // ── LIFECYCLE: onViewCreated ───────────────────────────────────────────
    // Called AFTER onCreateView — view is ready, safe to find and bind Views
    // view = the View returned by onCreateView
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        // Find views within this fragment's view hierarchy
        greetingText = view.findViewById(R.id.tvGreeting);
        actionButton = view.findViewById(R.id.btnAction);

        // Set text using data received via arguments
        greetingText.setText("Hello, " + userName + "!");

        actionButton.setOnClickListener(v -> {
            // requireActivity() = get the host Activity safely (never null here)
            // If you just need Context, use requireContext()
            greetingText.setText("Button clicked by " + userName);
        });
    }

    // ── LIFECYCLE: onDestroyView ───────────────────────────────────────────
    // Called when Fragment's view is destroyed (e.g. navigating away but fragment in backstack)
    // NULL OUT view references here to avoid memory leaks
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        // If using ViewBinding: binding = null;
        greetingText = null;   // allow GC to clean up the view
        actionButton = null;
    }
}
```

### Corresponding Fragment XML

```xml
<!-- res/layout/fragment_home.xml -->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"     <!-- fill available width -->
    android:layout_height="match_parent"    <!-- fill available height -->
    android:orientation="vertical"          <!-- stack children top to bottom -->
    android:padding="16dp">               <!-- inner spacing on all sides -->

    <TextView
        android:id="@+id/tvGreeting"        <!-- id used in onViewCreated to find this view -->
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Hello!"
        android:textSize="22sp" />

    <Button
        android:id="@+id/btnAction"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click"
        android:layout_marginTop="16dp" />

</LinearLayout>
```

### Adding Fragment to Activity — Two Ways

**Way 1: Static (XML — simpler)**
```xml
<!-- In activity_main.xml -->
<!-- Fragment declared directly in XML — always shown, can't be swapped -->
<fragment
    android:id="@+id/homeFragment"
    android:name="com.example.myapp.HomeFragment"   <!-- full class path -->
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

**Way 2: Dynamic (Code — flexible, most common)**
```java
// In MainActivity.java — dynamically add/replace/remove fragments at runtime

// FragmentManager manages the fragment back stack
// getSupportFragmentManager() = use support library version (not deprecated)
getSupportFragmentManager()

    // beginTransaction() = start a batch of fragment operations
    .beginTransaction()

    // replace() = remove any fragment in the container, add this one
    // R.id.fragment_container = FrameLayout in activity_main.xml that holds fragments
    // HomeFragment.newInstance("John") = create fragment with argument
    .replace(R.id.fragment_container, HomeFragment.newInstance("John"))

    // addToBackStack() = pressing Back will pop this fragment (go back to previous)
    // null = no name for this transaction (can name it to pop to specific point)
    .addToBackStack(null)

    // commit() = execute all the operations queued in this transaction
    .commit();
```

```xml
<!-- activity_main.xml — needs a container for fragments -->
<FrameLayout
    android:id="@+id/fragment_container"   <!-- id used in replace() above -->
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

### Fragment ↔ Activity Communication

```java
// ── Fragment tells Activity something happened ──────────────────────────
// Define interface inside Fragment
public class HomeFragment extends Fragment {

    // Interface = contract. Activity must implement this.
    public interface OnUserActionListener {
        void onUserSelected(String userName);  // method Activity must provide
    }

    private OnUserActionListener listener;  // reference to Activity

    // onAttach = called when Fragment is attached to Activity
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        // Cast context (the Activity) to the listener interface
        // Throws ClassCastException if Activity doesn't implement it
        if (context instanceof OnUserActionListener) {
            listener = (OnUserActionListener) context;
        } else {
            throw new RuntimeException(context + " must implement OnUserActionListener");
        }
    }

    // Somewhere in Fragment, trigger the callback
    private void userTappedItem(String name) {
        if (listener != null) {
            listener.onUserSelected(name);   // notify Activity
        }
    }
}

// In MainActivity — implement the interface
public class MainActivity extends AppCompatActivity
        implements HomeFragment.OnUserActionListener {

    // Must implement this method
    @Override
    public void onUserSelected(String userName) {
        // Handle the event from Fragment — e.g. open DetailActivity
        Toast.makeText(this, "Selected: " + userName, Toast.LENGTH_SHORT).show();
    }
}
```

### Activity vs Fragment — The Most Important Interview Topic

| Topic | Activity | Fragment |
|---|---|---|
| UI | Full screen | Portion of screen |
| Dependency | Independent | Needs an Activity |
| Reuse | Cannot reuse across Activities | Can reuse in multiple Activities |
| Back stack | System managed | FragmentManager managed |
| Communication | Intent | Interface or ViewModel |
| lifecycle | Full lifecycle | Lifecycle + `onCreateView`/`onDestroyView` |
| When to use | Top-level screens | Tabs, bottom nav, reusable UI panels |

### Interview Q&A

**Q: Why `onViewCreated` instead of `onCreateView` for UI setup?**
A: In `onCreateView` the view is just being built. In `onViewCreated` the view is fully ready. It's safer and cleaner.

**Q: What is the Fragment back stack?**
A: A stack maintained by FragmentManager. When you call `addToBackStack()`, pressing Back pops the fragment — like navigating back between Activities.

**Q: Why null view references in `onDestroyView`?**
A: Fragment can stay alive in the back stack with no view. View references would create memory leaks — the GC can't free the view because the Fragment still holds it.

---

## PART 5: PERMISSIONS (DEEP DIVE)

### Problem It Solves
Apps should not silently access your camera, location, or contacts. Android requires apps to declare and **request permission** at runtime for sensitive resources.

### Two Permission Types

```
Normal Permissions:   Low risk. Auto-granted when app installed.
                      INTERNET, VIBRATE, NFC

Dangerous Permissions: Access sensitive data. Must be requested AT RUNTIME.
                      CAMERA, LOCATION, READ_CONTACTS, RECORD_AUDIO, READ_SMS
```

### Complete Permission Flow

```java
package com.example.myapp;

import android.Manifest;                                    // Manifest = constants for permission strings
import android.content.pm.PackageManager;                   // PackageManager = check permission status
import android.os.Bundle;
import android.widget.Button;
import android.widget.Toast;
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;                 // ContextCompat = backward-compatible utility

public class CameraActivity extends AppCompatActivity {

    // ActivityResultLauncher for permission request
    // String = the permission string we're requesting
    // Boolean = was it granted?
    private ActivityResultLauncher<String> requestPermissionLauncher;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_camera);

        // ── Register permission result callback ─────────────────────────
        // This must be registered BEFORE the activity starts (in onCreate)
        requestPermissionLauncher = registerForActivityResult(
            new ActivityResultContracts.RequestPermission(),  // contract for single permission
            isGranted -> {                                    // lambda: called with result
                if (isGranted) {
                    // Permission was granted by user
                    openCamera();
                } else {
                    // Permission was denied by user
                    Toast.makeText(this,
                        "Camera permission required",
                        Toast.LENGTH_SHORT).show();
                }
            }
        );

        Button cameraBtn = findViewById(R.id.btnCamera);

        // ── Check and request permission on button click ────────────────
        cameraBtn.setOnClickListener(v -> checkAndRequestCameraPermission());
    }

    private void checkAndRequestCameraPermission() {

        // ContextCompat.checkSelfPermission = check if we already have this permission
        // this = context  |  Manifest.permission.CAMERA = "android.permission.CAMERA"
        // Returns PackageManager.PERMISSION_GRANTED or PERMISSION_DENIED
        int permissionStatus = ContextCompat.checkSelfPermission(
            this,
            Manifest.permission.CAMERA
        );

        if (permissionStatus == PackageManager.PERMISSION_GRANTED) {
            // Already have permission — proceed directly
            openCamera();

        } else {
            // Don't have permission — show rationale if needed, then request

            // shouldShowRequestPermissionRationale = true if user previously denied
            // We should show an explanation before asking again
            if (shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {
                // Show a dialog explaining WHY we need camera
                Toast.makeText(this,
                    "Camera is needed to take profile pictures",
                    Toast.LENGTH_LONG).show();
            }

            // Launch the permission request dialog
            // Android shows the system dialog — we can't customize it
            requestPermissionLauncher.launch(Manifest.permission.CAMERA);
        }
    }

    private void openCamera() {
        // Camera permission confirmed — safe to use camera
        Toast.makeText(this, "Opening camera!", Toast.LENGTH_SHORT).show();
    }
}
```

### Requesting Multiple Permissions

```java
// Launcher for multiple permissions at once
// String[] = array of permission strings
// Map<String, Boolean> = each permission mapped to granted/denied
private ActivityResultLauncher<String[]> multiplePermissionsLauncher =
    registerForActivityResult(
        new ActivityResultContracts.RequestMultiplePermissions(),
        results -> {
            // results is a Map — key=permission string, value=was granted
            Boolean cameraGranted   = results.get(Manifest.permission.CAMERA);
            Boolean locationGranted = results.get(Manifest.permission.ACCESS_FINE_LOCATION);

            if (Boolean.TRUE.equals(cameraGranted) &&
                Boolean.TRUE.equals(locationGranted)) {
                // Both granted
            } else {
                // One or more denied
            }
        }
    );

// Launch both requests at once
multiplePermissionsLauncher.launch(new String[]{
    Manifest.permission.CAMERA,
    Manifest.permission.ACCESS_FINE_LOCATION
});
```

### Interview Q&A

**Q: Normal vs Dangerous permissions?**
A: Normal permissions (INTERNET) are auto-granted. Dangerous permissions (CAMERA, LOCATION) require explicit user approval at runtime.

**Q: What if user selects "Don't ask again"?**
A: `requestPermissionLauncher` immediately returns `false`. `shouldShowRequestPermissionRationale()` returns `false`. You must redirect user to app Settings.

---

## PART 6: SERVICES (DEEP DIVE)

### Problem It Solves
Some work must continue even when the user leaves the screen — playing music, downloading a file, syncing data. Services run in the **background without a UI**.

### Three Types of Services

```
Started Service:      You call startService(). Runs until it stops itself.
                      Example: download a file.

Bound Service:        Components bind to it and communicate directly.
                      Stops when all clients unbind.
                      Example: music player where Activity controls playback.

Foreground Service:   Like Started Service but shows a persistent notification.
                      OS is much less likely to kill it.
                      Example: GPS tracking, music player.
```

### Started Service (IntentService Pattern)

```java
package com.example.myapp;

import android.app.Service;          // Service = base class for all services
import android.content.Intent;
import android.os.IBinder;           // IBinder = interface for bound services (null if not bound)
import android.util.Log;

public class DownloadService extends Service {

    private static final String TAG = "DownloadService";

    // ── onBind ────────────────────────────────────────────────────────────
    // Required override — return null if this is a Started Service (not Bound)
    @Override
    public IBinder onBind(Intent intent) {
        return null;    // null means: this service does not support binding
    }

    // ── onStartCommand ────────────────────────────────────────────────────
    // Called every time startService() is called with an intent
    // intent = the Intent passed from startService() — contains instructions
    // flags  = additional info about how the service was started
    // startId = unique ID for this start request (if multiple startService calls)
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "Service started");

        // Read data passed via intent extras
        String fileUrl = intent.getStringExtra("FILE_URL");

        // Do work on a background thread — Service runs on main thread by default!
        // Never do network/disk IO directly in onStartCommand
        new Thread(() -> {
            downloadFile(fileUrl);  // simulate long-running task

            // stopSelf() = service stops itself after completing work
            // Pass startId to ensure we only stop THIS specific request
            stopSelf(startId);
        }).start();

        // Return value tells Android what to do if service is killed
        // START_STICKY = restart service with null intent if killed
        // START_NOT_STICKY = don't restart if killed
        // START_REDELIVER_INTENT = restart and redeliver last intent
        return START_STICKY;
    }

    private void downloadFile(String url) {
        Log.d(TAG, "Downloading: " + url);
        // ... actual download logic
    }

    // ── onDestroy ─────────────────────────────────────────────────────────
    // Clean up resources when service is stopping
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "Service destroyed");
    }
}
```

### Starting the Service from Activity

```java
// Create intent targeting the Service class
Intent serviceIntent = new Intent(this, DownloadService.class);

// Pass data to service via extras
serviceIntent.putExtra("FILE_URL", "https://example.com/file.zip");

// startService = tell Android to start the service
// Android creates it if not running, or calls onStartCommand if already running
startService(serviceIntent);

// Later — stop the service from Activity
stopService(serviceIntent);
```

### Foreground Service (For Long Running Important Tasks)

```java
import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.Service;
import android.content.Intent;
import android.os.Build;
import android.os.IBinder;
import androidx.core.app.NotificationCompat;

public class MusicService extends Service {

    // Notification channel ID — required for Android 8.0+
    private static final String CHANNEL_ID = "MusicServiceChannel";

    // Notification ID — must be > 0 for foreground service
    private static final int NOTIFICATION_ID = 1;

    @Override
    public void onCreate() {
        super.onCreate();
        createNotificationChannel();  // must create channel before showing notification
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        // Build a notification — foreground service MUST show one
        Notification notification = new NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle("Music Player")          // title line in notification
            .setContentText("Playing: Song Name")     // body text
            .setSmallIcon(R.drawable.ic_music)        // icon shown in status bar
            .build();                                 // create the Notification object

        // startForeground = promote this service to foreground
        // Shows persistent notification — OS won't kill it
        startForeground(NOTIFICATION_ID, notification);

        // Do actual work here (or in a thread)
        playMusic();

        return START_STICKY;
    }

    private void createNotificationChannel() {
        // NotificationChannels required on Android 8.0 (API 26+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

            NotificationChannel channel = new NotificationChannel(
                CHANNEL_ID,                              // unique channel id
                "Music Service",                         // channel name shown in Settings
                NotificationManager.IMPORTANCE_LOW       // low = no sound for this notification
            );

            // Register channel with system — done once, safe to call multiple times
            NotificationManager manager = getSystemService(NotificationManager.class);
            manager.createNotificationChannel(channel);
        }
    }

    private void playMusic() { /* ... */ }

    @Override
    public IBinder onBind(Intent intent) { return null; }
}
```

---

## PART 7: BROADCAST RECEIVERS (DEEP DIVE)

### Problem It Solves
Your app needs to react to system events — device is charging, WiFi connected, screen turned off — without being in the foreground. BroadcastReceivers listen for these system-wide events.

### Definition
A BroadcastReceiver is a component that responds to system-wide **broadcast announcements** (Intents sent to all apps). It wakes up when a matching broadcast is received.

### Two Ways to Register

```
Static Registration (Manifest):   Works even when app is not running.
                                  Limited in Android 8.0+ (only certain system broadcasts).

Dynamic Registration (Code):      Only works while the component is running.
                                  More flexibility — can register/unregister as needed.
```

### Static Receiver — Boot Completed

```java
package com.example.myapp;

import android.content.BroadcastReceiver;   // BroadcastReceiver = base class
import android.content.Context;              // Context = access to system resources
import android.content.Intent;
import android.util.Log;

// Extend BroadcastReceiver — must override onReceive()
public class BootReceiver extends BroadcastReceiver {

    private static final String TAG = "BootReceiver";

    // onReceive = called when a matching broadcast arrives
    // context = app context (do NOT use async operations — onReceive must complete quickly)
    // intent  = the broadcast intent (contains action and data)
    @Override
    public void onReceive(Context context, Intent intent) {

        // intent.getAction() = the broadcast action string that triggered us
        if (Intent.ACTION_BOOT_COMPLETED.equals(intent.getAction())) {
            Log.d(TAG, "Device booted — starting services");

            // Start our service when device boots
            Intent serviceIntent = new Intent(context, DownloadService.class);
            context.startService(serviceIntent);  // use context, not 'this' (not an Activity)
        }
    }
}
```

```xml
<!-- In Manifest — declare the receiver and what it listens for -->
<receiver
    android:name=".BootReceiver"
    android:exported="true">               <!-- true = other apps and system can send to it -->
    <intent-filter>
        <!-- The broadcast action this receiver responds to -->
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

### Dynamic Receiver — Battery and Network Changes

```java
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;       // IntentFilter = specifies which broadcasts to receive
import android.os.Bundle;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    // Declare receiver as a field so we can unregister it later
    private BroadcastReceiver batteryReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onStart() {
        super.onStart();

        // ── Create the receiver ────────────────────────────────────────
        batteryReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                // Check which broadcast triggered us
                String action = intent.getAction();

                if (Intent.ACTION_BATTERY_LOW.equals(action)) {
                    Toast.makeText(context, "Battery Low!", Toast.LENGTH_SHORT).show();
                } else if (Intent.ACTION_POWER_CONNECTED.equals(action)) {
                    Toast.makeText(context, "Charger Connected", Toast.LENGTH_SHORT).show();
                }
            }
        };

        // ── Create the filter — what broadcasts this receiver cares about ─
        IntentFilter filter = new IntentFilter();
        filter.addAction(Intent.ACTION_BATTERY_LOW);      // add each action we want
        filter.addAction(Intent.ACTION_POWER_CONNECTED);

        // ── Register the receiver with this Activity's context ─────────
        // registerReceiver = start listening for broadcasts
        registerReceiver(batteryReceiver, filter);
    }

    @Override
    protected void onStop() {
        super.onStop();

        // ── Unregister when Activity is not visible ────────────────────
        // MUST unregister — if you don't, you get memory leaks + crash after Activity destroyed
        if (batteryReceiver != null) {
            unregisterReceiver(batteryReceiver);
        }
    }
}
```

### Custom Broadcast — Your App Broadcasting to Itself

```java
// ── Sending a custom broadcast ─────────────────────────────────────────
Intent broadcastIntent = new Intent("com.example.myapp.DATA_UPDATED");
broadcastIntent.putExtra("count", 42);     // attach data to the broadcast
sendBroadcast(broadcastIntent);            // send to all registered receivers

// ── Receiving your custom broadcast ────────────────────────────────────
IntentFilter customFilter = new IntentFilter("com.example.myapp.DATA_UPDATED");

BroadcastReceiver customReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        int count = intent.getIntExtra("count", 0);
        // update UI with count
    }
};

registerReceiver(customReceiver, customFilter);
```

### Interview Q&A

**Q: What is the time limit for `onReceive()`?**
A: 10 seconds. If you need longer, start a Service from `onReceive()`.

**Q: Ordered vs Normal Broadcast?**
A: Normal = sent to all receivers simultaneously. Ordered = receivers get it one by one in priority order; can abort or modify.

---

## PART 8: PARCELABLE (DEEP DIVE)

### Problem It Solves
Intent extras can't directly pass custom objects like `User` or `Product`. You need to serialize the object. Android's `Parcelable` is **faster than Java Serializable** and preferred for Android.

### Definition
`Parcelable` is an Android interface for object serialization. It converts your object into a stream of bytes (a `Parcel`) so it can be passed between components via Intents or Bundles.

### Serializable vs Parcelable

| | Serializable | Parcelable |
|---|---|---|
| Source | Java standard | Android specific |
| Speed | Slow (uses reflection) | Fast (manual, no reflection) |
| Code | Just `implements Serializable` | Must implement write/create manually |
| Boilerplate | None | Yes (reduced by `@Parcelize` in Kotlin) |
| Use | Simple, small objects | Always prefer in Android |

### Implementing Parcelable — Every Line Explained

```java
package com.example.myapp;

import android.os.Parcel;          // Parcel = container for message data (byte buffer)
import android.os.Parcelable;      // Parcelable = interface your class must implement

// Implement Parcelable to allow this class to be passed via Intent
public class User implements Parcelable {

    // ── Fields ─────────────────────────────────────────────────────────
    private int id;
    private String name;
    private String email;
    private boolean isPremium;

    // ── Normal Constructor ──────────────────────────────────────────────
    public User(int id, String name, String email, boolean isPremium) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.isPremium = isPremium;
    }

    // ── Parcelable Constructor ──────────────────────────────────────────
    // Called by Android to RECREATE this object from a Parcel (during deserialization)
    // ORDER MUST MATCH writeToParcel() — read in same order as written
    protected User(Parcel in) {
        id        = in.readInt();       // read int (matches writeInt below)
        name      = in.readString();    // read String (matches writeString below)
        email     = in.readString();
        // readByte() returns 0 or 1 — convert to boolean
        isPremium = in.readByte() != 0;
    }

    // ── CREATOR ────────────────────────────────────────────────────────
    // Static field that Android uses to create instances of this Parcelable
    // Must be named exactly CREATOR with this type
    public static final Creator<User> CREATOR = new Creator<User>() {

        // createFromParcel = Android calls this to rebuild your object from the Parcel
        @Override
        public User createFromParcel(Parcel in) {
            return new User(in);   // calls the Parcelable constructor above
        }

        // newArray = creates an array of your type with given size
        @Override
        public User[] newArray(int size) {
            return new User[size];  // needed for arrays of Parcelable
        }
    };

    // ── describeContents ───────────────────────────────────────────────
    // Return 0 unless your Parcel contains a FileDescriptor (almost never)
    @Override
    public int describeContents() {
        return 0;
    }

    // ── writeToParcel ──────────────────────────────────────────────────
    // Serialize this object INTO a Parcel
    // dest  = the Parcel buffer we're writing into
    // flags = additional flags about how the object should be written (usually 0)
    // ORDER HERE must match ORDER in Parcelable constructor
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(id);            // write int field
        dest.writeString(name);       // write String field
        dest.writeString(email);      // write another String
        // boolean → write as byte (1=true, 0=false) because Parcel has no writeBoolean
        dest.writeByte((byte) (isPremium ? 1 : 0));
    }

    // ── Getters ─────────────────────────────────────────────────────────
    public int getId()        { return id; }
    public String getName()   { return name; }
    public String getEmail()  { return email; }
    public boolean isPremium(){ return isPremium; }
}
```

### Passing Parcelable via Intent

```java
// ── Sender (MainActivity) ──────────────────────────────────────────────
User user = new User(1, "John", "john@example.com", true);

Intent intent = new Intent(this, DetailActivity.class);

// putExtra with Parcelable — object is serialized into the Intent's Bundle
intent.putExtra("USER_DATA", user);

startActivity(intent);

// ── Receiver (DetailActivity) ──────────────────────────────────────────
Intent intent = getIntent();

// getParcelableExtra = deserialize User from Intent
// Android calls CREATOR.createFromParcel() internally
// "USER_DATA" must match the key used in putExtra
User user = intent.getParcelableExtra("USER_DATA");

if (user != null) {
    Log.d("TAG", "Received: " + user.getName());
}
```

### Passing ArrayList of Parcelable

```java
// ── Send list ──────────────────────────────────────────────────────────
ArrayList<User> users = new ArrayList<>();
users.add(new User(1, "John", "j@j.com", true));
users.add(new User(2, "Alice", "a@a.com", false));

intent.putParcelableArrayListExtra("USER_LIST", users);

// ── Receive list ───────────────────────────────────────────────────────
ArrayList<User> users = intent.getParcelableArrayListExtra("USER_LIST");
```

---

## ANDROID ARCHITECTURE — BIG PICTURE

```
┌─────────────────────────────────────────────────────┐
│                    USER INTERFACE                    │
│          Activity / Fragment / XML Layouts           │
├─────────────────────────────────────────────────────┤
│                    VIEW MODEL                        │
│       Survives rotation. Holds UI data.              │
├─────────────────────────────────────────────────────┤
│              REPOSITORY (data layer)                 │
│     Decides: fetch from network or local DB?         │
├──────────────────────┬──────────────────────────────┤
│    REMOTE DATA       │      LOCAL DATA               │
│  Retrofit / OkHttp   │   Room Database / SharedPrefs │
└──────────────────────┴──────────────────────────────┘
```

This is MVVM — Model View ViewModel. It's the official Google recommended architecture. You will see this in every production Android codebase. Day 3 covers Room and SharedPreferences. Day 4 covers Networking. Day 5 covers ViewModels.

---

## DAY 2 MASTER SUMMARY

```
Manifest:     Declares every component. No declaration = crash.
Activity:     One screen. Lifecycle: onCreate→onStart→onResume→onPause→onStop→onDestroy.
              Save state in onSaveInstanceState. Restore in onCreate.

Intent:       Explicit = specific class. Implicit = action + let OS decide.
              putExtra / getXxxExtra for data passing.

Fragment:     Reusable UI. Lives inside Activity. Has own lifecycle.
              Use onViewCreated for UI setup. Null views in onDestroyView.
              Factory method newInstance() for arguments.

Permissions:  Declare in Manifest. Request dangerous ones at runtime.
              registerForActivityResult + RequestPermission contract.

Service:      No UI. Started (fire and forget), Bound (two-way), Foreground (persistent).
              Runs on main thread — use new Thread() or Executor for work.

Broadcast:    Receive system events. Static (Manifest) or Dynamic (register/unregister).
              Must complete onReceive in < 10 seconds.

Parcelable:   Pass custom objects via Intent. Faster than Serializable.
              Implement: constructor(Parcel), writeToParcel(), CREATOR.
              Read order must match write order.
```

---

## DAY 2 CODING EXERCISES

Write these completely from scratch:

1. Activity with a counter — survives rotation using `onSaveInstanceState`
2. Two activities — first sends name + age via Intent, second displays them
3. Fragment with a button that sends the clicked item name back to Activity via interface
4. Camera permission flow — check, request, handle grant/deny
5. Started Service — receives a URL via Intent, logs "downloading URL" on background thread
6. BroadcastReceiver that toasts when charger is connected (dynamic registration)
7. `User` class implementing `Parcelable` with 4 fields — pass via Intent and read back

---

*Day 3: Android UI — XML Layouts, ConstraintLayout, RecyclerView, Adapters, ViewBinding*
