Introduction to datalad & GIN
---

Datalad is a piece of software that runs on your computer (like git). It takes care of dataset versioning and many other stuff. Find out more in [this handbook](http://handbook.datalad.org/en/latest/). 

GIN is a remote repository (like Github) that can store data. Dalatad can interact with GIN, so you can use it to push and pull data back and forth between your local computer and the remote storage. Make an account [here](https://gin.g-node.org/). 

> **Notes**
> Parts of code that are in < > brackets suggests that you should insert your own name or path or whatever. 

<br> 

---

### create / clone / install

Create an empty folder and run: 
```
datalad create --description "<dataset description blablabla>" -c text2git <datasetFolderName>
```
This will turn that directory into a datalad dataset (same as `git init` in git...). 

If we already have some data in the directory, we just need to use the `--force` option when creating new datalad dataset. Don't be scared. 

If we just want to clone an existing dataset with its original name: 
```
datalad clone </path/to/to-be-cloned-dataset> --description "<blabla>"
```

Description is not really necessary (it can only be used for `git annex whereis` later).

> **Note**  
> `clone` and `install` are pretty much the same thing (clone calls install under the hood). `install` has more options, e.g. directly retrieve subdatasets (otherwise you must call `datalad get -n -r` after cloning the superdataset)

<br> 

### save

This does equivalent of `git add` and `git commit` in one step: 
```
datalad save -m "<commit message>" 
```

To commit only change in a  specific file (otherwise all changes will be committed together automatically): 
```
datalad save -m "<commit messgage>" </path/to/file.ext>
```

Only save modified (not untracked) files: 
```
dl save -m "<message>" --updated
```

Save explicitely to git (not annex!): 
```
datalad save -m "message" --to-git <path/to/file>
```

You can tag the last datalad save (after calling `datalad save -m "blablabla"`)
```
datalad save --version-tag <my_version_name>
```

<br>

### siblings + merge

Sibling is just a copy of the same dataset (same as `remote` in git). It can be placed on the same computer (just in a different folder), or in a repository on a remote server (e.g. GIN). Datalad allows synchronization of changes across these copies. 

To add a sibling (equivalent to `git remote add ...`)
```
datalad siblings add -d . --name <nameOfRemote> --url <localPathOrUrlToTheRemote>
```

Equivalent to `git fetch`. 

```
datalad update -s siblingName 
```
> **Note** 
> Without the `-s` option all remotes are updated (saved in orgin/master). You can use `git checkout` and `git diff` to explore before merging)

Equivalent to also merge (equivalent to "git pull")
```
datalad update --merge -s siblingName
```



### annex

Annexed "files" are just symlinks pointing to a hidden folder (.git/annex/objects) where the actual files are (as long as datalad get has been performed).

So a dataset without retrieved content looks like a pile of broken symlinks! 

remove from annex
```
git annex unannex  <pathToFile>
```
or
```
datalad unlock <pathToFile>
```

Both commands remove the annex, but the origin links are retained in the background. So if we don't do anything and run `datalad save`, the original annex will be re-established. 

In fact, `datalad run` performs unlock in the backgound. 


The "unannexed" files become untracked, you can save them to git
```
datalad save --to-git -m "commit message" pathToFile 
```

Otherwise they will be saved using *`annex.largefile` rules* on the next `datalad save`

If annexed files are copying across dataset boundaries, the metadata about their origin is preserved, so you should be able to get them in the new dataset too. 

<br>


### get

Just after `clone` and before running `get`, all the annex symlinks are "broken". 

Beware that some file browsers (e.g. on OSX) might not show broken symlinks! 

Check annex size for all files (including subdatasets) together
```
datalad status --annex all
```

To check the size of annexed content for the root dataset:
```
datalad status --annex
```
> **Note** 
> If you want to show annex info about a subdataset, you have to be in the subdataset folder (otherwise you only get info for the master dataset). 

Show the path to origin of the file link (annexed file)
```
git annex whereis <filename>
```

Show where source data is for a list of annexed files
```
git annex list .
```

> **Note** 
> After `get` from web, the local path is added to the origin of the file (so there are 2 possibilities, local + web)

To obtain the actual content of the annexed files
```
datalad get pathtofile
```

To obtain _all_ the annexed files (careful with large data)
```
datalad get .
```

Remove the content of annexed files, i.e. opposite of `get` (careful with the option `--no-check`, this can remove data permanently!)
```
datalad drop <pathtofile>
```


<br>

### GIN

always use ssh NOT http!!!

!!! when creating new repo on gin, make sure it's empty (i.e. no LICENSE or README files)!!! otherwise, you won't be able to push to it...

add remote as a sibling
```
datalad siblings add -d . --name gin --url <URL>
```

push to it
```
datalad push --to gin
```


push also subdatasets (--recursive flag pushes all subdatasets to their respective reponame, if they have one)
```
datalad push --to reponame --recursive
```

publish subdatasets see: 
http://handbook.datalad.org/en/latest/basics/101-139-gin.html#subdspublishing


!!! when pushing, you need to tell datalad the repo has annex (for some reason it doesn't know with gin)
see -> https://handbook.datalad.org/en/latest/basics/101-141-push.html

```
datalad push --to gin --data anything --force all
```


<br> 

### procedures 

List available procedure scripts and their locations 
```
datalad run-procedure --discover
```

a procdure (e.g. yoda) can be directly applied at the creation of a dataset
```
datalad create -c yoda newDatasetName
```

or later within the dataset
```
datalad run-procedure cfg_yoda
```

<br> 

### config 

add new parameter to config file the scope can be: 
--global 
--local 
--system

```
git config --global --add section.variable value
```

general command format: 
```
git config --local/--global/--system --add/remove-all/--list section.[subsection.]variable "value"
```

the sections are defined in the config file, e.g. [core], [user], [annex], [submodule] etc. 

each section can have subsection, e.g. [remote "ceren"], in that case, need to use section.subsection.variable format

to change a variable in the config
```
git config --global --replace-all section.variable value
```

to remove a variable
```
git config --unset section.variable
```

list the currently set config parameters
```
git config --list --show-origin
```

.git/config is not version controlled

.gitattributes, .gitmodules, .datalad/config are version controlled

change them using 
```
git config -f targetConfigFile 
```

careful: git config can't change .gitattributes (it has different format)

<br> 

### gitattributes 

!!! IMPORTANT !!!
If you’re creating datalad dataset in a folder where you already have some files, and you want certain rules to be applied to these files, .gitattributes files have to be ready before you run the first datalad save command !! (otherwise its a pain to annex the files once they have been saved to git, unless the files themselves are edited...which you often don’t want with datafiles)

On each line, the scope of the rule across the directory structure is defined first 
* glob `*` means current directory
* glob `**` means all subdirectories recursively

Then, different attributes of the annex are set, e.g. what is considered a large file based on: 
*  file type `(mimetype=text/*)`
* encoding `(mimencoding=binary)` 
* size `(largerthan=100kb)`

e.g. 
```
((largerthan=100kb)and(not((include=*.c)or(include=*.h))))
```

To get a negated rule, use e.g. 
```
(not(mimetype=text/*))
```

no .git subdirectory (in any subdirectory) should be considered large: 
```
**/.git* annex.largefiles=nothing
```

to make an exception for a file (so it's not annexed, this can be done e.g. for README.md)
```
<path/filename> annex.largefiles=(nothing)
```

if you want everything in a folder to be annexed, set 
```
<path>/* annex.largefiles=(everything)
```

a separate .gitattributes file can be in each subdirectory of the dataset (e.g. in .datalad subdir) and will over-rule the higher level gitattributes! 

While Windows relies on extension (e.g. ".txt") to open a file, UNIX relies on MIME-type, which is a file property, which you can find out with: 
```
file --mime-type <pathToFile>
```

e.g. 
* bash scripts are text/plain
* python scripts are text/x-python
* jpg file is image/jpg
* pdf is application/pdf


<br> 

### go back in time

detach HEAD
```
git checkout SHA
```

go back to tip of branch 
```
git checkout branchname
```

print file contents without git checkout SHA
```
git cat-file SHA
```


<br> 

### undo things

This is exactly the same as for git  
https://www.atlassian.com/git/tutorials/undoing-changes/git-reset


check anthony explaining reflogs and reset
https://youtu.be/R8R9_eT2law

add staged content to the last commit (and change the commit message)
```
$ git commit --amend 
```

if there is a new untracked file or non-added changes in the working directory that you want to remove
```
$ git checkout -- filename
```

the "--" is to separate the commits (there are none in this case cause we want the HEAD on the current branch) and filenames that should be checked out (note: generally in bash, "--" is used to signify end of command options, so this is a bit different)


**reset**   
remove last commit, but keep the modifications as non-added (you can refer to that commit with anything, HEAD~1, SHA, branch_name, etc.) 
```
$ git reset --mixed SHA
```

remove last commit, but keep the modifications as staged
```
$ git reset --soft SHA
```

remove last commit and discard the changes altogether (!dangerous)
```
$ git reset --hard SHA
```

!!! hard reset can be undone using reflog (careful, this is regularly deleted by git)

see where the HEAD was before and find the SHA you need
```
$ git reflog
```

**revert**  
revert a single file to previous state (at commit <SHA>) and get the file modification from that point onward back to working directory (don't change history or anything)
```
$ git checkout <SHA> -- filename
```

to undo changes but don't change history (this creates a new commit)  
```
$ git revert <SHA>
```


**clean**   
git clean removes new untracked files (e.g. after trying code that produces output, we need to clean this "testing" output before using "datalad run")

dry run (see what would be removed)
```
git clean -n 
```

remove files (-f), directories (-d), ignored files (-X), ignored and nonignored files (-x)
```
git clean -fx
```

git clean and reset can be both used equivalently to clean working tree only git reset can reset things in the staging index 


<br> 

### move 

-> it's okay to use `git mv` for non-annexed files

!!! Make sure to not use `git mv`, especially for subdatasets. Use UNIX `mv` and then `datalad -save` to clean up afterwards (otherwise annex links get messed up). 

<br> 

### remove

to remove a file, but keep it in history, just use UNIX rm and datalad save (annexed files can be removed (or moved) with UNIX rm and mv, and subsequent datalad save will fix the links etc.)

to remove and re-write history (dangerous), use 
```
git reset --hard HEAD~
```

permanently remove annexed content (but keep links)
```
datalad drop pathToFile
```

use flag --recursive to remove whole subdatasets

--nocheck flag (dangerous) allows dropping even when no source info available to get the content again

remove a subdataset but keep the url to re-install it anytime with datalad get --recursive
```
datalad uninstall -d . subdatasetName --recursive
```

remove datasets or subdatasets completely(!!!)
http://docs.datalad.org/en/stable/generated/man/datalad-remove.html
```
datalad remove -m "remove obsolete subds" -d . subdatasetName
```

removing whole dataset will be a problem because annex locks the write permission of the files. Can change with: 
`chmod` (or just use `sudo`) 
```
rm -rf datasetNamw
```

alternatively (dangerous): 
```
datalad remove --nocheck --recursive datasetName
```

<br> 

### log 

show last commit log (with additional info)
```
git log -n 1 -p --word-diff 
```
see option `-S` of git log to search for commits that change a 
specific word 

show only specific commit (using SHA)
```
git show <SHA>
```

short log report across all commits
```
git log --oneline
```

see the first commit
```
git log --reverse
```

get history of a particular file (what created and modified it)
```
git log -- filename
```

show log of subdataset without cd
```
git -C path/to/subdataset lg
```

<br> 

### diff

datalad diff only shows which files are different
git diff shows what exactly changed in the files

compare HEAD~1 to latest commit
```
git diff HEAD~1
```

compare two commits
```
git diff oldCommit..newCommit
```

two commits can be compared using --from (by default latest commit) and --to 
(whatever other commit is being compared against --from)

compare latest commit to remote branch
```
datalad diff --to remotes/remotename/branchname
```

get diff for previous commit compared to the current (newest) one
```
datalad diff --to HEAD~1
```

additionally, you can provide a subfolder (or file/s) to report on 
```
datalad diff --to HEAD~1 filenames
```

specify range
```
datalad diff --from HEAD~2 --to HEAD~1 
```

check if folder has changed (e.g. imported files we just wanted to read from and we can delete them before archiving)
```
git diff -- folderName
```

<br> 

### run

```
datalad run -m "commit message" --input someFiles --output someFiles 'commands to run'
```

if there are uncommited changes you'll get an error "clean dataset required"

either datalad save, or force the run with --explicit flag
```
datalad run -m "" --explicit 'commands to run'
```

note: the flag must be before the actual command!!!


Linux run matlab script from command line
```
matlab -nodisplay -nosplash -nodesktop -r "run('path/to/your/script.m');exit;"
```

!!! don't use quotes when calling matlab in datalad run !!! 
instead, just do: 
```
datalad run --input file1 --output dir1 matlab -nodisplay -nosplash -nodesktop -r "run('path/to/your/script.m');exit;"
```

if there's lot to run in bash, specify the command part as
```
bash -c 'all the commands and options for bash to run'
```

if there are multiple inputs (outputs)
```
datalad run -m "" --input file1 --input file2 --output fileN
```

you can also use globing
```
datalad run -m "" --input ./file*.txt 
```

can use substitution for files in commnad {inputs} {outputs}
if you want a specifi input, index them
```
{inputs[0]}
{inputs[1]} 
```

for example
```
datalad run -m 'message' --input 'file1' --output 'file2' "script.sh {inputs[0]} {outputs[0]}"
```


if output file(s) already exists and is not specified explicitely, you will get a permission errror (if they are annexed)

<br> 

### rerun

rerun from commit SHA1 (can be timepoint when everything was ready for analysis) till the end 
 
```
datalad rerun --branch branchName --onto SHA1 --since SHA1
```

the option --onto will base the new branch on the commit SHA1 (and execute from there)

This can be used to check that the results after rerunning everything match what we got in our master (name the branch e.g. "verify"). 

Then, compare if the resulting files after re-runing are equivalent with master. 
```
git diff master --stat
```


<br> 

### containers

add a container
```
datalad containers-add containername --url shub://blablablacontainer
```

list added containers 
```
datalad containers-list
```

datalad containers-run -m "commit message" --container-name containername 

if a container is just added as as subdataset (no with containers-add)
```
datalad containers-run -m "" --container-name pathToContainer
```




<br> 

### download

This just downloads a single file and automatically commits it: 
```
datalad download-url https://blablablabla.pdf --dataset . -O datasubfolder/filename.pdf
```

If multiple files are downloaded (i.e. multipe urls passed), you can specify output folder as: 
```
datalad download-url --path myFolder/ url1 url2 
```
 
<br> 

### subdatasets

list all subdatasets (only one level depth)
```
datalad subdatasets
```

Subdatasets can be just imported like libraries and not touched after. 

clone a dataset into someDir as a subset of parent dataset specified after the --dataset flag 
```
datalad clone --dataset . https://linktodataset someDir/outputDirName
```
 
note that clone and install are pretty much the same thing (clone calls install under the hood)

you can use shortcut to the root dir of the current dataset (you don't have to cd to the root before installing/cloning
```
datalad clone -d^ <url>
```


to get subdatasets after cloning a parent dataset (same as git submodule init+update)
```
datalad get -n -r --recursion-limit 2 pathToSubdataset
```

`-n` option means no data download (only metadata)
`-r` option means recursive (all nested subdatasets)
`--recursion-limit` (only go N levels deep, dont need to use this)


But we can create subdataset within the master dataset and then work within that independently! All the master dataset knows is whether or not there was a change in the subdataset. 

create a new subdataset for a specific analysis (e.g. source2raw, or preprocessing)
```
datalad create -c yoda -d . new_subdataset_name
```

Output of such subdataset (which should be in the root level of the subdataset, e.g. as sub-01, sub-02, etc) can be used as input to another analysis step within the master dataset!!!


if you don't want to cd to the subfolder (note that this would also work with git)
```
datalad -C subproject/path status --annex all 
```

save changes recursively within all subdatasets (using the same commit message across levels) 
```
datalad save --recursive -m "commit message"
```

to fix issues with submodules (which are called subdatasets in dl) (see section on git for more info about these issues), try updating them 
```
datalad update -s subdatasetName
```



<br> 

### github

creates a remote on github (new repo and link it as remote)
```
datalad create-sibling-github -d . repoName
```

default repo name is "github"
```
datalad push --to reponame 
```

to also push tags
```
git push reponame --tags
```


!subdatastes need to have valid url (no local path, otherwise they can't get)

set it before pushing
```
datalad subdatasets --contains midterm_project --set-property url <URL>
```

!!! a subdataset on remote must be set with default branch to master and NOT git-annex !!! otherwise you will not get the directory structure you want and instead something like: 

```
2ba/2019
34a/38f
...
uuid.log
```

-> change the default branch on github (go to repo settings, branches, set default)

-> also remember that annexed files will not be avaiable from github repo unless it was explicitely pushed that way, see section 8.1.4. "Use GitHub for sharing content" of the datalad handbook


