# Crate

A simple DJ library manager for macOS (it also runs on Windows and Linux). Import your music, fix tags, build playlists and smartlists, then send everything to **Engine DJ** or **rekordbox**.

It's a simplified take on what apps like Lexicon do. It's written from scratch and has only one dependency, Electron.

**Version 1.9.** The version shows next to the name in the window, in Crate › About Crate, and in the window title. What changed in each version is in [CHANGELOG.md](CHANGELOG.md).

## Install it (macOS)

You need [Homebrew](https://brew.sh). Everything else is installed for you.

**One click:** double-click **Install.command** in the project folder. Terminal opens, builds Crate, installs it in /Applications and opens it. The first time, macOS may ask whether to open a file from Terminal; choose Open. If it says the file can't be opened, right-click Install.command, choose Open, then Open again.

**From Terminal** (the same thing):

```bash
cd ~/Downloads/crate          # wherever the folder is
./build-mac.sh --install
```

The script:

1. Checks for Homebrew.
2. Installs or upgrades Node.js with `brew` if yours is older than 22.13.
3. Downloads Electron (about 100 MB, first run only) and unpacks it with macOS's own tools.
4. Runs the tests.
5. Builds `Crate.app` for your Mac's chip (Apple Silicon or Intel) and signs it ad hoc so macOS will open it.

   Why not a standard packaging tool: the zip library that Electron's installer and `@electron/packager` use stops silently part-way on recent Node versions, leaving no app, so the script uses macOS's `ditto` instead.
6. With `--install`, copies the app to `/Applications`.

Then open it from Launchpad or with `open /Applications/Crate.app`.

| Command | Does |
| --- | --- |
| `./build-mac.sh` | Build only; the app ends up in `dist/Crate-darwin-arm64/` (or `-x64`) |
| `./build-mac.sh --install` | Build and install to `/Applications` |
| `./build-mac.sh --skip-tests` | Build without running the tests |
| `npm start` | Run straight from the source folder, no build (after one `npm install`) |

Optional: `brew install ffmpeg` lets the tag tests double-check written files with ffprobe.

To update after changing the code, run `./build-mac.sh --install` again. Your library is kept separately in `~/Library/Application Support/Crate/library.db`, so rebuilding never touches it.

## What it does

**Library and tags**
- Import folders or files with the toolbar button, or drag them from Finder onto the window. Re-importing only re-reads files that changed.
- Supports MP3, FLAC, M4A/AAC/ALAC, AIFF and WAV.
- Edit title, artist, remixer, mix, album, genre, label, BPM, key, year, track number, composer, grouping, comment and rating. Changes are written into the files, so other DJ apps see them too.
- Select several tracks to edit them together. Fields left as "Mixed" keep each track's own value.
- Other apps' data inside your files is left alone: artwork, Serato cue and beatgrid data, Mixed In Key cues, and so on. Small edits are made in place; bigger ones are written to a temporary file that then replaces the original, so a crash can't leave a half-written track.
- Keys can be shown as Camelot (8A), Open Key (1m) or musical (Am). Key chips are coloured by position on the Camelot wheel, so tracks that mix well have similar colours.
- Columns: drag a header sideways to move it, drag its edge to resize, double-click the edge to fit the contents. Right-click the header to show or hide columns, or to reset the order or widths.
- Search across title, artist, album, remixer, label, genre and comment. Accents are ignored and partial words match.
- Preview player: double-click a track, or press Space. AIFF files are played by converting them to WAV as they stream; the file itself isn't changed.

**Playlists**
- Playlists, folders, and drag-and-drop: drag tracks onto a playlist, drag rows to reorder them, drag a playlist onto a folder.
- Smartlists are rule-based and update themselves. Rules can use any field, BPM ranges, date added, playlist membership, and "key mixes with 8A" (harmonic matches in any notation). They can also be limited and sorted.

**Clean up** (every tool shows a preview before anything changes)
- Fix titles & artists: tidy spacing, underscores to spaces, remove web addresses and promo text, remove "01 - " number prefixes, Title Case all-caps or all-lowercase titles, fill in the remixer from "(Someone Remix)", and split "Artist - Title".
- Find duplicates: matches on artist, title and remix, plus length. It suggests keeping the best-quality copy, and can optionally move the extras to the Trash.
- Rename genres: each spelling is listed separately, so "house" and "House" can be merged.
- Find & replace in any field.
- Write tags to file (right-click tracks): saves Crate's Artist, Title, Album, Genre, Year and Comment into the music files, and leaves every other tag in the file alone. Useful after an import that didn't save to the files.
- Save to file (track panel) writes your edits into the file and renames it to “Artist - Title”, with Undo. To rename many files at once, use Clean up › Rename files to “Artist - Title”…, which previews every new name and also has Undo. Files stay in their folder. Crate's next Engine DJ export moves Engine's tracks to the new names, so their cues and analysis stay. In rekordbox, use its own relocate feature for renamed files.
- Check for missing files, and relink them.

**Import from other DJ apps** (Import music › From rekordbox… / From Engine DJ…). Playlists go straight into your playlists, with no wrapper folder.
- **rekordbox 6/7:** reads `master.db` from `~/Library/Pioneer/rekordbox` (or a folder you choose) and decrypts it in memory; rekordbox itself is never changed. `masterPlaylists6.xml` decides which playlists exist, since `master.db` can still hold deleted or cloud-only ones. Playlists, folders and their order come across, and so do smart playlists (see below). rekordbox's own sampler sounds and demo tracks are skipped unless you tick the box. Quit rekordbox first so its latest changes are saved.
- **Engine DJ:** reads `Engine Library/Database2/m.db` (from a copy, so Engine can stay open). Tracks, keys, ratings, BPM, playlists, smartlists and folders come across, in Engine's order.
- Tracks match your Crate library by file path. By default, Crate keeps its own tags and only fills empty fields; you can choose to take the DJ app's tags instead, and optionally save them into the files.
- Files that aren't where the DJ app says, for example a library made on another computer, are added as missing so you can relink them. The dialog can also fix the paths: replace `C:/Users/me/Music` with `/Users/me/Music`, for example.
- Imported playlists, folders and smart playlists go straight to the top level of your playlists. Importing from the same app again replaces the ones the last import made.
- A playlist you already have in Crate with the same name is replaced by the imported one, so a round trip from Crate to the DJ app and back never shows a playlist twice. Playlists with other names that you made in Crate stay.
- Cue points come across (see below). Beatgrids aren't imported.

**Cue points**
- Crate keeps hot cues A–H, memory cues, and saved loops (Engine's and Serato's 8 loop pads), each with a name and colour. Hot cues and memory cues can also be loops.
- **Read from:** rekordbox's `master.db`, Engine DJ's database, and the Serato tags inside your music files (written by Serato DJ and Mixed In Key). When a track has cues from more than one place, cues imported from rekordbox or Engine win over the file's Serato cues. In the import dialog, "only fill what's empty" keeps cues a track already has; "use the DJ app's" replaces them.
- **Shown:** in a pane at the bottom left, above the play button, for the selected track, or for the playing track when the selected one has none (click a cue to play from it); as marks on the player's progress bar, and as a count in the optional Cues column.
- **To rekordbox:** the XML includes them. rekordbox takes cues when a track is added to your collection from the XML. For tracks already in your collection, use Import To Collection on them in the rekordbox xml panel.
- **To Engine DJ:** written into Engine's database. Engine has 8 cue pads, 8 loop pads and one main cue, but no memory cues:
  - Hot cues go to the cue pads, and saved loops to the loop pads.
  - Hot loops and memory loops fill any free loop pads.
  - The first memory cue becomes the main cue. Any other memory cues are left out, and the export message counts them.
  - Only tracks that have cues in Crate are changed. Cues you made in Engine on other tracks stay.
- Cue times are stored to the microsecond, so a trip through Engine lands on the same sample. MP3s can be off by a few milliseconds between apps, because apps differ in how they count the encoder's silent start; AIFF, WAV and FLAC are exact.

**Smart playlists between apps**
- **From rekordbox:** each smart playlist becomes a Crate smartlist with the same rules (artist, title, album, genre, label, comment, BPM, key, rating, year, length, date added, and so on, with "all" or "any" matching), so it keeps updating in Crate. A few rekordbox rules have no Crate equivalent: My Tag, colour and play count. Smart playlists that use them are skipped, and the import dialog lists them with the reason.
- **From Engine DJ:** Engine smartlists become Crate smartlists in the same folders, including their rules for genre, artist, BPM, key, rating, date added, file type and so on.
- **To Engine DJ:** Crate smartlists are written as real Engine smartlists, so they keep updating in Engine. Engine's rules are simpler than Crate's, so a smartlist that uses a limit or sort, playlist membership, "is empty", or a field Engine doesn't filter on (remixer, composer, grouping, bitrate) is exported as a normal playlist with its current tracks instead. The export message lists which ones. Older Engine libraries that have no smartlists get normal playlists instead.
- **To rekordbox XML and M3U8:** these formats can't hold rules, so smartlists are exported as normal playlists with their current tracks.

**Export** (all exports put your playlists at the top level, with no wrapper folder)
- **Engine DJ:** writes straight into your Engine Library (`~/Music/Engine Library/Database2/m.db`). It adds or updates tracks, writes cue points, and makes Engine's playlists match Crate's exactly: every playlist, folder and smartlist in Engine is replaced with yours, at the top level. Engine's Prepare list is kept. Quit Engine DJ first. A backup (`m.db.crate-backup-<date>`) is saved next to the database before every export. Engine analyses beatgrids and waveforms itself.
- **rekordbox library (native):** writes straight into rekordbox 6/7's own library (`~/Library/Pioneer/rekordbox/master.db` and `masterPlaylists6.xml`). Your tracks, tags, playlists, smart playlists and cue points then appear in rekordbox without importing anything.
  - rekordbox's playlists are made to match Crate's exactly: every playlist, folder and smart playlist there is replaced with yours, at the top level. rekordbox's built-in lists (Cloud Library Sync, CUE Analysis Playlist) are kept.
  - Quit rekordbox first. Every export saves a copy of both files in “Crate backups” next to them. To roll back, quit rekordbox and copy them back.
  - Cue points aren't written for FLAC or VBR MP3 files: rekordbox stores extra seek data for those that only it can create.
- **rekordbox XML:** exports an XML file of your whole library and every playlist. In rekordbox, open Preferences › Advanced › Database › rekordbox xml and choose the file. Your playlists appear under "rekordbox xml" in the sidebar (turn on that panel in Preferences › View › Layout).
- **M3U8:** exports one playlist file per playlist, with folders mirrored as directories.

## Keyboard

| Key | Does |
| --- | --- |
| ⌘F | Search |
| ↑ ↓ (⇧ to extend) | Move through tracks |
| ⌘A | Select all |
| Space / ↵ | Play/pause / play selected |
| 0–5 | Set rating |
| ⌫ | Remove from playlist (or from the library in All tracks) |
| ⌘I / ⌘N / ⇧⌘N | Import / new playlist / new smartlist |
| ⌘E / ⌥⌘E / ⇧⌘E | Export rekordbox XML / export to rekordbox library / export to Engine DJ |

## Known limits

- There's no audio analysis: BPM and key come from your tags. Analyse in your DJ app or Mixed In Key first; Crate reads what they write.
- Beatgrids are not transferred between apps, and cue points can't be edited in Crate yet; they're carried between apps as they are.
- Big batch edits (thousands of files) pause the window while files are written.
- The app is signed ad hoc for your own Mac, not notarised by Apple, so it isn't set up to be shared with other people.
- When run with `npm start` (not the built app), the macOS menu bar says "Electron" rather than "Crate".
- The Engine DJ export was tested against Engine's 3.0.x database schema and a real Engine library's cue data, not against a running copy of Engine DJ. Do your first export with a library you have backed up. Tracks Engine hasn't analysed yet get their cue points too, but if Engine replaces them when it first analyses a track, export again afterwards.

## How it's built

```
main.js               Electron main process: window, menus, file dialogs, audio streaming, API bridge
preload.js            The only bridge to the window (sandboxed, context-isolated)
src/main/api.js       Every action the window is allowed to call
src/main/windowstate.js  Remembers the window's size and position between launches
src/core/tags/        Tag reader/writer: ID3v2.2–2.4, FLAC Vorbis comments, MP4 atoms, WAV/AIFF chunks
src/core/library.js   SQLite library (Node's built-in node:sqlite), import, playlists, full-text search
src/core/smartlist.js Smartlist rules → parameterised SQL
src/core/cleanup.js   Duplicates, title fixes, find & replace, genre renaming
src/core/keys.js      Camelot / Open Key / musical key conversion
src/core/aiffstream.js    AIFF → WAV on the fly for the player
src/core/rename.js        “Artist - Title” file renaming
src/core/cues.js          Cue points ⇄ rekordbox, Engine DJ and Serato formats
src/core/smartconvert.js  Smartlist rules ⇄ rekordbox and Engine DJ formats
src/core/import/      rekordbox master.db (SQLCipher decryption), Engine DJ m.db
src/core/export/      rekordbox library (master.db, SQLCipher), rekordbox XML, M3U8, Engine DJ
src/renderer/         The interface (plain HTML/CSS/JS, no build step)
```

Security choices: the window runs sandboxed with context isolation and a strict Content Security Policy. It can only call the actions listed in `api.js`. Audio streams through a private `crate-audio://` scheme that serves library files only. Crate opens no network ports and sends no analytics.

## Tests

```bash
npm test   # tag engine (checked against ffmpeg/ffprobe when they're installed), library, smartlists, clean-up, exports, importers
```

`test/ui-harness.js` drives the real interface in Chromium through Playwright for end-to-end checks. It needs `npm i -D playwright`.
