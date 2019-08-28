<h1>Ground-up tutorial of how to create packages for use with NI Package Manager</h1>
The purpose of this tutorial is to provide basic tutorial of how to create .nipkg files. It explores a couple of the different uses of the files including creating the packages with several methods, including multiple packages in a package, creating and using feeds, and a couple of other tricks.

<h2>Create a Simple Test Application</h2>
We want to create a simple test application. This application will consist of three main parts:
- A payload (In this case is an executable)
- Dependencies, which we will have two types:
    - NI Software (LabVIEW Run-Time Engine)
    - Third-party Software (A custom shared library (dll) that we are going to create a NIPM package to distribute)

<h3>Create the Shared Library</h3>
For this example, we are going to create a very simple library. Or you can download it from the resources file.

1. Create a new blank project
1. Right-click **My Computer** in the Project Explorer and select **New » VI**
1. Create a VI that displays a dialog box containing the contents of a string input and outputs the string afterwards

    ![Code Snippet](/Resources/displayDialog.png)
1. Connect the front panel controls and indicators to the connector pane
1. Save the VI as `displayDialog.vi`
1. Right-click on **Build Specifications** and select **New » Shared Library** to generate a dynamic link library (`.dll`)
1. Define the Packed Library Properties:
    1. Build Specification name: SimpleDLL
    1. Target filename: SimpleLib.dll
    1. Destination directory: `C:\Users\<user>\Documents\NIPM Package Tutorial\builds\SharedLibrary\My DLL`

    ![SimpleDLL Properties](/Resources/SimpleDLL%20Properties.PNG)
1. Under **Source Files** select the `displayDialog` and press arrow directing towards **Exported VIs**
1. A window will pop-up called *Define VI Prototype*, but if does not, press the **Define Prototype** button. It should all be the default values below:
    - StringIn

        ![Define VI Prototype - StringIn](/Resources/Define%20VI%20Prototype%20-%20StringIn.PNG)
    - StringOut

        ![Define VI Prototype - StringOut](/Resources/Define%20VI%20Prototype%20-%20StringOut.PNG)
    - Len

        ![Define VI Prototype - Len](/Resources/Define%20VI%20Prototype%20-%20len.PNG)
1. Select **OK**
1. Select **Build**. You should now be able to find your packed library in the builds folder
1. In Windows Explorer, create a folder in `C:\Program Files(x86)` called `SimpleLibrary`
1. Move `SimpleLib.dll` to `C:\Program Files(x86)\SimpleLibrary`

<h3>Create the Payload Executable</h3>
This will be the main application we want to distribute. It will depend on LabVIEW Run-Time Engine and our Simple Library.

1. Create a new blank project and save it as `mainApplication.lvproj`
1. For this example we are going to create a UI that takes in a text input, then displays it in a dialog box when we press the okay button. We will also have an exit button that stops the application. Save the vi as `main.vi`

    ![main.vi front panel](/Resources/mainVI.PNG)
1. Add a **Call Library Function Node** to the VI
1. Double-click the node and add the Library Path and Function Name

    ![Call Library Function](/Resources/Call%20Library%20Function.PNG)
1. Fill in the parameters to match the VI Prototype from the previous section
1. Create the block diagram as shown below.

    ![main.vi block diagram OK event case](/Resources/mainOKevent.png)
1. Here is the Exit event case

    ![main.vi block diagram EXIT event case](/Resources/mainEXITevent.png)
1. Test the VI and save
1. Right-Click **Build Specifications** and select **New » Application (EXE)**
1. In the My Application Properties fill out the following information:
    1. Build Specification Name: NIPM test
    1. Target filename: NIPMtest.exe
    1. Destination directory: `C:\Users\<user>\Documents\NIPM Package Tutorial\builds\mainApplication\NIPM test`
1. Under Source Files set the main.vi as the Startup VI.
1. Under Additional Exclusions check **Exclude dependent shared libraries**. This will prevent the dll from being added to executable.
1. Select build, then test the output to ensure that it works.
1. Move the executable to another location on the computer and test that it works again.

<h2>Create the NI Packages</h2>
There are three main ways of creating a package

1. Through the LabVIEW Application Builder
1. Using the standalone NI Package Builder application
1. Automating through the NI Package Manager command line interface

Lets start by creating a basic package for our dll using the LabVIEW Application Builder

<h3>Building a Basic Package - LabVIEW Application Builder</h3>

1. Open SharedLibrary.lvproj
1. Right-click **Build Specifications** and select **New » Package**
1. In the properties fill out the following information:
    1. Build Specification Name: SimpleLibraryPackage
    1. Package name: simple-library
    1. Destination directory: `C:\Users\<user>\Documents\NIPM Package Tutorial\builds\SharedLibrary\SimpleLibraryPackage\Package`
1. In **Destinations** create a new folder under the Program Files folder called `SimpleLibrary`
1. Under **Source Files** select `BuildSpecification » SimpleDLL` on the left and select `Program Files\SimpleLibrary` on the right, then press the arrow pointing to the right. If you select this, you cannot add individual files from the build specification, and you have to ensure that the build specification has already been built.
  1. Alternatively, add the `SimpleLib.dll` from `ProgramFiles(x86)\SimpleLibrary` to your project and select that from `My Computer » SimpleLib.dll` instead of selecting `BuildSpecification » SimpleDLL`
1. Under Package fill out the following:
    1. Display Name: Simple Library
    1. Package Synopsis: Simple Lib for NIPM Tutorial
    1. Section: Infrastructure (hidden)
1. Look under Dependencies and note that *LabVIEW Runtime (32-bit)* has already been selected. This means that LabVIEW Runtime will be installed when this package is installed; if it isn't already present. You see the specific package name under Package name (ni-labview-2019-runtime-engine-x86) we will use this later for the CLI version.
1. Note the version in the **Version** window
1. Under the **Feed** window we can add our package to a feed, which will let remotes fetch updates easier. We will skip this for now
1. Select Build
1. Note that the version number in the name of the package matches the version number in the build spec. As we update this build spec we can have the version number update automatically. This is useful for systems that are installing the package to know what the latest version is.
1. Double-click to install the package. Pass through the prompts until it says Installation complete.
1. Check `C:\Program Files (x86)\SimpleLibrary` to ensure the files are present. You can run the executable now too to ensure that it works as expected.

<h3>Uninstalling the Package</h3>
We are going to build this package again using the command line interface, so we have to know how to uninstall the package.

1. Open NI Package Manager from the start menu.
1. Your library should show up after following these steps:
    1. Select the **Installed** tab
    1. Under *Category* select **Infrastructure**
    1. Under *Maintainer* select **Unregistered**
    1. Deselect **Products only**
    1. Select the gear in the top right and check **Show full version numbers and hidden packages**
1. Select and remove the package

<h3>Build a Basic Package - NI Package Manager Command Line Interface</h3>

1. First, we are going to follow the directions here in the [help file](http://www.ni.com/documentation/en/ni-package-manager/latest/manual/assemble-file-package/). These directions are repeated below with some more hand holding.
1. I created a directory in my NIPM Package Tutorial folder called `\CLI Package\SimpleLibPkg` to store all of the files I will use.
1. Inside the root directory (`SimpleLibPkg`), create a folder called control.
1. Now lets create the control file using the directions from the [help](http://www.ni.com/documentation/en/ni-package-manager/latest/manual/assemble-file-package/)  
    1. Using a text editor, create a new document.
    1. Enter all control file attributes and values needed for your package. You can find a description of these [here](http://www.ni.com/documentation/en/ni-package-manager/latest/manual/control-file-attributes/). Enter the following for our purposes:
    ```
      Package: simple-library
      Version: 1.0.0.0
      Section: Infrastructure
      Architecture: windows_all
      Depends: ni-labview-2019-runtime-engine-x86
      Maintainer: Michael Bilyk <33259264+mbilyk@users.noreply.github.com>
      Description: Simple Lib for NIPM Tutorial
       A simple library for use in a NIPM Tutorial (github.com/mbilyk/NIPM-tutorial)
      XB-Plugin: file
    ```
    1. Note that the package has the same name as the LabVIEW Application Builder package properties. We also are setting the the rest of the attributes to be the same as in the LabVIEW Application Builder method. We set a dependency on ni-labview-2019-runtime-engine-x86. The maintainer is a required field, you will want to replace my name with your own and my email with yours.
    1. Save the file as control and omit the file extension.
1. Inside the root directory (`SimpleLibPkg`), create a folder called data. This is going to be our main location for what we want to be contained in the package.
1. In the data folder create a folder called `Program Files_32`
1. In `SimpleLibPkg\data\Program Files (x86)` create a folder called SimpleLibrary. Notice that this is the same directory we installed our previous package to.
1. Place our output dll into `SimpleLibPkg\data\Program Files (x86)\SimpleLibrary`
1. We are also going to skip creating an [instructions file](http://www.ni.com/documentation/en/ni-package-manager/latest/manual/instructions-xml-file-packages/) for now, since we will do it when we create a package for the executable.
1. Inside the root directory, create a Debian binary file. This file declares conformity to the Debian standard. For more information, visit [debian.org].
    1. Using a text editor, create a new document.
    1. Enter `2.0`
    1. Save the file as debian-binary and omit the file extension.
1. At this point your file directory should look like this:

    ![Package Directory Structure](/Resources/FileDirectory.png)
1. Now that we have our package directory set up we can use the command line to create the package.
1. Create a new folder in the `\CLI Package` called `Package Output`
1. Open a command prompt.
1. Change directories to the location of Package Manager. The default location is C:\Program Files\National Instruments\NI Package Manager. You can also add NI Package Manager to the %PATH% environment variable for your system.
1. Run the following command: nipkg pack <directory containing package source files> <destination of .nipkg file>. For example
    ```
        nipkg pack "C:\Users\<USER>\Documents\NIPM Package Tutorial\CLI Package\SimpleLibPkg" "C:\Users\<USER>\Documents\NIPM Package Tutorial\CLI Package\Package Output"
    ```
1. You should now have a package in the `NIPM Package Tutorial\CLI Package\Package Output` directory
1. Run the package and ensure that it installs correctly.
1. Your done! Now remove it using the above [instructions](#uninstalling-the-package) so we can do it again. Jump to here if you want to install a package using the [command line interface](#Installing-a-package-using-the-command-line-interface).

<h3>Build a Basic Package - NI Package Builder</h3>

1. Download and install NI Package Builder from NI Package Manager.
1. Open NI Package Builder from the start menu. I am using NI Package Builder 32-bit.
1. Click the New Package button in the middle of the screen.

    ![New Package Button](/Resources/NewPackage.png)
1. Start typing the name of your package. In this case, it is *Simple Library*. If you accidentally click off of the display name, you can right-click on the **Display name** in the *Packages* pane and select **Rename**.
1. With the `SimpleLibrary` package selected, in the *Properties* pane click on the `Package name` field (it should say `mycompany-myproduct`) we are going to name it `simple-library`. This will name the .nipkg file that same as in our other examples. You can also fill out any other properties you would like such as the Maintainer.

    ![Edit Properties](/Resources/EditProperties.png)
1. Change the **Category** to `Infrastructure (hidden)`.
1. Next we need inputs. In the *Inputs* pane click the **New Input** button and select **Add Folder**

    ![Add Item Button](/Resources/AddItem.png)
1. Select the folder that includes the built library: `NIPM Package Tutorial\builds\SharedLibrary\SimpleDLL`
1. Drag the `SimpleDLL` folder from the *Inputs* pane into the `[Program Files (32-bit)]` directory.

    ![Drag SimpleDLL into \[Program Files (32-bit)\]](/Resources/DragFolder.png)
1. Right-click on the `SimpleDLL` folder, select **Rename**, and rename it to `SimpleLibrary`.
1. Save the solution. I saved it here `NIPM Package Tutorial\builds\SharedLibrary\SimpleLibraryPackage\Package Builder`, and called it `SimpleLibrary.pbs`. You will have to create the folder.
1. Select **Build Solution**.

    ![Build Solution](/Resources/BuildSolution.png)
1. You should now have a file called `simple-package_1.0.0.0_windows_all.nipkg` in the folder `NIPM Package Tutorial\builds\SharedLibrary\SimpleLibraryPackage\Package Builder\Packages`
1. Double-click the package to install it and test it out.
1. Uninstall as usual.

<h2>Installing a Package Using the Command Line Interface</h2>
Generally you would want to do this for automating installation for a CI system or for IT deployment.

1. Open a command prompt.
1. Change directories to the location of Package Manager. The default location is C:\Program Files\National Instruments\NI Package Manager. You can also add NI Package Manager to the %PATH% environment variable for your system.
1. At this point you will probably want to open the help for nipkg install. Run the command `nipkg help install`. This will give the following information (at least in NI Package Manager 19.0):
    ```
    C:\Program Files\National Instruments\NI Package Manager>nipkg help install

    install - Installs a package

    nipkg install [OPTIONS]... PACKAGES...

       Install packages

       PACKAGES... - A list of all the packages to install.
                 A package can be specified by its name or by its path.
                 If a package name is specified, that name must exist in
                 one of the configured feeds.
                 If a package path is specified (a path to a file with
                 a .nipkg extension), the contents of this file are
                 passed to the install.

    OPTIONS
       --accept-eulas
          Confirm acceptance of license agreements.
          Packages that require license agreements cannot be installed unless
          this option is specified.
       -y,--yes,--assume-yes
          Execute the installation without user confirmation.
       --simulate
          Only simulate the installation.
       --force-locked
          Force the installation of locked packages.
       --suppress-incompatibility-errors
          Suppress forward incompatibility errors.
          WARNING: Suppressing errors could leave your system in an unusable state.
       --verbose, -v
          Display verbose output such as detailed package information on
          the current operation.
       --proxy=[PROXYHOST]
          Use the specified proxy server for requests to remote machines.
          Specify the server using the format [protocol://]proxyhost[:port]
          Supported protocols include socks4, socks5, and http.
       --proxy-user=[CREDENTIALS]
          Use the specified credentials to authenticate with the proxy server.
          Specify the credentials using the format [user]:[password]
       --allow-downgrade
          Allow Package Manager to downgrade installed packages to fulfill
          the request.
       --allow-uninstall
          Allow Package Manager to remove installed packages to fulfill
          the request.
       --install-also-upgrades
          Allow Package Manager to upgrade installed packages to fulfill
          an install request.


    GLOBAL OPTIONS
       --config=[PATH],-c=[PATH]
      Allows the caller to specify an absolute path to a custom configuration file.

    ALIASES
       pkg-install
       install-pkg
    ```
1. With this information, we know that in order to install our package we need to use the command:
    ```
    nipkg install --accept-eulas -y "C:\Users\<USER>\Documents\NIPM Package Tutorial\builds\SharedLibrary\SimpleLibraryPackage\Package\simple-library_1.0.0-0_windows_all.nipkg"
    ```
1. And that is it, it is installed. You do more complex operations that we will cover later when we want to install multiple packages at once.
