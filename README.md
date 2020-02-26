# Allo-Media tech blog

This is where we write about things we learn.

## Local Jekyll setup

```
$ sudo apt install nodejs ruby ruby-dev
$ bundle install --path vendor/bundle
$ npm install
```

To serve the blog locally:

```
$ npm start
```

The blog is served at [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

> Note: the first time you load this URL, the styles might be broken; in such a
> situation, simply save an html or scss file from the sources.

## Publish to github pages

```
$ npm run publish
```

The blog is published at [http://tech.allo-media.net/](http://tech.allo-media.net/).

## Update dependencies

To update the dependencies (for example to fix a security alert), run:

```
$ bundle update github-pages
```

This will update the `Gemfile.lock` file with the updated Gems compatible with github-pages.
