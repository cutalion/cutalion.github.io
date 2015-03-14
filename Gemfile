source "https://rubygems.org"

group :development do
  gem 'rake'
  gem 'jekyll'
  gem 'rdiscount'
  gem 'pygments.rb'
  gem 'RedCloth'
  gem 'haml'
  gem 'compass'
  gem 'sass'
  gem 'sass-globbing'
  gem 'rubypants'
  gem 'rb-fsevent'
  gem 'stringex'
  gem 'liquid'
  gem 'directory_watcher'
end

gem 'sinatra'

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
