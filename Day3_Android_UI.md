# DAY 3 — ANDROID UI (DEEP DIVE)
### XML Layouts · ConstraintLayout · ViewBinding · RecyclerView · Adapter · DiffUtil · JSON Parsing

> Every line of code is commented. Format: Problem → Definition → How It Works → Code → Real Usage → Interview Q&A → Common Mistakes

---

## HOW ANDROID UI ACTUALLY RENDERS (Read First)

```
Your XML file (activity_main.xml)
          ↓
  LayoutInflater reads XML
          ↓
  Creates View objects in memory
          ↓
  View tree (parent → children)
          ↓
  Measure pass  → every View calculates its own size
          ↓
  Layout pass   → every View calculates its position
          ↓
  Draw pass     → Views draw themselves on Canvas
          ↓
  Screen shows result
```

Every UI element is a `View`. Every container is a `ViewGroup` (which extends `View`).
`ViewGroup` holds children. Nesting too deep → slow (measure/layout called recursively).

**Key Units:**
```
dp  (density-independent pixels) → use for sizes, margins, padding
    1dp = 1px on 160dpi screen, scales with screen density
    Always use dp for layout dimensions

sp  (scale-independent pixels) → use ONLY for text sizes
    Like dp but also respects user's font size setting in Accessibility

px  → NEVER use. Different screens have different densities.
```

---

## PART 1: XML LAYOUTS (DEEP DIVE)

### LinearLayout — Stack elements in a line

```xml
<!-- res/layout/activity_linear.xml -->

<!-- LinearLayout = arranges children in a single row (horizontal) or column (vertical) -->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"     <!-- fill the full screen width -->
    android:layout_height="match_parent"    <!-- fill the full screen height -->
    android:orientation="vertical"          <!-- stack children top to bottom -->
    android:padding="16dp"                  <!-- inner space between edges and children -->
    android:gravity="center_horizontal">    <!-- center children horizontally within this layout -->

    <!-- TextView — displays non-editable text -->
    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="match_parent" <!-- fill parent's width -->
        android:layout_height="wrap_content"<!-- height = just enough for content -->
        android:text="Student List"
        android:textSize="20sp"             <!-- text size in sp — respects accessibility -->
        android:textStyle="bold"            <!-- bold text -->
        android:textColor="#212121"         <!-- hex color — dark grey -->
        android:layout_marginBottom="16dp"  <!-- space below this view -->
        android:gravity="center" />         <!-- center text within the TextView -->

    <!-- EditText — user input field -->
    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter student name"   <!-- placeholder text shown when empty -->
        android:inputType="textPersonName"  <!-- keyboard type — shows name-optimized keyboard -->
        android:maxLines="1"               <!-- restrict to single line -->
        android:layout_marginBottom="8dp" />

    <!-- Button — tappable element -->
    <Button
        android:id="@+id/btnAdd"
        android:layout_width="wrap_content" <!-- width shrinks to fit the button text -->
        android:layout_height="wrap_content"
        android:text="Add Student"
        android:layout_gravity="end"        <!-- push this button to the right side -->
        android:layout_marginBottom="16dp" />

    <!-- layout_weight — divide remaining space proportionally -->
    <!-- Two views each with weight=1 → each gets 50% of remaining space -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">   <!-- children arranged left to right -->

        <Button
            android:id="@+id/btnSave"
            android:layout_width="0dp"      <!-- 0dp + weight = fill proportional space -->
            android:layout_height="wrap_content"
            android:layout_weight="1"       <!-- take 1 share of space -->
            android:text="Save"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/btnCancel"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"       <!-- take 1 share of space → 50/50 split -->
            android:text="Cancel" />

    </LinearLayout>

</LinearLayout>
```

---

### ConstraintLayout — The Most Powerful Layout

**Problem it solves:** LinearLayout needs nesting for complex UIs → slow. ConstraintLayout positions elements relative to each other in a **flat hierarchy** — no nesting needed, faster rendering.

**Key concept:** Every View must have horizontal AND vertical constraints, otherwise it collapses to position (0,0).

```xml
<!-- res/layout/activity_constraint.xml -->

<!-- ConstraintLayout = positions views by defining relationships between them -->
<!-- All views are direct children — NO nesting needed for complex layouts -->
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <!-- ─── Profile Image ─────────────────────────────────────────────── -->
    <ImageView
        android:id="@+id/ivAvatar"
        android:layout_width="80dp"         <!-- fixed 80dp square -->
        android:layout_height="80dp"
        android:src="@drawable/ic_person"   <!-- drawable resource -->
        android:scaleType="centerCrop"      <!-- crop image to fill bounds, maintain ratio -->

        <!-- HORIZONTAL constraint: left edge touches parent's left -->
        app:layout_constraintStart_toStartOf="parent"

        <!-- VERTICAL constraint: top edge touches parent's top -->
        app:layout_constraintTop_toTopOf="parent"

        android:layout_marginTop="24dp"
        android:layout_marginStart="0dp" />

    <!-- ─── Name TextView ─────────────────────────────────────────────── -->
    <TextView
        android:id="@+id/tvName"
        android:layout_width="0dp"          <!-- 0dp with horizontal constraints = fill space -->
        android:layout_height="wrap_content"
        android:text="John Doe"
        android:textSize="18sp"
        android:textStyle="bold"

        <!-- Constrain LEFT edge to RIGHT edge of avatar + 12dp gap -->
        app:layout_constraintStart_toEndOf="@id/ivAvatar"
        android:layout_marginStart="12dp"

        <!-- Constrain RIGHT edge to parent's right -->
        app:layout_constraintEnd_toEndOf="parent"

        <!-- Constrain TOP edge to TOP edge of avatar (vertically aligned) -->
        app:layout_constraintTop_toTopOf="@id/ivAvatar" />

    <!-- ─── Email TextView ─────────────────────────────────────────────── -->
    <TextView
        android:id="@+id/tvEmail"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="john@example.com"
        android:textSize="14sp"
        android:textColor="#757575"         <!-- lighter grey for secondary info -->

        app:layout_constraintStart_toEndOf="@id/ivAvatar"
        android:layout_marginStart="12dp"
        app:layout_constraintEnd_toEndOf="parent"

        <!-- Constrain TOP to BOTTOM of tvName — email sits below name -->
        app:layout_constraintTop_toBottomOf="@id/tvName"
        android:layout_marginTop="4dp" />

    <!-- ─── Divider Line ─────────────────────────────────────────────── -->
    <View
        android:id="@+id/divider"
        android:layout_width="0dp"          <!-- 0dp + both horizontal constraints = full width -->
        android:layout_height="1dp"         <!-- 1dp thin line -->
        android:background="#E0E0E0"        <!-- light grey color -->

        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"

        <!-- Place divider below the avatar -->
        app:layout_constraintTop_toBottomOf="@id/ivAvatar"
        android:layout_marginTop="16dp" />

    <!-- ─── Bio Section ─────────────────────────────────────────────── -->
    <TextView
        android:id="@+id/tvBio"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Bio goes here..."
        android:textSize="14sp"

        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@id/divider"
        android:layout_marginTop="16dp" />

    <!-- ─── Button centered horizontally ─────────────────────────────── -->
    <Button
        android:id="@+id/btnEdit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Edit Profile"

        <!-- Constrain both left and right to parent → centers horizontally -->
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"

        app:layout_constraintTop_toBottomOf="@id/tvBio"
        android:layout_marginTop="24dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### ConstraintLayout — Chains and Guidelines

```xml
<!-- CHAIN: links multiple views together for even distribution -->
<Button android:id="@+id/btn1" ...
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toStartOf="@id/btn2"   <!-- right of btn1 → left of btn2 -->
    app:layout_constraintHorizontal_chainStyle="spread" />  <!-- evenly spread -->

<Button android:id="@+id/btn2" ...
    app:layout_constraintStart_toEndOf="@id/btn1"   <!-- left of btn2 → right of btn1 -->
    app:layout_constraintEnd_toEndOf="parent" />

<!-- GUIDELINE: invisible line used as anchor for other views -->
<androidx.constraintlayout.widget.Guideline
    android:id="@+id/guideline"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"          <!-- vertical line -->
    app:layout_constraintGuide_percent="0.5" />  <!-- at 50% of parent width -->

<!-- Views can constrain to this guideline -->
<Button ...
    app:layout_constraintEnd_toStartOf="@id/guideline" />  <!-- button ends at center -->
```

---

## PART 2: VIEWBINDING (DEEP DIVE)

### Problem It Solves
`findViewById()` returns a generic `View` — you must cast it. Typos in IDs compile fine but crash at runtime. `ViewBinding` generates a typed class per XML file — **no casts, no NullPointerException, compile-time safety**.

### Definition
ViewBinding is a feature that generates a binding class for each XML layout file. Each field in the binding class corresponds directly to a View with an ID in the XML — type-safe, null-safe.

### Enable ViewBinding

```groovy
// In app/build.gradle — inside android {} block
android {
    ...
    buildFeatures {
        viewBinding true    // enable ViewBinding — generates binding classes for all layouts
    }
}
// Sync project after adding this
```

### Naming Rule
```
XML file name           →  Generated class name
activity_main.xml       →  ActivityMainBinding
fragment_home.xml       →  FragmentHomeBinding
item_user.xml           →  ItemUserBinding
```

### ViewBinding in Activity

```java
package com.example.myapp;

import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import com.example.myapp.databinding.ActivityMainBinding;  // auto-generated — import your package

public class MainActivity extends AppCompatActivity {

    // Declare binding as a field — nullable (null after onDestroy)
    private ActivityMainBinding binding;  // auto-generated class for activity_main.xml

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // inflate() = read XML + create all View objects + return binding object
        // getLayoutInflater() = returns the inflater attached to this Activity
        binding = ActivityMainBinding.inflate(getLayoutInflater());

        // binding.getRoot() = returns the root View of the layout (the outermost element)
        // setContentView with the root view sets this as the Activity's screen
        setContentView(binding.getRoot());

        // ── Access Views DIRECTLY through binding — no findViewById, no cast ──
        // XML id: android:id="@+id/tvTitle"
        // Binding field: binding.tvTitle (camelCase conversion of the id)
        binding.tvTitle.setText("Hello ViewBinding");

        // XML id: android:id="@+id/btnSubmit"
        // Binding field: binding.btnSubmit
        binding.btnSubmit.setOnClickListener(v -> {
            // XML id: android:id="@+id/etName"
            String name = binding.etName.getText().toString().trim();
            // .trim() removes whitespace from start and end of string

            if (name.isEmpty()) {
                binding.etName.setError("Name cannot be empty");
                // setError shows a red error popup below the EditText
                return;  // early return — stop further execution
            }

            binding.tvTitle.setText("Welcome, " + name);
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;  // release binding reference to prevent memory leak
                         // Activity is gone — binding holds view references — must null them
    }
}
```

### ViewBinding in Fragment

```java
package com.example.myapp;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import androidx.fragment.app.Fragment;
import com.example.myapp.databinding.FragmentHomeBinding;  // generated from fragment_home.xml

public class HomeFragment extends Fragment {

    // _binding = backing field (nullable) — the real storage
    // binding  = non-null accessor — only valid between onCreateView and onDestroyView
    // This pattern prevents accidental use outside the view's lifetime
    private FragmentHomeBinding _binding;

    // Safe accessor — throws if called when view doesn't exist
    private FragmentHomeBinding getBinding() {
        if (_binding == null) throw new IllegalStateException("Binding accessed outside view lifetime");
        return _binding;
    }

    @Override
    public View onCreateView(LayoutInflater inflater,
                             ViewGroup container,
                             Bundle savedInstanceState) {

        // inflate() for Fragment needs 3 params:
        // inflater   = passed in from Android
        // container  = parent ViewGroup (needed for correct LayoutParams)
        // false      = don't attach yet — FragmentManager does that
        _binding = FragmentHomeBinding.inflate(inflater, container, false);

        return _binding.getRoot();  // return root View to Android (required)
    }

    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        // Safe to use _binding here — view exists
        _binding.tvGreeting.setText("Hello from Fragment");

        _binding.btnAction.setOnClickListener(v -> {
            _binding.tvGreeting.setText("Clicked!");
        });
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        _binding = null;    // CRITICAL — Fragment outlives its View in backstack
                            // Not nulling = View leak = memory leak
    }
}
```

### Why Not DataBinding?
DataBinding is more powerful (two-way binding) but heavier and slower to compile. ViewBinding is simpler, faster, and covers 95% of needs.

---

## PART 3: RECYCLERVIEW (THE MOST IMPORTANT UI COMPONENT)

### Problem It Solves
Imagine displaying 1000 users in a scrolling list. Creating 1000 Views at once → out of memory. `ListView` (old) tried to recycle but had no strict pattern. `RecyclerView` enforces **strict recycling** — only creates ~10-15 Views (what fits on screen), recycles them as you scroll.

### Definition
RecyclerView is a flexible, high-performance widget for displaying large datasets in a scrollable list, grid, or custom layout by **recycling Views that scroll off-screen** instead of creating new ones.

### How RecyclerView Works Internally

```
Screen shows items 0-9 (10 visible at a time)

User scrolls DOWN:
  Item 0 goes off-screen top
    → RecyclerView takes that View
    → Puts it in the "Recycle Pool"
  Item 10 comes on-screen bottom
    → RecyclerView takes View from Pool (no new View created!)
    → Adapter.onBindViewHolder() fills it with Item 10's data
    → Shows on screen

This is why it's called RecyclerView — it recycles Views.
Without recycling: 1000 items = 1000 Views = OutOfMemoryError
With recycling:    1000 items = ~15 Views  = smooth scrolling
```

### RecyclerView Architecture

```
RecyclerView                → the scrollable container
    │
    ├── LayoutManager       → HOW to arrange items (list, grid, staggered)
    │     ├── LinearLayoutManager      → vertical or horizontal list
    │     ├── GridLayoutManager        → grid (N columns)
    │     └── StaggeredGridLayoutManager → Pinterest-style
    │
    └── Adapter             → WHAT to show (provides Views and data)
          │
          └── ViewHolder    → represents ONE item's View (avoids repeated findViewById)
```

### Step-by-Step Implementation

#### Step 1 — Add dependency (already included in most modern projects)
```groovy
// In app/build.gradle
dependencies {
    implementation 'androidx.recyclerview:recyclerview:1.3.1'
}
```

#### Step 2 — Create the data model
```java
package com.example.myapp;

// Plain Java class — represents one item in the list
// No Android imports needed here — pure data
public class Student {

    private int id;         // unique identifier for this student
    private String name;    // student's full name
    private String course;  // enrolled course name
    private double gpa;     // grade point average
    private boolean isActive; // is the student currently enrolled

    // Constructor — all fields required
    public Student(int id, String name, String course, double gpa, boolean isActive) {
        this.id = id;
        this.name = name;
        this.course = course;
        this.gpa = gpa;
        this.isActive = isActive;
    }

    // ── Getters — ViewHolder reads these to populate UI ────────────────
    public int getId()        { return id; }
    public String getName()   { return name; }
    public String getCourse() { return course; }
    public double getGpa()    { return gpa; }
    public boolean isActive() { return isActive; }

    // toString — useful for debugging
    @Override
    public String toString() {
        return "Student{id=" + id + ", name='" + name + "'}";
    }
}
```

#### Step 3 — Create item layout XML
```xml
<!-- res/layout/item_student.xml -->
<!-- This XML describes ONE row in the RecyclerView list -->

<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"     <!-- card takes full width of parent -->
    android:layout_height="wrap_content"    <!-- height = content height -->
    android:layout_margin="8dp"             <!-- space between cards -->
    app:cardCornerRadius="8dp"              <!-- rounded corners -->
    app:cardElevation="4dp">               <!-- drop shadow depth -->

    <!-- Inner layout with padding -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp"
        android:gravity="center_vertical"> <!-- center children vertically -->

        <!-- Status indicator dot — green if active, grey if not -->
        <View
            android:id="@+id/viewStatus"
            android:layout_width="12dp"
            android:layout_height="12dp"
            android:background="@drawable/circle_indicator"
            android:layout_marginEnd="12dp" />

        <!-- Text section — takes remaining space -->
        <LinearLayout
            android:layout_width="0dp"          <!-- 0dp + weight = fill remaining space -->
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical">

            <TextView
                android:id="@+id/tvStudentName"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="#212121" />

            <TextView
                android:id="@+id/tvCourse"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:textSize="13sp"
                android:textColor="#757575"
                android:layout_marginTop="2dp" />

        </LinearLayout>

        <!-- GPA badge on right side -->
        <TextView
            android:id="@+id/tvGpa"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="14sp"
            android:textStyle="bold"
            android:textColor="#1976D2"         <!-- blue color -->
            android:layout_marginStart="8dp" />

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

#### Step 4 — Create the Adapter (The Core)

```java
package com.example.myapp;

import android.content.Context;
import android.graphics.Color;
import android.view.LayoutInflater;          // converts XML to View objects
import android.view.View;
import android.view.ViewGroup;               // parent container passed during item creation
import androidx.annotation.NonNull;          // @NonNull = compiler warns if null is passed
import androidx.recyclerview.widget.RecyclerView;
import com.example.myapp.databinding.ItemStudentBinding;  // binding for item_student.xml
import java.util.ArrayList;
import java.util.List;

// RecyclerView.Adapter<VH> — VH is the ViewHolder type this adapter uses
// Adapter is the bridge between your data (List<Student>) and the RecyclerView
public class StudentAdapter extends RecyclerView.Adapter<StudentAdapter.StudentViewHolder> {

    // ── Interface for click callbacks ───────────────────────────────────
    // Allows Activity/Fragment to respond to item clicks without adapter knowing about them
    public interface OnStudentClickListener {
        void onStudentClick(Student student);       // user tapped an item
        void onStudentLongClick(Student student);   // user long-pressed an item
    }

    // ── Fields ─────────────────────────────────────────────────────────
    private List<Student> studentList;              // the data to display
    private OnStudentClickListener clickListener;   // callback to caller
    private Context context;                        // needed for color resources, inflation

    // ── Constructor ─────────────────────────────────────────────────────
    public StudentAdapter(Context context, OnStudentClickListener listener) {
        this.context = context;                     // store context reference
        this.clickListener = listener;              // store click callback
        this.studentList = new ArrayList<>();       // initialize empty — data set later
    }

    // ═══════════════════════════════════════════════════════════════════
    // VIEWHOLDER — Represents ONE item's View references
    // static = doesn't need outer class reference → avoids memory leak
    // ═══════════════════════════════════════════════════════════════════
    static class StudentViewHolder extends RecyclerView.ViewHolder {

        // Store binding to the item layout — gives typed access to all Views
        private ItemStudentBinding binding;

        // Constructor receives the root View (required by RecyclerView.ViewHolder)
        // We get this root View from our binding
        StudentViewHolder(ItemStudentBinding binding) {
            super(binding.getRoot());       // pass root View to parent ViewHolder
            this.binding = binding;         // store binding for use in bind()
        }

        // bind() = fill this ViewHolder's Views with data from one Student object
        // Called by onBindViewHolder() every time this ViewHolder is (re)used
        void bind(Student student, OnStudentClickListener listener) {

            // Set student name text
            binding.tvStudentName.setText(student.getName());

            // Set course text
            binding.tvCourse.setText(student.getCourse());

            // Format GPA to 1 decimal place: 3.85 → "GPA: 3.9"
            binding.tvGpa.setText(String.format("GPA: %.1f", student.getGpa()));

            // Set status dot color based on active status
            // Color.parseColor converts hex string to Android Color int
            if (student.isActive()) {
                binding.viewStatus.setBackgroundColor(Color.parseColor("#4CAF50")); // green
            } else {
                binding.viewStatus.setBackgroundColor(Color.parseColor("#9E9E9E")); // grey
            }

            // ── Click listener on the whole card ───────────────────────
            // getRoot() = the CardView — clicking anywhere on the item triggers this
            binding.getRoot().setOnClickListener(v -> {
                // Notify caller via the interface
                if (listener != null) {
                    listener.onStudentClick(student);
                }
            });

            // ── Long press listener ─────────────────────────────────────
            binding.getRoot().setOnLongClickListener(v -> {
                if (listener != null) {
                    listener.onStudentLongClick(student);
                }
                return true;    // true = event consumed — don't trigger further handlers
            });
        }
    }

    // ═══════════════════════════════════════════════════════════════════
    // REQUIRED OVERRIDES — These 3 methods are mandatory for every Adapter
    // ═══════════════════════════════════════════════════════════════════

    // onCreateViewHolder — called when RecyclerView needs a NEW ViewHolder
    // NOT called for every item — only called when pool is empty (typically ~15 times)
    // parent  = the RecyclerView itself (parent of the item Views)
    // viewType = for multiple view types (see section below)
    @NonNull
    @Override
    public StudentViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {

        // LayoutInflater.from(context) = get the system layout inflater
        // inflate() = parse item_student.xml and create View hierarchy
        // parent = passed for correct LayoutParams calculation
        // false = do NOT attach to parent yet — RecyclerView does that
        ItemStudentBinding binding = ItemStudentBinding.inflate(
            LayoutInflater.from(parent.getContext()),   // inflater from context
            parent,                                     // parent ViewGroup
            false                                       // don't attach yet
        );

        return new StudentViewHolder(binding);  // wrap in ViewHolder and return
    }

    // onBindViewHolder — called when a recycled ViewHolder needs to show NEW data
    // Called MANY times — every time an item scrolls into view
    // holder   = the recycled ViewHolder (may have previously shown a different Student)
    // position = index in the list (0-based)
    @Override
    public void onBindViewHolder(@NonNull StudentViewHolder holder, int position) {

        // Get the Student object for this position
        Student student = studentList.get(position);

        // Tell the ViewHolder to fill its Views with this student's data
        holder.bind(student, clickListener);
    }

    // getItemCount — tells RecyclerView how many items exist
    // RecyclerView calls this to decide how long the scroll is
    @Override
    public int getItemCount() {
        return studentList.size();  // return current list size
    }

    // ═══════════════════════════════════════════════════════════════════
    // DATA UPDATE METHODS
    // ═══════════════════════════════════════════════════════════════════

    // Replace the entire list and refresh the UI
    public void setData(List<Student> newList) {
        studentList.clear();            // remove old data
        studentList.addAll(newList);    // add all new items
        notifyDataSetChanged();         // tell RecyclerView: everything changed, redraw all
        // NOTE: notifyDataSetChanged is the sledgehammer — use DiffUtil for efficiency
    }

    // Add a single item and animate its appearance
    public void addStudent(Student student) {
        studentList.add(student);                           // add to list
        notifyItemInserted(studentList.size() - 1);         // tell RecyclerView: item added at end
        // notifyItemInserted = smooth insertion animation (better UX than notifyDataSetChanged)
    }

    // Remove an item at position and animate its removal
    public void removeStudent(int position) {
        studentList.remove(position);           // remove from list
        notifyItemRemoved(position);            // tell RecyclerView: item removed at this position
    }

    // Update a single item (e.g., GPA changed)
    public void updateStudent(int position, Student updated) {
        studentList.set(position, updated);     // replace item in list
        notifyItemChanged(position);            // tell RecyclerView: this one item changed
    }
}
```

#### Step 5 — Set Up RecyclerView in Activity

```java
package com.example.myapp;

import android.os.Bundle;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import com.example.myapp.databinding.ActivityMainBinding;
import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity
        implements StudentAdapter.OnStudentClickListener { // implement click interface

    private ActivityMainBinding binding;     // ViewBinding for activity_main.xml
    private StudentAdapter adapter;          // our custom adapter
    private List<Student> studentList;       // data list

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        setupRecyclerView();   // configure the RecyclerView
        loadData();            // populate with initial data
    }

    private void setupRecyclerView() {

        // ── Create Adapter ──────────────────────────────────────────────
        // Pass context and this Activity as the click listener
        adapter = new StudentAdapter(this, this);

        // ── Create LayoutManager ────────────────────────────────────────
        // LinearLayoutManager = displays items as a vertical list (default)
        // VERTICAL = scroll direction
        // false = don't reverse the list (false = top to bottom)
        LinearLayoutManager layoutManager =
            new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);

        // ── Configure RecyclerView ──────────────────────────────────────
        // setLayoutManager = tell RecyclerView HOW to arrange items
        binding.recyclerView.setLayoutManager(layoutManager);

        // setAdapter = tell RecyclerView WHERE to get its data and Views
        binding.recyclerView.setAdapter(adapter);

        // setHasFixedSize(true) = optimization: if item size doesn't change, skip re-measuring
        // Only set true if all items are the same height
        binding.recyclerView.setHasFixedSize(true);
    }

    private void loadData() {
        studentList = new ArrayList<>();

        // Create sample students
        studentList.add(new Student(1, "John Doe",    "Computer Science", 3.8, true));
        studentList.add(new Student(2, "Alice Smith", "Electronics",      3.5, true));
        studentList.add(new Student(3, "Bob Jones",   "Mechanical",       3.2, false));
        studentList.add(new Student(4, "Sara Lee",    "Computer Science", 3.9, true));
        studentList.add(new Student(5, "Mike Chen",   "Civil",            2.9, false));

        // Pass data to adapter — triggers onBindViewHolder for visible items
        adapter.setData(studentList);
    }

    // ── OnStudentClickListener implementation ───────────────────────────
    @Override
    public void onStudentClick(Student student) {
        // Show a Toast message with student name
        // Toast = brief pop-up message that auto-dismisses
        Toast.makeText(this,
            "Clicked: " + student.getName(),
            Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onStudentLongClick(Student student) {
        Toast.makeText(this,
            "Long press: " + student.getName(),
            Toast.LENGTH_SHORT).show();
    }
}
```

#### Activity XML (contains RecyclerView)
```xml
<!-- res/layout/activity_main.xml -->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- RecyclerView — the scrollable list container -->
    <!-- android:id used to find it in code -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:clipToPadding="false"      <!-- items can draw in padding area while scrolling -->
        android:padding="4dp" />

</LinearLayout>
```

---

## PART 4: DIFFUTIL (EFFICIENT LIST UPDATES)

### Problem It Solves
`notifyDataSetChanged()` redraws **everything** even if only one item changed. No animation, bad performance. DiffUtil computes the **minimum set of changes** between two lists and animates only what actually changed.

### Definition
`DiffUtil` is a utility class that calculates the difference between two lists using the Myers diff algorithm and dispatches targeted updates to the RecyclerView adapter.

```java
package com.example.myapp;

import androidx.recyclerview.widget.DiffUtil;
import java.util.List;

// DiffUtil.Callback tells DiffUtil HOW to compare your items
public class StudentDiffCallback extends DiffUtil.Callback {

    private final List<Student> oldList;    // the list currently shown
    private final List<Student> newList;    // the new list to show

    // Constructor — receives both lists
    public StudentDiffCallback(List<Student> oldList, List<Student> newList) {
        this.oldList = oldList;
        this.newList = newList;
    }

    // getOldListSize — how many items in the old list
    @Override
    public int getOldListSize() {
        return oldList.size();
    }

    // getNewListSize — how many items in the new list
    @Override
    public int getNewListSize() {
        return newList.size();
    }

    // areItemsTheSame — are these two items the SAME entity?
    // Use a unique identifier (like database ID) — NOT the content
    // DiffUtil uses this to detect moves (item moved position)
    @Override
    public boolean areItemsTheSame(int oldPos, int newPos) {
        // Compare IDs — same ID means same student, even if data changed
        return oldList.get(oldPos).getId() == newList.get(newPos).getId();
    }

    // areContentsTheSame — if it's the same item, has the content changed?
    // Only called if areItemsTheSame returned true
    // If this returns false → onBindViewHolder called for this item
    @Override
    public boolean areContentsTheSame(int oldPos, int newPos) {
        Student old = oldList.get(oldPos);
        Student neo = newList.get(newPos);
        // Compare every field that's displayed in the UI
        return old.getName().equals(neo.getName())
            && old.getCourse().equals(neo.getCourse())
            && old.getGpa() == neo.getGpa()
            && old.isActive() == neo.isActive();
    }
}
```

### Using DiffUtil in Adapter

```java
// Replace the naive setData() with DiffUtil-powered version in StudentAdapter:

public void setData(List<Student> newList) {

    // Create callback — compares old list (current) with new list
    StudentDiffCallback callback = new StudentDiffCallback(studentList, newList);

    // calculateDiff = run the diff algorithm (can be run on background thread for large lists)
    // Returns a DiffResult containing all the changes
    DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(callback);

    // Update our data list
    studentList.clear();
    studentList.addAll(newList);

    // dispatchUpdatesTo = apply the calculated changes to the RecyclerView
    // This triggers: notifyItemInserted, notifyItemRemoved, notifyItemChanged as needed
    // Items animate smoothly — only changed items update
    diffResult.dispatchUpdatesTo(this);   // 'this' = the adapter
}
```

---

## PART 5: MULTIPLE VIEWHOLDER TYPES (ADVANCED)

### Problem It Solves
Real lists have heterogeneous items — headers, footers, ads, different card styles. RecyclerView supports multiple ViewHolder types natively.

```java
// Adapter with Header + Student items
public class MultiTypeAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

    // ── Item type constants ─────────────────────────────────────────────
    private static final int TYPE_HEADER  = 0;   // int constant for header type
    private static final int TYPE_STUDENT = 1;   // int constant for student type

    private String headerTitle;          // data for header
    private List<Student> students;      // data for student rows

    public MultiTypeAdapter(String headerTitle, List<Student> students) {
        this.headerTitle = headerTitle;
        this.students = students;
    }

    // ── getItemViewType ─────────────────────────────────────────────────
    // Called before onCreateViewHolder — RecyclerView asks "what type is item at position?"
    @Override
    public int getItemViewType(int position) {
        if (position == 0) {
            return TYPE_HEADER;   // position 0 = header
        } else {
            return TYPE_STUDENT;  // all others = student rows
        }
    }

    // ── onCreateViewHolder ──────────────────────────────────────────────
    // viewType = value returned by getItemViewType
    // Create DIFFERENT ViewHolders based on type
    @NonNull
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {

        LayoutInflater inflater = LayoutInflater.from(parent.getContext());

        if (viewType == TYPE_HEADER) {
            // Inflate header layout for header items
            View view = inflater.inflate(R.layout.item_header, parent, false);
            return new HeaderViewHolder(view);    // return header-specific ViewHolder

        } else {
            // Inflate student layout for student items
            ItemStudentBinding binding = ItemStudentBinding.inflate(inflater, parent, false);
            return new StudentViewHolder(binding);  // return student-specific ViewHolder
        }
    }

    // ── onBindViewHolder ────────────────────────────────────────────────
    // Cast to the right ViewHolder type and bind data
    @Override
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, int position) {

        if (holder instanceof HeaderViewHolder) {
            // Cast to HeaderViewHolder — safe because getItemViewType returned TYPE_HEADER
            ((HeaderViewHolder) holder).bind(headerTitle);

        } else if (holder instanceof StudentViewHolder) {
            // Adjust position: subtract 1 for the header at position 0
            Student student = students.get(position - 1);
            ((StudentViewHolder) holder).bind(student, null);
        }
    }

    // ── getItemCount ────────────────────────────────────────────────────
    @Override
    public int getItemCount() {
        return students.size() + 1;   // +1 for the header item
    }

    // ── ViewHolder for Header ───────────────────────────────────────────
    static class HeaderViewHolder extends RecyclerView.ViewHolder {
        private TextView tvHeader;  // direct view reference (no binding for simplicity)

        HeaderViewHolder(View itemView) {
            super(itemView);
            tvHeader = itemView.findViewById(R.id.tvHeader);
        }

        void bind(String title) {
            tvHeader.setText(title);
        }
    }
}
```

---

## PART 6: GRID LAYOUT AND DECORATION

```java
// ── GridLayoutManager — N-column grid ──────────────────────────────────
// 2 = number of columns
GridLayoutManager gridLayoutManager = new GridLayoutManager(this, 2);

binding.recyclerView.setLayoutManager(gridLayoutManager);

// ── Horizontal list ─────────────────────────────────────────────────────
LinearLayoutManager horizontal = new LinearLayoutManager(
    this,
    LinearLayoutManager.HORIZONTAL,   // scroll direction = horizontal
    false
);
binding.recyclerView.setLayoutManager(horizontal);

// ── ItemDecoration — add spacing between items ───────────────────────────
// DividerItemDecoration adds a line between each item
DividerItemDecoration decoration = new DividerItemDecoration(
    this,
    DividerItemDecoration.VERTICAL     // dividers between vertical items
);
binding.recyclerView.addItemDecoration(decoration);

// ── Custom spacing decoration ────────────────────────────────────────────
class SpaceDecoration extends RecyclerView.ItemDecoration {

    private int space;  // space in pixels between items

    SpaceDecoration(int spaceDp) {
        // Convert dp to pixels using display density
        this.space = (int) (spaceDp * Resources.getSystem().getDisplayMetrics().density);
    }

    // getItemOffsets — called for each item; set the margins around it
    // outRect = the margin rect we're filling in (top/bottom/left/right)
    // view     = the item View
    // parent   = the RecyclerView
    // state    = RecyclerView state
    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent,
                               RecyclerView.State state) {
        outRect.bottom = space;   // add space below each item
        outRect.left   = space;
        outRect.right  = space;

        // Only add top space for the first item
        if (parent.getChildAdapterPosition(view) == 0) {
            outRect.top = space;
        }
    }
}

binding.recyclerView.addItemDecoration(new SpaceDecoration(8));
```

---

## PART 7: JSON PARSING (DEEP DIVE)

### Problem It Solves
Data from servers arrives as a JSON string. You need to convert it to Java objects to use in your RecyclerView. And you need to convert Java objects back to JSON to send to the server.

### Definition
JSON (JavaScript Object Notation) is a text-based data format. Parsing = converting JSON text → Java objects. Serializing = converting Java objects → JSON text.

### JSON Basics

```json
// JSON Object — key-value pairs in curly braces
{
  "id": 101,
  "name": "John Doe",
  "gpa": 3.8,
  "isActive": true,
  "course": "Computer Science"
}

// JSON Array — list of values in square brackets
[
  { "id": 101, "name": "John" },
  { "id": 102, "name": "Alice" }
]

// Nested JSON
{
  "student": {
    "id": 101,
    "name": "John",
    "address": {
      "city": "Chennai",
      "state": "Tamil Nadu"
    }
  },
  "grades": [90, 85, 92, 88]
}
```

### Way 1 — Manual Parsing (org.json — built into Android, no library needed)

```java
import org.json.JSONArray;      // JSONArray = parses JSON arrays [ ]
import org.json.JSONException;  // JSONException = thrown if parsing fails (checked exception)
import org.json.JSONObject;     // JSONObject = parses JSON objects { }

public class JsonParser {

    // Parse a single Student from JSON string
    public Student parseStudent(String jsonString) throws JSONException {

        // JSONObject constructor — parses the JSON string into a Java object
        // Throws JSONException if the string is not valid JSON
        JSONObject json = new JSONObject(jsonString);

        // getInt("id") = read the value for key "id" as an integer
        int id = json.getInt("id");

        // getString("name") = read as String
        String name = json.getString("name");

        // getString with null check using optString (returns default if key missing)
        // optString = returns the default if key not found (no exception)
        String course = json.optString("course", "Unknown");

        // getDouble = read floating point value
        double gpa = json.getDouble("gpa");

        // getBoolean = read true/false value
        boolean isActive = json.getBoolean("isActive");

        // Construct and return the Student object with parsed values
        return new Student(id, name, course, gpa, isActive);
    }

    // Parse a list of Students from JSON array string
    public List<Student> parseStudentList(String jsonArrayString) throws JSONException {

        List<Student> students = new ArrayList<>();

        // JSONArray constructor — parses [ {...}, {...}, ... ]
        JSONArray jsonArray = new JSONArray(jsonArrayString);

        // Loop through each element in the array
        for (int i = 0; i < jsonArray.length(); i++) {

            // getJSONObject(i) = get the i-th element as a JSONObject
            JSONObject studentJson = jsonArray.getJSONObject(i);

            int id       = studentJson.getInt("id");
            String name  = studentJson.getString("name");
            String course= studentJson.optString("course", "");
            double gpa   = studentJson.optDouble("gpa", 0.0);   // optDouble = default 0.0 if missing
            boolean active = studentJson.optBoolean("isActive", true);

            // Add parsed Student to the list
            students.add(new Student(id, name, course, gpa, active));
        }

        return students;
    }

    // Build JSON from a Student object (serialize)
    public String studentToJson(Student student) throws JSONException {

        JSONObject json = new JSONObject();   // create empty JSONObject

        // put() = add a key-value pair to the JSON object
        json.put("id", student.getId());
        json.put("name", student.getName());
        json.put("course", student.getCourse());
        json.put("gpa", student.getGpa());
        json.put("isActive", student.isActive());

        // toString() = convert JSONObject to JSON string
        return json.toString();
    }

    // Build nested JSON
    public String buildRequestJson(Student student, String token) throws JSONException {

        JSONObject root = new JSONObject();

        // Nested object
        JSONObject studentObj = new JSONObject();
        studentObj.put("id", student.getId());
        studentObj.put("name", student.getName());

        root.put("student", studentObj);   // nest studentObj inside root
        root.put("authToken", token);

        // Array of values
        JSONArray subjects = new JSONArray();
        subjects.put("Math");      // put() adds to the array
        subjects.put("Physics");
        subjects.put("Java");

        root.put("subjects", subjects);    // nest array inside root

        return root.toString(2);   // toString(2) = pretty-print with 2-space indent
    }
}
```

### Way 2 — Gson (Google's Library, Industry Standard)

```groovy
// In app/build.gradle
dependencies {
    implementation 'com.google.code.gson:gson:2.10.1'  // Gson library
}
```

```java
import com.google.gson.Gson;            // main Gson class
import com.google.gson.GsonBuilder;     // builder for custom Gson configuration
import com.google.gson.reflect.TypeToken; // TypeToken = captures generic type info at runtime
import java.lang.reflect.Type;

// ── Annotate your model with @SerializedName ───────────────────────────
// @SerializedName tells Gson which JSON key maps to this field
// If field name matches JSON key exactly, annotation is optional
public class Student {

    @SerializedName("id")           // JSON key "id" → this field
    private int id;

    @SerializedName("name")         // JSON key "name" → this field
    private String name;

    @SerializedName("course_name")  // JSON key "course_name" → field named "course"
    private String course;          // field name doesn't need to match JSON key

    @SerializedName("gpa")
    private double gpa;

    @SerializedName("is_active")    // snake_case in JSON → camelCase in Java
    private boolean isActive;

    // Constructors, getters... (same as before)
}

public class GsonExample {

    // Create Gson instance — reuse this, don't recreate every time
    private static final Gson gson = new GsonBuilder()
        .setPrettyPrinting()            // format output with line breaks and indentation
        .serializeNulls()               // include null fields in output (default: exclude)
        .create();                      // build the Gson instance

    // ── JSON String → Java Object (Deserialization) ──────────────────
    public Student jsonToStudent(String json) {
        // fromJson = parse JSON string and create a Student object
        // Student.class = tells Gson the target type
        return gson.fromJson(json, Student.class);
    }

    // ── Java Object → JSON String (Serialization) ────────────────────
    public String studentToJson(Student student) {
        // toJson = convert student object to JSON string
        return gson.toJson(student);
    }

    // ── JSON Array → List<Student> ────────────────────────────────────
    public List<Student> jsonToStudentList(String json) {
        // Problem: Gson can't know generic type List<Student> at runtime due to type erasure
        // Solution: TypeToken captures the generic type at compile time
        Type listType = new TypeToken<List<Student>>(){}.getType();
        // getType() = extracts the runtime type token: "List<Student>"

        // fromJson with the type token — Gson knows to make a List of Students
        return gson.fromJson(json, listType);
    }

    // ── List<Student> → JSON Array String ────────────────────────────
    public String studentListToJson(List<Student> students) {
        return gson.toJson(students);
        // Output: [{"id":1,"name":"John",...}, {"id":2,...}]
    }

    // ── Practical example: parse API response ─────────────────────────
    public void handleApiResponse(String responseBody) {

        // Suppose server returns: { "success": true, "data": [...], "total": 25 }
        try {
            // Parse the outer wrapper
            JSONObject wrapper = new JSONObject(responseBody);
            boolean success = wrapper.getBoolean("success");

            if (success) {
                // Get the "data" array as a string, then use Gson to parse the list
                String dataArray = wrapper.getJSONArray("data").toString();

                Type listType = new TypeToken<List<Student>>(){}.getType();
                List<Student> students = gson.fromJson(dataArray, listType);

                // Update RecyclerView with parsed list
                adapter.setData(students);
            }
        } catch (JSONException e) {
            Log.e("TAG", "Parse error: " + e.getMessage());
        }
    }
}
```

### Real Flow: Network Response → RecyclerView

```java
// Simulates the complete data flow in a real Android app

public class MainActivity extends AppCompatActivity
        implements StudentAdapter.OnStudentClickListener {

    private ActivityMainBinding binding;
    private StudentAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        // Set up RecyclerView
        adapter = new StudentAdapter(this, this);
        binding.recyclerView.setLayoutManager(
            new LinearLayoutManager(this));
        binding.recyclerView.setAdapter(adapter);

        // Show loading indicator while fetching data
        binding.progressBar.setVisibility(View.VISIBLE);
        binding.recyclerView.setVisibility(View.GONE);

        // Fetch data on background thread (simulated)
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Handler mainThread = new Handler(Looper.getMainLooper());

        executor.execute(() -> {
            // ── Background Thread ──────────────────────────────────────
            // Simulate network call (replace with OkHttp call — covered Day 4)
            String jsonResponse = fetchFromServer();   // blocks until done

            // Parse JSON on background thread — don't do heavy work on main thread
            Gson gson = new Gson();
            Type listType = new TypeToken<List<Student>>(){}.getType();
            List<Student> students = gson.fromJson(jsonResponse, listType);

            // ── Switch to Main Thread for UI updates ──────────────────
            mainThread.post(() -> {
                // Hide loading, show list
                binding.progressBar.setVisibility(View.GONE);
                binding.recyclerView.setVisibility(View.VISIBLE);

                // Update adapter with parsed data
                adapter.setData(students);
            });
        });
    }

    // Simulated JSON response — in real app this comes from OkHttp
    private String fetchFromServer() {
        return "[" +
            "{\"id\":1,\"name\":\"John\",\"course\":\"CS\",\"gpa\":3.8,\"isActive\":true}," +
            "{\"id\":2,\"name\":\"Alice\",\"course\":\"EE\",\"gpa\":3.5,\"isActive\":true}," +
            "{\"id\":3,\"name\":\"Bob\",\"course\":\"ME\",\"gpa\":3.2,\"isActive\":false}" +
            "]";
    }

    @Override
    public void onStudentClick(Student student) {
        Toast.makeText(this, "Tapped: " + student.getName(), Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onStudentLongClick(Student student) { }
}
```

---

## PART 8: COMMON INTERVIEW QUESTIONS

**Q: What is the ViewHolder pattern and why is it needed?**
A: ViewHolder caches `findViewById()` results inside the View tag. Before ViewHolder, `onBindViewHolder` called `findViewById` every time — a slow tree traversal. ViewHolder stores direct references, making binding O(1).

**Q: Difference between `notifyDataSetChanged()` and `notifyItemChanged()`?**
A: `notifyDataSetChanged()` — rebuilds everything, no animation, heavy. `notifyItemChanged(pos)` — updates one item with animation, efficient. Use DiffUtil to compute which specific notifications to fire.

**Q: How does RecyclerView recycling work?**
A: Views that scroll off-screen go into a `RecycledViewPool`. When a new item needs a View, the pool is checked first. If found, `onBindViewHolder` fills it with new data. Only if pool is empty does `onCreateViewHolder` create a new View.

**Q: `match_parent` vs `wrap_content`?**
A: `match_parent` = take all available space. `wrap_content` = take only as much as content needs.

**Q: Why use `sp` for text and `dp` for everything else?**
A: `sp` scales with both screen density AND the user's font size setting in Accessibility options. `dp` only scales with density.

**Q: What is `getItemId()` used for?**
```java
// setHasStableIds(true) + getItemId() lets RecyclerView track items across updates
// Enables smarter animations and prevents flickering
@Override
public long getItemId(int position) {
    return studentList.get(position).getId();  // return unique ID
}
```

---

## DAY 3 MASTER SUMMARY

```
Layouts:
  LinearLayout   → stack horizontally or vertically; use weight for proportional sizing
  ConstraintLayout → flat hierarchy; position by constraints between views; zero nesting

ViewBinding:
  Enable in build.gradle → generates typed class per XML
  Activity: binding = XxxBinding.inflate(inflater); setContentView(binding.getRoot())
  Fragment: inflate with container; null binding in onDestroyView()
  Access: binding.tvName.setText() — no cast, no NPE

RecyclerView:
  LayoutManager → how to arrange (Linear, Grid)
  Adapter       → what data to show; 3 required methods: onCreateViewHolder, onBindViewHolder, getItemCount
  ViewHolder    → caches view references for one item; bind() fills data

Update notifications:
  notifyDataSetChanged()  → nuclear option, no animation
  notifyItemInserted(pos) → smooth insert animation
  notifyItemRemoved(pos)  → smooth remove animation
  notifyItemChanged(pos)  → smooth update animation
  DiffUtil                → auto-compute minimal changes; always prefer

JSON:
  org.json (built-in) → JSONObject, JSONArray; getInt/getString/optString
  Gson (library)      → fromJson/toJson; @SerializedName; TypeToken for List<T>
```

---

## DAY 3 CODING EXERCISES

Write completely from scratch without looking:

1. ConstraintLayout screen: Avatar image (left) + Name + Email (right of avatar) + Bio (below) + Button (centered at bottom)
2. Enable ViewBinding in a project; access 3 Views without a single `findViewById`
3. `Product` model class with: id, name, price, category, inStock
4. `item_product.xml` — card with name, price badge, category tag, colored stock indicator
5. `ProductAdapter` with: `onCreateViewHolder`, `onBindViewHolder`, `getItemCount`, click listener interface
6. Wire RecyclerView in Activity: GridLayoutManager (2 columns), sample data, click shows product name in Toast
7. `ProductDiffCallback` implementing all 4 DiffUtil.Callback methods
8. Parse this JSON manually with `org.json`:
   ```json
   {"id":5,"name":"Laptop","price":45000.0,"category":"Electronics","inStock":true}
   ```
9. Parse a JSON array of 3 products with Gson into `List<Product>`
10. Full flow: fake JSON string → Gson parse → adapter.setData() → RecyclerView shows items

---

*Day 4: Data Handling — SharedPreferences · SQLite · Room Database · DAO Pattern · File Storage*
