# Electron + Python

This sample shows how to build Python Flask apps that run in Electron. I'm using it to combine Python backends with React frontends but if you prefer to use handle your frontend entirely from Flask that should be simple to do. The Electron main (backend) process spawns a Python Flask webserver and provides a randomly generated authentication token to both the webserver and the Electron renderer (frontend) process for use in authenticating messages sent between the frontend and the webserver. 

The webserver currently exposes a GraphQL endpoint for the frontend to interact with but the backend is just a plain old Flask webserver so you can tweak it to host whatever sort of REST or other Flask web services as might be needed by your application. The React frontend part of the sample is similarly based on a stock create-react-app site, so it should be easy to customize as needed. The only significant embelishments to the stock cra app are (1) the bare minimal amount of https://github.com/sharegate/craco to support hooking into electron without needing to eject the create react app and (2) typescript support, which you don't have to use but I personally can't imagine building a serious javascript project without it so it's there if you need it.

This example builds a stand-alone Electron + Create-React-App + Python application and installer. On Windows it builds the app into `./dist/win-unpacked/My Electron Python App.exe` and the installer into `./dist/My Electron Python App Setup 1.0.0.exe` (OSX and Linux destinations are similar). You can change the name of the application by changing the `name` property in `package.json`.

# Installation

Tested with Anaconda Python v3, should work fine with Anaconda Python v2 (should also work fine with whatever python environment you use if you have the correct packages installed).

NOTE: On windows you will need to [install anaconda](https://www.anaconda.com/download/) (which installs python and pip) and potentially configure environment variables to add python and/or pip to the path if you don't have it installed already.

```bash
# start with the obvious step you always need to do with node projects
npm install

# Depending on the packages you install, with Electron projects you may need to do 
# an npm rebuild to rebuild any included binaries for the current OS. It's probably
# not needed here but I do it out of habit because its fast and the issues can be
# a pain to track down if they come up and you dont realize a rebuild is needed
npm rebuild
```

**VERY IMPORTANT:** Windows users, if you use VS Code or use Powershell as your shell, you need to type `cmd` inside the VS Code terminal or inside your Powershell window before running the conda commands because conda's environment switcher will not work under Powershell (much of it works, but the critical parts that don't work, like activating evironments, fail silently while appearing to work),

```bash
# install Anaconda if not already installed

cmd # Only needed if you're coding on Windows in VS Code or Powershell, as discussed above
conda env create -f environment.yml
conda activate electron-python-sample
conda env list 
# in the list, make sure the electron-python-sample has a * in front
# indicating it is activated (under Powershell on Windows the activate
# command fails silently which is why you needed to run the conda commands
# in a cmd prompt)

# run the unpackaged python scripts from a dev build of electron
npm run start # must be run in the same shell you just conda activated
```

**NOTE** if you see the following error message when trying to `npm run start` it means you did not successfully `conda activate electron-python-sample` in the shell from which you are trying to `npm run start`. On Windows under VS Code that could be because you forgot to go into a `cmd` shell as discussed above before trying to conda activate.

```
Traceback (most recent call last):
  File "python/api.py", line 3, in <module>
    from graphene import ObjectType, String, Schema
ModuleNotFoundError: No module named 'graphene'
```

```bash
# use pyinstaller to convert the source code in python/ into an 
# executable in pythondist/, build the electron app into a subdirectory 
# of dist/, and run electron-packager to package the electron app as a 
# platform-specific installer in dist/
npm run build # must be run in the same shell you just conda activated

# double-click to run the either the platform-specific app that is built 
# into a subdirectory of dist/ or the platform-specific installer that is 
# built and placed in the dist/ folder
```

# Debugging the Python process

To test the Python GraphQL server, in a conda activated terminal window run `npm run python-build`, cd into the newly generated `pythondist` folder, and run `api.exe --apiport 5000 --signingkey devkey` then browse to `http://127.0.0.1:5000/graphiql/` to access a GraphiQL view of the server. For a more detailed example, try `http://127.0.0.1:5000/graphiql/?query={calc(math:"1/2",signingkey:"devkey")}` which works great if you copy and paste into the browser but which is a complex enough URL that it will confuse chrome if you try to click directly on it.

# Notes

The electron main process both spawns the Python child process and creates the window. The electron renderer process communicates with the python backend via GraphQL web service calls.

The Python script `python/calc.py` provides a function: `calc(text)` that can take text like `1 + 1` and return the result like `2.0`. The calc functionality is exposed as a GraphQL api by `python/api.py`.

The details of how the electron app launches the Python executable is tricky because of differences between packaged and unpackaged scenarios. This complexity is handled by `main/with-python.ts`. If the Electron app is not packaged, the code needs to `spawn` the Python source script. If the Electron app is packaged, it needs to `execFile` the packaged Python executable found in the app.asar. To decide whether the Electron app itself has been packaged for distribution or not, `main/with-python.ts` checks whether the `__dirname` looks like an asar folder or not.

# Important

Killing spawned processes under Electron can be tricky so the electron main process sends a message to the Python server telling it to exit when Electron is shutting down (and yes, that does mean that if you are debugging and control-c to kill the npm process hosting the app you can leave a zombie python process, so it's better to close the app normally by closing the window before killing your npm process).
