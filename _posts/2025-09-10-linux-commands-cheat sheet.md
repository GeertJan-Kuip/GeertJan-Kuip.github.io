# Linux commands cheat sheet

Briljant [video of NetwerkChuck](https://www.youtube.com/watch?v=gd7BXuUQ91w) turned into a cheat sheet. I have added links to pages where flags are explained.

- ssh
- clear

## Files

- ls (-l, -al)  _- list files_
- pwd  _- print working directory_
- cd  _- change directory_
- touch {filename}  _- create file_
- echo {text} {> filename}  _- print to console or add text to file_
- nano {filename}  _- create/open and edit file. exit by ctrl x, type y for save_
- vim {filename} _- create/open and edit file. i to start edit, escape :wq to save and exit_
- cat {filename} _- read file_
- shred {filename} _- overwrite file so it becomes unrecoverable_
- mkdir {directory name} _- create directory_
- cp {source} {destination} _- copy file_
- mv {source} {destination} _- move file_
- rm {file} _- remove file_
- rmdir (-r) {directory} _- remove directory. -r removes recursively_
- ln (-s) {file} {link} _- create link to a file_

## Users, groups, permissions

- whoami
- useradd {username} _- add user_
- adduser {username) _- add user and set all params_
