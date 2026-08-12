source "https://rubygems.org"

# Deployed via a custom GitHub Actions workflow (.github/workflows/pages.yml),
# not GitHub's legacy hosted Pages builder — so we don't need the `github-pages`
# gem's exact version-matching guarantee, and shouldn't pay for it. That gem
# pinned a 2017-era Jekyll/kramdown/liquid stack whose gems assumed stdlib
# libraries (rexml, base64, bigdecimal, csv) were always on the load path and
# never `tainted?`/`taint` got removed from String — both no longer true on
# current Ruby. Plain, current Jekyll declares its own dependencies properly.
gem "jekyll", "~> 4.3"

group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-remote-theme"
  gem "jekyll-seo-tag"
  gem "jekyll-include-cache" # optional dep of just-the-docs; theme require fails without it
end

# `jekyll serve` (local dev only) needs this explicitly on Ruby 3+ — removed
# from the stdlib default gems.
gem "webrick", "~> 1.8"

# Windows and JRuby compatibility (safe to include on macOS/Linux)
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", platforms: [:mingw, :x64_mingw, :mswin]

# Lock `http_parser.rb` gem to a compatible version on JRuby builds
gem "http_parser.rb", "~> 0.6.0", platforms: [:jruby]
