# Prep Notes site

Static site, no build step. Deploy directly to GitHub Pages.

## Structure
- index.html - home page, lists all notes as cards
- doc.html - reader page, renders any markdown file passed as ?f=docs/filename.md
- style.css - all styling
- manifest.json - the list of notes shown on the home page
- docs/ - your actual .md files go here

## Add a new note
1. Drop the .md file into docs/
2. Add one entry to manifest.json:
   { "file": "docs/YourFile.md", "title": "Shown on card", "desc": "One line description" }
3. Commit and push - GitHub Pages updates automatically

## Deploy on GitHub Pages
1. Create a new GitHub repo (public), e.g. "prep-notes"
2. Upload all these files/folders to the repo root (index.html, doc.html, style.css, manifest.json, docs/)
3. Go to repo Settings > Pages
4. Under "Build and deployment", set Source = "Deploy from a branch", Branch = main, folder = / (root)
5. Save. Your site goes live at https://YOUR-USERNAME.github.io/prep-notes/ within a minute or two
6. Bookmark that URL / add to your phone's home screen for one-tap access
