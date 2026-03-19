# Smart Aid – AI Emergency Assistant

> This project is based on an original implementation by PAGADALA-MOKSHAGNA.  
> I enhanced it by adding data tracking, analytics, and AI-based improvements.
## MongoHelper Function 

## 🧠 **Purpose of MongoHelper**

This class acts as a **singleton utility** to handle **MongoDB connections** from your Android app.
It:

* Creates a **single reusable MongoDB client connection**.
* Connects to a **specific database (`Smart_First_Aid`)**.
* Provides **references (handles)** to specific collections (`procedures` and `UserDetails`).
* Optionally allows you to **close the connection** when the app quits.

---

## 🔍 Let’s Go Through It Step by Step

### 🏷️ Package Declaration

```java
package com.example.smartfirstaid.data.db;
```

This means the helper is organized under your app’s data layer, specifically under the database module (`db`).
📘 *Good practice — keeps your code modular and maintainable.*

---

### ⚙️ Imports

```java
import com.mongodb.MongoClientSettings;
import com.mongodb.ServerAddress;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;
import java.util.Collections;
```

These are from the **MongoDB Java Driver** (`org.mongodb:mongodb-driver-sync`).
They allow your Android app to:

* Configure connection settings (`MongoClientSettings`)
* Specify where your MongoDB server is (`ServerAddress`)
* Open client sessions and access databases/collections
* Use BSON documents (the native MongoDB format)

---

### 🧱 Class Definition

```java
public final class MongoHelper {
    private static MongoClient client;
    private static MongoDatabase db;

    private MongoHelper() {}
```

* `final` → prevents subclassing (ensures it stays a strict helper class).
* `private static MongoClient client;` → static reference, shared across entire app.
* `private static MongoDatabase db;` → cached database object.
* `private MongoHelper()` → private constructor to prevent creating instances.
  👉 *This enforces a **singleton pattern** — only one connection across the app.*

---

### 🌐 The procedures() Method

```java
public static MongoCollection<Document> procedures() {
    if (client == null) {
        client = MongoClients.create(
                MongoClientSettings.builder()
                        .applyToClusterSettings(b ->
                                b.hosts(Collections.singletonList(
                                        new ServerAddress("10.0.2.2", 27017)
                                )))
                        .build());
        db = client.getDatabase("Smart_First_Aid");
    }
    return db.getCollection("procedures");
}
```

#### 🔧 Step-by-step explanation

1. **`if (client == null)`**
   Checks if a MongoClient has already been created.

   * If *not*, it creates a new connection.
   * This avoids reopening multiple database sessions unnecessarily.

2. **`MongoClients.create()`**
   Creates the client using a **connection configuration object** (`MongoClientSettings`).

3. **`applyToClusterSettings()`**
   Configures how the app connects to the MongoDB cluster.

   * In this case, it connects to a **single host**:

     ```java
     new ServerAddress("10.0.2.2", 27017)
     ```

   * `10.0.2.2` is a **special IP for Android Emulator** that routes to your **host PC’s localhost**.
     So if MongoDB is running on your laptop (`localhost:27017`), the emulator can reach it via `10.0.2.2`.

4. **`db = client.getDatabase("Smart_First_Aid");`**
   Selects (or creates if not existing) the database `Smart_First_Aid`.

5. **`return db.getCollection("procedures");`**
   Provides a handle to the `"procedures"` collection inside your database.
   You can now perform:

   ```java
   MongoCollection<Document> coll = MongoHelper.procedures();
   coll.insertOne(new Document("title", "CPR Steps"));
   ```

---

### 👤 The userDetails() Method

```java
public static MongoCollection<Document> userDetails() {
    if (client == null) {
        client = MongoClients.create(
                MongoClientSettings.builder()
                        .applyToClusterSettings(b -> b.hosts(
                                java.util.Collections.singletonList(
                                        new ServerAddress("10.0.2.2", 27017)
                                )))
                        .build());
        db = client.getDatabase("Smart_First_Aid");
    }
    return db.getCollection("UserDetails");
}
```

This is nearly identical — except it points to a **different collection**: `UserDetails`.

💡 **Note**:

* The comment:

  ```java
  // Emulator → 10.0.2.2 | Real device → Your Laptop IPv4 (e.g., 192.168.1.23)
  ```

  means:

  * On **emulator**, use `10.0.2.2` to access host MongoDB.
  * On a **real Android phone**, you must connect over Wi-Fi using your laptop’s **IPv4 address**.

Example:

```java
new ServerAddress("192.168.1.23", 27017);
```

---

### 🧹 Closing Connection

```java
public static void close() {
    try { if (client != null) client.close(); } catch (Exception ignore) {}
    client = null; db = null;
}
```

This safely closes the MongoDB connection (useful when the app shuts down).
Though in Android, it’s not strictly required unless you maintain long-running background processes.

---

## ⚙️ Workflow Summary

| Step | What Happens                            | Code Reference                   |
| ---- | --------------------------------------- | -------------------------------- |
| 1️⃣  | App calls `MongoHelper.procedures()`    | Static method invoked            |
| 2️⃣  | Checks if client is initialized         | `if (client == null)`            |
| 3️⃣  | Creates MongoDB connection              | `MongoClients.create(...)`       |
| 4️⃣  | Selects Database                        | `getDatabase("Smart_First_Aid")` |
| 5️⃣  | Returns collection handle               | `getCollection("procedures")`    |
| 6️⃣  | Performs CRUD using returned collection | e.g., `insertOne`, `find`, etc.  |

---

## ⚡ Common Usage in Android (Example)

```java
MongoCollection<Document> userColl = MongoHelper.userDetails();

// Insert
Document newUser = new Document("name", "John")
        .append("phone", "9876543210");
userColl.insertOne(newUser);

// Read
for (Document doc : userColl.find()) {
    Log.d("MongoDB", doc.toJson());
}
```

---

## 🧠 Professor’s Notes & Best Practices

1. ✅ **Singleton connection** = efficient memory & connection handling.
2. ⚠️ **Never use `localhost` directly** in Android Emulator — always `10.0.2.2`.
3. ⚡ If you connect from a **real device**, use your laptop’s **IPv4 address**, and ensure **MongoDB’s port (27017)** is accessible in your firewall.
4. 🧩 For production, you’d use **MongoDB Atlas** (cloud-hosted) + credentials (username/password).
5. 🧼 Add error handling for `MongoTimeoutException` (network failure).

---

Would you like me to show you how to **modify this MongoHelper** so it can **automatically detect whether the app is running on an emulator or real device**, and adjust the IP (`10.0.2.2` vs. your local network IPv4) accordingly?
That would make it fully portable without manual IP edits.
