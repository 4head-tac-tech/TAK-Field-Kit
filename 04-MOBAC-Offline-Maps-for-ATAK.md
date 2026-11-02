# MOBAC (Mobile Atlas Creator) for ATAK — Offline Maps Guide

How to use **Mobile Atlas Creator (MOBAC)** on a PC to download map tiles for a chosen area, package them into an offline map file, and load that file into **ATAK** on an Android phone. Useful when you will be outside cell coverage, or when you do not want the phone pulling maps over the network.

> **Use at your own risk. No warranty.** This guide is an unofficial, community-written resource provided "as is." It is not affiliated with or endorsed by the TAK Product Center or the MOBAC project. Map providers have **terms of use that often forbid bulk downloading**, and violating them can get you blocked or create legal exposure. Read **Section 2** and the full **[Legal notice, license and disclaimer](#legal-notice-license-and-disclaimer)** before you download anything.

---

## What you need

| Item | Notes |
|---|---|
| A Windows, Mac or Linux PC | Downloads and file building happen here, not on the phone |
| Java runtime | MOBAC is a Java program. Use a current, supported Java build; if it will not start, try a different Java version |
| MOBAC | From the project page at mobac.sourceforge.io. Extract (unzip) it fully before running |
| Free disk space | Offline maps get large quickly. Plan for gigabytes |
| A map source you are **allowed** to bulk download | See Section 2 |
| An Android phone with **ATAK-CIV** | To test the finished map |
| A USB cable or other way to move files to the phone | |

---

## 1. What MOBAC does

MOBAC pulls map tiles (small square images) from an online map server for the area and zoom levels you choose, then bundles them into one file that a mobile app can read offline. ATAK can then show those tiles with no internet connection.

Key terms:

| Term | Meaning |
|---|---|
| **Map source** | Where the tiles come from. Defined in small XML files in MOBAC's `mapsources` folder |
| **Tile** | One small map image at a given zoom level and position |
| **Zoom level** | How detailed the map is. Each step up roughly quadruples the number of tiles |
| **Atlas** | The finished output: the offline map file(s) |
| **Atlas format** | The file type MOBAC builds. Pick the one your app can read |
| **Tile store / cache** | MOBAC keeps tiles it already downloaded so repeat downloads are faster |

## 2. Read this first: legality and map-provider rules

This is the part people skip and later regret.

- **Most free online tile servers prohibit bulk downloading.** The OpenStreetMap Foundation's published tile policy, for example, prohibits downloading tiles in advance to build an archive or for offline use from its own servers, and it blocks heavy users. Many commercial providers (including large web mapping companies) also forbid scraping tiles in their terms.
- **Having a technical way to download something is not permission to do it.** MOBAC can fetch from many sources; that does not mean the provider allows it.
- **Safer sources:**
  - Providers that explicitly allow offline use or bulk download, often with a paid plan or API key.
  - Data you already have a licence for.
  - Imagery and map data released under open licences, served from a tile server **you run yourself** or one that explicitly permits bulk download.
  - Tile packages the provider publishes for download.
- **Check each source's terms yourself** before adding it. Terms change.
- **Be gentle.** Keep your area and zoom range as small as you actually need. Huge downloads burden the provider and are the fastest way to get blocked.
- **Do not share** downloaded atlases of a provider's tiles with others unless the provider's licence allows redistribution.
- **Copyright and attribution** may apply to the map data. Keep any required credit.

If you cannot confirm that a source allows what you are about to do, do not use it for bulk downloads.

## 3. Install MOBAC

1. Download MOBAC from the official project page (mobac.sourceforge.io). Avoid reposted copies from unknown sites.
2. **Extract the whole zip** to a folder you control, e.g. `C:\MOBAC` (Windows) or `~/mobac` (Mac/Linux). Do not run it from inside the zip.
3. Install a supported Java runtime if you do not have one.
4. Start MOBAC:
   - **Windows:** run `Mobile Atlas Creator.exe`
   - **Mac / Linux:** run the start script `start.sh` from a terminal (you may need `chmod +x start.sh` first)
5. The first launch may show a short setup dialog. Accept the defaults and choose a language if asked.

Folders to know inside the MOBAC folder:

| Folder | Purpose |
|---|---|
| `mapsources/` | Map source XML files (add custom ones here) |
| `atlases/` | Default output location for finished atlases |
| `tilestore/` | MOBAC's own tile cache |

## 4. Add or check a map source

MOBAC ships with some sources. Many may no longer work, since providers change or block access. Others you add yourself.

To add a source:
1. Get the source's XML definition from a provider that permits what you intend to do.
2. Save it in the `mapsources` folder with a lowercase name and the `.xml` extension.
3. **Restart MOBAC.** The new source appears in the map source drop-down.

Tips:
- When copying XML from a web page, paste into a plain text editor first. Smart quotes and stray formatting will break the file.
- Save as **All Files** so the editor does not append `.txt`.
- Keep the XML exactly as the provider documents it, including any API key field you are entitled to use.
- If the source does not appear, check the filename, the folder, and the XML for typos.

## 5. Build an atlas, step by step

1. **Open MOBAC.**
2. **Choose the atlas format** (the drop-down near the top left). See the format guidance in Section 6.
3. **Name the atlas** (optional but useful), e.g. `Area01-Terrain`.
4. **Pick the map source** from the drop-down.
5. **Navigate** the preview map to your area (scroll, zoom, or search by coordinates).
6. **Select the area.** Click and drag a rectangle (or use the polygon/circle tools, if your version offers them).
7. **Choose zoom levels.** Tick only the levels you need. Lower numbers are zoomed out, higher numbers are zoomed in. Start with a small range.
8. **Check the tile count and size estimate.** MOBAC shows how many tiles you are about to fetch. If the number is huge, shrink the area or drop high zoom levels.
9. Click **Add Selection** (add it to the atlas). You can add several areas or zoom ranges as separate entries.
10. Open **Settings** and confirm the **output directory** and the tile-store/cache settings. Also check the **thread count and delay settings**, and keep them modest so you do not hammer the provider.
11. Click **Create Atlas.**
12. Wait. MOBAC downloads the tiles, then builds the file. Do not close it early.
13. When it finishes, choose **Open Atlas Folder** to find the result. By default it is in the `atlases` folder.

## 6. Choosing the atlas format for ATAK

MOBAC can write many formats. For ATAK, the usual choices are SQLite-based tile files. Formats commonly used with ATAK and similar Android apps include:

| Format in MOBAC | Notes |
|---|---|
| **Osmdroid SQLite** | A common choice for ATAK because ATAK is built on osmdroid-style tile handling |
| **MBTiles SQLite** | A widely supported standard container |
| **RMaps SQLite** | Works with several Android apps; check ATAK support in your version |

> ATAK versions differ in which map file types they accept and how they are recognised. **Test a very small atlas first** (one tiny area, two or three zoom levels) to confirm your ATAK version reads your chosen format before you spend hours on a large download.

## 7. Loading the map into ATAK, iTAK and WinTAK

The finished atlas is just a file. Each TAK client has its own way of picking it up. **Do the small test atlas on each client you plan to use** before building anything large. Supported file types, folder names and menu wording differ between app versions, so treat the paths below as the usual starting point and confirm them against your version's documentation.

### 7.0 Rules that apply to every client

- Use **simple file names**: letters, numbers and dashes only. No spaces or unusual characters.
- Copy the **whole file**. A half-copied file will not load. Compare file sizes on both sides.
- Put the map somewhere on the device **before** you go offline, and test with **airplane mode / network off**.
- Keep a **backup** of the atlas on your PC.
- Do not upload an atlas built from a provider's tiles to a shared server or public location unless that provider's licence allows redistribution (see Section 2).
- Large atlases are slow and heavy to send. Prefer several smaller files.

### 7.1 ATAK (Android)

**Method A: copy the file into the imagery folder (most common)**
1. Connect the phone to the PC by USB (set the phone to **File transfer** mode), or use any file transfer method you trust.
2. Copy the finished atlas file (for example a `.sqlite` or `.mbtiles` file) to the phone.
3. Place it in ATAK's imagery folder: **`atak/imagery`** on the phone's internal storage. If the folder does not exist, open ATAK once so it creates its folders, then look again.
4. Open ATAK. Open the **map/imagery selector** (layers menu) and look for your atlas by name. Select it.
5. If it does not appear, restart ATAK so it rescans the folder.

**Method B: Import Manager**
1. Copy the file anywhere on the phone (for example the Downloads folder).
2. In ATAK, go to **Settings/Menu → Import Manager → Import → Local SD** and pick the file.
3. Choose the import type if ATAK asks (imagery/map).
4. Open the map selector and choose the new map.

**Verify:** zoom to the area you downloaded and confirm tiles load with **airplane mode on**.

### 7.2 iTAK (iPhone / iPad)

iTAK keeps files inside its own app storage, so you cannot just drop a file into a folder. You send the file **to** iTAK, and iTAK imports it. iTAK's map file support is narrower than ATAK's, so **build and test a small atlas first, and prefer the MBTiles SQLite format** from MOBAC for iTAK.

**Method A: Files app / "Open in iTAK"**
1. Get the atlas onto the iPhone or iPad: AirDrop from a Mac, iCloud Drive or another cloud drive, a USB connection through the Finder (Mac) or iTunes/Apple Devices app (Windows), or a private link.
2. Open the **Files** app and find the file.
3. Tap the file (or touch and hold, then **Share**) and choose **iTAK** from the share sheet or **Open in iTAK**.
4. iTAK imports it. Watch for a confirmation message.

**Method B: iTAK's own import/maps area**
1. Open iTAK and go to the **Maps / Layers** area (the exact tab name varies by version).
2. Choose the option to **import** a map or add a map source, and pick the file from Files.
3. Select the imported map as your base map.

**Method C: data package**
If the map is wrapped in a TAK data package (Section 7.4), import the package through iTAK's package import.

**Verify:** switch the base map to your atlas, turn on airplane mode, and pan and zoom around the downloaded area.

> If iTAK refuses the file, rebuild a small test in a different MOBAC format. iTAK may not read every format ATAK does.

### 7.3 WinTAK (Windows)

WinTAK reads map files from folders in its data directory, usually under your **Documents** folder in a `wintak` folder (with an `imagery` or `Maps` subfolder depending on version). Confirm the exact location on your install: in WinTAK, the **Import Manager** and **Settings → Maps** screens usually show or open it.

**Method A: copy into the imagery folder**
1. Close WinTAK, or be ready to restart it.
2. Open File Explorer and go to your WinTAK data folder (typically `Documents\wintak`).
3. Open the **imagery** folder (create it if it does not exist, matching the name WinTAK uses on your version).
4. Copy the atlas file in, then start (or restart) WinTAK.
5. Open the **map/layers** selector and choose the new map.

**Method B: Import Manager**
1. In WinTAK, open the **Import Manager** (or the import option in the menu or toolbar).
2. Choose **Import**, select the atlas file, and confirm the import type if asked.
3. Open the map selector and choose the map.

**Method C: MOBAC map source XML**
For online sources that you are permitted to use, WinTAK can also load a MOBAC-format map source XML instead of a prebuilt atlas. Place the XML in the maps/imagery folder and it will appear as a selectable map. That pulls tiles live (and caches them) rather than working fully offline, so it does not replace a prebuilt atlas for a no-signal trip.

**Verify:** disconnect the PC from the network and confirm the map still draws in your downloaded area.

### 7.4 Sharing one atlas with several people (data package)

Instead of copying files by hand to every device, you can wrap the atlas in a **TAK data package** (a zip with a manifest) and distribute it.

1. In ATAK, use the **data package / mission package tool** to create a package, add the atlas file to it, give it a clear name, and save it.
2. Share the package in one of these ways:
   - **Send it to contacts** from ATAK (it moves over the TAK Server).
   - **Upload it to the TAK Server's file/data package area**, if your server operator allows it, so others can download it.
   - **Copy the zip directly** to each device, then import it through that client's Import Manager (ATAK, WinTAK) or **Open in iTAK**.
3. Each person imports the package on their own client, then selects the map in the map selector.

Cautions:
- **Size.** Big packages can overload a small server or a slow link. Prefer hand-copying for large files, and keep packages as small as you can.
- **Everyone with access can download it.** Only share atlases you are allowed to redistribute.
- **Location sensitivity.** The area you choose can reveal where you operate. Share with only the people who need it.
- Ask the server operator before uploading large files to the TAK Server.

### 7.5 Which method should I use?

| Situation | Best method |
|---|---|
| One Android phone | Copy to `atak/imagery` |
| One iPhone/iPad | Share the file to iTAK from the Files app |
| One Windows PC | Copy into the WinTAK imagery folder or use Import Manager |
| Several devices, small file | Data package sent through TAK Server |
| Several devices, large file | Copy the file to each device by hand or from a shared drive |
| Mixed ATAK, iTAK and WinTAK | Build one MBTiles atlas, test on every client first |

### 7.6 Test on every client

1. Import the small test atlas.
2. Select it as the map.
3. Turn on **airplane mode** (phones) or disconnect the network (PC).
4. Pan and zoom around the downloaded area at several zoom levels.
5. Note any gaps, blur or missing areas, then rebuild with a larger area or more zoom levels if needed.
6. Only then build and load the full atlas.

## 8. Managing size and storage

- **Zoom is the main driver of size.** Each additional zoom level roughly multiplies the tile count by four. Dropping the highest zoom level often cuts the size dramatically.
- Use **several small atlases** rather than one huge one. They are easier to copy, test and replace.
- Consider **lower image quality** (JPEG compression) for aerial/satellite imagery if MOBAC offers it, to save space.
- Keep an eye on phone storage and on the file-size limits of the file system you copy through.
- Delete old atlases you no longer need.
- Keep a **backup of finished atlases** on your PC, since re-downloading is slow and may be restricted.

## 9. Updating maps

Tiles are a snapshot. Roads, trails and imagery change. Rebuild your atlas periodically for areas you rely on, and note the date in the file name, e.g. `Area01-Terrain-2026-10`.

Clear MOBAC's tile store (`tilestore/`) if you want fresh tiles instead of cached ones. Do that only if your provider's terms allow re-downloading.

## 10. Practical checklist

Before you leave coverage:

- [ ] Source's terms checked and bulk download is allowed
- [ ] Test atlas built and loaded in ATAK first
- [ ] Final atlas covers the **whole** area you will travel, plus a margin
- [ ] Highest zoom level is sufficient for navigation
- [ ] File loaded on **every** client you will use: ATAK (`atak/imagery`), iTAK (Open in iTAK) and WinTAK (imagery folder or Import Manager)
- [ ] Map visible in each client's map selector
- [ ] Tested in airplane mode / with the network off on each device
- [ ] Backup copy on PC
- [ ] Phone storage has space left over
- [ ] You do not depend on this as your only navigation aid

---

## Common mistakes and corrections

| Mistake | What happens | Correction |
|---|---|---|
| Bulk downloading from a provider that forbids it | Blocked, throttled or in breach of terms | Use a source that allows offline use, or run your own tile server |
| Selecting a huge area at high zoom | Enormous download, days of work, provider complaints | Shrink the area or drop the top zoom levels; check the tile count first |
| Running MOBAC from inside the zip file | Errors or it won't start | Extract everything first |
| Missing or wrong Java version | MOBAC won't launch | Install a supported Java runtime; try another version if needed |
| Pasting source XML with smart quotes or formatting | The source fails to load | Paste into plain text, retype quotes, save as `.xml` |
| Saving the XML as `google.xml.txt` | Source never appears | Choose **All Files** when saving; check the extension |
| Not restarting MOBAC after adding a source | New source missing | Restart MOBAC |
| Putting the XML in the wrong folder | Source missing | It goes in `mapsources/` |
| Picking an atlas format ATAK doesn't read | Map doesn't show in ATAK | Test a tiny atlas first; try Osmdroid SQLite or MBTiles |
| Never testing a small atlas | Hours wasted on an unusable file | Always build and test a tiny one first |
| Forgetting to click **Add Selection** | Atlas is empty or wrong | Add the selection, then create the atlas |
| Not checking the output directory | You can't find the file | Use **Open Atlas Folder** or set the directory in Settings |
| Aggressive thread count / no delay | Provider blocks you | Keep settings modest |
| Copying the file to the wrong phone folder | ATAK doesn't see it | Use `atak/imagery`, then restart ATAK or import it |
| Trying to drop a file into an iPhone folder for iTAK | iTAK never sees it | Share the file to iTAK from the Files app (**Open in iTAK**) |
| Assuming iTAK reads every format ATAK does | iTAK rejects the file | Build a small MBTiles test and check it in iTAK first |
| Copying into the wrong WinTAK folder | Map not listed in WinTAK | Check the data folder path shown in WinTAK, then restart it |
| Leaving WinTAK open while copying files | New map not picked up | Restart WinTAK after copying |
| Using a MOBAC source XML in WinTAK and expecting offline use | Map is blank with no network | Use a prebuilt atlas for offline trips |
| Sending a huge atlas through the TAK Server | Slow, stalled or overloaded server | Hand-copy large files; keep packages small |
| Uploading a provider's tiles to a shared server | Breaches the provider's licence | Only share atlases you are allowed to redistribute |
| Half-copied or interrupted transfer | Corrupt or missing map | Re-copy and compare file sizes |
| Phone set to charge-only over USB | PC can't see the phone's files | Set USB mode to File transfer |
| Spaces or odd characters in the file name | Import problems | Simple names only |
| Testing with Wi-Fi on | Tiles load from the internet and hide a failed offline map | Test with airplane mode |
| Stale cache | Old imagery in the new atlas | Clear the tile store when permitted |
| Phone storage full | Copy fails or ATAK misbehaves | Free space first |
| Cutting the area too tight | Map vanishes at the edge of your route | Add a margin around the planned area |
| Using this as your only navigation | Dead battery or file error leaves you stuck | Carry independent backups |

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| MOBAC won't open | Java missing or the wrong version. Reinstall Java; on Mac/Linux run `start.sh` from a terminal to see error messages |
| Map source list is empty or missing a source | XML in the wrong folder, bad filename, or not restarted |
| Preview map is grey or blank | The source is offline, blocking you, or needs an API key |
| Download stalls or errors | Provider throttling or blocking. Pause, reduce threads, or switch to a source that permits your use |
| Atlas builds but it is tiny | Selection wasn't added, or the wrong zoom levels were ticked |
| ATAK doesn't list the map | Wrong folder, unsupported format, or ATAK hasn't rescanned. Restart ATAK; try another format |
| iTAK imports but shows no map | Unsupported format or wrong selection. Rebuild as MBTiles and select it as the base map |
| WinTAK doesn't list the map | Wrong folder, WinTAK not restarted, or unsupported format. Check the data folder path and restart |
| Package imports but the map is missing | The atlas wasn't added to the package, or the import type was wrong. Rebuild the package and re-import |
| Map shows but with gaps | Area or zoom levels didn't cover that region. Rebuild with a larger selection |
| Map is blurry when zoomed in | Highest downloaded zoom is too low. Add higher levels (expect a bigger file) |
| File too big to transfer | Split into smaller atlases or reduce zoom |

---

## Credits and links

- MOBAC (Mobile Atlas Creator): https://mobac.sourceforge.io
- TAK Product Center (ATAK): https://tak.gov
- OpenStreetMap tile usage policy: https://operations.osmfoundation.org/policies/tiles/

*MOBAC and ATAK versions differ. Menu names, supported formats and folder behaviour may change. Verify against current documentation and test with a small atlas first.*

---

## Legal notice, license and disclaimer

**This is general boilerplate, not legal advice.** If you need certainty about your obligations, ask a qualified lawyer.

### Use at your own risk; no warranty

This guide, and any scripts, configuration files or other code distributed with it (together, the "Materials"), are provided **"as is" and "as available," without warranty of any kind**, express or implied. That includes, without limitation, any warranty of merchantability, fitness for a particular purpose, accuracy, completeness, security, reliability or non-infringement.

To the fullest extent permitted by law, the authors and contributors are **not liable** for any claim, damages or other liability, whether in contract, tort or otherwise, arising from or connected with the Materials or their use. This includes data loss, corrupted or wiped storage, hardware damage, downtime, being blocked by a service provider, financial loss, injury, or any other harm. You follow this guide entirely at your own risk.

### Accuracy and AI assistance

Parts of this guide were drafted with the help of an AI assistant and may contain errors, omissions or outdated steps. Software, file formats, folder locations and menu names change. Always check against the official documentation for MOBAC, ATAK and any map provider you use, and test on non-critical data first.

### Map data, tile servers and terms of use

- **You are responsible for the terms of every map source you use.** Many providers prohibit bulk downloading, scraping, offline caching or redistribution of their tiles. Using MOBAC against such a source may breach those terms and may lead to blocking or legal claims.
- This guide does not grant any right to use any provider's map data, imagery or tiles. Map data and imagery are usually protected by copyright and database rights. Respect attribution and licence requirements.
- Do not redistribute atlases built from a provider's tiles unless that provider's licence permits it.
- Third-party source definitions (XML files) are not endorsed or checked by this guide.

### Commands and file operations can be destructive

Copying, overwriting or deleting files on a phone or PC can cause data loss. Back up anything you care about first.

### Not affiliated; trademarks

This is an unofficial, independent community guide. It is **not affiliated with, endorsed by or sponsored by** the TAK Product Center, the MOBAC project, OpenStreetMap, any government agency, any map provider, or any other company or project mentioned. TAK, ATAK, WinTAK, iTAK, MOBAC, OpenStreetMap, Android, Java, Windows and all other names, logos and trademarks belong to their respective owners and are used here only to identify the products.

### Third-party software and content

- **ATAK and TAK software** are not included. Obtain them from the official source (tak.gov) and follow the terms you accept there. Do not redistribute them.
- **MOBAC, Java and other tools** have their own licenses, maintainers and risks. You are responsible for reviewing and complying with them.
- Links point to external sites that the authors do not control.

### Privacy, consent and the law

Offline maps and the places you choose to download can reveal where you operate and travel. Protect finished atlases and the files that describe your areas of interest as you would any location data. You are responsible for complying with all laws and rules that apply to you, including copyright and database-right law, privacy and data-protection law, and export-control and other legal requirements for the software and data you use. Do not use these Materials for any unlawful purpose.

### Not for safety-critical use

Do not rely on a phone, an offline map or any software described here as your **only** means of navigation, communication or emergency response. Maps can be out of date or wrong, files can corrupt, batteries die and apps fail. Carry independent backups (paper maps, a compass, a charged spare power source), test your setup in advance, and use sound judgment, especially outdoors or anywhere a failure could put someone at risk.

### License

Unless stated otherwise, and subject to the disclaimers above:

- **The text of this guide** is licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license.
- **Any code, scripts or configuration files** distributed with this guide are licensed under the **MIT License**.

Copyright (c) 2026 4head-tac-tech. Third-party software and content keep their own licenses. By contributing, you agree that your contribution is licensed on the same terms. The warranty disclaimer and limitation of liability above apply to everyone who uses, copies, modifies or redistributes the Materials.

### Reporting problems

Found a mistake, a security issue or an outdated step? Please open an issue or a pull request in the project repository: https://github.com/4head-tac-tech/TAK-Field-Kit.
