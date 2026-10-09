# Git & GitHub: Simple Practice Guide

A very simple, step-by-step guide to everything practised in this repo, from setup to rebuilding the project from scratch. Run through it as many times as you like.

Replace `YOUR-USERNAME` with your GitHub username.

## The idea in one line

```
edit files  ->  git add  ->  git commit  ->  git push  ->  pull request  ->  merge  ->  git pull
(your folder)   (staging)    (local history)  (GitHub)      (review)                    (back to you)
```

---

## 1. Setup (do this once)

```bash
sudo apt install git                              # install git
git config --global user.name "Your Name"         # name shown on your commits
git config --global user.email "you@example.com"  # email shown on your commits
git config --global init.defaultBranch main       # new repos start on a branch called main
sudo apt install gh                               # install the GitHub command line tool
gh auth login                                     # log in to GitHub
gh auth status                                    # check you are logged in
```

At the login prompts choose: **GitHub.com**, **HTTPS**, **Yes**, **Login with a web browser**. Open https://github.com/login/device on any device, enter the one-time code and approve.

---

## 2. Create the project

```bash
mkdir quotes && cd quotes                 # make a folder and go into it
git init                                  # turn it into a git repo
echo 'print("Hello, git!")' > quotes.py   # create a file
echo '# Quotes' > README.md               # create another file
git status                                # see what git can see (both files untracked)
git add .                                 # stage everything here ('.' means this folder)
git commit -m 'Initial commit'            # save it to history
git log --oneline                         # show history, one line per commit
```

---

## 3. Put it on GitHub

```bash
gh repo create quotes --public --source=. --remote=origin --push
```

- `--public` anyone can see it (use `--private` to hide it)
- `--source=.` upload this folder
- `--remote=origin` call the GitHub link `origin`
- `--push` upload the commits straight away

Check it worked:

```bash
git remote -v     # origin should point at your GitHub repo
git status        # should say 'up to date with origin/main'
```

---

## 4. The everyday loop (branch, change, merge)

Always work on a branch, never directly on `main`. Create the branch **before** you edit anything.

```bash
git switch main                  # 1. start on main
git pull                         # 2. get the latest from GitHub
git switch -c my-change          # 3. create a branch and move onto it

# 4. edit your files

git status                       # see what changed
git diff                         # see exactly what changed (- removed, + added)
git add .                        # 5. stage the changes
git commit -m 'Describe change'  # 6. save them
git push -u origin my-change     # 7. upload the branch
gh pr create --fill              # 8. open a pull request
```

Merge the pull request on GitHub (green **Merge pull request** button), or from the terminal:

```bash
gh pr merge --merge --delete-branch   # merge it and delete the branch on GitHub
```

Then tidy up:

```bash
git switch main                  # 10. back to main
git pull                         # 11. download the merged result
git branch -d my-change          # 12. delete the old local branch
git branch -a                    # list all branches (local and on GitHub)
```

---

## 5. Trash it and rebuild it

Make a mess:

```bash
echo 'junk' > junk.txt                             # new untracked file
mkdir scratch && echo 'temp' > scratch/notes.txt   # new untracked folder
echo '# broken' > README.md                        # change a tracked file
rm quotes.py                                       # delete a tracked file
git status                                         # shows modified, deleted and untracked
```

Clean it up:

```bash
git restore .     # put tracked files back to the last commit
git clean -n      # dry run: show untracked files that WOULD be deleted
git clean -fd     # really delete untracked files and folders (permanent!)
git status        # should say 'working tree clean'
```

Rebuild everything from GitHub:

```bash
cd ..                 # go up out of the quotes folder
pwd                   # check where you are BEFORE deleting
rm -rf quotes         # delete the whole folder (no undo!)
git clone https://github.com/YOUR-USERNAME/quotes.git   # download it again
cd quotes
git log --oneline     # all the history is back
python3 quotes.py     # and it runs
```

---

## 6. Undo a commit

**Not pushed yet** (only on your machine): use `reset`.

```bash
git reset --hard HEAD~1    # remove the last commit AND its changes
git reset --soft HEAD~1    # remove the last commit but keep the changes
```

**Already pushed**: use `revert`, which adds a new commit that cancels the old one.

```bash
git revert HEAD            # an editor opens: save and exit (nano: Ctrl+O, Enter, Ctrl+X)
git push                   # upload the revert
```

---

## 7. Merge conflicts

A conflict happens when two branches change the same line differently. Git stops and marks the file:

```
<<<<<<< HEAD
the version from your current branch
=======
the version from the incoming branch
>>>>>>> other-branch
```

To fix it:

```bash
nano quotes.py                                       # delete the 3 marker lines, keep the line you want
grep -n '<<<<<<<\|=======\|>>>>>>>' quotes.py        # should print nothing: no markers left
python3 quotes.py                                    # check it still runs
git add quotes.py                                    # tell git it is resolved
git commit                                           # finish the merge (save and exit the editor)
git push                                             # upload it
```

Stuck halfway? Cancel the merge:

```bash
git merge --abort
```

See the history as a diagram:

```bash
git log --oneline --graph
```

---

## 8. Keep files out of git, and park unfinished work

**.gitignore** lists files git should never track (logs, secrets, caches):

```bash
printf 'logs/\n.env\n__pycache__/\n' > .gitignore   # one name per line
git add .gitignore
git commit -m 'Add .gitignore'
git push
git rm --cached FILENAME                            # stop tracking a file git already has
```

**stash** puts unfinished changes on a shelf:

```bash
git stash          # shelve your changes (folder becomes clean)
git stash list     # see what is on the shelf
git stash pop      # bring the newest one back
git stash clear    # delete everything on the shelf (permanent!)
```

---

## 9. Add a license

Pick one. Run on a new branch (`git switch -c add-license`).

```bash
gh api licenses/mit --jq .body > LICENSE                       # MIT: simple, permissive
sed -i 's/\[year\]/2026/; s/\[fullname\]/Your Name/' LICENSE   # fill in year and name (MIT only)

gh api licenses/apache-2.0 --jq .body > LICENSE                # Apache 2.0: like MIT plus patent grant
gh api licenses/gpl-3.0 --jq .body > LICENSE                   # GPL 3.0: derived work must stay GPL

gh api licenses --jq '.[].key'                                 # list every license key available
```

Then the normal loop: `git add LICENSE`, `git commit`, `git push -u origin add-license`, `gh pr create --fill`, merge, `git switch main`, `git pull`.

---

## 10. Check your login, log out, start again

```bash
gh auth status                                   # who am I logged in as?
git config --global --get-regexp credential      # which helper gives git my login (should show gh)
gh auth logout                                   # remove the login token from this machine
```

To revoke it on GitHub as well: Settings, Applications, Authorized OAuth Apps, GitHub CLI, Revoke.

**Wipe everything and start again:**

```bash
cd ~                                              # leave the project folder
rm -rf quotes                                     # delete the local folder
gh auth refresh -h github.com -s delete_repo      # one-time: allow gh to delete repos
gh repo delete YOUR-USERNAME/quotes --yes         # delete the repo on GitHub (permanent!)
```

Then go back to section 2.

---

## Handle with care

These cannot be undone. Pause and check first.

| Command | Why it is risky |
|---|---|
| `rm -rf folder` | Deletes a folder and everything in it, no recycle bin |
| `git clean -fd` | Permanently deletes untracked files (run `git clean -n` first) |
| `git reset --hard` | Throws away uncommitted changes and moves the branch back |
| `git stash clear` | Deletes every stash |
| `gh repo delete` | Deletes the repo on GitHub |

---

## Command summary

| Group | Command | What it does |
|---|---|---|
| Setup | `git config --global user.name "Name"` | Sets the name on your commits |
| Setup | `git config --global user.email "email"` | Sets the email on your commits |
| Setup | `gh auth login` | Logs in to GitHub |
| Setup | `gh auth status` | Shows who you are logged in as |
| Setup | `gh auth logout` | Removes the login from this machine |
| Start | `git init` | Turns a folder into a git repo |
| Start | `git clone URL` | Downloads a repo from GitHub |
| Start | `gh repo create NAME --public --source=. --remote=origin --push` | Creates the GitHub repo and uploads your project |
| Daily | `git status` | Shows what changed |
| Daily | `git add FILE` / `git add .` | Stages one file / everything |
| Daily | `git commit -m 'msg'` | Saves staged changes to history |
| Daily | `git commit -am 'msg'` | Stages tracked files and commits in one go |
| Daily | `git push` | Uploads commits to GitHub |
| Daily | `git pull` | Downloads new commits from GitHub |
| Look | `git log --oneline` | History, one line per commit |
| Look | `git log --oneline --graph` | History as a branch diagram |
| Look | `git diff` | Shows unstaged changes |
| Look | `git diff --staged` | Shows staged changes |
| Look | `git remote -v` | Shows where origin points |
| Look | `git branch -a` | Lists all branches |
| Branch | `git switch -c NAME` | Creates a branch and moves onto it |
| Branch | `git switch NAME` | Moves to an existing branch |
| Branch | `git push -u origin NAME` | Uploads a new branch |
| Branch | `git merge NAME` | Combines that branch into the current one |
| Branch | `git merge --abort` | Cancels a merge that went wrong |
| Branch | `git branch -d NAME` | Deletes a finished local branch |
| Branch | `git push origin --delete NAME` | Deletes a branch on GitHub |
| Pull request | `gh pr create --fill` | Opens a pull request |
| Pull request | `gh pr merge --merge --delete-branch` | Merges the pull request and deletes its branch |
| Undo | `git restore .` | Puts tracked files back to the last commit |
| Undo | `git clean -n` | Dry run: lists untracked files that would be deleted |
| Undo | `git clean -fd` | Deletes untracked files and folders |
| Undo | `git reset --hard HEAD~1` | Removes the last local commit and its changes |
| Undo | `git reset --soft HEAD~1` | Removes the last local commit, keeps the changes |
| Undo | `git revert HEAD` | Adds a commit that cancels the last one (safe after pushing) |
| Stash | `git stash` / `git stash list` | Shelves changes / lists the shelf |
| Stash | `git stash pop` / `git stash clear` | Restores the newest / deletes all |
| Ignore | `git rm --cached FILE` | Stops tracking a file without deleting it |
| License | `gh api licenses/KEY --jq .body > LICENSE` | Fetches a license (`mit`, `apache-2.0`, `gpl-3.0`) |
| Wipe | `gh repo delete USER/NAME --yes` | Deletes the repo on GitHub |
