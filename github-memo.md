git fetch --all --prune
git reset --hard origin/main
git clean -fd

git add -A
git commit -m "before implement todo.md"
git push origin main