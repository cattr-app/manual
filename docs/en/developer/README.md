# Developer documentation

## Build the Cattr desktop client :id=cattr-client-build

The desktop client is an Electron application for Windows, macOS, and Linux. Its current native dependency stack requires the Node.js and npm versions listed below.

### Build requirements

- x64 Windows, macOS, or Linux
- Node.js 14.21.x
- npm 9.9.4
- Python 3.10 and a native C/C++ toolchain
- Git

On macOS, install Xcode from the [Apple Developer website](https://developer.apple.com/xcode/). On Debian/Ubuntu, install the build dependencies:

~~~bash
sudo apt-get update
sudo apt-get install -y git cmake curl python3 build-essential pkg-config \
  libsecret-1-0 libsecret-1-dev ca-certificates openssh-client dpkg-dev dpkg-sig
~~~

On Windows, install Python 3.10 and Visual Studio 2022 Build Tools with the Desktop development with C++ workload. A native Windows build does not require Docker.

Use a Node version manager such as [nvm](https://github.com/nvm-sh/nvm) to install Node.js 14.21.x, then install the npm version used by the project:

~~~bash
nvm install 14.21
nvm use 14.21
npm install --global npm@9.9.4
~~~

### Get the source and run development mode

~~~bash
git clone https://github.com/cattr-app/desktop-application.git
cd desktop-application
npm ci
npm run build-development
npm run dev
~~~

On Windows, use npm run dev-win instead of npm run dev. Development mode stores its data separately from the regular client profile.

### Create production packages

Set the application version, build the renderer, and package for the current platform:

~~~bash
npm ci
npm --no-git-tag-version version 1.0.0
npm run build-production
npm run package-linux
~~~

Choose the packaging command for your target:

| Target | Command | Output |
| --- | --- | --- |
| macOS, signed and notarized | npm run package-mac | DMG; requires Apple signing credentials |
| macOS, unsigned | npm run package-mac-unsigned | DMG |
| Linux | npm run package-linux | AppImage, DEB, and tar.gz |
| Windows | npm run package-windows | NSIS installer and portable executable |

Packages are written to target/. macOS packages must be built on macOS. Linux can also build Windows packages when Wine is installed; Windows builds Windows packages only.
