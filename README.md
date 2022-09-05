# Personal blog
The theme is based on [Minimal Mistakes Jekyll theme](https://mmistakes.github.io/minimal-mistakes/). I use a custom homepage located in `_pages/home.md` instead of default `index.html` at the root of the project. The rest of the stuff is the regular configuration with reference to [the offical documentation](https://mmistakes.github.io/minimal-mistakes/docs/configuration/).

## Deployment
### Gem-based method
1. `Gemfile` content:
```
source "https://rubygems.org"

gem "minimal-mistakes-jekyll"
gem "webrick", "~> 1.7"
```

2. use `bundle` command.

3. set the `theme` attribute in `_config.yml` file.

### Remote theme method
1. `Gemfile` content:
```
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
gem "jekyll-include-cache", group: :jekyll_plugins
gem "webrick", "~> 1.7"
```

2. make sure `jekyll-include-cache` in `plugins` array of `_config.yml`.

3. use `bundle` command.

4. set the `remote_theme` attribute in `_config.yml` file while comment others.