## git 命令

- git commit --amend
  
  - 修改最后一次commit信息
  
- 批量删除本地分支

  - git branch \| grep 'releases*' \| xargs git branch -d    (-D删除远程分支)

- 放弃本地修改，强制远程覆盖本地：
  git fetch --all && git reset --hard origin/feat/UserGroup && git pull

- 本地分支关联远程分支

  - 远程不存在

    git push -u origin branchName 

  - 远程分支存在

    git branch --set-upstream-to origin branchName

 - 不新增commit
     - git add . && git commit --amend --no-edit  

 - 合并commit   

   git rebase -i commitId (不包含该commitid)

- 删除分支
  git branch -d branchName

- 删除远程分支
  git push origin --delete origin/branchName

