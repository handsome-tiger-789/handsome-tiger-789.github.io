# frozen_string_literal: true

source "https://rubygems.org"

gemspec

gem "html-proofer", "~> 5.0", group: :test

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]

gem "jekyll-remote-theme"
gem "jekyll-sitemap"

# Pin to avoid sass-embedded 1.100.0 source-build bug (NameError: JSON::Fragment) on Ruby 3.1 CI
gem "sass-embedded", "~> 1.93.0"