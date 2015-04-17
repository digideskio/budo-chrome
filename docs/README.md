# usage

How to use Chrome Script Injection with budō-chrome.

You need Chrome installed for this to work. Only tested on OSX and Linux so far ([tweet me](https://twitter.com/mattdesl) or post a bug if you find issues). 

### install

First, we need to install the CLI tools. They can be installed locally (preferred) or globally. For the sake of demonstration, we will use global installs in this article. `garnish` is optional but provides nicer logging in terminal.

```sh
npm install -g budo-chrome watchify garnish
```

### simple use

![screenshot](http://i.imgur.com/LJP7d9I.png)

The `--open` command is included for convenience, and is the easiest way to get things up and running.

```sh
budo-chrome index.js --open | garnish
```

This will open a new Chrome instance connected to remote debugging port 9222. You can specify a different port with the `--remote-port` option.

Now, when you save `index.js`, you should see `info budo: bundle.js (inject)` appear in the terminal. If it doesn't, you may need to refresh the tab first, and then try again.

You can quit the process and Chrome instance with `Ctrl + C`.

### connecting to an existing Chrome instance

If you don't specify `--open`, the remote interface will try to connect to an existing instance (defualt port 9222). 

First, you need to run Chrome in [remote debugging mode](http://www.chromium.org/developers/how-tos/run-chromium-with-flags) and open the `localhost:9966` tab. In OSX it might look like this:

```sh
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome http://localhost:9966/ --remote-debugging-port=9222
```

**Note:** You may need to quit chrome first, otherwise it will open a new tab in your existing instance and not work.

Once it's open on `localhost:9966`, open another terminal tab (Cmd + T in OSX) and run the following command:

```sh
budo-chrome index.js | garnish
```

Refresh the chrome tab and then start saving your file to see it injected.

### drawbacks

There are some "gotchas" to know about when you are working with script injection.

#### no console

If you open the DevTools or console you will lose your Remote Chrome session! 

#### stateful code

Live injection will leave your application state unchanged, and so it's best suited for animations and render loops. For example:

```js
//you need a hard refresh to change these
ball.color = 'red'
ball.fill = true

function render(dt) {
    //integrate position
    ball.position.x += dt * speed
    
    //the values in your render loop
    //can be changed during runtime
    ball.radius = 30
}
```

In some cases, if your bundle only includes "state" and no render or update functions, it may get collected and not appear at all during remote script parsing. 

#### large bundles

When the bundle size reaches roughly over 600kb, you may start seeing very long pauses during script injection. This is a [known issue](https://code.google.com/p/v8/issues/detail?id=3894&q=liveedit&colspec=ID%20Type%20Status%20Priority%20Owner%20Summary%20HW%20OS%20Area%20Stars) and will hopefully be fixed in time. 