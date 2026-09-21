# Prayer Library (GitHub Pages)

A small static website that hosts your prayers for the Prayer App.

- The **app** downloads one small file, `prayers.json`, which lists every prayer.
- When the user taps download on a prayer, the app downloads that prayer's **audio file** from the same site and keeps it on the phone. After that it plays offline, so scheduled prayers work without internet.
- The **website** (`index.html`) lets anyone listen in a browser and shows the link the app uses.

```
prayer-library/
  index.html            the page people see
  prayers.json          the list the app downloads (generated, do not edit by hand)
  prayers.source.json   YOU edit this: id, title, category, description, file
  site.config.json      your public site address (filled in by the build script)
  audio/                your audio files (.m4a, .mp3, .aac)
  tools/build.py        creates prayers.json from prayers.source.json
  .github/workflows/    optional check that runs on every push
  .nojekyll             tells GitHub to serve files as they are
```

## 1. Prepare on your computer

1. Install Python 3 from python.org (tick "Add Python to PATH"), then run once:
   ```
   pip install mutagen
   ```
2. Put your audio files in `audio/`. Use simple names: letters, numbers, `-`, `_`, `.` (no spaces). Example: `morning_gratitude.m4a`.
3. Open `prayers.source.json` and add one entry per prayer:
   ```json
   {
     "id": "morning_gratitude",
     "title": "Morning Gratitude",
     "category": "morning",
     "description": "A gentle start to the day.",
     "file": "morning_gratitude.m4a"
   }
   ```
   - `id`: lowercase letters, numbers, `_` or `-`. **Never change an id after people have downloaded it.**
   - `category`: one of `morning`, `evening`, `meal`, `protection`, `gratitude`, `custom`.
   - To hide a prayer without deleting it, add `"enabled": false`.
4. Build the list. The first time, give your public address (your GitHub username and repository name):
   ```
   python tools/build.py --base-url https://YOUR-USERNAME.github.io/prayer-library
   ```
   The script reads each audio file, fills in `audio_url`, `duration` and `file_size`, and stops with a clear message if something is wrong. After the first time you only run `python tools/build.py`.

The included `test_chime.m4a` is only a sample sound so you can test the whole download flow. Remove it when you add real prayers.

## 2. Put it on GitHub

1. On github.com choose **New repository**. Name it `prayer-library` and choose **Public**. (Free GitHub Pages needs a public repository, so anyone can see the files.)
2. Click **uploading an existing file** and drag in everything from this folder, including the `audio` and `tools` folders and the hidden files `.nojekyll` and `.github` (in Windows Explorer turn on View > Show > Hidden items). Then click **Commit changes**.
3. Go to **Settings > Pages**. Under **Build and deployment** choose **Deploy from a branch**, select branch `main` and folder `/ (root)`, and press **Save**.
4. Wait one or two minutes. Your site is at `https://YOUR-USERNAME.github.io/prayer-library/`.

## 3. Test it

Open these in a browser:

- `https://YOUR-USERNAME.github.io/prayer-library/` shows the page with your prayers and players.
- `https://YOUR-USERNAME.github.io/prayer-library/prayers.json` shows the list.
- Press play on a prayer on the page. If it plays, the app can download it.

## 4. Connect the app

In the Flutter project, open `lib/core/constants/app_constants.dart` and set:

```dart
static const String prayersJsonUrl =
    'https://YOUR-USERNAME.github.io/prayer-library/prayers.json';
```

Then in the app press **Download Prayers**. The prayers appear under **Predefined**. Tap the download button on a prayer to save its audio to the phone.

## Adding or changing prayers later

1. Add the audio file to `audio/` and an entry to `prayers.source.json`.
2. Run `python tools/build.py`.
3. Upload the changed files (or `git add . && git commit -m "Add prayer" && git push`).
4. GitHub Pages updates in about a minute. Phones may see the old list for up to about 10 minutes because GitHub caches files.

If you replace the audio of an existing prayer, keep the same id and file name. Users who already downloaded it keep their old copy until they download it again.

## Making good audio files

`.m4a` (AAC) is best. For speech, mono at 64 to 96 kbps sounds clear and stays small. With ffmpeg:

```
ffmpeg -i input.wav -c:a aac -b:a 96k -ac 1 -movflags +faststart audio/my_prayer.m4a
```

Add `-af loudnorm` to even out the volume between prayers.

## Good to know

- **Rights:** only publish audio you made or have permission to share. The repository is public and anyone can download the files.
- **Size:** GitHub allows about 1 GB per site, 100 MB per file, and roughly 100 GB of traffic per month. That is plenty for spoken prayers.
- **HTTPS:** GitHub Pages uses `https://`, which Android requires.
- **Not for secrets:** everything here is public. Do not put private recordings in this repository.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `404` on the site | Pages is not enabled yet, or the branch/folder is wrong (Settings > Pages). Wait two minutes. |
| `prayers.json` shows `YOUR-USERNAME` | Run `python tools/build.py --base-url https://<your-username>.github.io/prayer-library`, then upload again. |
| Build says "audio/... not found" | The `file` in `prayers.source.json` must match the file name exactly, including capital letters. |
| The page shows the list but audio will not play | Open the audio URL from `prayers.json` directly. A 404 means the file was not uploaded. |
| A red cross next to your commit on GitHub | Run `python tools/build.py`, upload the new `prayers.json`. |
| Changes do not show in the app | Wait about 10 minutes for the GitHub cache, then press Download Prayers again. |
