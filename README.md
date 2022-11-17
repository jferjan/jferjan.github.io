
# Serve

docker compose up -d

# GitHub deploy

docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 gh-deploy


# EXTRA

## New website

docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 new .

## Build - Export static website to /site folder 

docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 build

## New Lorem page

curl 'https://jaspervdj.be/lorem-markdownum/markdown.txt' > docs/about.md

## HELP

docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 -h

