# Blog
My german Blog at [https://mstroh76.github.io/Blog/](https://mstroh76.github.io/Blog/)



## Create/Update:

```
apt install hugo
mkdir Blog
cd ../Blog
git clone git@github.com:mstroh76/Blog.git
mkdir -p ../HugoBlog/themes
cp -v update.sh git.sh config.toml ../HugoBlog
cd  ../HugoBlog/themes
unzip ../../Blog/hugo-clarity.zip
cd ..
chmod +x *.sh

./update.sh
```