# erkan000.github.io

Kişisel notlarımı yayınladığım web siteme ait repodur.



# Altyapı
 jekyll + jekyll-theme-midnight
 

# 
bundle exec jekyll serve

## Docker

```sh

export JEKYLL_VERSION=3.8
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:$JEKYLL_VERSION jekyll build
docker run --rm --volume="$PWD:/srv/jekyll" -it -p 4000:4000 jekyll/jekyll:$JEKYLL_VERSION jekyll serve --watch --drafts

```
