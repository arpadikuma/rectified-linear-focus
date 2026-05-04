# Setup Instructions

Complete guide to get your Obsidian vault synced to GitHub.

---

## Option A: Initial Setup (if you don't have git installed)

### 1. Install Git
- **Mac:** `brew install git`
- **Windows:** Download from [git-scm.com](https://git-scm.com)
- **Linux:** `sudo apt install git` or equivalent

### 2. Create GitHub Repository
1. Go to [github.com](https://github.com)
2. Sign in or create account
3. Click "+" → "New repository"
4. Name: `rectified-linear-focus`
5. Description: "Building real AI products for creators"
6. Choose **Public** (so it's part of your portfolio)
7. Click "Create repository"

### 3. Initial Commit

**On Mac/Linux:**
```bash
# Navigate to your vault folder
cd ~/path/to/rectified-linear-focus

# Initialize git
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: roadmap structure"

# Add remote (replace USERNAME with your GitHub username)
git remote add origin https://github.com/USERNAME/rectified-linear-focus.git

# Push to GitHub
git branch -M main
git push -u origin main
```

**On Windows (Command Prompt):**
```cmd
cd path\to\rectified-linear-focus
git init
git add .
git commit -m "Initial commit: roadmap structure"
git remote add origin https://github.com/USERNAME/rectified-linear-focus.git
git branch -M main
git push -u origin main
```

### 4. After First Push

Verify it worked by going to your GitHub repo URL and seeing all files there.

---

## Weekly Workflow

### Every Friday (5 minutes):

**Mac/Linux:**
```bash
cd ~/path/to/rectified-linear-focus
git add .
git commit -m "Weekly update: [date] - [brief summary]"
git push
```

**Windows:**
```cmd
cd path\to\rectified-linear-focus
git add .
git commit -m "Weekly update: [date] - [brief summary]"
git push
```

### Examples of good commit messages:
- `Weekly update: 2025-04-29 - Caption Generator MVP architecture`
- `Weekly update: 2025-05-06 - FastAPI backend complete, beta recruiting`
- `Weekly update: 2025-05-13 - Frontend deployed, 20 beta users`

---

## Automated Weekly Sync (Optional)

If you want a script to auto-commit, save this as `sync.sh` (Mac/Linux) in your vault root:

```bash
#!/bin/bash
cd "$(dirname "$0")"
git add .
git commit -m "Weekly update: $(date +%Y-%m-%d) - auto-sync"
git push
echo "✓ Synced to GitHub"
```

Then run: `bash sync.sh`

(Or set up GitHub Actions to auto-commit daily, but this is overkill to start)

---

## Obsidian Setup

### 1. Open This Folder as Vault

1. Open Obsidian
2. "Open folder as vault" (or similar button)
3. Navigate to `rectified-linear-focus` folder
4. Click open

### 2. Optional: Install Git Plugin

1. Settings → Community plugins
2. Search "Git"
3. Install "Obsidian Git"
4. Now you can commit from Obsidian UI instead of terminal

(Not necessary, but convenient if you prefer not to use terminal)

### 3. Enable Backlinks & Graph

1. Settings → Core plugins
2. Enable "Backlinks"
3. Enable "Graph view"
4. Enable "Daily notes"

Now you can:
- See links between projects (graph view)
- Quick link between files: `[[Caption Generator]]`
- See what links to each file

---

## Sharing Your Repo

### In LinkedIn Bio/Posts
```
Building AI products in public: 
github.com/USERNAME/rectified-linear-focus
```

### In Twitter Bio
```
Building real AI products for creators
Repository: github.com/USERNAME/rectified-linear-focus
```

### In Every Post
Mention: "Follow along in the roadmap: [link]"

---

## Directory Structure

This is what you should have locally:

```
rectified-linear-focus/
├── README.md
├── .gitignore
├── PROJECTS/
│   ├── visionair/
│   │   └── README.md
│   ├── caption-generator/
│   │   └── README.md
│   ├── aesthetic-scorer/
│   │   └── README.md
│   ├── repo-analyzer/
│   │   └── README.md
│   ├── fashion-assistant/
│   │   └── README.md
│   └── mlops-energy-forecasting/
│       └── README.md
├── CONTENT/
│   └── calendar.md
├── WEEKLY_LOGS/
│   ├── template.md
│   └── [weekly logs as you add them]
└── PRIVATE/
    ├── revenue-tracking.md
    └── notes.md
```

The `PRIVATE/` folder won't sync to GitHub because of `.gitignore`.

---

## Making Changes

### In Obsidian
1. Edit any `.md` file normally
2. Save (auto-saves in Obsidian)
3. Weekly: Run git sync (Friday)

### Adding New Files
1. Create new file in Obsidian
2. Save in appropriate folder
3. When you sync, it auto-includes new files

---

## Troubleshooting

### "git: command not found"
→ Git not installed. Install from git-scm.com

### "fatal: not a git repository"
→ You're not in the right folder. Use `cd` to navigate to `rectified-linear-focus`

### "Permission denied"
→ May need to set git credentials. Run: `git config --global user.name "Your Name"`

### "Everything is staged, nothing to commit"
→ This is fine. Just means no changes since last sync.

---

## Next Steps

1. Follow Option A above to set up the repo
2. Open vault in Obsidian
3. Customize the README with your actual name/links
4. Update the CONTENT/calendar dates for real
5. Make your first commit this Friday
6. Share link in your LinkedIn/Twitter bios

---

## Questions?

If something doesn't work:
- Check git is installed: `git --version`
- Check you're in right folder: `pwd` (Mac/Linux) or `cd` (Windows)
- Google the error message + "git" + "github"
- GitHub docs: github.com/git-guides/
