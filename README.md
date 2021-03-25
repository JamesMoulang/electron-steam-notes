# electron-steam-notes
My notes on getting electron html5 apps working with Steamworks, Greenworks, Overlay, etc.

So, at this point you have a working electron app that you can run 'locally' (i.e. not as a distributable) and you want to:

1. Add your game files
2. Create a distributable
3. Get Steam Overlay running

I used [electron-forge](https://github.com/electron-userland/electron-forge) to package up my app, but other options exist.

## Setting up electron-forge

Install electron-forge globally and create a new folder than contains your project.

```bash
npm install -g @electron-forge/cli
electron-forge init my-new-app
cd my-new-app
npm start
```

Have a look inside the folder. The important files are:

* package.json
* src (folder that contains your app)

First, we need to ensure we're using the right version of electron (v11.4.0)

So let's uninstall the version of electron that electron-forge installed automatically, in this folder

```bash
npm uninstall electron
```

And install the version we want

```bash
npm install electron@11.4.0
```

By this point, your dependencies in package.json should look like this:

```json
"dependencies": {
    "electron-squirrel-startup": "^1.0.0"
},
"devDependencies": {
    "@electron-forge/cli": "^6.0.0-beta.54",
    "@electron-forge/maker-deb": "^6.0.0-beta.54",
    "@electron-forge/maker-rpm": "^6.0.0-beta.54",
    "@electron-forge/maker-squirrel": "^6.0.0-beta.54",
    "@electron-forge/maker-zip": "^6.0.0-beta.54",
    "electron": "11.4.0"
}
```

Now, copy the entire app you had working already into the .src folder. The whole lot, node_modules and all.

To save file size you can uninstall electron **within the src folder** because we only need the electron version running in the root directory of the electron-forge project.

```bash
npm uninstall electron
```

**Note:** you could have used electron-forge from the start, but, it's not the order I figured this out in, so, it's not the order I'm presenting it in either :) If you're like me, and you haven't read this far before starting to spam commands in to the terminal, this comment probably isn't very useful either. Oh well!

At this point you should be able to run your app like before. From within the root directory of your electron-forge project runb:

```bash
npm start
```

A few errors are possible at this point:

### Uncaught Error: Steam initialization failed. Steam is running,but steam_appid.txt is missing.

This is pretty self-explanatory. The steam_appid.txt file needs to go in the root directory of the electron-forge project. Just copy it to where it needs to be.

### Cannot find module '...index.js'. Please verify that the package.json has a valid "main" entry

Make sure that the value of main in package.json is pointing at the right file. The default value was

```json
"main": "src/index.js",
```

for me, which doesn't match the structure of my electron app. It should match the file inside which you create the new BrowserWindow, and do all that funky electron stuff that I don't understand.

```json
"main": "src/start.js",
```

At this point, as long as you're using a valid steam_appid.txt file, and Steam is running, you should see yourself playing your game in Steam. Hooray!

## Building an app

In order to get Steam Overlay running in your game, you have to launch it via Steam. In order to launch it via steam, it has to be an app. So let's make an app.

**Note:** Assuming you still have the 'hello world' Greenworks app, you could take this moment to actually include your game files to the src folder. For me, that's just a matter of updating the index.html file, and including my assets folder and bundle.js file. Your project structure may vary. Just make sure, as you do this, you don't get rid of the code that initialises greenworks.

```bash
npm run make
```

Once this has completed you should see 'your-game-win32-x64' within the 'out' folder.

## Launching your App through Steam

From the Steam Client, click Games > Add a Non-Steam Game to My Library...

Find your .exe and add it. Now, you should be able to run your game through Steam Client, which is necessary to get Steam Overlay running.

**Note:** you only need to do this once. Once 

But first, Steam Overlay won't work without these two launch options (right click on your game in your Steam Library and click Properties...)

```bash
--in-process-gpu --disable-direct-composition
```

What do these do, you ask? Look, don't ask difficult questions. Feel free to edit this page if you have a better idea:

* **--in-process-gpu** allows Steam to find the render process, to hook in the Overlay (somehow)
* **--disable-direct-composition** "Microsoft DirectComposition is a Windows component that enables high-performance bitmap composition with transforms, effects, and animations." If we don't disable this, the Steam Overlay renders strangely (completely white), although resizing the window does seem to fix this. It's possible there's other workarounds.

You also have to copy steam_appid.txt into the root directory of the built app, otherwise you'll have the same "missing steam_appid.txt" error as earlier.

That's it!
