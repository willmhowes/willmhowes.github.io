+++
title = "Automated static site generation with Dreamhost and Zola"
date = 2022-11-14

[taxonomies]
tags = ["rust", "ssg", "zola", "dreamhost"]

+++

# Backstory

I had been squatting on the [willm.how](https://willm.how) domain for a number of months until today I decided to finally put it to use. I skimmed through the [jamstack](https://jamstack.org/generators/) static site generator list as I do everytime I think about starting a blog, staring down a sea of javascript-backends. When I spotted [Zola](https://www.getzola.org/), a static site generator that uses rust on the back-end (woo-hoo), I had a feeling this attempt might be different. An hour or two later, my blog was live! Quickly, the real challenge revealed itself: whether or not I would be able to automate deployment of the statically-generated site using only my dreamhost server, a shared tier on which I do NOT have root access (Spoiler: it not only possible, it's also pretty straightforward).

# Binary trickery and bash scripting

My initial hurdle was not knowing how I would run Zola's build process without installing it on the server. I considered integrating GitHub Actions, but the learning curve for that seemed too high for what I was trying to do. Then, I read [here](https://help.dreamhost.com/hc/en-us/articles/216994417-Is-root-sudo-access-available-) that I can just run a binary from the home folder of the server.

{{ image(src="homer_doh.jpg",
      position="center", alt="Homer Simpson smacking his head and saying 'doh'") }}

I copied [this](https://github.com/getzola/zola/releases/tag/v0.14.0) tarball onto the server. I had to use an older version because of an [incompatibility](https://github.com/chrismaltby/gb-studio/issues/1083) between the particular linux image used by Dreamhost shared servers and the precompiled binary provided by Zola. In theory, this issue resolves if I compile the binary myself but I'm okay running the old binary for now.

After I SSH'd into the server, I extracted the tarball with `tar -xvzf zola-v0.14.0-x86_64-unknown-linux-gnu.tar.gz`, a command I sourced from [here](https://phoenixnap.com/kb/extract-tar-gz-files-linux-command-line). I then used [this](https://help.dreamhost.com/hc/en-us/articles/216445197-Pushing-your-local-Git-repository-to-a-DreamHost-server-Linux-Mac-OS-X) dreamhost blog post to setup a git repository on the server that was ready to receive the repository I had on my laptop. While I was at it, I followed [this](https://jigarius.com/blog/multiple-git-remote-repositories) blog post to configure git-remote to be able to push to both my server and a GitHub repository in one command.

Then I wrote the following bash script, called compile.sh:

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

The script just clones the repo into a folder, runs the Zola build command, and then removes the clone folder. The clone folder is necessary because the repository does not automatically have a directory associated with it here. We call the directory associated with a repository a [working tree](https://craftquest.io/articles/what-is-the-working-tree-in-git) . I ran `chmod u+x compile.sh` to make the script executable, and it worked just as I had hoped! This was exciting for me because I can honestly say I've never written a bash script that was actually useful to me until now. I was satisfied with the progress I had made, and I would be okay to manually run compile.sh after every push. But it would be that much better if I could have the server to run the script for me...

# God bless git hooks

In my research I had come across [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks), but I was only coming across hooks that were triggered on push (client-side) when what I needed was a hook that triggered on receipt of a push (server-side). I found [this](https://stackoverflow.com/questions/32053967/git-automatically-run-bash-script-on-the-server-after-any-client-pushes) stack overflow post and followed its link to [this](https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps) DigitalOcean tutorial on git-based site deployment automation. It was here that I discovered the existence of the lovely post-receive hook. I ran `touch post-receive` inside the .git/hooks/ directory. The code inside post-receive looks like this:

```bash
#!/bin/bash
~/compile.sh
```

because all it does is run the script I wrote above. I could just keep the code from compile.sh inside the post-receive hook, but if I ever want to manually run compile.sh it will be fewer keystrokes to run from the home folder rather than inside the hooks folder of a git repo (I imagine someone who knows better than me is screaming at their monitor about how poorly organized my files are, feel free to email me at [me@willm.how](mailto:me@willm.how) with your suggestions). After this, I was finished before you can say `chmod u+x post-receive` three times fast.

I euthanized a stray .DS_Store in order to change the state of the repository and pushed the change to both remotes, and bada-bing bada-boom, the push triggered a Zola build like I had hoped for. I hope someone else confused about what can you and cannot do with a Dreamhost shared hosting plan finds themselves as pleased as I am.

Thank you to [this](https://robinforest.net/post/hugo-questions/) article for inspiration.

# Future Improvements

1. Improve the accessibility of the theme's HTML with ARIA tags. It's also possible the current theme is poor for accessibility so there is probably work to be done there. Using the [WAVE](https://wave.webaim.org/) tools should help.
1. The DigitalOcean article above sets out an interesting way of staging beta and live sites changes, and it should be easy enough to replicate this process on a seperate subdomain.
1. I realized that I should be able to add zola to the $PATH variable on my server, which would make it easier to invoke because I wouldn't have to locate the binary before running it. Of course, it's just in the home folder but I digress.
1. [Compile](https://www.getzola.org/documentation/getting-started/installation/#from-source) the zola binary for the server

# Update - 2022-11-15

I updated compile.sh to read as follows:

```bash
#!/bin/bash
cd ~/
echo "Refreshing clone..."
rm -rf ~/clone.willm.how.git
mkdir ~/clone.willm.how.git
git clone ~/willm.how.git ~/clone.willm.how.git/
cd ~/clone.willm.how.git/
../zola build -o ~/willm.how/blog/
rm -rf ~/clone.willm.how.git
cd ~/
```

You'll notice the only change is that Zola builds the site into a subdirectory of the willm.how directory. This is simply to make it so that my blog live inside a "blog" slug when a user visits the website. I just like the way it looks. The only other change to ensure that Zola compiled properly is to update the base_url variable in config.toml to include the new slug. With that, everything worked as I wanted. However, if somebody goes to just https://willm.how, they would find nothing. So I added the following index.html to the root directory of the site:

```html
<!DOCTYPE html>
<html>
   <head>
      <title>HTML Meta Tag</title>
      <meta http-equiv = "refresh" content = "0; url = https://willm.how/blog" />
   </head>
   <body>
      <p>Redirecting to another URL</p>
   </body>
</html>
```

[This](https://www.tutorialspoint.com/How-to-automatically-redirect-a-Web-Page-to-another-URL) is a simple HTML file which redirects the user directly the blog page. In the future I intend to have a better landing site, but a simple redirect will do for now.
