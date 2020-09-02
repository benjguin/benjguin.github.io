#blog.3-4.fr source repo

The maintenance of this site moves to GitHub so that GitHub Actions can publish without having to store secrets

`markdown-sources` branch contains the sources.  
Github actions generate `gh-pages` branch content which is what is published to the blog.3-4.fr Github Pages.

##Hexo

https://hexo.io

```
npm install -g hexo-cli
```

alternate engines: cf https://staticsitegenerators.net/

##how to generate and test locally

```
cd C:\dev\_git\vsonline\blog34frsources
hexo server
```

then test on http://localhost:4000/

## how to publish

```
cd C:\dev\_git\vsonline\blog34frsources
hexo generate
internal\publish.cmd
```

then from Ubuntu command line

```
cd /mnt/c/dev/_git/GitHub/benjguin.github.io
git add --all && git commit -m "new blog post" && git push origin master

cd /mnt/c/dev/_git/vsonline/blog34frsources
git add --all && git commit -m "new blog post" && git push origin master
```
