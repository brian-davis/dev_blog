---
layout: post
title: "Hello, World!"
date: 2024-09-27 10:40:55 -0700
categories: writing
---

```ruby
def hello_world(name = "World")
  "Hello, #{name}!"
end
puts hello_world
#=> Hello, World!
```

I am reviving an old tech blog here, now using a [Jekyll](https://jekyllrb.com/) back end and [GitHub Pages](https://pages.github.com/) deployment. Previously I had built out a full Ruby on Rails 7 web app, backed by a PostgreSQL database, a Redis store, Sidekiq background jobs, AWS S3 assets hosting, deployed to Heroku, the whole nine yards. Although it was a fun experience, as that is my preferred tech stack, it was really overkill for a simple, personal blog which realistically did not receive much traffic. It was costing me a small monthly fee, which was ultimately not an expense I really think I need. Going with DigitalOcean rather than Heroku would have been about the same price. Using a simple Markdown-based system with free-tier deployment solutions is much more sensible for my current needs.

How has it been going so far? Well, I've found that the GitHub and Jekyll documentation was not quite up-to-date for 2024, or at least is missing an obvious "gotcha" or FAQ for an issue I ran into. The recommended GitHub Actions "Jekyll" deployment strategy was failing on the Ruby `bundle` step. I had set up the bare-bones repo, built an empty app with `jekyll new`, was up and running locally with `jekyll serve`. I had followed the GitHub docs and built a _jekyll.yml_ file with the default configurations. Yet the builds were failing on `git push`. The magic wasn't happening 😔. I did some troubleshooting <sup>[1](https://stackoverflow.com/a/77854067/21928926), [2](https://talk.jekyllrb.com/t/build-error-at-setup-ruby-need-help/8791)</sup> and determined that the default Ruby version in the configuration file was to blame. It had defaulted to `ruby-version: "3.1"`, whereas I am running version 3.2.2 locally, and had built my _Gemfile.lock_ as such. updating that line to `ruby-version: "3.2"` in _jekyll.yml_ did the trick, and now pushing to GitHub does indeed deploy smoothly. So watch out for that if you are in a similar situation.

Similarly, the themes I have tried so far have been a bit troublesome, again because of versioning and outdated dependencies.

I'm running the latest version:

```
$ jekyll --version
jekyll 4.3.4
```

But it seems that most of the popular gem-distributed Jekyll themes don't like version 4 (or the _github-pages_ gem) and won't `bundle`. What I might have to do is fork the default `minima` theme, make my own changes, distribute as my own gem. I will post updates on that process. For the time being, the default theme is fine, and the only real complaint is that it is just so generic and GitHub-y in style. Not really a problem. Maybe if Jekyll gets tedious I will switch to the Golang-based [Hugo](https://gohugo.io/) static-site framework/generator. Maybe. Or maybe [MetalSmith](https://metalsmith.io/) which seems to be a new NodeJS-based option.

Out of the box, there were some deprecation warnings on running `jekyll serve`, such as:

```
Deprecation Warning: lighten() is deprecated.
```

This is coming from some outdated SASS styling syntax in the default _Minima_ theme. Run this to discover the source code:

```
$ code $(bundle show minima)
```

Another snag, important for tech-blogging, is the _Liquid_ templating engine used by Jekyll. I noticed that in the token code-snippet shown above, the code whitespace was not being preserved, which was not acceptable. I had been using the default Liquid `highlight ruby` syntax, but this was not intuitively building `code` or `pre` markup, without bugs, or working with the auto-save and Markdown parsers in VSCode to preserve the indentation. I did some more troubleshooting <sup>[1](https://stackoverflow.com/a/21105616/21928926)</sup> and abandoned the Liquid syntax, and just used the intuitive triple-backtick Markdown syntax <code>```ruby</code>. This seemed to work, and even worked on deployment to Github Pages, without any extra configuration, as mentioned by the helpful Stack Overflow posters. Apparently this is something else that has changed in more recent versions.

So thanks for reading, and check back in, as I will be re-posting old pieces and adding new ones. 👋
