---
title: Ruby Tips & Tricks
date: 2025-10-13
description: Ruby Tips & Tricks to Solve Annoying Issues
draft: false
category: Ruby
---

# Helpful Tips to Setup Ruby tools

Over the years, I've had multiple issues setting up ruby in nvim. Sometimes the lsp wouldn't work, and other times, the toolings wouldn't work. These are few tips I've picked up over the years. I will add more as I discover them.

### Installing Ruby with SSL

This is how we can install ruby with a different version manager other than rbenv. In this case, mise. Here, we are also using the `RUBY_CONFIGURE_OPTS`. 

Install ruby and configure specific open ssl version (1.1) in this case

```rb
RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@1.1)" mise install ruby@3.3.0
```
Now, Check the ssl version used on installing ruby. 

```rb
ruby -ropenssl -e 'puts OpenSSL::OPENSSL_LIBRARY_VERSION'
```

### CA certificates

Sometimes, re-installing ca-certificates helps fixing Rail's certificates issues.

```bash
brew install ca-certificates
# if reinstalling
brew reinstall ca-certificates
```
### Incompatible rubocop version

If your editor uses a rubocop version (globally installed) that is not compatible with the project dependencies and have issues. 

First, find out the installed globally installed rubocop version

```rb
rubocop -v
1.81.1 # should print something like this
```

Find out the rubocop version in dependency list. Cat the Gemfile

```bash
cat Gemfile
```
If the version you see in shell is different than what you see in Gemfile then there might be conflicts. To resolve this, you need to reinstall the Gemfile version.

First un-install the rubocop from the project. Comment out the dependency and run bundle install. Then re-install the gem, rubocop in this case. 
