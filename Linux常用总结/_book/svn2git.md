* 依赖的工具包  
```bash  
    yum install git-svn
```  
* 获取SVN历史提交用户信息，创建SVN用户与GIT用户关系映射文件(提交过的用户都需要获取，否则任务会被中断 )  
```bash    
    svnuser = gituser <mail>  
    svnuser1 = gituser1 <mail>  
```  
* 拉取svn仓库    
```bash   
    拉取所有的trunk、branch和tags
    git svn clone --stdlayout --authors-file=authors.txt --no-metadata svn://172.16.16.4/wxapi.soyoung.com wxapi.soyoung.com  
    仅拉取trunk和branck  
    git svn clone --trunk=trunk --branches=branches --authors-file=authors.txt --no-metadata svn://172.16.16.4/wxapi.soyoung.com wxapi.soyoung.com  
```   
* 分支处理  
```bash  
    git branch -r
    git branch -d <branchname> # 删除不需要的分支
```  
* 添加和push git仓库  
```bash  
    #处理trunk，提交至git的master
    cd projectname  
    git remote add origin git@xxx.xxx.xxx.xxx:root/projectname.git  
    git push -u origin master  
    #切换到分支，并提交到git的分支中  
    git checkout remotes/spuwxapi.soyoung.com  
    git branch -a  
    git checkout -b spuwxapi  
    git push --all  
```  
* 遇到的报错  
```bash
Use of uninitialized value $u in substitution (s///) at /usr/share/perl5/vendor_perl/Git/SVN.pm line 106.
Use of uninitialized value $u in concatenation (.) or string at /usr/share/perl5/vendor_perl/Git/SVN.pm line 106.
#解决方案，修改SVN.pm line 106.
$u =~ s!^\Q$url\E(/|$)!! or die 
        "$refname: '$url' not found in '$u'\n"; 
修改为：
if(!$u) { 
        $u = $pathname; 
}else { 
        $u =~ s!^\Q$url\E(/|$)!! or die 
        "$refname: '$url' not found in '$u'\n"; 
} 
```
