# QtIFW-Advanced-Setup
Create "Qt Installer Framework" installers from your project via qmake

## Features
- Easily creation of installers via qmake
- Windows: Automatic inclusion of msvc libs as installation (if required)
- Extends default installer functionality:
	- provide uninstall only for offline installers
	- Hide uninstall/... by special command line switch
	- Adds local vs global installation
		- Global installation requires admin/root and chooses global location
		- Local installation uses user directories
		- Maintenancetool ensures it's privileged if installed globally
	- Windows: Allows to create a desktop shortcut
	- Windows: proper registration as "Installed Application"
	- Windows: Adds update/modify shortcuts to the windows startmenu directory
	- Linux: Automatic desktop file creation
- simple deployment as additional step before creating the installer
	- uses the qt deployment tools to deploy your application
	- option to include custom and Qt translations

## Installation
The package is providet as qpm package, [`de.skycoder42.qtifw-advanced-setup`](https://www.qpm.io/packages/de.skycoder42.qtifw-advanced-setup/index.html). To install:

1. Install qpm (See [GitHub - Installing](https://github.com/Cutehacks/qpm/blob/master/README.md#installing))
2. In your projects root directory, run `qpm install de.skycoder42.qtifw-advanced-setup`
3. Include qpm to your project by adding `include(vendor/vendor.pri)` to your `.pro` file

Check their [GitHub - Usage for App Developers](https://github.com/Cutehacks/qpm/blob/master/README.md#usage-for-app-developers) to learn more about qpm.

### Requirements
A pyhton script is used to generate the installer from the input. Thus, you need to have **Python 3** installed! If you want to use the deployment feature on linux, you need to have [linuxdeployqt](https://github.com/probonopd/linuxdeployqt) installed in your Qt installation

## Usage
Check the example application for a full demonstration. The idea is: You specify files and directories via your pro-file and run `make installer` to create the installer.

### Installer
This example shows the "minimal" input to create an installer:
```.pro
QTIFW_CONFIG = config.xml	#Configuration file, and other files for the config dir
QTIFW_MODE = online_all		#select the kind of installer to create

#define one package of your installer
sample.pkg = de.skycoder42.qtifwsample            #the package name
sample.meta = meta                                #directories with metadata (i.e. the "meta" directory of the package)
sample.dirs = data                                #directories with installation data (i.e. the "data" directory of the package)
win32: sample.files = $$shadowed($${TARGET}.exe)  #files to be copied to the data directory
else: ...

QTIFW_PACKAGES += sample #add all packages

# IMPORTANT! Setup the variables BEFORE including the pri file
include(vendor/vendor.pri)
```

To create the installer, simply run `make installer`.

#### Variable documentation
 Variable Name	| Default value								| Description
----------------|-------------------------------------------|-------------
 QTIFW_BIN		| `...`										| The directory containing the QtIFW Tools (repogen, binarycreator, etc.). The default value assumes you installed Qt and QtIFW via the online installer and that QtIFW is of version 2.0. Adjust the path if your tools are located elsewhere
 QTIFW_DIR		| `qtifw-installer`							| The directory (relative to the build directory) to place the installer files in
 QTIFW_MODE		| `offline`									| The type of installer to create. Can be:<br>`offline`: Offline installer<br>`online`: Online installer<br>`repository`: The remote repository for an online installer<br>`online_all`: Both, the online installer and remote repository
 QTIFW_TARGET	| `$$TARGET Installer`						| The base name of the installer binary
 QTIFW_TARGET_x	| win:`.exe`<br>linux:`.run`<br>mac:`.app`	| The extension of the installer binary
 QTIFW_CONFIG	| _must not be empty_						| Files for the configuration directory. **Must** contain a file named `config.xml` with the installer configuration
 QTIFW_PACKAGES	| _empty_									| A list of all packages to install. Must be of type `package`

 #### The `package` type
 All entries of the QTIFW_PACKAGES variable must be such entries. They are defined as "objects" with the following variables:

 Member Name	| Description
----------------|-------------
 pkg			| The unique name (identifier) of the package
 meta			| A list of directories to copy their contents into the packages "meta" directory
 dirs			| A list of directories to copy their contents into the packages "data" directory
 files			| A list of files to be copied into the packages "data" directory

### Deployment
As additional feature, you can generate a deployment target as well. This can be used by running `make deploy`. To use the feature, add the following to your pro file:
```pro
# automatically creates a default deployment target
CONFIG += qtifw_auto_deploy 

# if you have translations, specify the pro-file to be scanned for them
QTIFW_DEPLOY_TSPRO = $$_PRO_FILE_

# to include the deployed files into your installer either add the folder to <package>.dirs or do it automatically
sample.pkg = de.skycoder42.qtifwsample
sample.meta = meta
QTIFW_AUTO_INSTALL_PKG = sample

QTIFW_PACKAGES += sample
```

#### Variable documentation
 Variable Name			| Default value			| Description
------------------------|-----------------------|-------------
 QTIFW_DEPLOY_SRC		| _empty_				| The source file/directory to be deployed. Only one element, type defined by platform
 QTIFW_DEPLOY_OUT		| `$$OUT_PWD/deployed`	| The directory (relative to the build directory) to place the deployed files in
 QTIFW_DEPLOY_TSPRO		| _empty_				| `.pro` files, to be scanned for translations. They are generated automatically. If not empty, Qt-translations will be deployed automatically, too
 QTIFW_AUTO_INSTALL_PKG	| _empty_				| The name of a package to add the files generated by `qtifw_auto_deploy` to
 
#### The `qtifw_auto_deploy` configuration flag
If set (by adding `qtifw_auto_deploy` to your pro file), the deployment files are detected automatically. It's basically a shortcut for the code below:

```pro
win32:CONFIG(debug, debug|release): QTIFW_DEPLOY_SRC = $$shadowed(debug/$${TARGET}.exe)
else:win32:CONFIG(release, debug|release): QTIFW_DEPLOY_SRC = $$shadowed(release/$${TARGET}.exe)
else:mac: QTIFW_DEPLOY_SRC = $$shadowed($${TARGET}.app)
else: QTIFW_DEPLOY_SRC = $$shadowed($$TARGET)

!isEmpty(QTIFW_AUTO_INSTALL_PKG) { #NOTE: pseudo code, won't work like that
	mac: $$QTIFW_AUTO_INSTALL_PKG.dirs += $$OUT_PWD/deployed/$${TARGET}.app
	else: $$QTIFW_AUTO_INSTALL_PKG.dirs += $$OUT_PWD/deployed
}
```