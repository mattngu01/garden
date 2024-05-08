https://git-scm.com/book/en/v2/Git-Internals-Git-Objects

Git is content-addressable filesystem - key=`content hash`, value=`content`. 
```console
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4

$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

Git stores filenames & directories as tree objects, content as blob objects. Git stores commit metadata & info in its own tree.

![[Pasted image 20240222150022.png]]