# How to push this to GitHub

Same pattern as before. New repo, same five git commands.

## 1 · On GitHub (browser)

Open https://github.com/new and fill in:

- **Repository name**: one of these (your choice — short, hyphenated, descriptive):
  - `audio-deepfake-detection` ← recommended, most discoverable
  - `lightweight-cnn-audio-deepfake`
  - `on-device-deepfake-detection-mobile`
- **Description**: *Lightweight 2.10 MB CNN for real-time audio deepfake detection — mobile-banking liveness verification. ASVspoof 2019 LA.*
- **Visibility**: **Public** (recommended) or Private — your call
- **Add README**: **OFF** (we have one)
- **Add .gitignore**: **No .gitignore** (we have one)
- **Add license**: **No license** (we have an MIT `LICENSE`)

Click **Create repository**. Copy the URL GitHub shows.

## 2 · On your machine (Git Bash)

Download the zip I sent, extract it on your Desktop. You should end up with a folder like:

```
C:\Users\PC\Desktop\audio-deepfake-detection\audio-deepfake-detection\
```

(Windows nests the folder because the zip has a folder at its root. Verify the exact path with `find ~/Desktop -maxdepth 3 -type d -name "audio-deepfake-detection"` if you're not sure.)

In Git Bash, **type these commands yourself** (don't paste — pastes sometimes get a hidden Unicode character that breaks `cd`):

```
cd ~/Desktop/audio-deepfake-detection/audio-deepfake-detection
```

Verify:

```
pwd
ls
```

The prompt should end in `/audio-deepfake-detection` and `ls` should show `README.md`, `LICENSE`, `.gitignore`, `HOW_TO_PUSH.md`, and a `notebooks/` folder.

🛑 **If `pwd` doesn't end in `/audio-deepfake-detection` or `ls` doesn't show those files, STOP. Do not run git. Send me what you see.**

## 3 · The five git commands, one at a time

```
git init -b main
```

```
git add .
```

```
git commit -m "Initial release: lightweight on-device audio deepfake detection"
```

Replace `<your-repo-name>` below with whatever you called it on GitHub:

```
git remote add origin https://github.com/433-433/<your-repo-name>.git
```

```
git push -u origin main
```

The browser pop-up shouldn't appear this time — your GitHub credentials from the last two pushes are cached.

Expect the push to take **about 10–15 seconds** — the whole repo is only a few hundred kilobytes (two notebooks and some text files, no model weights, no audio).

---

## 4 · After it's up

**Topics** — ⚙ next to About → add:
```
audio-deepfake  anti-spoofing  asvspoof-2019  cnn  mobile-banking  on-device  pytorch  lightweight  liveness-detection
```

**Pin the repo** to your profile alongside the NTN and STAR-RIS ISAC repos. Three pinned repos → a strong portfolio page.

---

## 5 · Why there's no CITATION.cff in this repo

You asked me not to list author names. A `CITATION.cff` file requires a structured list of authors, so I've skipped it for this repo rather than list a single name that misrepresents the work. If the paper is published later and you want a formal citation entry, I can drop one in then.

---

## 6 · Why there's no PDF of the paper

You asked me not to include the ICAST label or author names. The paper PDF has both in its banner, so I haven't included it in the repo. If you want to add it later once it's publicly available, drop it into a `paper/` folder and commit.

---

## 7 · Normal update cycle

From inside the repo folder:

```
git add .
git commit -m "Short description of what changed"
git push
```

Three lines. Forever.
