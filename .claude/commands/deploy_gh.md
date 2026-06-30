Deploy this project to GitHub Pages. Follow every step in order.

## Steps

### 1. Detect repo info
Run `git remote get-url origin` to extract the GitHub username and repo name from the URL.
The final GitHub Pages URL will be `https://<username>.github.io/<repo-name>/`.

### 2. Set the Vite base path
Open `vite.config.ts`. Inside `defineConfig({ ... })`, add or update the top-level `base` option to `'/<repo-name>/'` (where `<repo-name>` is what you found in step 1). This makes all asset paths relative to the GitHub Pages subdirectory. Without it, assets 404.

Example — add `base` alongside the existing `plugins` key:
```ts
export default defineConfig({
  base: '/treasure-game-demo/',
  plugins: [react()],
  ...
})
```

### 3. Build
Run `npm run build`. The output lands in `./build` (configured in vite.config.ts).

### 4. Deploy the build folder to gh-pages
Use a git worktree to push *only* the `build/` directory as the root of the `gh-pages` branch:

```bash
# Remove stale worktree if it exists from a previous run
git worktree remove /tmp/gh-pages-deploy --force 2>/dev/null || true

# Attach (or create) the gh-pages branch as a worktree
git fetch origin gh-pages:gh-pages 2>/dev/null || true
git worktree add /tmp/gh-pages-deploy gh-pages 2>/dev/null \
  || git worktree add --orphan -B gh-pages /tmp/gh-pages-deploy

# Wipe old content and copy fresh build
rm -rf /tmp/gh-pages-deploy/*
cp -r build/. /tmp/gh-pages-deploy/

# Commit and push
cd /tmp/gh-pages-deploy
git add --all
git commit -m "Deploy to GitHub Pages $(date -u +%Y-%m-%dT%H:%M:%SZ)"
git push origin gh-pages --force

# Clean up
cd -
git worktree remove /tmp/gh-pages-deploy --force
```

### 5. Revert vite.config.ts base change (optional)
If the user works locally without `base` set, revert the `base` line after the push so local `npm run dev` continues to work without a path prefix. Ask the user whether to keep or revert before doing so.

### 6. Enable GitHub Pages (one-time manual step)
If the `gh-pages` branch was just created for the first time, remind the user to enable Pages in GitHub:
> Go to **github.com/\<username\>/\<repo-name\>** → Settings → Pages → Source: **Deploy from a branch** → Branch: **gh-pages** / **(root)** → Save.

### 7. Report the URL
Print the live URL: `https://<username>.github.io/<repo-name>/`
Note that GitHub takes ~1–2 minutes to publish after the push.
