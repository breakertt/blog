title: A note on tricky Git commands
date: 2023/3/2 13:20:00
categories:
- Linux
toc: true
---

- Check the changes on a specific function [1] \
  `git log -L :myfunction:path/to/myfile.c`
- Reset the commit date to the author date from a certian commit to latest commit \
  `git filter-branch -f --env-filter 'export GIT_COMMITTER_DATE="$GIT_AUTHOR_DATE"' <commit>..HEAD`  
- Restore file from old commit \
  `git checkout <commit> -- <path/to/file>
  
# Refs

[1] https://stackoverflow.com/a/33953022
