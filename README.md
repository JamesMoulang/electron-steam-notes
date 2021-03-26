# electron-steam-notes

My notes on getting electron html5 apps working with Steamworks, Greenworks, Overlay, etc.

If you're reading this you either want to get up and running as quickly as possible, or maybe you've managed to greenworks etc. working independently, but you need to set up your project to work with Steam Overlay.

## Quick start from scratch

Start by checking out this github repo.

The starting state of the project is basically a freshly initialised electron-forge project that you need to add Greenworks, and Steamworks SDK files to.

First get all the npm stuff you need to run electron-forge. Note that the package.json already specifies electron v11.4.0 which is the version I found to actually work with Greenworks.

```bash
npm install
```

Now we need to set up Greenworks in the src folder of our app (the folder that actually contains the app files, as opposed to electron-forge stuff). It's not as simple as just an npm install, though.

```bash
cd src
npm install --save --ignore-scripts git+https://github.com/greenheartgames/greenworks.git
```

Now we need to add the pre-built Greenworks binaries to Greenworks in node_modules

You can find the binaries here: https://greenworks-prebuilds.armaldio.xyz/

Look for the 64 bit, Electron, Windows, Steamworks v1.5.0 version.

11.0.0-beta.11 -> 11.4.1 (v85)

Copy greenworks-win64.node into node_modules/greenworks/lib (you may have to create this folder)

Two more files need to be added to this same folder, and they can be found at https://partner.steamgames.com/downloads/list (note you need to log in to get these files).

It's important for the Steamworks SDK version to match the prebuilt Greenworks verison. We need version 1.50

You need two files per platform:

* redistributable_bin/win64/steam_api64.dll
* sdkencryptedappticket from public/steam/lib e.g. public/steam/lib/win64/sdkencryptedappticket64.dll

Now you can

```bash
npm install
```

You should be all set up at this point. You can test by running:

```bash
cd ..
npm start
```

If this all works, you can skip to the "building an app" section at the bottom of this page to create a distributable, and get Steam Overlay running. If you want to set this up yourself, because this didn't work for some reason, or you actually want to understand the structure of the program, I recommend looking at [this page by @slosumo](https://github.com/slosumo/greenworks_electron/blob/main/greenworks_isntructions.txt).

Note that Steam Overlay probably won't work out of the box with this app, but if you add your own game files it probably will.

Good luck!

## Building an app that works with Steam Overlay

So, at this point you have a working electron app that you can run 'locally' (i.e. not as a distributable) and you want to:

1. Add your game files
2. Create a distributable
3. Get Steam Overlay running

I used [electron-forge](https://github.com/electron-userland/electron-forge) to package up my app, but other options exist.

### Setting up electron-forge

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
npm install --save-dev electron@11.4.0
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

#### Uncaught Error: Steam initialization failed. Steam is running,but steam_appid.txt is missing.

This is pretty self-explanatory. The steam_appid.txt file needs to go in the root directory of the electron-forge project. Just copy it to where it needs to be.

#### Cannot find module '...index.js'. Please verify that the package.json has a valid "main" entry

Make sure that the value of main in package.json is pointing at the right file. The default value was

```json
"main": "src/index.js",
```

for me, which doesn't match the structure of my electron app. It should match the file inside which you create the new BrowserWindow, and do all that funky electron stuff that I don't understand.

```json
"main": "src/start.js",
```

At this point, as long as you're using a valid steam_appid.txt file, and Steam is running, you should see yourself playing your game in Steam. Hooray!

### Building an app

In order to get Steam Overlay running in your game, you have to launch it via Steam. In order to launch it via steam, it has to be an app. So let's make an app.

**Note:** Assuming you still have the 'hello world' Greenworks app, you could take this moment to actually include your game files to the src folder. For me, that's just a matter of updating the index.html file, and including my assets folder and bundle.js file. Your project structure may vary. Just make sure, as you do this, you don't get rid of the code that initialises greenworks.

```bash
npm run make
```

Once this has completed you should see 'your-game-win32-x64' within the 'out' folder.

### Launching your App through Steam

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

If you're making a game that doesn't update the display frequently, you might also have issues with the Steam Overlay not updating, as it hooks into whatever render process is active. If that process isn't active, Steam Overlay can't update. In that case, there are [workarounds](https://github.com/greenheartgames/greenworks/wiki/Troubleshooting#steam-overlay-is-unresponsive--frozen). But if you're making a game that uses canvas, and requestAnimationFrame, this shouldn't be a problem.

That's it!