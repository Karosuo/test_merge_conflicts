# test_merge_conflicts
Just a repo to test some merge conflicts and other hopefully interesting git cases

## Test 1 - Create a file that gets merge conflicts by a copy that I don't want, then I will take git checkout --ours my_file.txt
to see if I still have to git add it or if it just gets removed from the index of conflicted files (add some unstaged files that are not part of the merge conflict)

**Resolution**
Created two files with conflicts, such that when running `git merge origin/main` while in `main`
the following output appears
`
Auto-merging my_file.txt
CONFLICT (add/add): Merge conflict in my_file.txt
Auto-merging my_second.txt
CONFLICT (add/add): Merge conflict in my_second.txt
Automatic merge failed; fix conflicts and then commit the result.
`
Observing that:
* git shows different sections for "Unmerged" or "Untracked" files
`
$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both added:      my_file.txt
        both added:      my_second.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        not_conflicted_file.txt

no changes added to commit (use "git add" and/or "git commit -a")
`

* Then using `git checkout --ours my_file.txt` prints that the path was updated
`
$ git checkout --ours my_file.txt
Updated 1 path from the index
`
but it still shows it as unmerged paths
`
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both added:      my_file.txt
        both added:      my_second.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        not_conflicted_file.txt

no changes added to commit (use "git add" and/or "git commit -a")
`

* Running `git diff my_file.txt` shows that it is still index as conflicted file, but it doesn't show any changes which prooves that `git checkout --ours` works correctly
`
diff --cc my_file.txt
index 9ba6124,20bb193..0000000
--- a/my_file.txt
+++ b/my_file.txt
`

* Also can be proven when we cat its content
`
// git cat my_file.txt
$ cat my_file.txt
Some extra changes from p2
`

* Once `git add my_file.txt` the `git status` doesn't show any changesets staged to be commited, but the file was indexed as not having conflicts anymore.
`
$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both added:      my_second.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        not_conflicted_file.txt

no changes added to commit (use "git add" and/or "git commit -a")
`

* The other conflicted file `my_second.txt` is still listed as unmerged path, and it doesn't allow to just commit and leave `my_second.txt` as untracked or unstaged or in a limbo of some sort
`
error: Committing is not possible because you have unmerged files.
hint: Fix them up in the work tree, and then use 'git add/rm <file>'
hint: as appropriate to mark resolution and make a commit.
fatal: Exiting because of an unresolved conflict.
U       my_second.txt
`

* If we use `git mergetool my_second.txt`, fix the merge conflict and save the file, git will automatically stage that file, resolving the conflict related to it
`
$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   my_second.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        not_conflicted_file.txt

* Let's be aware of the fact that if we start `git mergetool my_second.txt` and close without saving (I'm guessing this also applies to not clearing the conflicts, as for example beyond compare shows it, but haven't tested it), git will ask you if the merge was successful, to which if you select 'no', then the file will still be shown as Unmerged path
`
$ git mergetool my_second.txt
Merging:
my_second.txt

Normal merge conflict for 'my_second.txt':
  {local}: created file
  {remote}: created file
my_second.txt seems unchanged.
Was the merge successful [y/n]? n
merge of my_second.txt failed
`

`
$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both added:      my_second.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        not_conflicted_file.txt

no changes added to commit (use "git add" and/or "git commit -a")
`

If we select 'yes', then the file gets staged automatically and the conflict resolved:
`
$ git mergetool my_second.txt
Merging:
my_second.txt

Normal merge conflict for 'my_second.txt':
  {local}: created file
  {remote}: created file
my_second.txt seems unchanged.
Was the merge successful [y/n]? y
`

`
$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   my_second.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        not_conflicted_file.txt
`
This makes sense, because even when in our case this is totally wrong, git wouldn't know if the file was actually in the desired state.
If we check the current contents with `cat my_second.txt` we can see that the actual text includes the conflict guards, ready to be staged and commited, so let's be very careful of not say 'yes' to that git prompt of `was the merge successful?` if we're not 100% sure
`
$ cat my_second.txt
<<<<<<< HEAD
Second file, some secondary edits from p2
=======
second file, first change in p1
>>>>>>> origin/main
`

* Fixing the changes to my_second.txt as follows when running `git mergetool my_second.txt`, and allowing git to auto stage the changes, allows us to directly commit the merge

**Note:** I realized that I introduced some untracked file, instead of make it an unstaged file, which might behave differently in this scenario

## Test 2 - Create a merge conflict where one of the two parties changes the directory name, and the same file's content

## Test 3 - Testing auto commit when no fast forward is available, create a feature branch from a base branch, where fast forward is possible, then use --no-ff and see if a commit is still created

**Resolution:**
According to the documentation (2.41.0)
>--no-commit
Perform the merge and commit the result. This option can be used to override --no-commit.

>With --no-commit perform the merge and stop just before creating a merge commit, to give the user a chance to inspect and further tweak the merge result before committing.

>Note that fast-forward updates do not create a merge commit and therefore there is no way to stop those merges with --no-commit. Thus, if you want to ensure your branch is not changed or updated by the merge command, use --no-ff with --no-commit

There's some cases where --no-commit is required, let's test if it's the one with no conflicts (and --no-ff)
