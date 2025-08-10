#!/usr/bin/env bash
# Usage: bash publish.sh <github-username/repo-name>   e.g., bash publish.sh youruser/neon-racer
set -euo pipefail

REPO="${1:-}"
if [ -z "$REPO" ]; then
  echo "Usage: bash publish.sh <github-username/repo-name>"; exit 1
fi

if ! command -v git >/dev/null 2>&1; then
  echo "Git not installed. Please install git."; exit 1
fi

DEFAULT_BRANCH="main"

# Init repo if needed
if [ ! -d .git ]; then
  git init
fi

# Ensure main branch
if git rev-parse --abbrev-ref HEAD >/dev/null 2>&1; then
  git branch -M "$DEFAULT_BRANCH"
else
  git checkout -b "$DEFAULT_BRANCH" || git branch -M "$DEFAULT_BRANCH"
fi

git add .
git commit -m "Initial public release of Neon Racer" || true

if command -v gh >/dev/null 2>&1; then
  # With GitHub CLI (recommended)
  gh repo create "$REPO" --public --source=. --remote=origin --push
else
  # Manual remote (create empty public repo at https://github.com/$REPO first)
  git remote remove origin 2>/dev/null || true
  git remote add origin "https://github.com/$REPO.git"
  git push -u origin "$DEFAULT_BRANCH"
fi

echo "Published: https://github.com/$REPO"
