git fetch --all --prune
git reset --hard origin/main
git clean -fd

git add -A
git commit -m "after codex to modeling 1"
git push origin main