git fetch --all --prune
git reset --hard origin/main
git clean -fd

git add -A
git commit -m "before trim & respond"
git push origin main