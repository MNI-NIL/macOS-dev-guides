# 🧭 Reverse-Engineered Guide to macOS Help System (Tips.app, HelpViewer, and Caching)

This guide summarizes everything discovered while reverse-engineering the macOS Help Book system as of macOS 15 (“Sequoia”), especially with regard to the new `Tips.app`-based viewer, persistent caching behavior, and command-line inspection tools.

---

## 📘 Help Viewer Architecture (macOS 15+)

- `Tips.app` **is the new user interface** for Help Books.
- It is located at:
  ```
  /System/Applications/Tips.app
  ```
- Despite the new name, its bundle identifier is still:
  ```
  com.apple.helpviewer
  ```
- Help Books are **registered** with the system and then opened via `Tips.app`.

---

## 📁 Help Book Location and Registration

When a Help Book is first opened (usually via `Help > [AppName] Help`):

- macOS **copies the entire `.help` bundle** from the app into:
  ```
  ~/Library/Group Containers/group.com.apple.helpviewer.content/Library/Caches/
  ```
  Example:
  ```
  net.endoquant.MRIQuantSim.net.endoquant.MRIQuantSim.help*1.0.help
  ```

This copy is **persisted across app builds** and must be manually removed to reflect updates.

---

## 🔧 Help Book Cache Management

### ✅ Delete cached Help Book:
```bash
rm -rf ~/Library/Group\ Containers/group.com.apple.helpviewer.content/Library/Caches/net.endoquant.MRIQuantSim.*
```

### ✅ Kill Help Viewer daemons:
```bash
killall -9 helpd Tips
```

### ✅ Reset Help system preferences:
```bash
rm ~/Library/Preferences/com.apple.help.plist
```

---

## 🛠️ Command-Line Tools

### 🔎 View registered Help Books:
```bash
plutil -p ~/Library/Preferences/com.apple.help.plist
```

This lists all Help Books known to the system, including:
- `appBundleUrl`
- `helpBookId`
- `version`
- `cached url`

---

### 🔎 Inspect Help system logs:
```bash
log stream --predicate 'subsystem == "com.apple.helpviewer"' --info
```

Use this to monitor registration, loading, and Tips activity.

---

### 🔎 Confirm the identity of `Tips.app`:
```bash
osascript -e 'id of app "Tips"'
# Returns: com.apple.helpviewer
```

---

## 📦 Indexing Help Books

Modern Help Books use `.cshelpindex` files (Core Spotlight format).

### ✅ Generate a new index:
```bash
hiutil -I corespotlight -Caf MRIQuantSim.cshelpindex MRIQuantSimHelp.help/Contents/Resources/
```

### ✅ Purge system Help Viewer caches:
```bash
hiutil -P
```

---

## 🧪 Help Book Build Automation

### ✅ Smart script to conditionally clear Help Viewer cache:
- Computes a hash of `.html` and `.plist` files
- Clears cache only when contents change
- Stores hash in:
  ```
  $BUILD_DIR/help_cache_last_hash.txt
  ```

**Benefits**:
- Doesn’t rely on Xcode’s dependency system
- Avoids fragile timestamps
- Works reliably in sandboxed builds

---

## 🪛 Files and Paths Summary

| Purpose                        | Path |
|-------------------------------|------|
| Cached Help Books             | `~/Library/Group Containers/group.com.apple.helpviewer.content/Library/Caches/` |
| Help Book preferences         | `~/Library/Preferences/com.apple.help.plist` |
| Help Viewer logs              | `log stream --predicate 'subsystem == "com.apple.helpviewer"'` |
| Help Viewer UI                | `/System/Applications/Tips.app` |
| Help Book copy source         | `YourApp.app/Contents/Resources/YourHelp.help` |
| Help Index (new format)       | `.cshelpindex` |
| Index generator               | `hiutil` |

---

## ✅ Recommendations

- Never assume Help Books are read live from your bundle — macOS copies them.
- Clear the cache whenever HTML changes, or use a smart script to detect changes.
- Avoid relying on `Use discovered dependency file` — timestamp precision is not reliable.
- Use command-line tools (`hiutil`, `plutil`, `log stream`) to inspect system behavior.
