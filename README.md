
# git add the files size less than a number (e.g. 900000 bytes):
----------------------------------------------------------------
var=`du -a --max-depth=1 . | awk '$1*512 > 900000 {print $2}' | sed -e ':a' -e 'N' -e '$!ba' -e 's/\n/ /g'`; git add ${var%??}


# List the files sizes according to their sizes in the whole git history
--------------------------------------------------------------------
git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | sed -n 's/^blob //p' | sort --numeric-sort --key=2 | cut -c 1-12,41- | $(command -v gnumfmt || echo numfmt) --field=2 --to=iec-i --suffix=B --padding=7 --round=nearest


# Remove files from the whole git history:
------------------------------------------
git filter-branch --force --index-filter   'git rm --cached --ignore-unmatch <file-path>'   --prune-empty --tag-name-filter cat -- --all

git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d

git reflog expire --expire=now --all && git gc --prune=now --aggressive


# Show the sorted file/folder sizes
----------------------------------- 
du -sh ./* -c | sort -h


# Check for the large files:
----------------------------
git lfs migrate info


# Migrate the large files to git lfs (Rewrites the git history):
----------------------------------------------------------------
git lfs migrate import --include="*.psd"


# Push commits to remote in batches of N:
-----------------------------------------
	git rev-list --reverse --all | ruby -ne 'x ||=0; x += 1; print $_ if x % N == 0;' | xargs -I{} git push origin +{}:refs/heads/master

# Create an orphan branch to split the root commit:
---------------------------------------------------
git checkout --orphan <branch-name>


# Interactive root commit rebasing:
-----------------------------------
git rebase -i --root
