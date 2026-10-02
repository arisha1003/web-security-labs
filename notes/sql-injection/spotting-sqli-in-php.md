# How to Spot SQL Injection in PHP Code

SQL Injection happens when **user input is mixed directly into a SQL query** and can change the query's meaning.

## 1. Find where user input enters

For example:

```php
$id = $_GET['id'];
```

This means:

```text
Browser → ?id=5 → $_GET['id'] → $id
```

The user controls `$id`.

---

## 2. Follow the input

Now look where `$id` goes:

```php
$query = "SELECT first_name, last_name
          FROM users
          WHERE user_id = '$id'";
```

Here, `$id` is placed **directly inside the SQL string**.

The flow is:

```text
User input
    ↓
$id
    ↓
SQL query
    ↓
Database
```

This is the pattern we investigate for SQL Injection.

---

## 3. Why is this dangerous?

The developer expects `$id` to be normal data:

```text
5
```

But if user input can be interpreted as SQL syntax, the user may be able to change what the database executes.

The problem is:

```text
SQL code + User input
       ↓
    mixed together
```

---

## 4. The safer way: Prepared Statements

Instead of putting `$id` directly into the SQL:

```php
$query = "SELECT * FROM users WHERE user_id = '$id'";
```

use a placeholder:

```php
$stmt = $db->prepare(
    "SELECT * FROM users WHERE user_id = :id"
);

$stmt->bindParam(':id', $id);
$stmt->execute();
```

Think of `:id` as an **empty box**:

```text
SELECT * FROM users
WHERE user_id = [ BOX ]
                    ↑
                   :id
```

Then:

```php
$stmt->bindParam(':id', $id);
```

puts the value of `$id` into that box as **data**.

So:

```text
SQL structure  +  Data
     ↓             ↓
SELECT ... :id    5
```

They stay separate.

---

## 5. Important: `$id` vs `:id`

They are NOT the same thing.

```text
$id
 ↓
PHP variable containing the user's value
```

```text
:id
 ↓
SQL placeholder (empty box)
```

`bindParam()` connects them:

```text
$id ─────────→ :id
value           placeholder
```

---

## 6. What About `mysqli_real_escape_string()`?

You may also see:

```php
$id = $_POST['id'];

$id = mysqli_real_escape_string($connection, $id);

$query = "SELECT * FROM users WHERE user_id = $id";
```

Here PHP **escapes special characters** before putting the value into the query.

So don't assume:

```text
$id directly in SQL = automatically vulnerable
```

You must check **what happened to the input before it reached the query**.

Prepared statements are generally preferred because they keep SQL code and user data separate.

---

## 7. The Pentester's Simple Rule

When reading PHP code, follow the input:

```text
Where does it come from?
        ↓
Can the user control it?
        ↓
Where does it go?
        ↓
Does it reach SQL?
        ↓
Is it safely separated from the SQL?
```

### Example

```php
$id = $_GET['id'];       // User input

$query = "SELECT *
          FROM users
          WHERE user_id = '$id'";   // 🚨 Directly in SQL
```

vs.

```php
$id = $_GET['id'];       // User input

$stmt = $db->prepare(
    "SELECT * FROM users WHERE user_id = :id"
);

$stmt->bindParam(':id', $id);        // ✅ Separate
$stmt->execute();
```

---

## 8. One Thing to Remember

> **SQL Injection = untrusted user input becoming part of SQL code.**

When reading code, don't just look for `$id`.

Look for the **whole journey**:

```text
USER INPUT
    ↓
PHP VARIABLE
    ↓
SQL QUERY
    ↓
DATABASE
```

Then ask:

**"Can the user's data become SQL code?"**

