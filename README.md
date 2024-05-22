# Quartz v4

> “[One] who works with the door open gets all kinds of interruptions, but [they] also occasionally gets clues as to what the world is and what might be important.” — Richard Hamming

Quartz is a set of tools that helps you publish your [digital garden](https://jzhao.xyz/posts/networked-thought) and notes as a website for free.
Quartz v4 features a from-the-ground rewrite focusing on end-user extensibility and ease-of-use.

🔗 Read the documentation and get started: https://quartz.jzhao.xyz/

[Join the Discord Community](https://discord.gg/cRFFHYye7t)


## Personal Notes

Majority of the repository contents is a clone of quartz.

### Build & host locally

```shell
npx quartz build --serve
```

### Sync content to remote

```shell
npx quartz sync --no-pull
```

You can also do the traditional `git add .` and `git push`. The quartz command is a wrapper around git.

### Process to sync Dropbox to content folder

TODO: automate this process in Github actions on a cron?

Use [dropdl](https://github.com/mattngu01/dropdl) to download all of the contents of '/Apps/remotely-save/Personal Vault' to `content` dir, then sync it up with the github repository.
