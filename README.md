# docs.robotical.io
Documentation Site

# Install

Make sure you have the system packages `ruby` and `rubygems` installed

```
$ sudo dnf install ruby rubygems
```

Then install the Ruby gems `jekyll` and `jekyll-paginate`:

```
$ gem install jekyll
$ gem install jekyll-paginate
```

Finally, run a development server (in the root dir of the repo):

```
$ jekyll serve -w
Configuration file: /.../_config.yml
            Source: /.../docs
       Destination: /.../docs/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.747 seconds.
 Auto-regeneration: enabled for '/.../docs'
Configuration file: /.../docs/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

And go to http://127.0.0.1:4000/ to see the site. Changes made to the source will be refreshed into the build by Jekyll


## NOTE

The CSS source for this site is the same as that for the main website, and is managed there. To make changes, they must be made there then copied in here.
