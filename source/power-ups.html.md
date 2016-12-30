---
title: Trello Power-Ups Reference

language_tabs:
  - javascript

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---
# Power-Ups Introduction

Power-Ups are powerful independent modules that allow developers to modify the Trello experience while still maintaining member control. Power-Ups represent the ability to enrich the experience of members by modifying and improving their interactions with Boards and Cards.

Our current set of Power-Ups has some great examples of what the platform can do. Learn more about our GitHub Power-Up, or our Salesforce Power-Up.

Power-Ups gain read-only access to member information as members interact with the Power-Up. 

Power-Ups can make Attachments to Cards and can store Power-Up specific data, but are restricted from making other changes.

### Looking for more help or clarifications?

Already got access, have read the documentation, and are looking for more help or clarification? We are trying out holding office hours for Power-Up development over video chat. To sign up for an office hours appointment, please select from the available slots on [this calendar](#).

# Sample Projects

Trello has built two sample Power-Ups that you can use to get started developing right away. These samples demonstrate the capabilities of the platform and should help you to get started.

[Remix the Gomix Sample Project]() or Clone the [GitHub Pages Sample]()

# Getting Started

Interested in building your own Power-Up? 
1. Clone or download our samples above. Or get started by building your own manifest and index connector - find out how, [here](#manifest.json).
2. Make sure your project is hosted via HTTPS.
3. Hook up your Power-Up to Trello for Local Development or Team Development.

## Local Development Only

Visit [https://trello.com/power-up-preview/](https://trello.com/power-up-preview/) and enter an HTTPS-based manifest URL. This will allow you to see and enable your Power-Up only for your current browser and member. Other members of your team won't be able to see your Power-Up, and you will see your Power-Up on all boards belonging to Teams that you have access to.

## Team Development

You can start developing a Power-Up for your Team right now by creating a new Power-Up. Click below to get started. We'll need your Power-Up's Name and an HTTPS based manifest to get you setup.

[Create a Power-Up](#)

# Architecture Overview

Fundamentally, Power-Ups are run client-side within the Trello experience. Power-Ups are registered within the Trello server, which makes them available to members. Each time a member looks at the list of available Power-Ups, we reload the manifest of your Power-Up and render this information to the member.
Once a user has chosen to add a Power-Up to a board, the Power-Up’s functionality is loaded via an iframe. The Trello client posts messages to your Power-Up’s iframe and expects any declared capabilities to be handled in a reasonable timeframe.

With each message that we post to your Power-Up, we will also give you access to a Trello object that allows you to interact with member content, and limited data storage.

The current Power-Ups architecture makes extensive use of callbacks and promises because the core Trello web architecture is heavily asynchronous. You will most often be defining methods that Trello will invoke at appropriate times, and you will need to supply and handle promises that are provided to you.

<aside class="notice">
All hosted assets must be hosted over HTTPS.
</aside>

Virtually everything about Power-Ups is configurable (outside of the manifest) in terms of filenames and structure.

## manifest.json

```javascript
// Example manifest.json
{
  "name": "Power-Up Name",
  "details": "The [Power-Up Name](https://example.org) is awesome because it can...",
  "icon": {
    "url": "./images/icon.png"
  },
  "author": "Your Name Here",
  "capabilities": [
    "board-buttons"
  ],
  "connectors": {
    "iframe": {
      "url": "./"
    }
  }
}
```

We load your Manifest each time a member tries to add or manage their Power-Ups, or every time your Power-Up is activated by a Member loading a board. Your manifest determines which capabilities we attempt to execute from your Power-Up.

We require name, details, icon, author, capabilities, and connectors parameters as shown in the example.

Your manifest location will be one of the things required as part of setting up your Power-Up. You will not be able to change your manifest location after it is set. Additionally, all files hosted on your servers must be served over HTTPS.

The URL referenced within connectors.iframe.url is relative to the path of your manifest.json file.

## Index Connector

```javascript
TrelloPowerUp.initialize({
  'board-buttons': function(t, options){
    return [{
    icon: './images/icon-white.svg',
    text: 'My Button',
    callback: function(t){
      // do something when the button is clicked
    }}];
  },
  'show-settings': function(t, options){
    return t.popup({
      title: 'Settings',
      url: './settings.html',
      height: 184
    });
  }
});
```

Once a user has added your Power-Up to one of their boards, each time the board is loaded, we will load your specified connector URL in an iframe. It is this page that we will be using to expose functionality, and to invoke capabilities.

Your index connector should use a `<script>` tag to include the Trello hosted Power-Up library.

Your index connector is also responsible for initializing your Power-Up and providing the callback functions for each of your supported capabilities.

## Additional iframes

Your Power-Up will often need to show additional iframes, such as when using `t.overlay()` or `t.popup()`. These pages should also include the Power-Up client library script in order to enable communications with Trello.

To initialize the Power-Up from an iframe other than your connector you should call:

`var t = TrelloPowerUp.iframe();`

After you have initialzed the Power-Up in your iframe, you can use all of the t helper methods such as `t.render()`, `t.sizeTo()`, `t.get()`, etc.

## Examined Power-Up - Appear.in

```javascript
var AppearIn = require('appearin-sdk');

var appearin = new AppearIn();
var roomUrl = function(path) {
  return "https://appear.in/trello.com/" + path;
}

// Initialize the Trello PowerUp with our capabilities
TrelloPowerUp.initialize({
  // Specify the board-buttons method to call, taking both
  // a “t” object that exposes Trello member information and
  // a reference to the board context for any potential buttons
  'board-buttons': function(t, board) {
    // We verify via the Appear.in SDK that the member is 
    // capable of using Appear.in. If not, show no button
    if(!appearin.isWebRtcCompatible()) {
      return [];
    }
    // We create a button by specifying that when the board 
    // details promise has been resolved, we return an object 
    // with the contents of the button once the board details 
    // have been loaded
    return t.board()
    .then(function(board){
      var url = roomUrl('b/' + board.id);
      return [
        {
          icon: './images/icon.svg',
          text: "Appear.in",
          url: url,
          target: url
        }
      ]
    })
  }
})
```

To understand the general architecture of Capabilities, we will use a sample Appear.in plugin to demonstrate one of the simplest Capabilities, “board-buttons”. This Capability adds a button to the top of the board.

## Examined Power-Up - Dropbox

```javascript
var api = require('src/api');
var dataForUrl = require('src/data-for-url');
var dropboxButton = require('src/dropbox-button');
var DropboxError = require('src/dropbox-error');
var DropboxUrls = require('src/dropbox-urls');
var xtend = require('xtend');

var Promise = Trello.Promise;

var ICON_SVG = './images/icon.svg';

var uniqueClaims = function(list) {
  var seen = {};
  return list.filter(function(entry){
    if(!seen[entry.url]) {
      seen[entry.url] = true;
      return true;
    }
    return false;
  });
}

TrelloPowerUp.initialize({
  'attachment-sections': function(t, options) {
    var entries = options.entries;

    var claimed = entries.filter(function(entry){
      return DropboxUrls.isFolder(entry.url);
    })

    if(claimed.length == 0) {
      return [];
    }

    return Promise.map(uniqueClaims(claimed), function(claim){
      var parsed = DropboxUrls.parse(claim.url);

      var eventualTitle = dataForUrl(t, claim.url, claim.name)
      .get('title')
      .then(function(title){
        if(title && !/^https:/.test(title)) {
          return title;
        } else {
          return "Dropbox Folder";
        }
      });

      return {
        claimed: [ claim ],
        icon: ICON_SVG,
        title: function(){
          return eventualTitle
        },
        content: {
          type: 'iframe',
          url: t.signUrl('./folder.html', { url: claim.url })
        }
      }
    })
  },
  'attachment-thumbnail': function(t, options) {
    return dataForUrl(t, options.url, options.name, { thumbnail: true })
    .then(function(data){
      data.url = options.url;
      data.openText = "Open in Dropbox";
      data.image = {
        url: data.thumbnail || ICON_SVG,
        logo: !data.thumbnail
      }
      if(data.uninitialized) {
        data.initialize = {
          type: 'iframe',
          url: t.signUrl(TrelloPowerUp.util.relativeUrl('authorize-link.html'))
        }
      }

      return data;
    })
    .catch(DropboxError, function(){
      throw t.NotHandled();
    })
  },
  'format-url': function(t, options) {
    return dataForUrl(t, options.url, options.name)
    .then(function(data){
      return {
        icon: ICON_SVG,
        text: data.title
      }
    })
    .catch(DropboxError, function(){
      throw t.NotHandled();
    })
  },
  'card-buttons': function(t) {
    return [{
      icon: ICON_SVG,
      text: 'Dropbox',
      callback: dropboxButton
    }]
  }
});
```

Dropbox is a more complex Power-Up. Our Dropbox Power-Up takes advantage of attachment-sections, attachment-thumbnail, format-url, and card-buttons.

# Capabilities

There are currently 9 capabilities. Capabilities represent areas of the UX that you can hook into. For each capability you state that you support in your manifest, the web application will make a request to your Power-Up for UX and functionality to be rendered within Trello, at the time when that UX item would normally be rendered.

Capability | Description
---------- | -----------
[board-buttons](#buttons) | Allows you to add one or more buttons to a Board at the top right of the screen
card-badges | Allows you to add badges to the front of cards. Card badges can also be refreshed on an interval that you specify
card-detail-badges | Allows you to add badges on the back of cards Card detail badges can also be refreshed on an interval that you specify
attachment-sections | Allows you to add sections to the back of a card, each containing an iframe. You will need to iterate through a list of Card attachments stored in options.entries. Return list of attachments that you are claiming per section
attachment-thumbnail | Allows you to improve the attachment preview on the back of a card, for when you don't need a whole iframe. Specify the link, the text shown for “opening” the attachment, and the thumbnail
card-buttons | Allows you to add one or more buttons on the back of cards. Each button should have `{ icon, text, callback -> method }`
format-url | Allows you to modify the rendering of a URL any time we would display a URL such as in a Card comment. Rendered in the sidebar, in markdown, and anywhere else URLs are shown. Make URLs nicer and easier to understand by replacing the raw URL with an icon and text
card-from-url |  Allows you to use information that you know (via authentication or algorithmically), to handle drag and dropped URLs. Set the name and description of the card based on the URL being dropped
show-settings | Allows you to have a gear button in the menu bar for configuring your Power-Up. Useful if you are setting Board-level attributes such as which GitHub repository is on a Board

## Buttons

IPhone ennui pug master cleanse cred cardigan. Jean shorts XOXO freegan thundercats, tofu tousled helvetica tumblr you probably haven't heard of them vice mumblecore. Lo-fi tattooed ennui la croix. Swag jianbing cronut cred subway tile, vaporware messenger bag hashtag pour-over mixtape. Twee synth vice kombucha. Before they sold out master cleanse cornhole crucifix twee. 

Fields | Description
------ | -----------
icon | Path to the icon to display on the button
text | Text to show on the button
url (optional) | The URL that a user should be taken to upon clicking the button
target (optional) | The target for the a tag (learn about targets)
callback (optional) | A method to call onclick.

Please note that callbacks require that you state “callback” as one of your capabilities in your manifest.json file.

### board-buttons

```javascript
// Static example

'board-buttons': function(t, board) {
  return [
    {
      icon: './images/icon.svg',
      text: "Button Text",
      url: 'https://developers.trello.com/'
    }
  ];
}
```

90's jean shorts flannel, poke master cleanse subway tile normcore man braid fashion axe selfies. Bitters pug wayfarers authentic actually. Woke marfa tumeric tote bag, typewriter lumbersexual raw denim VHS salvia deep v. Food truck gentrify pour-over, hashtag authentic 90's air plant venmo fap paleo. 

<img src="https://www.fillmurray.com/400/400">

Vexillologist neutra jianbing, polaroid synth hexagon kinfolk knausgaard locavore. Vaporware meggings tumblr, distillery cronut cray brunch green juice af copper mug. Humblebrag tacos thundercats, poutine sartorial venmo normcore disrupt irony tattooed drinking vinegar blog banjo chicharrones crucifix.

### card-buttons

```javascript
// Callback Example

'card-buttons': function(t, options){
  return [{
    icon: './images/icon.svg',
    text: 'Button Text',
    callback: function(t){
      return t.popup({
        title: "Card Button Popup",
        url: './card-button-popup.html'
      });
    }
  }];
}
```

Chillwave af bitters etsy, readymade farm-to-table kinfolk. VHS listicle ethical vegan, 3 wolf moon lo-fi waistcoat yr cray pour-over pork belly. Pok pok ennui hell of subway tile paleo small batch. Irony flexitarian paleo tacos. Pok pok YOLO kickstarter, woke direct trade austin hot chicken selfies crucifix bitters semiotics gochujang. 

<img src="https://www.fillmurray.com/500/400">

Pork belly freegan semiotics, salvia vegan dreamcatcher glossier seitan heirloom blue bottle kombucha banjo aesthetic. Paleo gastropub everyday carry meditation meggings disrupt.
You are expected to return a list of of buttons.
Static URL Example


## Badges

```javascript
// Static Example

'card-badges': function(t, card) {
  return [
    {
      icon: './images/icon.svg',
      text: '3'
    }
  ];
}
```

```javascript
// Dynamic Example
'card-badges': function(t, card) {
  return {
    dynamic: function(){
      return {
        title: 'Detail Badge', // for detail badges only
        text: 'Dynamic ' + (Math.random() * 100).toFixed(0).toString(),
        icon: icon, // for card front badges only
        color: badgeColor,
        refresh: 10
      }
    }
  }
}
```

Tattooed brooklyn tbh squid, lyft biodiesel ethical street art bespoke letterpress tacos offal. Street art unicorn hot chicken, edison bulb asymmetrical yuccie pinterest yr stumptown. Hella gentrify lumbersexual helvetica, paleo kitsch celiac. Man braid mixtape art party fanny pack, heirloom hot chicken kombucha tumeric hell of DIY pop-up.

Fields | Description
------ | -----------
icon | Path to the icon to display on the button
text | Text to show on the button
url (optional) | The URL that a user should be taken to upon clicking the button
target (optional) | The target for the a tag (learn about targets)
callback (optional) | A method to call onclick.

Fields
text
Text to show on the badge
icon (card-badges only)
Path to the icon to display on the badge.
Style Note: This image should be solid SVG
Style Note: If using a PNG, the image should be greyscale, matching existing card badges.
title (detail-badges only)
Title to show above te badge
color (optional)
Should be one of: ‘red’, ‘yellow’, ‘green’, or ‘’
refresh (optional)
The number of seconds between refreshes. The system will ignore a refresh of less than 10 seconds. In order to use refresh, the object to be returned should instead look like:
{dynamic: callbackFunction(t, context)}
By returning an object with the property dynamic specified as a function that returns these parameters, your card badge will be regenerated based on your supplied refresh rate.

### card-badges

Everyday carry schlitz PBR&B tumblr blog, chillwave quinoa pour-over yuccie you probably haven't heard of them chambray ennui beard subway tile. Cray next level offal, chambray raw denim tumblr microdosing. Master cleanse biodiesel banjo, lyft coloring book occupy paleo fam taxidermy scenester kale chips. Lo-fi selvage photo booth slow-carb, lyft woke swag offal man bun hashtag pok pok schlitz drinking vinegar hoodie quinoa.

<img src="www.fillmurray.com/600/300">

Raw denim church-key cray, knausgaard umami bespoke artisan vaporware chartreuse. Tumeric bespoke pinterest schlitz. Synth cardigan pug YOLO, plaid craft beer church-key prism migas pitchfork mustache farm-to-table PBR&B man bun.

### card-detail-badges

Mustache DIY craft beer, small batch seitan unicorn celiac pork belly banh mi quinoa. Locavore mlkshk before they sold out truffaut affogato. Enamel pin 8-bit letterpress tattooed heirloom yuccie hexagon, vinyl art party cornhole swag messenger bag pug normcore.

<img src="www.fillmurray.com/500/300">

Farm-to-table iceland 8-bit messenger bag, actually bicycle rights kombucha bespoke kitsch tumblr poke ramps. Live-edge lumbersexual tumeric mlkshk, small batch poutine kombucha artisan retro bicycle rights edison bulb. La croix bushwick tbh, forage kinfolk drinking vinegar salvia organic cray authentic vape pickled affogato. Everyday carry hot chicken plaid shabby chic occupy ugh copper mug.


## Attachments

### attachment-sections

Example
'attachment-sections': function(t, options) {
  return [
    {
      icon: './images/icon.svg',
      title: "Attachment Section",
      claimed: [arrayOfClaimedAttachments],
      content: {
        type: 'iframe',
        url: t.signUrl('./attachment-section.html')
      }

    }
  ];
}
Fields
icon
Path to the icon to display on the button
title
A string to use as the header for the section, or a function that returns the title
claimed
An array of claimed attachments. You can use objects from options.entries in this array.
content
An object with content.type set as “iframe”, and a URL that has been wrapped in t.signUrl().
height (optional)
The height of the attachment-section iframe to render
id (optional)
Unique id for your section, best practice is to provide one if you provide a function for your section's title

## attachment-thumbnail

Example
'attachment-thumbnail': function(t, attachment) {
  if(attachment.url == "https://developers.trello.com/") {
    return {
      title: "Trello Developer Site",
      url: "https://developers.trello.com/",
      openText: "Open With Sample",
      image: {
        url: './images/trello-icon.png',
        logo: true // false if this is a thumbnail
      },
      initialize: {
        type: 'iframe',
        url: TrelloPowerUp.util.relativeUrl('authorize-link.html')
      }
    };
  } else {
          throw t.NotHandled("Not a handled URL");
  }
},

Fields
title
The name to display instead of the raw url
url
The URL of the file that should be linked to
openText (optional)
The text shown to the user such as "Open in New Tab" or "Open in DropBox"
image (recommended)
The URL of the thumbnail to be shown, and a boolean parameter 'logo' indicating whether you are showing a logo, or a thumbnail of the content
modified (Date) (optional)
The date of last modification of the attached file
created (Date) (optional)
The creation date of the attached file
createdBy (optional)
The name of the creator of the attached file
modifiedBy (optional)
The name of the last modifier of the attached file
initialize (optional)
If you would like to also include an iframe, such as for authentication purposes, specify a {type:'iframe', url:url} object within the intialize parameter.

## format-url

Example
'format-url': function(t, options) {
  if(options.url.length > 20) {
    return {
      icon: './images/trello-icon.png',
      text: options.url

    };
  } else {
    throw t.NotHandled("Not a handled URL");
  }
},
Fields
icon
Path to the icon to display to the left of the URL
text
Text to show instead of the URL
Context Fields
The following fields are available as part of the context object:
url
The URL of the attachment
locale
The locale of the member

## card-from-url

Example
'card-from-url': function(t, options) {
  // return a name, and optionally a description
  // based on options.url
  return {
    name: 'Suitable name based on options.url',
    desc: 'Suitable description based on options.url'
  };
},
Fields
name
The name to use for the new Card
desc (optional)
The description to use for the new Card

## show-settings

Example
'show-settings': function(t, options) {
  return t.popup({
    title: "Power-Up Settings",
    url: 'settings.html',
    height: 250
  });
}
No fields need to be returned with this capability. Standard behavior is to open a popup as shown in the example above.
Menus
Often you will want to render Menus to the member, such as in response to clicking on a card-button. This code is invoked as the callback when a member clicks on the card-button returned as part of the plugin initialization. Initially the callback runs the t.popup method and provides two hardcoded options. If the user clicks on one of these options, we then create an overlay via t.overlay that loads the appropriate interface in a new iframe.
Below is a sample callback from our Dropbox integration:
var attachFile = function(t) {
  return t.overlay({
    url: 'attach.html'
  })
  .then(function(){
    return t.closePopup();
  })
}

var attachFolder = function(t) {
  return t.overlay({
    url: 'attach-folder.html'
  })
  .then(function(){
    return t.closePopup();
  })
}

module.exports = function(t) {
  return t.popup({
    title: 'Dropbox',
    items: [
      {
        text: "Attach a File…",
        callback: attachFile
      },
      {
        text: "Attach a Folder…",
        callback: attachFolder
      }
    ]
  });
}
 
Popups can be nested, which will automatically provide a back button in the corner of the popup. You can also request that search be allowed on the popup. This will provide native web client functionality for performing full text searches of your supplied items.

Alternatively, popups can be created containing an iframe by supplying a title, URL, and height instead of a set of items.

## Attachment

Attachments can be created with the t.attach method, but they can only be removed by the user using the standard web experience in the attachments section, or by clicking in the upper right hand corner of any attachment-section claiming an attachment.
t.attach({ url: url });

The names of each attachment in the “Remove Attachments” popup are rendered using format-url.

# Authentication

You should only require authentication when it is required by the user. Here are some samples of ways that Power-Ups can trigger an authentication workflow:
1. For our Dropbox Power-Up, “Show Details from Dropbox…” has been added to the attachment-thumbnail by detecting that the link to Dropbox has not been initialized for this member. Once the Power-Up has been initialized and authenticated, we show a true thumbnail.

For our Github Power-Up, the attachment-sections iframe managed by the Power-Up includes a prompt to authenticate to see additional information.

We also trigger authentication after a user attempts to perform an action that requires it, such as when attaching a GitHub Issue to a card.

When developing a Power-Up that requires authentication, try to offer as much functionality in a non-authenticated state as possible, and add non-intrusive links to authenticate the user to provide more functionality.
To achieve Authentication, you will often need to use the t.authenticate method. Here’s a code sample of how you might handle the OAuth workflow from within a Power-Up popover iframe.
Client side we need to wire up what should happen when the authenticate button or link is clicked, which is handled by the following code.
```javascript
var Promise = TrelloPowerUp.Promise;
var t = TrelloPowerUp.iframe();

var oauthUrl = 'https://trello.com/1/authorize?expiration=never' +
  '&name=[APPNAME]&scope=read&key=[APIKEY]&callback_method=fragment' +
  '&return_url=[RETURNURL]';

var tokenLooksValid = function(token) {
  return /^[0-9a-f]{64}$/.test(token);
}

var authorizeOpts = {
  height: 680,
  width: 580,
  validToken: tokenLooksValid
};

var authBtn = document.getElementById('authorize');
authBtn.addEventListener('click', function() {
  t.authorize(oauthUrl, authorizeOpts)
  .then(function(token) {
    return t.set('organization', 'private', 'token', token)
    .catch(t.NotHandled, function() {
      // fall back to storing at board level
      return t.set('board', 'private', 'token', token);
    });
  })
  .then(function() {
    // now that the token is stored, we can close this popup
    // you might alternatively choose to open a new popup
    return t.closePopup();
  });
});
```javascript
 
The authorize handler opens a new window. Once the auth flow is complete in that window, you should land on a page you control (generally your return_url). That page needs to contain a small amount of javascript to send the resulting token back to the Power-Up.
Here is an example of how that would be achieved for the Trello oauth flow from above.

```javascript
var token = window.location.hash.substring(7);
if (window.opener && typeof window.opener.authorize === 'function') {
  window.opener.authorize(token);
} else {
  localStorage.setItem('token', token);
}
setTimeout(function(){ window.close(); }, 1000);
```
# Advanced Topics

## Data Storage
There are various scopes that can be used to store data. These scopes are determined by the visibility level, and the entity it is attached to.

Card ('card')
Board ('board')
Organization ('organization')
Private ('private')
This data is available to only this member on this card.
This data is available to only this member on this board.
This data is only available to this member on any board belonging to the current organization.
Shared ('shared')
This data is available to any member with access to this card.
This data is available to any member with access to this board.
This data is available to any member on any board belonging to the current organization.

You can store data two ways:
t.set('board', 'private', 'key', value)
or
t.set('board', 'private', { key1: value1, key2: value2 })

Currently, there is no external API for accessing Power-Up data. Power-Up stored data can only be retrieved when the UI is loaded by the member with the Power-Up enabled.

Members can reset their private data at any time via the Power-Ups menu.

## Recoloring

Trello expects you to change the color of any icons you use based on a query parameter that we will provide. This parameter is designed to ensure that icons have great usability even when the background behind an image may change.
For example, card badge icons are expected to be grey (#999) in their default state, but when a color is applied to the badge (such as ‘red’), the icon is passed ?color=fff. This behavior means that the icon should be recolored such that the primary color used is white, ensuring the image will look good and have good usability on a red background.

## Timeouts

Your Power-Up should respond within 1 second to all capability methods. If your method takes more than 1 second to complete, we will consider the callback as failed.
## Internationalization

Trello Power-Ups were designed with Internationalization in mind. When creating a plugin or iframe, you can optionally pass in the options argument one of the following:

localization object
localization: {
  defaultLocale: 'en',
  supportedLocales: ['en', 'fr'],
  resourceUrl: './strings/{locale}.json'
}
localizer object
localizer: {
  localize: function(key, data){
    ... synchronously returns a localized string for the given key and data ...
  }
}
loadLocalizer function
loadLocalizer: function(locale){
    ... returns a Promise that resolves to a localizer object ...
  }

Additionally, you can set window.localizer to your own custom localizer and the HostHandlers will use it.

Once a localizer has been initialized, you can call the following new handlers (which behind the scenes make use of the localizer). We make the current user locale available to you via window.locale. Additionally, you can grab it off the options object from any of the interfaces or callbacks as well.

localizeKey(key, data) synchronously returns the output of window.localizer.localize(key, data)

localizeKeys(key) synchronously localizes multiple keys. Input can be either an array of keys ['key1', 'key2'] or an arrary of key, data arrays [['key1', data1], ['key2', data2]]

localizeNode(node) synchronously inserts localized texts into DOM nodes starting at the provided node and all of it's children. Tag a node in your HTML with data-i18n-id="key" for its text content to be replaced based on that key. You can pass args with data-i18n-args='{ "arg1": "data" }'. You can also have placeholder text replaced with data-i18n-attrs='{ "placeholder": "key" }'. This will also use the args defined in data-i18n-args.

Currently there is no support for dot notation when replacing args.

Argument replacement assumes that your resource file follows this format:
{
  "key": "value with replaceable {arguments}"
}

Again, dot notation is not supported for arguments. This also limits how you can use braces {} in that there is no escaping currently supported.
## What are callbacks?

Sometimes a Power-Up wants the web client to create a bit of UI, and the Power-Up wants to know when the user interacts with that UI (e.g. when the user clicks on a button) For example, the Trello web client might run card-button on a Power-Up, and the Power-Up could respond with
{
  icon: … url of an icon to display …
  text: "Activate"
  callback: … a callback to run when the button is clicked …
}

### How are callbacks used?

For example, a Power-Up might request that a popover list be displayed, with, say 20 items in it. Each of the items has a callback associated with it, but only the one corresponding to the thing that gets clicked on would be called. Or maybe the popup would just get closed and none of them would get called.
Callbacks can also be used to e.g. supply the most recent state of the badge on a card (in that case, the same callback might be called many times) … or a callback could be associated with the button in the board header (and the same callback would be triggered every time the button was clicked).

## Webhooks & Offline Access

Webhooks and Offline Access are not currently part of the Power-Up architecture. Power-Ups are only active while a member is in the Trello web client experience. It is possible for your Power-Up and service to access a user’s information and create Webhooks using the standard API.
The way to accomplish this is for you to Authorize your application to the Trello API as a part of your Authorization flow, or in response to a user action. You would then need your service to use the standard Trello API to make offline requests.

For example, if you were building a Power-Up where you wanted to react to any change made to a card, the member’s experience could be as follows.
- Trello Member enables Magic Service Power-Up
- Trello Member clicks on the card-button to attach a Magic Service to a card.
- You prompt the member via an overlay iframe to authorize your service to access the member’s Trello account
- Your service stores the user’s token as part of a one-time authorization step
- Your service stores the Card’s ID as part of your card-button callback
- Your service creates a webhook based on Card ID
- Your Power-Up renders a new card-button for any cards that are already attached to your service
- When the user clicks the “detach” card-button, your service deletes the webhook

## Power-Ups Troubleshooting
As our platform grows, we are working on identifying the most common mistakes and problems.
Uncaught PluginRunner::NotHandled: attempt to run callback on plugin 55a5d917446f51777421000f failed(anonymous function) @ ltp.js:40603(anonymous function) @ ltp.js:37845
ltp.js:38699 ^--- With additional stack trace: PluginRunner::NotHandled: attempt to run callback on plugin 55a5d917446f51777421000f failed
Solution: Make sure you are specifying “callback” as one of your capabilities in your manifest.json file.

# Power-Up Client Library

We provide and host a Javascript library for all Power-Ups. This Library gives you access to a standardized communication structure between your Power-Up and the Trello web experience.

By using this library, your application-specific code can be simplified to the set of capabilities that you wish to expose. The library is hosted here:
https://trello.com/power-ups/power-up.min.jshttps://trello.com/power-ups/power-up.js

We also provide a CSS file which you should use to automatically achieve a consistent style with Trello:
https://trello.com/power-ups/power-up.css

## initialize({'[capability-name]':function() {}, ...})

The most important method of the Client Library which is required for all Power-Ups, is the initialize method. This method should be called within your index connector. Initialize accepts only one parameter, a JavaScript object containing any of the capabilities you wish to use in your Power-Up, with the name of the capability as the key, and the function you would like us to call when a capability is executed as the value.
```javascript
TrelloPowerUp.initialize({
  'board-buttons': function(t, options){
    return [{
    icon: './images/icon-white.svg',
    text: 'My Button',
    callback: function(t){
      // do something when the button is clicked
    }}];
  },
  'show-settings': function(t, options){
    return t.popup({
      title: 'Settings',
      url: './settings.html',
      height: 184
    });
  }
});
```
Each capability that you include in your initialization will take a reference to the t object and a options or context object. This object will include various things depending on the capability.

## Board Capability Options
{
  "context":{
    "board":"55db14fd3e105ac84105bd75",
    "command":"board-buttons",
    "plugin":"564ddf493f183488ea5ddc0e"
  },
  "locale":"en-US"
}

## Card Capability Options
{
  "context":{
    "board":"55db14fd3e104ac8b105bd75",
    "card":"563b532e4e998440d0d88e62",
    "command":"card-detail-badges",
    "plugin":"564ddf493f184b88ea5ddc0e"
  },
  "locale":"en-US"
}

## Attachment Sections Options
{
  "entries":[
    {"id":"56a2467476812315cbbbcbe4","url":"https://example.org/url","name":"https://example.org/url"},
    ...
    {"id":"569e5ab156d89775e853ebb4","url":"https://example.org/url","name":"Attachment Name"}
  ],
  "context":{
    "board":"55db14fd3e105ac8b105bd75",
    "card":"569d3e15729379ff9b9f6f90",
    "command":"attachment-sections",
    "options":{
      "entries":[
        {"id":"56a2467476812315cbbbcbe4","url":"https://example.org/url","name":"https://example.org/url"},
        ...
        {"id":"569e5ab156d89775e853ebb4","url":"https://example.org/url","name":"Attachment Name"}
      ]
    },
    "plugin":"564ddf493f183b88ea5ddc0e"
  },
  "locale":"en-US"
}

## UX Methods
render(function renderer) Define a method that is called by the web client when there are updates, such as a new attachment. Attachment-Sections (and other capabilities) are not re-initialized upon changes to content such as attachments of a card to prevent flashing. render is also an important method because we may not know the Member’s locale at time of initialization, but we will know it at render time.
popup(options) Use this method to have Trello display a popup. It will be displayed adjacent to the element in the current context. Pop-ups are designed for lists of capabilities or content that a member is expected to click on. The options object can contain: title callback url items search
closePopup() This method should be used within a popup callback to close the current popup
back() This method should be used within a popup callback to return to the prior popup, if it exists.
overlay({url:’’}) Place an iframe on top of the Member’s experience
closeOverlay() This method should be used within an overlay to close the existing overlay.
boardBar(options) Used to render an iframe that is visible at the bottom of the standard board view. This option is currently under development.
The options object can contain: url args - A list of arguments to be passed to the iframe as query parameters heightcloseBoardBar() Used to close the bottom Board Bar
authorize(urlWithSecret, options) Pass a URL to open as a new window for OAuth authentication purposes.
NOTE: This must be run within an onclick event from one of your iframes, otherwise the browser popup blocker will be triggered and may break your ability to communicate between your authorization window and the parent Trello window.sizeTo(selector) - Sizes the current iframe based on the height of the element referenced via your selector.
Data Methods
iframe() Get access to the t object within an iframe
set(scope, visibility, name, value) Used to store some persistent data in Trello's database. The Power-Up provides a scope (organization,board or card) and the visibility of the data (i.e. whether it's ‘shared’ with everyone that can see the org/board/card or if it's ‘private’) and a string to store. (Power-Ups are allowed to store 1K of data for each scope/visibility pair … the current Power-Ups store stringified JSON) Using t.set will re-initialize your Power-Up in it’s entirety by re-executing all capabilities.
get(scope, visibility, name, defaultValue) Used to get all of the stored data for the Power-Up
attach({url:URL, name:Name}) Attach a new URL to the card in the current context.
signUrl(URL) Sign a URL for use with attachment-sections only.
localizeKey(key, data) Localize a string by key
localizeKeys(keys) Localize an list of strings or objects
board(field1, field2, ...) Returns a promise with information about the board for the current context Valid fields include: 'id', 'name', 'url', 'shortLink'
list(field1, field2, ...) Returns a promise with information about the list for the current context Valid fields include: 'id', 'name'
card(field1, field2, ...) Returns a promise with information about the card for the current context. The information available is only when a specific card is in context. For instance, this function will return information about a card successfully inside of the capabilities 'attachment-sections' and 'card-badges', but not 'show-settings' nor 'board-buttons'. The latter are not within the context of a single card.

Valid fields include: 'id', 'name', 'desc', 'due', 'closed', 'cover', 'attachments', 'members', 'labels', 'url', 'shortLink', 'idList'

Example usage:
t.card('id', 'name', 'url').then(function(promiseResult){console.log(promiseResult)})
// <- {id:"72hejh3iuruyr87rhiu", name: "Card Name", url: "https://trello.com/c/38UCRu1a/card-name"}
t.closePopup, t.closeOverlay, t.iframe can only be called within an appropriate context.
t.popup can only be called within a capability, not currently from an iframe such as an attachment-section. This is due to the relative placement of the popup to an item within the Trello Web Client.

