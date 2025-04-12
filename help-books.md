ng stubborn system caches
- Development-time workflows

---

## 📁 1. Help Book Folder Structure

Your Help Book must be structured like this inside your app bundle:

```
MRIQuantSim.app/
└── Contents/
    └── Resources/
            └── MRIQuantSimHelp.help/
	                └── Contents/
			                ├── Info.plist
					                └── Resources/
							                    ├── en.lproj/
									                        │   └── index.html
												                    └── MRIQuantSim.cshelpindex (optional but recommended)
														    ```

> Place `index.html` inside a language-specific `.lproj` folder (e.g., `en.lproj`) for reliability.

---

## 🧾 2. Help Book Info.plist (inside `.help/Contents/Info.plist`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleHelpBookName</key>
        <string>MRIQuantSimHelp</string>
	    <key>CFBundleIdentifier</key>
	        <string>net.endoquant.MRIQuantSim.help</string>
		    <key>HPDBookAccessPath</key>
		        <string>index.html</string>
			</dict>
			</plist>
			```

---

## 🧾 3. App’s `Info.plist` (main bundle)

```xml
<key>CFBundleHelpBookFolder</key>
<string>MRIQuantSimHelp.help</string>
<key>CFBundleHelpBookName</key>
<string>MRIQuantSimHelp</string>
```

These tell macOS where to find the Help Book and what to call it in the menu.

---

## 🧪 4. Generate the Help Index (optional but recommended)

```bash
hiutil -I corespotlight -Caf MRIQuantSim.cshelpindex MRIQuantSimHelp.help/Contents/Resources/
```

Place the resulting `MRIQuantSim.cshelpindex` into the same Resources folder.

---

## 🧹 5. Clear System Help Caches (during development)

If your help page isn't updating, run:

```bash
hiutil -P
killall -9 Tips helpd
rm -f ~/Library/Preferences/com.apple.help.plist
```

This purges Help system processes and preference caches — but does **not** clear the most persistent cache...

---

## 💣 6. The Secret Cache You *Must* Clear

The real culprit is here:

```
~/Library/Group Containers/group.com.apple.helpviewer.content/Library/Caches/
```

macOS creates a full copy of your Help Book on **first launch**, named like:

```
<your-app-id>.<help-book-id>*<version>.help
```

To force an update:

```bash
rm -rf ~/Library/Group\ Containers/group.com.apple.helpviewer.content/Library/Caches/net.endoquant.MRIQuantSim.*
```

---

## 🧰 7. Troubleshooting Flowchart

1. ❓ Content not updating?
2. ✅ Confirm index.html is correct in `.help`
3. ✅ Confirm app Info.plist keys are correct
4. ✅ Clean and rebuild
5. ✅ Run `hiutil -P` and clear Preferences
6. ✅ **Delete cached copy in Group Containers**
7. ✅ Relaunch app and re-test

---

## 🧪 8. Dev Tip: Rename Help Book to Force Fresh Load

When you hit a wall, try:

- Renaming `MRIQuantSimHelp.help` to `MRIQuantSimHelpV2.help`
- Updating all `Info.plist` keys accordingly
- Changing `index.html` to `index2.html` and updating `HPDBookAccessPath`

This tricks macOS into treating your Help Book as completely new.

---

## 📌 9. Summary: Do’s and Don’ts

✅ DO:
- Use `en.lproj/index.html`
- Use `hiutil -P` regularly during testing
- Include `.cshelpindex` if pre-indexing content
- Clear Group Container cache manually

❌ DON’T:
- Assume `index.html` is live once it appears in Help Viewer
- Trust `hiutil -Tvf` to validate modern `.cshelpindex` files (it no longer works)

---

## 📎 License

MIT
~