---
layout: single
title:  ""
date:   2021-07-27 23:52:55 +0530
tags: ruby gems jekyll linux manjaro arch
---

# Getting up to speed with Ruby 2.7 and Jekyll on Manjaro

I've recently switched from the `Debian` verse (Ubuntu, specifically) to the world of Arch Linux based distribution, Manjaro. It has been a fun roller-coaster ride so far and still learning something new every day while doing development on this system. 

In this blog, I am going to share the experience of building and running my `Jekyll` based blog site(yes, the one you are reading this blog post on right now 😉)

At first, I installed `ruby` and `jekyll` on `Manjaro` using

```bash
yay -S ruby
gem install jekyll bundler
```

and tried to build the website.

```bash
bundle install && bundle exec jekyll serve
```

The command failed with the error message specified below-

```bash
# Your user account isn't allowed to install to the system RubyGems.
# You can cancel this installation and run:

bundle config set --local path 'vendor/bundle'
bundle install
```

As expected, I ran the suggested commands and everything went smooth.

Now comes the time to build the website again and run the server. This time I encountered another error stating-

```bash
jekyll 3.9.0 | Error:  no implicit conversion of Hash into Integer
/home/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/pathutil-0.16.2/lib/pathutil.rb:502:in `read': no implicit conversion of Hash into Integer (TypeError)
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/pathutil-0.16.2/lib/pathutil.rb:502:in `read'
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/jekyll-3.9.0/lib/jekyll/utils/platforms.rb:75:in `proc_version'
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/jekyll-3.9.0/lib/jekyll/utils/platforms.rb:40:in `bash_on_windows?'
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/build.rb:77:in `watch'
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/build.rb:43:in `process'
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:93:in `block in start'
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:93:in `each'
	from /home/rishabh/dev/rishabh96b.github.io/vendor/bundle/ruby/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:93:in `start'
	from /home/rishabh/dev/rishabh96b.github.io/vendor
```

After some research, I found out that the `Github-Pages` uses Jekyll `3.9`, which isn’t compatible with Ruby 3 and downgrading to Ruby 2.7 should solve this problem.

I removed `ruby` binaries from the system and searched to install the specific version of a package using `pacman` . This surprisingly led me to the conclusion that specifying a version while installing a package is not easy in arch-based systems as it is with `apt` package manager in `Debian` based distributions. So we cannot do something like `sudo pacman -S ruby=2.7` in Manjaro.

Solution? There are "time machine" repositories out there where people just store old packages. We just need to find those and install them using `pacman` 

I found out that Ruby `v2.7` is available as package `ruby2.7` and installed it along with the `bundler`

```bash
yay -S ruby2.7 ruby2.6-bundler
```

There was another caveat with this as checking the ruby installation with `ruby -v` and `gem -v` and hooray, it did not work. Instead, the binaries are named as `ruby2.7` and `gem2.7`. The workaround is to alias it to `ruby` and `gem` respectively.

```bash
alias ruby=ruby2.7
alias gem=gem2.7
```

It is time to install `jekyll` again as I removed it along with the latest `ruby` binary with the command `gem install jekyll bundler` and got slapped with permission error this time.

```bash
Permission denied @ rb_sysopen - /usr/lib/ruby/gems/2.7.0/cache/eventmachine-1.2.7.gem (Errno::EACCES)
```

As it is not recommended to install gems(`gem install`) as `root` , I turned to the `sudo` for my solution to this problem. I could have `chown`ed `/usr/bin` directory but chowning a `root` user owned directory is not recommended as it does not have any potential benefits and can break your system. As you might have guessed already, running the above command with `sudo` was successful.

```bash
sudo gem install jekyll bundler
```

And then I attempted to run the website server again and this time the service ran smoothly.