<div align="center">
  <img src="https://media.giphy.com/media/3Z15Ve7WEQGkLa1FwC/giphy.gif" alt="handsfree.js">
  <p>A platform for creating handsfree user interfaces, tools, games, and experiences for the web and IoT 🤯</p>
  <p>Made possible by <a href="https://github.com/Tastenkunst/brfv4_javascript_examples">BRFv4</a></p>
<p>
  <img class="mr-1" src="https://img.shields.io/github/release-pre/browsehandsfree/handsfreejs.svg"> <img class="mr-1" src="https://img.shields.io/github/last-commit/browsehandsfree/handsfreejs.svg"> <img class="mr-1" src="https://img.shields.io/github/commits-since/browsehandsfree/handsfreejs/0.0.1.svg">
  <img src="https://img.shields.io/github/repo-size/browsehandsfree/handsfreejs.svg">
</p>
<p>
  <img class="mr-1" src="https://img.shields.io/github/issues-raw/browsehandsfree/handsfreejs.svg"> <img class="mr-1" src="https://img.shields.io/github/issues/browsehandsfree/handsfreejs/✨ enhancement.svg"> <img src="https://img.shields.io/github/issues-pr-raw/browsehandsfree/handsfreejs.svg">
</p>
<p>
  <img class="mr-1" src="https://img.shields.io/github/issues/browsehandsfree/handsfreejs/🐞 bug.svg"> <img src="https://img.shields.io/github/issues/browsehandsfree/handsfreejs/👷 help wanted.svg">
</p>
<p>
  <img src="https://travis-ci.org/BrowseHandsfree/handsfreeJS.svg?branch=master">
  <img src="https://img.shields.io/codecov/c/github/BrowseHandsfree/handsfreeJS/master.svg?style=flat">
</p>
</div>

## License
> Please read the EULA ([eula.txt](eula.txt)) carefully before using Handsfree.js, which depends on [BRFv4](https://www.beyond-reality-face.com/). Once you decide to use handsfree.js commercially, you will need a separate license agreement from them that you must agree to. You can try handsfree.js free of charge to evaluate if it fits your projects' needs. Once you decided to use BRFv4 in your project, contact Taskenkunst for a commercial license: [http://www.tastenkunst.com/#/contact](http://www.tastenkunst.com/#/contact)











# `/handsfree.js/`

`handsfree.js` is a drop-in library that uses computer vision to help you create head-tracked experiences. You can find the library in `/handsfree.js/`, with the entry point being `/handsfree.js/Handsfree.js`. This file adds a global `Handsfree` class to your project.

`handsfree.js` works using plugins though, so you most likely don't need to edit these files. In fact, the fastest way to get started building with handsfree.js is via CDN!

## Quickstart
Choose one of the following `<script>`'s to add to your HTML page, which will add a `Handsfree` global class:

```html
<body>
  <!-- Latest with bug fixes (Recommended for production) -->
  <script defer async src="https://unpkg.com/handsfree@<3.1/dist/handsfree.js"></script>

  <!-- Latest with bug fixes and new features (Recommended for development) -->
  <script defer async src="https://unpkg.com/handsfree@<4/dist/handsfree.js"></script>

  <!-- Latest with potential backwards incompatability (Recommended for testers) -->
  <script defer async src="https://unpkg.com/handsfree/dist/handsfree.js"></script>
</body>
```

You can also include it via NPM:

```javascript
// From the terminal in the project root, type:
npm i handsfree

// Then inside your Node project:
import Handsfree from 'handsfree'
```

 Then **in both cases**, you'll then need to instantiate a Handsfree controller:

```js
// Start settings for your app
const config = {debug: false, settings: {}}
// Instantiate Handsfree
const handsfree = new Handsfree(config)
// Turn on webcam and start tracking
handsfree.start()
```

## Config
There are a number of ways to configure Handsfree.js, the easiest being during instantiation. Here are the available configs and their defaults as of [this release](https://github.com/BrowseHandsfree/handsfreeJS/releases):

```js
const handsfree = new Handsfree({
  // Whether to show (true) the "debugger" or not (false)
  // - For now the debugger is a simple canvas that shows a wireframe over tracked faces
  // - You can toggle the debugger later with handsfree.toggleDebugger(true|false)
  debug: false,

  // Available settings
  // - You can change any of these later with: handsfree.settings['settingName'] = newVal;
  settings: {
    // Maximum number of faces to track
    // - Performance drops with each additional face
    // - 🚧 This is experimental and not working with the core plugins yet
    maxFaces: 1,

    sensitivity: {
      // A factor to adjust the cursors move speed by
      // - Think of this as your user's "base cursor speed" which you
      //   can factor into your plugins to give your users a consistent experience regardless of their speed preference
      xy: 0.7,

      // How much wider (+) or narrower (-) a smile needs to be to click
      // - Good ranges are [-0.5, 0.5]
      // - Because this is based on mouth/eye ratio, this value should be adjustable by your user
      click: 0
    },

    // 🚧 Experimental
    // These settings help reduce cursor jitter
    stabilizer: {
      // How much stabilization to use: 0 = none, 3 = heavy
      factor: 1,
      // Number of frames to stabilizer over
      buffer: 30
    }
  }
})
```

## API

The following methods are available on your `handsfree` instance:

```js
// Starts a webcam stream, shows the debugger if active, and starts tracking
// - Also triggers the onStart event of enabled plugins
handsfree.start()

// Closes the webcam stream, hides the debugger, and stops tracking
// - Also triggers the onStop event of enabled plugins
handsfree.stop()

// Toggles the debugger on (true), off (false), or flips the state (null)
handsfree.toggleDebugger(true|false|null)
```

Changing settings is done with:

```js
handsfree.settings['path']['to']['setting'] = newVal
```

## Plugin Development

Handsfree makes heavy use of plugins, allowing you to extend Handsfree.js without modifying the core library!

To add a plugin, use the `handsfree.use({})` method with the following signature. Calling `use()` with the same `.name` overwrites that plugin (this lets you easily prototype in your browser's debug console).

Finally, this method returns a reference to the plugin:

```js
const myPlugin = handsfree.use({
  // Must be unique (spaces and special characters are fine)
  // - Using the same name as an existing one overwrites it
  name: '',

  // Set to true to have this plugin disabled by default
  // - Disabled plugins do not run any of their hooks
  _isDisabled: false,

  // Called once when the plugin is first added to handsfree with handsfre.use({})
  onUse: (handsfree) => {},

  // Called once per frame, after all calculations
  // - {Array} faces  A collection of
  // - {Return}       To overwrite/modify the properties of faces for use within other plugins, return faces array with modified values
  onFrame: (faces, handsfree) => {},

  // Called any time Handsfree.start() is called
  onStart: (handsfree) => {},

  // Called any time Handsfree.stop() is called
  onStop: (handsfree) => {},

  // Called when .disable() is explicitely called on this plugin
  onDisable: (handsfree) => {},

  // Called when .enable() is explicitely called on this plugin
  onEnable: (handsfree) => {}
})
```

Additionally, every plugin has a `.disable()` and an `enable()` method, which sets the `._isDisabled` flag to either true or false. These methods also fire the plugins `onDisable()` and `onEnable` hooks if they were provided:

```js
handsfree.plugin['my-plugin'].disable() // handsfree.plugin['my-plugin']._isDisabled === true
handsfree.plugin['my-plugin'].enable() // handsfree.plugin['my-plugin']._isDisabled === false
```

## The `faces` array

The `onFrame` hook recieves a `faces` array, which contains an object for each tracked face. The key properties of the a `face` object include:

```js
{
  cursor: {
    // Where to position the cursor on the screen, based on the users head pose
    x: 0,
    y: 0,

    // The HTML element currently under the above {x, y}
    $target: 0,

    // Cursor states for this face
    // - These states are activated via the Click Gesture,
    //   which is currently done by smiling wide
    state: {
      // True during the first frame of a click, false after (even if still held)
      mouseDown: false,
      // True after the first frame of a click and every frame until released
      mouseDrag: false,
      // True on the last frame of a click, immediately after the click is released
      mouseUp: false
    }
  },

  // A list of all 64 landmarks
  // - points[27] refers to the point between the eyes
  points: [{x, y}, ...],

  // The head's pitch (facing up/down)
  rotationX: 0,
  // The head's yaw (facing left/right)
  rotationY: 0,
  // The head's roll (think of an airplane doing a barrel roll)
  rotationZ: 0,

  // The overall size of the head relative to the video frame
  // - smaller values == further away
  // - larger values == close up
  scale: 0,

  // Where the head is relative to the left edge of the video feed
  // - 0 == left
  // - window.innerWidth == right
  translationX: 0,
  // Where the head is relative to the top edge of the video feed
  // - 0 == top
  // - window.innerHeight == bottom
  translationY: 0
}
```

There are 64 landmark points, reflected in the following image: 

![image from BRFv4](src/assets/img/brfv4_landmarks.jpg)

## Events
### handsfree-trackFaces
An alternative to plugins is to listen in on the window's `handsfree-trackFaces` event:

```js
/**
 * Bind to the handsfree-trackFaces event
 * @param {Handsfree} ev.detail.scope The handsfree instance
 * @param {Object}    ev.detail.faces An array of face objects
 */
window.addEventListener('handsfree-trackFaces', (ev) => {
  // Do code with the handsfree instance: ev.detail.scope
  // or with the faces ev.detail.faces.forEach(face => {})
})
```

### handsfree-injectDebugger
The `handsfree-injectDebugger` event is fired after the debugger is injected, but before handsfree is started. Use this event to draw into the canvas without the camera being turned on.

```js
/**
 * Bind to the handsfree-injectDebugger event
 * @param {Handsfree}       ev.detail.scope The handsfree instance
 * @param {Canvas2DContent} ev.detail.canvasContext The 2D debug canvas context
 */
window.addEventListener('handsfree-injectDebugger', (ev) => {
  // Do code with the handsfree instance: ev.detail.scope
  // or draw into the canvas with ev.detail.canvasContext
})
```

## Classes
The document body contains `.handsfree-stopped` when handsfree is stopped (this includes when it's been initialized but not started), and `.handsfree-started` when it's on. This lets you quickly hide, show, and style elements based on the state of handsfree.

## Visual Debugging

The debugger is loaded into the first element in the DOM with the `.handsfree-debug-wrap`. If one doesn't exist, then it's added as the last root element of `body`. You should rarely need to debug visually, and it's preferred that you don't draw into this canvas at all as it's used by the BRFv4 model for inferring your head pose.













# `/src/`

This is where [handsfree.js.org](https://handsfree.js.org) lives and is a good place to prototype quickly. I'll leave more instructions for this workflow soon.

## Workflows
### Handsfree YouTube

**Source:** `/src/components/youtube/`

**Preview:** https://handsfree.js.org/#/youtube

**Starter Kit:** https://glitch.com/~handsfree-youtube

![](https://media.giphy.com/media/4HgnusIh1i8MzRoaOm/giphy.gif)


## Building Prereqs
- [NodeJS](https://nodejs.org/en/)

## Scripts
Run the following from projects root directory:

``` bash
# Install dependencies
npm install

# Start a server with hot-reload at localhost:3000
npm run serve

# Test the library (not the documentation site)
npm run test

# Build for production
npm run build

# Build and deploy (see /deploy.js to configure for your own server)
npm run deploy
```

For detailed explanation on how things work, check out the [Vuetify.js](https://vuetifyjs.com/) and [CLI Plugin](https://github.com/vuetifyjs/vue-cli-plugin-vuetify) documentation.


# More coming soon

- [@Labofoz](https://twitter.com/labofoz)
- [handsfree.js.org](https://handsfree.js.org)

## Thanks for trying out Handsfree.js!
**March 4th 2018**: https://twitter.com/LabOfOz/status/970556829125165056

![](https://media.giphy.com/media/4HvjGXt2Jjwz5LG71Z/giphy.gif)