source "https://rubygems.org"

# Use the GitHub Pages gem for full compatibility with GitHub Pages builds.
# This pins Jekyll and all plugins to the versions used by GitHub Pages.
gem "github-pages", group: :jekyll_plugins

# Additional plugins
group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-remote-theme"
  gem "jekyll-seo-tag"
end

# Windows and JRuby compatibility (safe to include on macOS/Linux)
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", platforms: [:mingw, :x64_mingw, :mswin]

# Lock `http_parser.rb` gem to a compatible version on JRuby builds
gem "http_parser.rb", "~> 0.6.0", platforms: [:jruby]
