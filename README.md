# In-class demo and walkthrough (Git + GitHub)
This is written as an instructor script you can run live. It starts with SSH
verification, then does a quick local Git demo, then transitions into the pair
labs.
## 0. Pre-flight (2 minutes)
Ask students to open a terminal in a working directory where they can create a new
folder.
Quick sanity checks:
```bash
git --version
ssh -V
```
If Git is missing, have them install Git and restart their terminal.
## 1. Verify GitHub SSH (10 minutes)
### 1.1 The check
```bash
ssh -T git@github.com
```
Expected outcomes:
- If it prints a greeting and mentions successful authentication, they are good.
- If it asks to confirm a host fingerprint, type `yes`.
- If it says permission denied, continue below.
### 1.2 If permission denied: fix checklist
1) Check for an existing key:
```bash
ls -al ~/.ssh
```
Look for `id_ed25519` and `id_ed25519.pub` (or `id_rsa` / `id_rsa.pub`).
2) If no key exists, generate one:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
Accept defaults. Use a passphrase if you want, but warn that it adds prompts unless
they use a keychain.
3) Start agent and add the key:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
4) Copy the public key and add it to GitHub
macOS:
```bash
pbcopy < ~/.ssh/id_ed25519.pub
```
Linux:
```bash
xclip -selection clipboard < ~/.ssh/id_ed25519.pub
```
Windows Git Bash:
```bash
cat ~/.ssh/id_ed25519.pub
```
Then GitHub: Settings → SSH and GPG keys → New SSH key → paste.
5) Re-test:
```bash
ssh -T git@github.com
```
Optional debugging (when someone is still stuck):
```bash
ssh -vT git@github.com
```
## 2. Clone vs fork (5 minutes)
Say this out loud:
- Clone: downloads a repo to your machine.
- Fork: makes a copy on GitHub under your account.
Two common scenarios:
1) You have write access to the repo (class repo, your own repo): clone it
directly.
2) You do not have write access (open source, partner repo): fork on GitHub, then
clone your fork.
### 2.1 For today's exercise
1. Show the class the instructor repo on GitHub (for example,
`https://github.com/<your-org>/git-demo-class`).
2. Ask: "Given that you only need to read the code, should you clone directly or
fork first?" Let a few students reason it out.
3. Explain that even though cloning would technically work, you want them to fork
so they gain experience pushing to their own copy.
4. Walk through the fork + clone steps:
- Click **Fork** in GitHub, keep the default settings.
- Copy the SSH URL from *their* fork (not the instructor repo).
- In the terminal:
```bash
git clone git@github.com:<student-username>/git-demo-class.git
cd git-demo-class
git remote -v
```
5. Point out that `origin` now points to their fork. If they need the upstream
link, add it with `git remote add upstream git@github.com:<instructor>/git-demo-
class.git`.
6. Remind them they can still pull upstream changes later with `git fetch upstream
&& git merge upstream/main`.
## 3. Quick local Git demo (8 to 10 minutes)
### 3.1 Create a fresh demo repo
First navigate to a working directory you want to create a folder in and then:
```bash
mkdir git_demo
cd git_demo
git init
```
Explain: `git init` creates the hidden `.git` directory, the local database.
### 3.2 Add a file and commit
Create `hello.py`:
```python
print('Hello, Git!')
```
Then:
```bash
git status
git add hello.py
git commit -m "Initial commit: add hello.py"
```
### 3.3 Make a change, inspect diff, commit
Append:
```python
print('Learning version control')
```
Then:
```bash
git status
git diff
git add hello.py
git commit -m "Add learning message"
```
### 3.4 View history
```bash
git log --oneline
```
Optional (more visual):
```bash
git log --oneline --graph --all
```
### 3.5 Restore a previous version of a file
Copy the first commit hash from `git log --oneline` (the older one), then:
```bash
cat hello.py
git checkout <OLD_COMMIT_HASH> -- hello.py
cat hello.py
```
Bring it back to the latest:
```bash
git checkout <MOST_RECENT_COMMIT_HASH> -- hello.py
cat hello.py
```
### 3.6 `.gitignore` quick win
```bash
printf "*.log\n" > .gitignore
echo "temp" > temp.log
git status
```
Explain: ignored files do not show up as untracked, which protects you when using
`git add .`.
### 3.7 Branch and merge
```bash
git checkout -b feature
```
Add another line to `hello.py`:
```python
print('Working on a new feature')
```
Commit and show graph:
```bash
git add hello.py
git commit -m "Add feature print"
git log --oneline --graph --all
```
Merge into main:
```bash
cat hello.py
git checkout master
cat hello.py
git merge feature
```
### 3.8 Merge Conflict demo
Create a new branch to simulate a conflict.
```bash
git checkout -b test_merge
git checkout master
```
In `master`, change the last line of hello.py to:
```python
print('Working on an old feature')
```
```bash
git add hello.py
git commit -m "Modify feature line in master"
git checkout test_merge
```
On `test_merge`, change the last line of `hello.py` to:
```python
print('Working on an exciting feature')
```
```bash
touch testing.txt
git add .
git commit -m "Modify feature line in test_merge"
git checkout master
git merge test_merge
```
This will produce a conflict. Let's explore.
```bash
cat hello.py
git status
ls -al
```
Here is how to abort the merge:
```bash
git merge --abort
git status
ls -al
```
Now, let's try resolving it. Repeat the steps to create the conflict, then:
```bash
git merge test_merge
```
Now let's resolve it by editing `hello.py` to the desired final state. After
editing:
```bash
git add hello.py
git commit -m "Resolve merge conflict between master and test_merge"
git log --oneline --graph --all
```
## 4. Pair lab 1: Pull requests
Use the provided notebook instructions in CLass Lab - Pull Requests.
## 5. Pair lab 2: Conflicts (20 to 25 minutes)
Use the provided notebook instructions in CLass Lab - Conflicts.
### Instructor talking points
- Conflicts happen when Git cannot safely auto-merge.
- The conflict markers show two competing versions.
- The resolution is just: edit to the correct final file, stage, commit.
### Common student pitfalls
- Forgetting to `git pull` before merging
- Resolving conflict in GitHub UI but not pulling locally
- Leaving conflict markers in the file
## 6. Wrap-up
Exit ticket prompts:
1) Explain clone vs fork in one sentence.
2) Paste the first line of `ssh -T git@github.com`.
3) What caused a conflict today and what did you do to resolve it?
