# DAY 1 — JAVA FOUNDATIONS (DEEP DIVE)
### 7-Day Android SDE Survival Plan

> Format per topic: Problem → Definition → How It Works Internally → Code → Real Android Usage → Interview Q&A → Common Mistakes

---

## PART 1: JVM (Java Virtual Machine)

### Problem It Solves
Without JVM, you'd write different code for Windows, Linux, and Mac. JVM makes **one codebase run everywhere**.

### Definition
JVM is a virtual machine that **interprets Java bytecode** and converts it to machine code at runtime. It also manages memory automatically (Garbage Collection).

### How It Works — Step by Step

```
YourCode.java
     ↓  (javac — Java Compiler)
YourCode.class  ← bytecode (not machine code yet)
     ↓  (JVM loads .class)
Interpreter + JIT Compiler
     ↓
Machine Code (CPU executes)
```

**JIT (Just-In-Time) Compiler** — JVM doesn't interpret every line slowly. It compiles frequently-used bytecode into native machine code at runtime to make it fast.

### JVM Memory Areas

```
┌─────────────────────────────────────┐
│              JVM Memory             │
├────────────┬──────────┬─────────────┤
│   Heap     │  Stack   │  Metaspace  │
│            │          │             │
│ Objects    │ Methods  │ Class info  │
│ (GC cleans)│ Local    │ (static)    │
│            │ Variables│             │
└────────────┴──────────┴─────────────┘
```

- **Heap** → All objects live here. Garbage Collector cleans unused ones.
- **Stack** → Each method call gets a frame. Local variables live here. Auto-cleaned when method returns.
- **Metaspace** → Class definitions, static variables.

### Garbage Collection
You never call `free()` in Java. JVM's GC automatically removes objects with no references.

```java
String s = new String("Hello");  // object on heap
s = null;  // no reference → GC will delete it
```

### Interview Q&A

**Q: Why is Java platform-independent?**
A: Java compiles to bytecode (`.class`), not machine code. JVM on any OS interprets the same bytecode. Write once, run anywhere.

**Q: Difference between JDK, JRE, JVM?**
```
JDK = JRE + Development Tools (javac, debugger)
JRE = JVM + Standard Libraries
JVM = Engine that runs bytecode
```

**Q: What is Stack Overflow?**
A: Infinite recursion causes the call stack to exceed its limit.
```java
void bad() { bad(); }  // StackOverflowError
```

**Q: Heap vs Stack?**
| Stack | Heap |
|---|---|
| Method calls, local vars | Objects |
| Auto-cleaned | GC cleans |
| Fast | Slower |
| Thread-specific | Shared across threads |

### Common Mistakes
- Thinking `.run()` creates a new thread (it doesn't — covered in Threads section).
- Assuming GC runs immediately when you set object to null. It runs when JVM decides.

---

## PART 2: OOP — Object-Oriented Programming

### Problem It Solves
Without OOP, code is a long list of functions with no structure. OOP groups **related data and behavior** into classes, making large codebases manageable.

### The 4 Pillars (Memorize These)

---

### 1. Encapsulation — "Hide the internals"

**Problem:** Anyone can modify your data directly → bugs.

```java
// BAD — exposed data
class BankAccount {
    public int balance = 1000;
}

// Someone does: account.balance = -999;  // invalid!
```

**Solution — Encapsulation:**
```java
class BankAccount {
    private int balance;         // hidden

    public int getBalance() {    // controlled read
        return balance;
    }

    public void deposit(int amount) {  // controlled write
        if (amount > 0) {
            balance += amount;
        }
    }

    public void withdraw(int amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }
}

// Usage
BankAccount acc = new BankAccount();
acc.deposit(500);
System.out.println(acc.getBalance());  // 500
// acc.balance = -999;  // COMPILER ERROR
```

**Real Android usage:** Room database entity fields are private. SharedPreferences values are accessed via getters. Every Android class you'll read follows this.

---

### 2. Inheritance — "Reuse and extend"

**Problem:** Two classes have identical code. Copy-paste = bugs when you update one but forget the other.

```java
// Base class (Parent)
class Animal {
    String name;

    void eat() {
        System.out.println(name + " is eating");
    }

    void breathe() {
        System.out.println("Breathing");
    }
}

// Child class — inherits all Animal methods
class Dog extends Animal {

    void bark() {
        System.out.println("Woof!");
    }
}

class Cat extends Animal {

    void meow() {
        System.out.println("Meow!");
    }
}

// Usage
Dog d = new Dog();
d.name = "Rex";
d.eat();    // inherited from Animal
d.bark();   // Dog's own method
```

**Real Android usage:** Every Activity `extends AppCompatActivity`. Every Fragment `extends Fragment`. You inherit lifecycle methods like `onCreate()`, `onResume()`.

---

### 3. Polymorphism — "One interface, many forms"

**Problem:** You have a list of different animal types. You want to call `makeSound()` on each without caring about its actual type.

```java
class Animal {
    void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Woof");
    }
}

class Cat extends Animal {
    @Override
    void makeSound() {
        System.out.println("Meow");
    }
}

// Polymorphism in action
Animal a1 = new Dog();  // Dog stored as Animal reference
Animal a2 = new Cat();

a1.makeSound();  // "Woof"  — actual type decides
a2.makeSound();  // "Meow"

// Works with a list
List<Animal> animals = new ArrayList<>();
animals.add(new Dog());
animals.add(new Cat());

for (Animal a : animals) {
    a.makeSound();  // each one calls its own version
}
```

**Real Android usage:** RecyclerView Adapter uses polymorphism — different ViewHolder types, one `onBindViewHolder()`.

---

### 4. Abstraction — "Define contract, hide details"

**Problem:** You want to enforce that every subclass MUST implement certain methods, but you don't want to write the implementation in the base class.

```java
// Abstract class — cannot be instantiated directly
abstract class Shape {
    abstract double area();   // must be implemented by subclass

    void describe() {         // concrete method — shared
        System.out.println("Area = " + area());
    }
}

class Circle extends Shape {
    double radius;

    Circle(double r) { this.radius = r; }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

class Rectangle extends Shape {
    double width, height;

    Rectangle(double w, double h) { width = w; height = h; }

    @Override
    double area() {
        return width * height;
    }
}

// Usage
Shape s1 = new Circle(5);
Shape s2 = new Rectangle(4, 6);

s1.describe();  // Area = 78.53...
s2.describe();  // Area = 24.0
```

**Interface vs Abstract Class:**

| | Abstract Class | Interface |
|---|---|---|
| Can have constructor | YES | NO |
| Can have fields | YES | NO (only constants) |
| Multiple inheritance | NO | YES |
| Use when | Shared base behavior | Contract / capability |

```java
// Interface example
interface Clickable {
    void onClick();
}

interface Swipeable {
    void onSwipe();
}

// Class can implement multiple interfaces
class Button implements Clickable, Swipeable {
    public void onClick() { System.out.println("Clicked"); }
    public void onSwipe() { System.out.println("Swiped"); }
}
```

**Real Android usage:** `View.OnClickListener` is an interface. You implement it to handle button clicks.

---

### Interview Q&A — OOP

**Q: What is the difference between `abstract class` and `interface`?**
A: Abstract class can have constructors, state, and partial implementation. Interface is a pure contract. A class can implement multiple interfaces but extend only one abstract class.

**Q: What is method overriding vs overloading?**
```java
// Overloading — same name, different params (compile-time)
void print(int x) {}
void print(String s) {}

// Overriding — child redefines parent method (runtime)
@Override
void makeSound() { System.out.println("Woof"); }
```

**Q: Can you call a parent's method from child after overriding?**
```java
class Dog extends Animal {
    @Override
    void makeSound() {
        super.makeSound();  // calls Animal's version
        System.out.println("Also: Woof");
    }
}
```

---

## PART 3: COLLECTIONS (DEEP DIVE)

### Problem It Solves
Arrays have a fixed size. You don't know ahead of time how many users, messages, or items you'll have. Collections are **resizable, type-safe, feature-rich** containers.

### The Hierarchy (Draw This From Memory)

```
Iterable
    └── Collection
            ├── List (ordered, duplicates OK)
            │     ├── ArrayList
            │     └── LinkedList
            ├── Set (no duplicates)
            │     ├── HashSet (no order)
            │     └── TreeSet (sorted)
            └── Queue (FIFO)
                  └── LinkedList

Map (NOT a Collection — separate hierarchy)
    ├── HashMap (no order)
    ├── LinkedHashMap (insertion order)
    └── TreeMap (sorted by key)
```

---

### ArrayList — Deep Dive

**What problem:** Need a resizable list where you access elements by index frequently.

**Internal working:**
```
ArrayList internally wraps a plain array.

Initial state: Object[] data = new Object[10]

Add "A":  [A][ ][ ][ ][ ][ ][ ][ ][ ][ ]
Add "B":  [A][B][ ][ ][ ][ ][ ][ ][ ][ ]
...
Full:     [A][B][C][D][E][F][G][H][I][J]

Add one more?
  → Create new array of size 15 (1.5x)
  → Copy all 10 elements
  → Add new element
```

**Complete code:**
```java
import java.util.ArrayList;
import java.util.Collections;

public class ArrayListDemo {
    public static void main(String[] args) {

        // Create
        ArrayList<String> names = new ArrayList<>();

        // Add
        names.add("John");
        names.add("David");
        names.add("Alice");
        names.add(1, "Bob");  // insert at index 1

        // Read
        System.out.println(names.get(0));     // John
        System.out.println(names.size());     // 4

        // Update
        names.set(2, "Charlie");

        // Delete
        names.remove("John");      // by value
        names.remove(0);           // by index

        // Iterate (3 ways)
        for (String name : names) {
            System.out.println(name);
        }

        for (int i = 0; i < names.size(); i++) {
            System.out.println(names.get(i));
        }

        names.forEach(name -> System.out.println(name));

        // Sort
        Collections.sort(names);

        // Search
        boolean has = names.contains("Alice");
        int idx = names.indexOf("Alice");

        // Clear
        names.clear();
        System.out.println(names.isEmpty());  // true
    }
}
```

**Complexity:**
| Operation | Time |
|---|---|
| `get(index)` | O(1) |
| `add(end)` | O(1) amortized |
| `add(middle)` | O(n) — shifts elements |
| `remove(middle)` | O(n) — shifts elements |
| `contains(value)` | O(n) — linear scan |

---

### LinkedList — Deep Dive

**What problem:** Frequent insertions/deletions at head or middle. ArrayList shifts all elements — slow.

**Internal working:**
```
Each element = a Node object

class Node {
    Object data;
    Node next;
    Node prev;  // doubly linked
}

HEAD
 ↓
[A] ⇄ [B] ⇄ [C] ⇄ [D]
                       ↑
                      TAIL

Insert between B and C:
[A] ⇄ [B] ⇄ [X] ⇄ [C] ⇄ [D]
Only 2 pointer updates. No shifting.
```

**Code:**
```java
import java.util.LinkedList;

LinkedList<String> list = new LinkedList<>();

list.add("A");
list.addFirst("Z");   // O(1)
list.addLast("M");    // O(1)

System.out.println(list.getFirst());  // Z
System.out.println(list.getLast());   // M

list.removeFirst();
list.removeLast();
```

**ArrayList vs LinkedList — Decision Guide:**
```
Mostly reading → ArrayList
Mostly adding/removing at ends → LinkedList
Mostly adding/removing in middle → ArrayList (cache-friendly, still often faster)
Default choice → ArrayList (90% of cases)
```

---

### HashMap — Deep Dive (Most Important)

**What problem:** You need to look up a value by a key instantly. Not by scanning a list, but directly.

**Definition:** HashMap stores key-value pairs using a hash function to compute an index into an internal array (called a "bucket array").

**Internal working — step by step:**
```
map.put(101, "John");

Step 1: Compute hashCode of key
        101.hashCode() = 101

Step 2: Compute bucket index
        index = hashCode % capacity
              = 101 % 16 = 5

Step 3: Store Entry at bucket 5
        buckets[5] = Entry(101, "John")

map.get(101):
        Same hash → same bucket → return "John"
        Average: O(1)
```

**Collision handling:**
```
map.put(101, "John");
map.put(201, "David");

Both hash to bucket 5:

buckets[5] → [101,"John"] → [201,"David"]
             (linked list within bucket)

Java 8: When chain length > 8, converts to Red-Black Tree
        O(n) worst case → O(log n)
```

**Complete code:**
```java
import java.util.HashMap;
import java.util.Map;

public class HashMapDemo {
    public static void main(String[] args) {

        HashMap<Integer, String> map = new HashMap<>();

        // Put
        map.put(101, "John");
        map.put(102, "David");
        map.put(103, "Alice");
        map.put(null, "NullKey");   // null key allowed

        // Get
        System.out.println(map.get(101));        // John
        System.out.println(map.get(999));        // null (not found)

        // Check existence
        System.out.println(map.containsKey(102));    // true
        System.out.println(map.containsValue("Alice")); // true

        // Update
        map.put(101, "Johnny");   // overwrites

        // Delete
        map.remove(103);

        // Iterate
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " → " + entry.getValue());
        }

        // Keys only
        for (int key : map.keySet()) {
            System.out.println(key);
        }

        // Values only
        for (String val : map.values()) {
            System.out.println(val);
        }

        // getOrDefault — very common in Android
        String name = map.getOrDefault(999, "Unknown");
        System.out.println(name);  // Unknown

        // Size
        System.out.println(map.size());
    }
}
```

**HashMap vs LinkedHashMap vs TreeMap:**
| | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| Order | None | Insertion order | Sorted by key |
| Speed | O(1) | O(1) | O(log n) |
| Use when | Fast lookup | Predictable order | Need sorted keys |

---

### HashSet — Deep Dive

**What problem:** Store a collection of items but **never want duplicates**.

**Internal working:** HashSet internally uses a HashMap where your element is the key and a dummy `PRESENT` object is the value. Duplicate check is O(1).

```java
import java.util.HashSet;
import java.util.TreeSet;

HashSet<String> set = new HashSet<>();

set.add("John");
set.add("David");
set.add("John");   // ignored — duplicate

System.out.println(set.size());         // 2
System.out.println(set.contains("John")); // true

// Iterate (no guaranteed order)
for (String s : set) {
    System.out.println(s);
}

// Sorted set
TreeSet<String> sorted = new TreeSet<>();
sorted.add("Banana");
sorted.add("Apple");
sorted.add("Cherry");

System.out.println(sorted);  // [Apple, Banana, Cherry]
```

**Real Android usage:**
```java
// Tracking granted permissions — no duplicates, fast lookup
Set<String> grantedPermissions = new HashSet<>();
grantedPermissions.add("android.permission.CAMERA");
grantedPermissions.add("android.permission.LOCATION");

if (grantedPermissions.contains("android.permission.CAMERA")) {
    // proceed
}
```

---

### Student Management System (Day 1 Coding Task)

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

class Student {
    private int roll;
    private String name;
    private double gpa;

    public Student(int roll, String name, double gpa) {
        this.roll = roll;
        this.name = name;
        this.gpa = gpa;
    }

    public int getRoll() { return roll; }
    public String getName() { return name; }
    public double getGpa() { return gpa; }

    @Override
    public String toString() {
        return "Roll: " + roll + ", Name: " + name + ", GPA: " + gpa;
    }
}

class StudentManager {
    private ArrayList<Student> students = new ArrayList<>();
    private HashMap<Integer, Student> rollIndex = new HashMap<>();

    public void addStudent(Student s) {
        students.add(s);
        rollIndex.put(s.getRoll(), s);
        System.out.println("Added: " + s);
    }

    public void removeStudent(int roll) {
        Student s = rollIndex.get(roll);
        if (s != null) {
            students.remove(s);
            rollIndex.remove(roll);
            System.out.println("Removed roll: " + roll);
        } else {
            System.out.println("Not found: " + roll);
        }
    }

    public Student searchByRoll(int roll) {
        return rollIndex.get(roll);  // O(1)
    }

    public List<Student> searchByName(String name) {
        List<Student> result = new ArrayList<>();
        for (Student s : students) {
            if (s.getName().equalsIgnoreCase(name)) {
                result.add(s);
            }
        }
        return result;
    }

    public void printAll() {
        System.out.println("--- All Students ---");
        for (Student s : students) {
            System.out.println(s);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        StudentManager mgr = new StudentManager();

        mgr.addStudent(new Student(101, "John", 3.8));
        mgr.addStudent(new Student(102, "David", 3.5));
        mgr.addStudent(new Student(103, "Alice", 3.9));

        mgr.printAll();

        System.out.println(mgr.searchByRoll(102));

        mgr.removeStudent(102);
        mgr.printAll();
    }
}
```

---

## PART 4: GENERICS (DEEP DIVE)

### Problem It Solves
Without generics, collections store `Object` type. You get runtime `ClassCastException` crashes. Generics catch type errors **at compile time**.

### Definition
Generics allow you to write **type-parameterized** classes, methods, and interfaces. The type is specified at usage time, checked at compile time, and erased at runtime (type erasure).

### Without Generics — The Problem

```java
ArrayList list = new ArrayList();  // raw type
list.add("Hello");
list.add(42);           // no error yet

String s = (String) list.get(1);  // CRASH: 42 is not a String
```

### With Generics — The Fix

```java
ArrayList<String> list = new ArrayList<>();
list.add("Hello");
// list.add(42);  // COMPILER ERROR — caught early
String s = list.get(0);  // no cast needed
```

### Generic Class

```java
// T is a type parameter — placeholder for actual type
class Box<T> {
    private T content;

    public void put(T item) {
        this.content = item;
    }

    public T get() {
        return content;
    }
}

// Usage
Box<String> stringBox = new Box<>();
stringBox.put("Hello");
String val = stringBox.get();  // no cast

Box<Integer> intBox = new Box<>();
intBox.put(42);
int num = intBox.get();  // no cast
```

### Generic Method

```java
// Works with any type
public <T> void printArray(T[] arr) {
    for (T item : arr) {
        System.out.print(item + " ");
    }
    System.out.println();
}

// Usage
printArray(new String[]{"A", "B", "C"});
printArray(new Integer[]{1, 2, 3});
```

### Bounded Generics

```java
// Only accept Number or its subclasses (Integer, Double, etc.)
public <T extends Number> double sum(List<T> list) {
    double total = 0;
    for (T num : list) {
        total += num.doubleValue();
    }
    return total;
}

// Usage
List<Integer> ints = Arrays.asList(1, 2, 3);
System.out.println(sum(ints));  // 6.0
```

### Wildcards

```java
// ? = any type — used for flexibility in method params

// Read-only — accepts List<Integer>, List<Double>, etc.
public void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}

// Upper bounded — Number or subclass
public void addNumbers(List<? extends Number> list) {}

// Lower bounded — Integer or superclass
public void addIntegers(List<? super Integer> list) {}
```

### Real Android Usage

```java
// Retrofit response wrapper — very common pattern
class ApiResponse<T> {
    private T data;
    private String error;
    private boolean success;

    public T getData() { return data; }
    public boolean isSuccess() { return success; }
}

// Usage
ApiResponse<List<User>> userResponse;
ApiResponse<String> tokenResponse;

// RecyclerView Adapter — generic base
class BaseAdapter<T> extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    protected List<T> items = new ArrayList<>();

    public void setData(List<T> newItems) {
        items = newItems;
        notifyDataSetChanged();
    }
}
```

### Interview Q&A

**Q: What is type erasure?**
A: At compile time, Java uses generic type info for checking. At runtime, the type parameter is removed — `List<String>` and `List<Integer>` are both just `List` at runtime.

**Q: Can you create a generic array?**
```java
T[] arr = new T[10];  // COMPILER ERROR
// Generics and arrays don't mix well
```

**Q: What's the difference between `<?>` and `<T>`?**
- `<T>` — you can use T inside the method body, read and write.
- `<?>` — unknown type, mostly read-only, more flexible for input parameters.

---

## PART 5: EXCEPTION HANDLING (DEEP DIVE)

### Problem It Solves
Programs fail — network errors, null pointers, invalid input. Without handling these, your app **crashes**. Exception handling lets you catch failures and respond gracefully.

### Definition
An exception is an object thrown by Java when something goes wrong at runtime. You can catch it, log it, show a message to the user, and continue running.

### Exception Hierarchy

```
Throwable
    ├── Error              ← JVM-level, don't catch
    │     ├── OutOfMemoryError
    │     └── StackOverflowError
    │
    └── Exception
          ├── Checked Exceptions  ← must handle at compile time
          │     ├── IOException
          │     ├── SQLException
          │     └── FileNotFoundException
          │
          └── RuntimeException   ← unchecked, discovered at runtime
                ├── NullPointerException
                ├── ArrayIndexOutOfBoundsException
                ├── ArithmeticException
                ├── ClassCastException
                └── IllegalArgumentException
```

### Checked vs Unchecked

```java
// CHECKED — compiler forces you to handle this
// Reading a file can fail → must catch IOException
try {
    FileReader fr = new FileReader("file.txt");  // checked
} catch (IOException e) {
    System.out.println("File not found");
}

// UNCHECKED — compiler doesn't force you
// But it will crash at runtime if it happens
String s = null;
s.length();  // NullPointerException — unchecked
```

### try-catch-finally

```java
public int divide(int a, int b) {
    try {
        return a / b;                          // risky code

    } catch (ArithmeticException e) {          // specific catch
        System.out.println("Cannot divide by zero: " + e.getMessage());
        return -1;

    } catch (Exception e) {                    // general catch (always last)
        System.out.println("Unexpected: " + e.getMessage());
        return -1;

    } finally {                                // ALWAYS runs
        System.out.println("divide() finished");
    }
}
```

**Rule:** `finally` always runs — even if an exception is thrown or return is called. Use it to close resources.

### Try-With-Resources (Modern Java)

```java
// Old way — you must close manually in finally
FileReader fr = null;
try {
    fr = new FileReader("file.txt");
    // read
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fr != null) fr.close();  // forget this → resource leak
}

// Modern way — auto-closes
try (FileReader fr = new FileReader("file.txt")) {
    // read — fr is auto-closed after this block
} catch (IOException e) {
    e.printStackTrace();
}
```

### Custom Exceptions

```java
// Define
class InsufficientFundsException extends Exception {
    private double amount;

    public InsufficientFundsException(double amount) {
        super("Insufficient funds. Needed: " + amount);
        this.amount = amount;
    }

    public double getAmount() { return amount; }
}

// Use
class BankAccount {
    private double balance = 100.0;

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount);
        }
        balance -= amount;
    }
}

// Call
BankAccount acc = new BankAccount();
try {
    acc.withdraw(500.0);
} catch (InsufficientFundsException e) {
    System.out.println(e.getMessage());  // Insufficient funds. Needed: 500.0
}
```

### throws vs throw

```java
// throw — actually throws an exception object
throw new IllegalArgumentException("Age cannot be negative");

// throws — declares that this method CAN throw this exception
// (caller must handle or also declare throws)
public void setAge(int age) throws IllegalArgumentException {
    if (age < 0) throw new IllegalArgumentException("Age negative");
    this.age = age;
}
```

### Real Android Usage

```java
// Parsing JSON — can fail
try {
    JSONObject json = new JSONObject(responseString);
    String name = json.getString("name");
} catch (JSONException e) {
    Log.e("TAG", "JSON parse error: " + e.getMessage());
}

// Network call — can fail
try {
    Response response = okHttpClient.newCall(request).execute();
} catch (IOException e) {
    Log.e("TAG", "Network error: " + e.getMessage());
    showRetryUI();
}

// Room database — not found
try {
    User user = userDao.findById(id);
    if (user == null) throw new UserNotFoundException(id);
} catch (UserNotFoundException e) {
    showEmptyState();
}
```

### Interview Q&A

**Q: Checked vs Unchecked exception?**
A: Checked exceptions are verified by compiler — you must handle them. Unchecked (RuntimeException) are not checked by compiler but cause crashes if unhandled.

**Q: Can `finally` block be skipped?**
A: Only if `System.exit()` is called or JVM crashes.

**Q: What's the difference between `throw` and `throws`?**
A: `throw` is the action of throwing. `throws` is a method signature declaration saying "this method might throw this."

**Q: What is exception chaining?**
```java
try {
    // original error
} catch (IOException e) {
    throw new RuntimeException("Wrapped", e);  // e is the cause
}
```

### Common Mistakes
- Catching `Exception` too broadly — hides bugs.
- Empty catch blocks — silently swallows errors.
- Not closing resources in `finally` — memory leaks.

---

## PART 6: THREADS (DEEP DIVE)

### Problem It Solves
By default, code runs on one thread (the main thread). In Android, the main thread handles UI. If you run a network call on the main thread, **UI freezes**. Threads let you do work in parallel.

### Definition
A thread is an independent path of execution within a program. Multiple threads in one process share the same heap (objects) but each have their own stack.

### Creating Threads — 3 Ways

**Way 1: Extend Thread**
```java
class MyTask extends Thread {
    @Override
    public void run() {
        System.out.println("Running on: " + Thread.currentThread().getName());
    }
}

MyTask t = new MyTask();
t.start();  // creates new thread → calls run()
```

**Way 2: Implement Runnable (Preferred)**
```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Task running");
    }
};

Thread t = new Thread(task);
t.start();
```

**Way 3: Lambda (Modern Java 8+)**
```java
Thread t = new Thread(() -> {
    System.out.println("Lambda task");
});
t.start();
```

### run() vs start() — Critical Interview Question

```java
Thread t = new Thread(() -> System.out.println("Hello"));

t.run();   // WRONG — runs on CURRENT thread, no new thread created
t.start(); // RIGHT — JVM creates new thread, calls run() on it
```

### Thread Lifecycle

```
NEW
 ↓ start()
RUNNABLE ←→ BLOCKED (waiting for lock)
 ↓                  ↗
RUNNING → WAITING
 ↓ (run completes)
DEAD/TERMINATED
```

### Thread Methods

```java
Thread t = new Thread(() -> {
    try {
        Thread.sleep(2000);  // sleep 2 seconds
        System.out.println("Done");
    } catch (InterruptedException e) {
        System.out.println("Interrupted!");
    }
});

t.start();
t.join();    // main thread waits for t to finish
t.interrupt(); // interrupt a sleeping/waiting thread

System.out.println(t.isAlive());   // false after done
System.out.println(t.getName());   // Thread-0
t.setName("DownloadThread");
t.setPriority(Thread.MAX_PRIORITY); // 1-10
```

### Multiple Threads Example

```java
public class MultiThreadDemo {
    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("T1: " + i);
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("T2: " + i);
            }
        });

        t1.start();
        t2.start();

        t1.join();  // wait for t1
        t2.join();  // wait for t2

        System.out.println("Both done");
    }
}
```

Output is interleaved — T1 and T2 run simultaneously, order not guaranteed.

---

## PART 7: SYNCHRONIZATION (DEEP DIVE)

### Problem It Solves
Multiple threads accessing **shared mutable data** simultaneously causes **race conditions** — data corruption, wrong results.

### Race Condition Illustrated

```java
class Counter {
    int count = 0;

    void increment() {
        count++;  // NOT atomic! 3 steps: read, add 1, write
    }
}

Counter c = new Counter();

Thread t1 = new Thread(() -> {
    for (int i = 0; i < 10000; i++) c.increment();
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 10000; i++) c.increment();
});

t1.start(); t2.start();
t1.join();  t2.join();

System.out.println(c.count);  // Expected: 20000, Actual: RANDOM (18453, 19102...)
```

**Why?**
```
Thread1: read count=5
Thread2: read count=5   ← both read same value
Thread1: write count=6
Thread2: write count=6  ← Thread1's update lost!
Expected 7, got 6.
```

### synchronized keyword

```java
class Counter {
    int count = 0;

    // Only ONE thread can execute this at a time
    synchronized void increment() {
        count++;  // now atomic with the lock
    }
}
```

**How it works:**
- Every object in Java has a **monitor lock** (intrinsic lock).
- `synchronized` method acquires the lock on `this`.
- Other threads trying to call `increment()` wait until lock is released.

### synchronized block (More Precise)

```java
class Counter {
    int count = 0;
    private final Object lock = new Object();

    void increment() {
        // Only lock the critical section, not the whole method
        synchronized (lock) {
            count++;
        }
        // other non-critical code here runs without locking
    }
}
```

### volatile keyword

```java
class Flag {
    volatile boolean running = true;  // changes visible to all threads immediately
}
```

`volatile` ensures visibility — when one thread changes the value, all threads see the new value immediately. Does NOT prevent race conditions (use `synchronized` for that).

### Deadlock

```java
// DANGER — two threads locking each other's resources
Object lockA = new Object();
Object lockB = new Object();

Thread t1 = new Thread(() -> {
    synchronized(lockA) {
        synchronized(lockB) { /* ... */ }  // waits for t2 to release B
    }
});

Thread t2 = new Thread(() -> {
    synchronized(lockB) {
        synchronized(lockA) { /* ... */ }  // waits for t1 to release A
    }
});
// Both threads wait forever → DEADLOCK
```

**Fix:** Always acquire locks in the same order.

### Real Android Usage

```java
// Shared cache accessed from background threads
class ImageCache {
    private final HashMap<String, Bitmap> cache = new HashMap<>();

    public synchronized void put(String url, Bitmap bitmap) {
        cache.put(url, bitmap);
    }

    public synchronized Bitmap get(String url) {
        return cache.get(url);
    }
}
```

### Interview Q&A

**Q: What is a race condition?**
A: When the outcome of operations depends on the sequence or timing of multiple threads accessing shared data, leading to unpredictable results.

**Q: synchronized vs volatile?**
| | synchronized | volatile |
|---|---|---|
| Mutual exclusion | YES | NO |
| Visibility | YES | YES |
| Compound operations | Safe | Not safe |
| Performance | Slower | Faster |

**Q: What causes deadlock?**
A: Two or more threads each waiting for a resource locked by the other.

---

## PART 8: EXECUTORS (DEEP DIVE)

### Problem It Solves
Creating a raw `Thread` for every task is expensive — thread creation/destruction overhead. You'd also have no control over how many threads run simultaneously. **Executors manage a pool of reusable threads**.

### Definition
`ExecutorService` is a higher-level API for managing thread lifecycle, task submission, and result retrieval. It maintains a thread pool — threads are reused across tasks.

### Thread Pool Concept

```
Without pool:                 With pool:
Task1 → new Thread()          Thread1 ──┐
Task2 → new Thread()          Thread2 ──┼── Task queue
Task3 → new Thread()          Thread3 ──┘
(100 tasks = 100 threads)     (100 tasks, 3 threads, queue manages rest)
```

### Types of Executors

```java
import java.util.concurrent.*;

// Fixed pool — always N threads
ExecutorService fixed = Executors.newFixedThreadPool(4);

// Single thread — tasks run sequentially
ExecutorService single = Executors.newSingleThreadExecutor();

// Cached — creates threads as needed, reuses idle ones
ExecutorService cached = Executors.newCachedThreadPool();

// Scheduled — run tasks after delay or periodically
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
```

### execute() — Fire and Forget

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

executor.execute(() -> {
    System.out.println("Task 1 on: " + Thread.currentThread().getName());
});

executor.execute(() -> {
    System.out.println("Task 2 on: " + Thread.currentThread().getName());
});

executor.shutdown();  // no new tasks, finish current
// executor.shutdownNow();  // interrupt all
```

### submit() + Future — Get Result Back

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

// Callable returns a value (unlike Runnable)
Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);  // simulate work
    return 42;
});

System.out.println("Waiting for result...");
int result = future.get();  // BLOCKS until result is ready
System.out.println("Result: " + result);  // 42

executor.shutdown();
```

### Multiple Futures

```java
ExecutorService executor = Executors.newFixedThreadPool(3);

List<Future<Integer>> futures = new ArrayList<>();

for (int i = 0; i < 5; i++) {
    final int taskId = i;
    Future<Integer> f = executor.submit(() -> {
        Thread.sleep(500);
        return taskId * 10;
    });
    futures.add(f);
}

for (Future<Integer> f : futures) {
    System.out.println(f.get());  // 0, 10, 20, 30, 40
}

executor.shutdown();
```

### ScheduledExecutorService

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

// Run once after 3 seconds
scheduler.schedule(() -> {
    System.out.println("Delayed task");
}, 3, TimeUnit.SECONDS);

// Run every 2 seconds after initial 1-second delay
scheduler.scheduleAtFixedRate(() -> {
    System.out.println("Periodic: " + System.currentTimeMillis());
}, 1, 2, TimeUnit.SECONDS);
```

### Real Android Usage

```java
// Android — network call on background thread, update UI on main thread
ExecutorService executor = Executors.newSingleThreadExecutor();
Handler mainHandler = new Handler(Looper.getMainLooper());

executor.execute(() -> {
    // Background thread
    String result = fetchFromNetwork();  // slow operation

    // Switch to main thread to update UI
    mainHandler.post(() -> {
        textView.setText(result);  // UI update
    });
});
```

This is the standard pattern in Android before coroutines. You will see it in production code.

### Interview Q&A

**Q: Why use ExecutorService over raw Thread?**
A: Thread pool reuse avoids overhead of thread creation. Better control of concurrency (limit thread count). Built-in task queue. Easy result retrieval via Future.

**Q: What is Future.get() behavior?**
A: It's blocking — current thread waits until the result is ready. You can use `future.get(timeout, unit)` to avoid waiting forever.

**Q: Callable vs Runnable?**
| | Runnable | Callable |
|---|---|---|
| Return value | No (void) | Yes (generic type) |
| Exception | Cannot throw checked | Can throw checked |
| Use with | Thread / execute() | submit() |

**Q: What happens if you don't call shutdown()?**
A: The thread pool threads keep running — **JVM won't exit**. Always call `executor.shutdown()`.

---

## DAY 1 — 10 CODING EXERCISES (Write These Without Looking)

```java
// 1. ArrayList of integers 1-10, print even ones
ArrayList<Integer> nums = new ArrayList<>();
for (int i = 1; i <= 10; i++) nums.add(i);
nums.stream().filter(n -> n % 2 == 0).forEach(System.out::println);

// 2. HashMap of 5 students (roll → name), print all
HashMap<Integer, String> students = new HashMap<>();
students.put(101, "John"); students.put(102, "Alice");
// ... etc
students.forEach((roll, name) -> System.out.println(roll + ": " + name));

// 3. BankAccount with encapsulation
// (balance private, deposit/withdraw with validation)

// 4. Animal → Dog, Cat with makeSound() polymorphism

// 5. Generic method printAny(T value)

// 6. Divide by zero — catch ArithmeticException

// 7. Two threads counting from 1-5 simultaneously

// 8. Synchronized Counter incremented by 2 threads (verify = 20000)

// 9. ExecutorService with 3 threads running 6 tasks

// 10. submit() a task, get Future<String> result
```

---

## QUICK REFERENCE — Day 1

```
JVM:         bytecode → JVM → machine code. Heap=objects, Stack=calls.
OOP:         Encap(private+getters), Inherit(extends), Poly(@Override), Abstract(contract)
ArrayList:   Dynamic array. get O(1), insert-middle O(n). Default choice.
LinkedList:  Nodes. Insert-ends O(1). Use rarely.
HashMap:     Hash → bucket. O(1) avg. Key-value. Null key allowed.
HashSet:     No duplicates. Backed by HashMap.
Generics:    Type safety at compile time. <T> = type param.
Exceptions:  Checked(compile), Unchecked(runtime). try-catch-finally.
Threads:     start() creates thread. run() is just a method.
Sync:        synchronized prevents race conditions. volatile for visibility.
Executors:   Thread pool. execute()=fire, submit()=Future result. Always shutdown().
```

---

*Day 2: Android Activities, Fragments, Intents, Manifest, Permissions, Lifecycle, Parcelable*
