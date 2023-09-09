---
layout: post
title:  "Stopping CNN's autoplaying videos with a Chrome extension"
date: 2017-11-15 20:20:00 -0500
description: "I'm sick of CNN's autoplay videos, and I'd like to learn how to write a Chrome extension"
tags:
- chrome
- javascript
- programming
---
Even though I have uninstalled Flash and checked all the appropriate settings in Chrome, CNN still seems to be able to get its videos to start playing whether I want them to or not (I don't). And because of this, I decided I'd like to give building a Chrome extension a try in order to stop those videos from autoplaying.

I realize there may be other Chrome extensions out there (like [Disable HTML5 Autoplay](https://chrome.google.com/webstore/detail/disable-html5-autoplay/efdhoaajjjgckpbkoglidkeendpkolai?hl=en-US)), but I want something very simple and something I can update as needed. I don't want to disable videos across the whole web. For the moment, I just want to focus on CNN, and _maybe_ BBC News.

Luckily, I don't need to start completely from scratch, because I found an extension that does something very similar: [Make Medium Readable Again](https://github.com/thebaer/MMRA "The GitHub repo for the MMRA extension"). This extension removes some annoyances from [Medium](https://medium.com/ "The Medium site") articles, which is very much in line with what I'll be trying to achieve in my own extension. I'll try not to reuse any of its code, but instead just the ideas of how it accomplishes hiding elements on a page.

# Creating the necessary files

For the simplest script, it appears that we'll need just a couple of files to make it work. First, is a `manifest.json` file that stores the configuration for the extension, and second, a script file (named `content.js`) that will do the work of hiding the videos.

This looks to be the least amount of info needed for the `manifest.json` file:

```json
{
  "manifest_version": 2,
  "name": "Simple Video Autoplay Stopper",
  "description": "A very minimal extension to stop autoplaying videos on CNN",
  "version": "1.0",
  "permissions": [
    "tabs",
    "*://*.cnn.com/*"
  ],
  "content_scripts": [
    {
      "matches": [
        "*://*.cnn.com/*"
      ],
      "js": ["content.js"]
    }
  ]
}
```

And the script that does all the work, `content.js`. Just to make sure it'll work on CNN pages, I'll just have it print something to the dev console.

```javascript
console.log('Success! You are on a CNN page.');
```

# Install your simple extension

Now, with those two files in a directory on their own, you can add them in Chrome. On the Extensions page (via Windows menu, then choose Extensions), you will need to check the "Developer mode" checkbox, and then click the "Load unpacked extension..." button. From there choose the directory with those files, and then you should see something like this:

![Screenshot of extensions page with new extension's details](https://c1.staticflickr.com/5/4527/37715490044_5505310c8a_c.jpg)

Once installed, open up the Chrome dev console and if you go to CNN's website (any page), you should see the message above (possibly hard to find amongst a whole bunch of other messages):

![Screenshot of dev console log showing our success message](https://c1.staticflickr.com/5/4524/38398814842_723259448d_c.jpg)

And if you visit any other non-CNN website, you should _not_ see that message in the dev console.

# Let's block some video

The CNN HTML and CSS code isn't the greatest, but they do seem to follow some common patterns when it comes to their video players. It appears that many of the video players on the CNN site have a `<div>` with a class name of `media__video`, which looks something like this:

```html
<div class="js-media__video media__video" id="media__video_large-media_0--wrapper">
  ... evil video player ...
</div>
```

My goal at this point is to just wipe the video from the page completely. I don't want to temporarily disable it and provide a link to play it. And I don't want to replace the video with a "blocked" icon, or something like that. There has yet to be a time that I actually wanted to watch the CNN videos. I just want it gone.

So, let's just wipe the video player off the page completely. Here's just about the least amount of code needed to accomplish deleting the videos:

```javascript
// in case there are multiple videos, let's find all of 'em
const playerWrappers = container.querySelectorAll('.media__video');

// now, delete them from the DOM
playerWrappers.forEach(player => {
  player.parentNode.removeChild(player);
});

```

# Let's give it a try

Once you've installed an "unpacked extension" from local code, you can always update the extension by clicking the "Reload" link - shown in the first screenshot above with the Apple symbol next to it (it will look different depending on your OS).

Now if we visit a CNN page with video we should no longer see the video at the top of the article. Here's what it looks like with the extenstion disabled (or not installed):

![Screenshot of the CNN page with the video present](https://c1.staticflickr.com/5/4536/38418137892_aa206fbf5c_c.jpg)

And here's what it looks like with it enabled:

![Screenshot of the CNN page with the video completely removed](https://c1.staticflickr.com/5/4560/37563010225_9958211c6d_c.jpg)

Awesome.

# Clean it up a little bit

If you notice in the first screenshot included in this post, instead of a nice icon for the extension you'll see the puzzle piece logo instead. To remedy this, I created a super simple (and very ugly) image for the extension (named `icon.png`), and then added a reference to it in the `manifest.json`:

```json
{
  "manifest_version": 2,
  "name": "Simple Video Autoplay Stopper",
  "description": "A very minimal extension to stop autoplaying videos on CNN",
  "version": "1.0",
  "permissions": [
    "tabs",
    "*://*.cnn.com/*"
  ],
  "content_scripts": [
    {
      "matches": [
        "*://*.cnn.com/*"
      ],
      "js": ["content.js"]
    }
  ],
  "icons": {
    "128": "icon.png"
  }
}
```

# Success!

It's nice to see how little code it takes to get rid of those annoying autoplaying videos. When I do visit CNN, it's usually to get a quick impression of a story, and not any in-depth reporting. But when there are videos starting without me wanting them to, I want to check the CNN site less and less. But now, I have an easy solution.

It's likely there are cases that I will miss, due to different HTML and CSS identifiers surrouding the video. If so, I'll likely come back and update the extension to make sure those videos are removed as well. Even if I do, it's nice to know it isn't all that difficult to do.

# Where to find the source code

The code for this simple extension can be found here on GitHub: [simple-video-autoplay-stopper](https://github.com/billturner/simple-video-autoplay-stopper "The GitHub page for this extension"). If you'd like to fork it and add your own features or block videos from other sites, feel free. I think I'm going to keep it as simple as possible for myself.

# Some links that were helpful

* [Make Medium Readable Again](https://github.com/thebaer/MMRA) extension source
* [Content Scripts](https://developer.chrome.com/extensions/content_scripts) page from official extension docs
* [How to Create a Chrome Extension in 10 Minutes Flat](https://www.sitepoint.com/create-chrome-extension-10-minutes-flat/)

# Finally

I may revisit this sometime soon and add support for blocking video on other sites, and I may actually make it an option to play the disabled video. If so, I'll make a new post explaining the modifications I made. But for the time being, this works just as I had hoped!
