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

[![files](/assets/images/articles/GitHubPages/editor-files.png)](/assets/images/articles/GitHubPages/editor-files.png)

So I decided to place a button into the header bar besides the aux links in the upper right corner. I built a `themeswitcher.html` file:

```html
<div>
    <button class="btn js-toggle-dark-mode">🌞</button>
    <script> 
        const toggleDarkMode = document.querySelector('.js-toggle-dark-mode'); 
        jtd.addEvent(toggleDarkMode, 'click', function() { 
            if (localStorage.getItem('theme') === 'dark') {
                jtd.setTheme('light');
                localStorage.setItem('theme', 'light');
                toggleDarkMode.textContent = "🌛";
            }
            else if (localStorage.getItem('theme') === 'light') {
                jtd.setTheme('dark');
                localStorage.setItem('theme', 'dark');
                toggleDarkMode.textContent = "🌞";
            }
            else {
                jtd.setTheme('light');
                localStorage.setItem('theme', 'light');
                toggleDarkMode.textContent = "🌛";
            }
        }); 
    </script>
</div>
```

[![browser local storage](/assets/images/articles/GitHubPages/browser-local-storage.png)](/assets/images/articles/GitHubPages/browser-local-storage.png)

This file has the button with the script functionality in it and can store the preferred setting of light or dark in the browser local storage. It also changes the button content with basic moon and sun emoticons as icons. With these basic icons, the button still functions on every OS and can display the size properly, even for mobile displays.
The switcher code gets included in the `_layouts\default.html` file in the main class div container and after the aux-nav nav-item.

[![themeswitcher button](/assets/images/articles/GitHubPages/themeswitcher-button.png)](/assets/images/articles/GitHubPages/themeswitcher-button.png)


## Lightbox Integration

Lightbox is a JavaScript and css implementation to show linked images in a loaded lightbox, wich is like a overlay box to open the image in front and in focus of the site. The used lightbox JS Code is listed down below (Source: [jekyllcodex.org](https://jekyllcodex.org/without-plugin/lightbox/#)). To use this with the just-the-docs Jekyll theme, the JS and the CSS should be placed in the assets folder in each a JS folder and a css folder. These are loaded on top of the remote loaded just-the-docs theme folders and the new files are deployed besides the originals. You have to link the JS and CSS in the footer_custom.html file for just-the-docs to load them:

[**_includes/footer_custom.html**](/_includes/footer_custom.html)
```html
<link href="/assets/css/lightbox.css" rel="stylesheet" />
<script src="/assets/js/lightbox.js"></script>
```

After that you have to activate each image you wnat to be clickable with surrounding it with a md link to itself:

```md
![image alt text](/path to image)

to 

[![image alt text](/path to image)](/path to image)
```

Try the difference:

**Without lightbox**
![No Lightbox](/assets/images/Logo-BooksBrain.png)

**With lightbox**
[![No Lightbox](/assets/images/Logo-BooksBrain.png)](/assets/images/Logo-BooksBrain.png)

The JS script now enriches all the activated image links to be loaded with the lightbox.

The script and css were taken from the [Jekyll Codex](https://jekyllcodex.org/without-plugin/lightbox/):

**[lightbox.js](https://github.com/Skjoldrun/Skjoldrun.github.io/blob/main/assets/js/lightbox.js)**
```javascript
function is_youtubelink(url) {
  var p = /^(?:https?:\/\/)?(?:www\.)?(?:youtu\.be\/|youtube\.com\/(?:embed\/|v\/|watch\?v=|watch\?.+&v=))((\w|-){11})(?:\S+)?$/;
  return (url.match(p)) ? RegExp.$1 : false;
}
function is_imagelink(url) {
  var p = /([a-z\-_0-9\/\:\.]*\.(jpg|jpeg|png|gif))/i;
  return (url.match(p)) ? true : false;
}
function is_vimeolink(url, el) {
  var id = false;
  var xmlhttp = new XMLHttpRequest();
  xmlhttp.onreadystatechange = function () {
    if (xmlhttp.readyState == XMLHttpRequest.DONE) {   // XMLHttpRequest.DONE == 4
      if (xmlhttp.status == 200) {
        var response = JSON.parse(xmlhttp.responseText);
        id = response.video_id;
        console.log(id);
        el.classList.add('lightbox-vimeo');
        el.setAttribute('data-id', id);

        el.addEventListener("click", function (event) {
          event.preventDefault();
          document.getElementById('lightbox').innerHTML = '<a id="close"></a><a id="next">&rsaquo;</a><a id="prev">&lsaquo;</a><div class="videoWrapperContainer"><div class="videoWrapper"><iframe src="https://player.vimeo.com/video/' + el.getAttribute('data-id') + '/?autoplay=1&byline=0&title=0&portrait=0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe></div></div>';
          document.getElementById('lightbox').style.display = 'block';

          setGallery(this);
        });
      }
      else if (xmlhttp.status == 400) {
        alert('There was an error 400');
      }
      else {
        alert('something else other than 200 was returned');
      }
    }
  };
  xmlhttp.open("GET", 'https://vimeo.com/api/oembed.json?url=' + url, true);
  xmlhttp.send();
}
function setGallery(el) {
  var elements = document.body.querySelectorAll(".gallery");
  elements.forEach(element => {
    element.classList.remove('gallery');
  });
  if (el.closest('ul, p')) {
    var link_elements = el.closest('ul, p').querySelectorAll("a[class*='lightbox-']");
    link_elements.forEach(link_element => {
      link_element.classList.remove('current');
    });
    link_elements.forEach(link_element => {
      if (el.getAttribute('href') == link_element.getAttribute('href')) {
        link_element.classList.add('current');
      }
    });
    if (link_elements.length > 1) {
      document.getElementById('lightbox').classList.add('gallery');
      link_elements.forEach(link_element => {
        link_element.classList.add('gallery');
      });
    }
    var currentkey;
    var gallery_elements = document.querySelectorAll('a.gallery');
    Object.keys(gallery_elements).forEach(function (k) {
      if (gallery_elements[k].classList.contains('current')) currentkey = k;
    });
    if (currentkey == (gallery_elements.length - 1)) var nextkey = 0;
    else var nextkey = parseInt(currentkey) + 1;
    if (currentkey == 0) var prevkey = parseInt(gallery_elements.length - 1);
    else var prevkey = parseInt(currentkey) - 1;
    document.getElementById('next').addEventListener("click", function () {
      gallery_elements[nextkey].click();
    });
    document.getElementById('prev').addEventListener("click", function () {
      gallery_elements[prevkey].click();
    });
  }
}

document.addEventListener("DOMContentLoaded", function () {

  //create lightbox div in the footer
  var newdiv = document.createElement("div");
  newdiv.setAttribute('id', "lightbox");
  document.body.appendChild(newdiv);

  //add classes to links to be able to initiate lightboxes
  var elements = document.querySelectorAll('a');
  elements.forEach(element => {
    var url = element.getAttribute('href');
    if (url) {
      if (url.indexOf('vimeo') !== -1 && !element.classList.contains('no-lightbox')) {
        is_vimeolink(url, element);
      }
      if (is_youtubelink(url) && !element.classList.contains('no-lightbox')) {
        element.classList.add('lightbox-youtube');
        element.setAttribute('data-id', is_youtubelink(url));
      }
      if (is_imagelink(url) && !element.classList.contains('no-lightbox')) {
        element.classList.add('lightbox-image');
        var href = element.getAttribute('href');
        var filename = href.split('/').pop();
        var split = filename.split(".");
        var name = split[0];
        element.setAttribute('title', name);
      }
    }
  });

  //remove the clicked lightbox
  document.getElementById('lightbox').addEventListener("click", function (event) {
    if (event.target.id != 'next' && event.target.id != 'prev') {
      this.innerHTML = '';
      document.getElementById('lightbox').style.display = 'none';
    }
  });

  //add the youtube lightbox on click
  var elements = document.querySelectorAll('a.lightbox-youtube');
  elements.forEach(element => {
    element.addEventListener("click", function (event) {
      event.preventDefault();
      document.getElementById('lightbox').innerHTML = '<a id="close"></a><a id="next">&rsaquo;</a><a id="prev">&lsaquo;</a><div class="videoWrapperContainer"><div class="videoWrapper"><iframe src="https://www.youtube.com/embed/' + this.getAttribute('data-id') + '?autoplay=1&showinfo=0&rel=0"></iframe></div>';
      document.getElementById('lightbox').style.display = 'block';

      setGallery(this);
    });
  });

  //add the image lightbox on click
  var elements = document.querySelectorAll('a.lightbox-image');
  elements.forEach(element => {
    element.addEventListener("click", function (event) {
      event.preventDefault();
      document.getElementById('lightbox').innerHTML = '<a id="close"></a><a id="next">&rsaquo;</a><a id="prev">&lsaquo;</a><div class="img" style="background: url(\'' + this.getAttribute('href') + '\') center center / contain no-repeat;" title="' + this.getAttribute('title') + '" ><img src="' + this.getAttribute('href') + '" alt="' + this.getAttribute('title') + '" /></div><span>' + this.getAttribute('title') + '</span>';
      document.getElementById('lightbox').style.display = 'block';

      setGallery(this);
    });
  });

});
```

**[lightbox.css](https://github.com/Skjoldrun/Skjoldrun.github.io/blob/main/assets/css/lightbox.css)**
```css
#lightbox {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.85);
  z-index: 9999999;
  line-height: 0;
  cursor: pointer;
  display: none;
}

#lightbox .img {
  position: relative;
  top: 50%;
  left: 50%;
  -ms-transform: translateX(-50%) translateY(-50%);
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  max-width: 100%;
  max-height: 100%;
}

#lightbox .img img {
  opacity: 0;
  pointer-events: none;
  width: auto;
}

@media screen and (min-width: 1200px) {
  #lightbox .img {
    max-width: 1200px;
  }
}

@media screen and (min-height: 1200px) {
  #lightbox .img {
    max-height: 1200px;
  }
}

#lightbox span {
  display: block;
  position: fixed;
  bottom: 13px;
  height: 1.5em;
  line-height: 1.4em;
  width: 100%;
  text-align: center;
  color: white;
  text-shadow:
    -1px -1px 0 #000,
    1px -1px 0 #000,
    -1px 1px 0 #000,
    1px 1px 0 #000;
}

#lightbox span {
  display: none;
}

#lightbox .videoWrapperContainer {
  position: relative;
  top: 50%;
  left: 50%;
  -ms-transform: translateX(-50%) translateY(-50%);
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  max-width: 900px;
  max-height: 100%;
}

#lightbox .videoWrapperContainer .videoWrapper {
  height: 0;
  line-height: 0;
  margin: 0;
  padding: 0;
  position: relative;
  padding-bottom: 56.333%;
  /* custom */
  background: black;
}

#lightbox .videoWrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 0;
  display: block;
}

#lightbox #prev,
#lightbox #next {
  height: 50px;
  line-height: 36px;
  display: none;
  margin-top: -25px;
  position: fixed;
  top: 50%;
  padding: 0 15px;
  cursor: pointer;
  text-decoration: none;
  z-index: 99;
  color: white;
  font-size: 60px;
}

#lightbox.gallery #prev,
#lightbox.gallery #next {
  display: block;
}

#lightbox #prev {
  left: 0;
}

#lightbox #next {
  right: 0;
}

#lightbox #close {
  height: 50px;
  width: 50px;
  position: fixed;
  cursor: pointer;
  text-decoration: none;
  z-index: 99;
  right: 0;
  top: 0;
}

#lightbox #close:after,
#lightbox #close:before {
  position: absolute;
  margin-top: 22px;
  margin-left: 14px;
  content: "";
  height: 3px;
  background: white;
  width: 23px;
  -webkit-transform-origin: 50% 50%;
  -moz-transform-origin: 50% 50%;
  -o-transform-origin: 50% 50%;
  transform-origin: 50% 50%;
  /* Safari */
  -webkit-transform: rotate(-45deg);
  /* Firefox */
  -moz-transform: rotate(-45deg);
  /* IE */
  -ms-transform: rotate(-45deg);
  /* Opera */
  -o-transform: rotate(-45deg);
}

#lightbox #close:after {
  /* Safari */
  -webkit-transform: rotate(45deg);
  /* Firefox */
  -moz-transform: rotate(45deg);
  /* IE */
  -ms-transform: rotate(45deg);
  /* Opera */
  -o-transform: rotate(45deg);
}

#lightbox,
#lightbox * {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
```