# Git commands

### Creating new branch from Develop

Make sure Git Flow extension is installed in VS Code, and Git Flow CLI is installed (run `git flow` to check) and initialised. For 'develop' branch, rename to 'Develop' (capitalised 'D'). The rest to be default.

Always checkout to `Develop` branch before making feature branch.

When ready to create branch, click the Git Flow button at lower bar of VS Code:

Options should appear for what branch to make:

or green triangle at Source Control tab -> Gitflow section -> Features:

or

Select the appropriate branch type, then proceed to name it (example of bugfix branch):

Appropriate branch should then be created automatically.

More Info: <https://code.visualstudio.com/docs/sourcecontrol/overview#_vs-code-as-git-editor>

Checkout to the appropriate branch before adding code:

Or click Branch indicator:

Then select branch to checkout:

Add and commit as usual:

```bash
git add .
git commit .
```

### If local and origin out of sync and want to reset to origin:

```bash
git checkout Develop
git reset --hard origin/Develop
```

More info: <https://www.freecodecamp.org/news/git-reset-origin-how-to-reset-a-local-branch-to-remote-tracking-branch/>

### Workflow for feature branch

Create new branch from Develop: from Git bash/CLI

```bash
git checkout Develop
git pull
git checkout -b feature/[feature name]
```

Alternatively, use Git Flow extension to make branch.

Make commits:

```bash
git add .
git commit .
```

After implementation done, switch back to Develop branch and sync with remote branch (origin):

```bash
git checkout Develop
git fetch
git pull
```

Update local feature branch to be updated with local Develop branch:

```bash
git checkout feature/[feature name]
git merge Develop
```

Once conflicts resolved, push to remote:

```bash
git push
```

When ready to merge feature into Develop:

#### Option 1: from VS Code

```bash
git checkout Develop
git merge feature/[feature name]
git push
```

#### Option 2: from Gitlab website

Navigate to feature branch.  


Click create merge request.  


Check feature branch is correct.  
Check destination branch is correct **(Gitlab defaults to Main. ALWAYS CHANGE TO DEVELOP!!!)**.  


Make sure "delete feature branch" is UNCHECKED (i.e. do not delete feature branch after merge)  


Resolve all conflicts.  
Merge once all conflicts are resolved. Double check delete branch checkbox is UNCHECKED.  


Close merge request (may give error, but it's ok)
