# [https://grahamg.xyz](https://grahamg.xyz)
Blog about software development projects and related topics. Occasionally I’ll talk about something unrelated, like geopolitics; doing otherwise in these times is irresponsible.

## Theme

This blog uses a local Jekyll theme adapted from [The Proportional Web](https://owickstrom.github.io/the-proportional-web/) by Oskar Wickström. It retains the original narrow reading column, black-on-white palette, Alegreya typography, small-cap headings, indented paragraphs, and floral rules, with Jekyll layouts for posts, pages, archives, pagination, and an Atom feed.

- `_layouts/` and `_includes/` contain the theme templates.
- `assets/proportional/index.css` contains the upstream stylesheet, with its external font import removed. Its MIT license is included alongside it.
- `assets/main.css` adapts the original styles for this blog.
- `assets/fonts/` contains self-hosted Alegreya, Alegreya SC, and Courier Prime fonts and their SIL Open Font Licenses.
- `_data/navigation.yml` controls the header links.

To preview locally (Ruby 3.3 is used by the deployment workflow):

```sh
bundle install
bundle exec jekyll serve
```

Open `http://localhost:4000`. To generate the static site, run `bundle exec jekyll build`. The existing GitHub Pages workflow builds and deploys pushes to `main`.
