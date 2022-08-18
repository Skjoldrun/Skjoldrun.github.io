---
layout: page
title: GitHub - static page for knowledge base
parent: Other
---

# Introduction

With [GitHub Pages](https://pages.github.com/) you can create your own free hosted static website with little effort and interesting tooling, such as [jekyll](https://jekyllrb.com/). All you need is a GitHub account, a dedicated repo for the site, some editor like [VS Code](https://code.visualstudio.com/), a git client and maybe some theme pack to add a nice look and functionality to your site.

You can write markdown files or html files for articles and create your own blog, product website or whatever you want to display. The neat part is, it's completely free and your site is hosted on the GitHub CDN, so its super fast from everywhere on the globe. 
This knowledge base is my first site and playground to explore the functions and tools.



# Setup

For the basic site create a repo with a special naming for GitHub to know that this is a Pages site: username.github.io. This repo will contain all config files and articles of your site. If you want to generate a jekyll based site, i can recommend to read the documentation on [jekyll](https://jekyllrb.com/) and get in touch with it. But you can also copy someone's repo file (like mine 🙂) and customize everything, step by step until you like what you see.

The jekyll base config is the `_config.yml` file. It contains title, name, email, baseurl, theme pack and many more things to be configured. 

Cool thing is that you can even use remote themes, which will not be downloaded to your repo, but will be loaded by jekyll in the build pipeline and then being hosted and activated on your live website. I have configured it with `remote_theme: just-the-docs/just-the-docs` in my config file und have set some theme options in it under the section 'just-the-docs configuration'.



# Theming and Customization

I use a theme, called [Just the Docs](https://github.com/just-the-docs/just-the-docs) for this site and have customized it a little bit for my needs. 

AS remote theme it gets loaded and built within the build pipe and I have not to fiddle around with its theme source files for its basic display. Only if I want to change something about the theme, I can create the same folder structure and files to be overwritten on load. The theme files I create in my repo are then higher in rank and overwrite the base remote ones. You can customize basic things like CSS, scripts or even build a dark theme / light theme switch, like I did.


## custom theme switch

The original just-the-docs theme has no easy light / dark theme switch built in, so I decided to explore the theme structure and scripts to build this feature with my amateur HTML and JavaScript skills 😉.

I found that you can overwrite the original file with rebuilding the part of folder structure and filenames and put your custom code into the file. Furthermore, you can write a sort of plugin, which can be loaded with include statements in the HTML structure. The theme had basic light and dark CSS files and some basic script function to switch between them, but I wanted a button to be displayed on every page to enable the user to toggle at every time and from everywhere. 

![files](/assets/images/other/GitHubPages/editor-files.png)

So I decided to place a button into the header bar besides the aux links in the upper right corner. I built a `themeswitcher.html` file:

```html
<div>
    <button class="btn js-toggle-dark-mode">🌛</button>
    <script> 
        const toggleDarkMode = document.querySelector('.js-toggle-dark-mode'); 
        jtd.addEvent(toggleDarkMode, 'click', function() { 
            if (localStorage.getItem('theme') === 'dark') {
                jtd.setTheme('light');
                localStorage.setItem('theme', 'light');
                toggleDarkMode.textContent = "🌛";
            }
            else {
                jtd.setTheme('dark');
                localStorage.setItem('theme', 'dark');
                toggleDarkMode.textContent = "🌞";
            }
        }); 
    </script>
</div>
```

![browser local storage](/assets/images/other/GitHubPages/browser-local-storage.png)

This file has the button with the script functionality in it and can store the preferred setting of light or dark in the browser local storage. It also changes the button content with basic moon and sun emoticons as icons. With these basic icons, the button still functions on every OS and can display the size properly, even for mobile displays.

The switcher code gets included in the `_layouts\default.html` file in the main class div container and after the aux-nav nav-item:

```html
<div id="main-header" class="main-header">
    {% if site.search_enabled != false %}
    <div class="search">
        <!-- ... -->
    </div>
    {% endif %}
    {% include header_custom.html %}
    {% if site.aux_links %}
    <nav aria-label="Auxiliary" class="aux-nav">
        <!-- ... -->
    </nav>
    {% endif %}
    {% include themeswitcher.html %} <!-- themeswitcher include here -->
</div>
```

![themeswitcher button](/assets/images/other/GitHubPages/themeswitcher-button.png)

