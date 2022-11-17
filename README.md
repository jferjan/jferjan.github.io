# Commands

* `docker build -t jferjan/mkdocs-material:1 .` - Build custom docker image from latest mkdocs-material image.
* `docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 new [dir-name]` - Create a new project.
* `docker compose up -d` - Start the live-reloading docs server.
* `docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 build` - Build the documentation site.
* `docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 gh-deploy` - Deploy to GitHub
* `docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 -h` - Print help.

# Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.


For full documentation visit [mkdocs.org](https://www.mkdocs.org) and [squidfunk.github.io/mkdocs-material](https://squidfunk.github.io/mkdocs-material/).