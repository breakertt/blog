title: A note on tricky Git commands
date: 2023/3/2 13:20:00
categories:
- Linux
toc: true
---

- Check the changes on a specific function [1]
  `git log -L :myfunction:path/to/myfile.c`
- Commit with a specified date
  `git commit --amend --date="Wed Feb 16 14:00 2011 +0100" --no-edit`
- Rebase while keeping commit date as the author date
  `git rebase --committer-date-is-author-date -i <commit>`
- Reset the commit date to the author date from a certain commit to the latest commit
  `git filter-branch -f --env-filter 'export GIT_COMMITTER_DATE="$GIT_AUTHOR_DATE"' <commit>..HEAD`  
- Restore file from old commit
  `git checkout <commit> -- <path/to/file>`
- Delete files and dirs that are not tracked
  `git clean -fd` (`git clean -i` for interactive delete)
