Git uses SHA-1 value to address commits, files, etc. A "simple name" to reference these values instead of SHA-1 is a "reference".

See [[git objects]]

A branch is essentially a pointer / reference to head of a commit.

![[Pasted image 20240222150301.png]]

`HEAD` file is a symbolic reference to the branch you're currently on.


Remotes are the last HEAD value of a branch that a server received, living in `refs/remotes`