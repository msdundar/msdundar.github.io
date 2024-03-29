# msdundar.github.io

Compile site in minified version:

```sh
hugo --minify
```

Compile site and generate /public folder:

```sh
hugo -D
```

Build site and run server:

```sh
hugo server -D
```

Create a new post:

```sh
hugo new posts/my-new-post.md
```

Convert the site:

```sh
hugo convert toJSON # Convert front matter to JSON
hugo convert toTOML # Convert front matter to TOML
hugo convert toYAML # Convert front matter to YAML
```

Update theme:

```sh
git submodule update --remote --merge
```

Update hugo:

```sh
brew update && brew upgrade hugo
```

Cover image size: `1200x628px`
