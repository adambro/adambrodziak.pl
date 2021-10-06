## Wasted 10 years with Bash

*Originally I  [published this 3 years ago on Medium](https://adambrodziak.medium.com/wasted-10-years-with-bash-a7f6cb480419), but keeping a copy here too.*

When you're thinking of shell on any Linux system you probably think Bash (Bourne Again Shell). On Windows too actually, as Git Bash is quite popular (despite it has got an ugly terminal emulator) and WSL (Windows Subsystem for Linux) uses Bash too by default. At least those have quite modern 4.x versions of Bash, while MacOS X ships an outdated 3.2 version and there are tutorials how to [upgrade Bash on Mac to 5.0](https://itnext.io/upgrading-bash-on-macos-7138bd1066ba) which was released days ago.

The Bash 5.0 upgrade on Mac looks quite complicated and can yield unexpected errors, so that begs the question: why not use better shell instead?


## My history with Bash

My story with Linux started in 2007 and for 10 straight years I've been using Bash by default, as it came with Ubuntu. It's not *that bad*, as Ubuntu at least ships autocomplete configuration by default, so you can use Tab to complete command or it's params. Just try to type `cd /ho<Tab>` to see it in action. I've learned that very qickly and blessed Ubuntu for [making it work for me](https://www.tecmint.com/install-and-enable-bash-auto-completion-in-centos-rhel/).

Over the years I've learned [some useful things](https://zwischenzugs.com/2018/01/06/ten-things-i-wish-id-known-about-bash/), like Ctrl+R to search history or Ctrl+K to clear line. I did not know any better (cnconscious incompetence), becuase for me shell==Bash. Even after finding that there are other shells and relying on [Bash quirks in scripts is bad practice](https://linux.die.net/man/1/checkbashisms) I haven't looked for greener pastures. Somehow it did not even occur to me to alter Bash stupid defaults (case sensitive file completion - why?!), besides maybe increasing history buffer (huge productivity boost BTW). There is [sensible Bash configuration](https://github.com/mrzool/bash-sensible) that you should definately check out!


## Fish - The user-friendly command line shell

It was late 2017 when I stubled upon [Fish Shell](https://fishshell.com/). The tipping point was aptly named [The fish shell is awesome](https://jvns.ca/blog/2017/04/23/the-fish-shell-is-awesome/) post by Julia Evans. Then I've watched few [videos](https://www.youtube.com/watch?v=g_HoW4iek2Q) and was totally hooked. Fish comes with all bells and wistles out of the box, setup for you. No need to configure anything, install plugins, etc.

As you can see Fish (on the right) has got [the best tab-complation](https://www.youtube.com/watch?v=0NOAUogSSMo) compared to ZSH and Bash.

Of course Fish community created [plugins for more advanced features](https://github.com/jorgebucaran/awesome-fish) - I can recommend the following:
- [rafaelrinaldi/**pure**](https://github.com/rafaelrinaldi/pure) - Pure-fish port of [sindresorhus/pure](https://github.com/sindresorhus/pure) prompt
- [franciscolourenco/**done**](https://github.com/franciscolourenco/done) - Automatically receive notifications when a long process finish
- [jethrokuan/**z**](https://github.com/jethrokuan/z) - Pure-fish [rupa/z](https://github.com/rupa/z)-like directory jumping

Notice those plugins are *Pure-fish* implementations of 3rd party scripts. The reason is: Fish ofers sane scripting ([unlike other shells](https://twitter.com/fishpkg/status/1087159872561414145) some say), but it's not POSIX compliant. This is probably the biggest drawback (or downside) of using Fish. Over the years I've accumulated oneliners, scripts and habits from Bash (i.e. using `&&` to join commands), but most were POSIX-compliant.  Which led me to ZSH...

Note: [Fish 3.0.0](https://github.com/fish-shell/fish-shell/releases/tag/3.0.0) released month ago have added `&&` support. It is that important :)


## ZSH (Z shell) - designed for interactive use

ZSH has been recommended to me by my collegues at work as better Bash alternative, becuase it is POSIX-compliant. That means my Bash habits still work: `&&` to join commands, way of exporting ENV variables, storing command output in var using backticks, etc.

Since I've been using Fish before I've started to look for plugins that replicate features that Fish provides out of the box. To get some of the Fish coolness install the following:
* [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) - Fish-like fast/unobtrusive autosuggestions for zsh.
* [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - Fish shell-like syntax highlighting for Zsh.
* [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) - This is a clean-room implementation of the Fish shell's history search feature

There's plenty of [awesome ZSH plugins](https://github.com/unixorn/awesome-zsh-plugins), but I tend use only few, not to overload [my .zshrc](https://github.com/adambro/dotfiles/blob/master/home/.zshrc) file. The reason: it is on me to make sure everything works well. I'm using [Antigen plugin manager](http://antigen.sharats.me/) that is supposedly solving plugins installation issues (see motivation section), but problems still happen. To be honest I have never tried installing [oh-my-zsh](https://ohmyz.sh/) directly, because it does not ship Fish-like plugins listed above and [configuring custom plugins in OMZ is awful](https://joshldavis.com/2014/07/26/oh-my-zsh-is-a-disease-antigen-is-the-vaccine/).

## Bash as a scripts runtime

OK, so we've established that there are better interactive shells than Bash, but Bash is still useful for scripts. It's becuase of it's ubiqoutness obviously - it's available on (almost) every Linux/Unix system those days. However Bash has it's gotachas as scripting language, so beware. Here are few links to make your life easier with Bash scripts:
* http://redsymbol.net/articles/unofficial-bash-strict-mode/
* https://zwischenzugs.com/2018/01/06/ten-things-i-wish-id-known-about-bash/
* https://github.com/dylanaraps/pure-bash-bible

## Partying words

If you're working with shell do yourself a favour and try something more modern than Bash. Do not waste a decade as I did. My suggestions are as follows:
* Try Fish as it's awesome out of the box. If you know something even cooler - let me know!
* If you like to tinker or need POSIX shell give ZSH a go. No idea which plugin manager to recommend though ;)
* Use [Bash sensible config](https://github.com/mrzool/bash-sensible) if you really need Bash, i.e. on remote host you do not control.

Let me know if I missed anything or there's even better shell that I'm not aware of. Basically rising awareness of Bash alternatives is my point here, so I'm open to learn.