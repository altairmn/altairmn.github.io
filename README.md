To make changes, do the following steps:

1. Install hugo, and run `hugo server` in the base directory
2. Write post using typora. Transfer all images to `./static/images`
3. Run `hugo` in the basedir to generate all output
4. Replace image paths in the generated html with `x/y/z/images/<image>` to `/images/<image>`. This makes sure that it runs in the browser.
