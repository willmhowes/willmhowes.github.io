+++
title = "Automated static site generation with Dreamhost and Zola"
date = 2022-11-14

[taxonomies]
tags = ["rust", "ssg", "zola", "dreamhost"]

+++

Today I decided to post my thesis notes on this blog. But I first had to make the blog, which only took an hour or two. The much more interesting challenge is whether or not I would be able to automate deployment of the statically-generated site using my dreamhost server, a shared tier on which I do NOT have root access. I wasn't sure how restricted I would be in running the ssg's build command. I am using [Zola](https://www.getzola.org/), which is a static site generator that uses rust on the back-end (woo-hoo). I read [here](https://help.dreamhost.com/hc/en-us/articles/216994417-Is-root-sudo-access-available-) that I can just run a binary from the home folder, so I copied [this](https://github.com/getzola/zola/releases/tag/v0.14.0) tarball over to the home folder on my server. I had to use an older version because of an  [incompatibility](https://github.com/chrismaltby/gb-studio/issues/1083) between the particular linux image used by Dreamhost shared servers and the precompiled binary.

After I SSH'd into the server, I extracted the tarball with `tar -xvzf zola-v0.14.0-x86_64-unknown-linux-gnu.tar.gz`, a command I sourced from [here](https://phoenixnap.com/kb/extract-tar-gz-files-linux-command-line). I then used [this](https://help.dreamhost.com/hc/en-us/articles/216445197-Pushing-your-local-Git-repository-to-a-DreamHost-server-Linux-Mac-OS-X) dreamhost blog post to setup a git repo on server that was ready to receive the repo I had on my laptop. While I was at it, I followed [this](https://jigarius.com/blog/multiple-git-remote-repositories) blog post to setup a remote which pushed to both my server and a GitHub repository.

Then I wrote the following bash script, called `compile.sh`:

```bash
#!/bin/bash
cd ~/
echo "Refreshing clone..."
rm -rf ~/clone.willm.how.git
mkdir ~/clone.willm.how.git
git clone ~/willm.how.git ~/clone.willm.how.git/
cd ~/clone.willm.how.git/
../zola build -o ~/willm.how
rm -rf ~/clone.willm.how.git
cd ~/
```

The script just clones the repo into a folder, runs the Zola build command, and then removes the clone folder. I ran `chmod u+x compile.sh` to make the script executable, and it worked just as I had hoped! This was exciting for me because I can honestly say I've never written a bash script that was actually useful to me until now. Now I had to figure out how to get the server to run the script for me...

In my research I had come across [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks), but I was only coming across hooks that were triggered on push (client-side) when what I needed was a hook that triggered on receipt of a push (server-side). I found [this](https://stackoverflow.com/questions/32053967/git-automatically-run-bash-script-on-the-server-after-any-client-pushes) stack overflow post and followed its link to [this](https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps) DigitalOcean tutorial on git-based site deployment automation. It was here that I discovered the existence of the lovely post-receive hook. I ran `touch post-receive ` inside the `.git/hooks/` directory. The code inside post-receive looks like this:

```bash
#!/bin/bash
~/compile.sh
```

because all it does is run the script I wrote. I could just keep the code from compile.sh inside the post-receive hook, but if I ever want to manually recompile it will be easier to run from the home folder rather than inside the hooks folder of a git repo (I imagine someone who knows better than me is screaming at their monitor about how poorly organized my files are, feel free to email me at [me@willm.how](mailto:me@willm.how) with your suggestions). After this, I was finished before you can say `chmod u+x post-receive` three times fast.

I euthanized a stray `.DS_Store` and pushed the change to both remotes, and bada-bing bada-boom, the site recompiled just like I had hoped for. The DigitalOcean article above sets out an interesting way of staging beta and live sites changes, and it should be easy enough to replicate this process on a subdomain. But for now, I hope someone else confused about what can you and cannot do with a Dreamhost shared hosting plan finds themselves as pleased as I am.

Thank you to [this](https://robinforest.net/post/hugo-questions/) article for inspiration.