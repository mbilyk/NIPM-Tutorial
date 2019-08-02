This tutorial is based off this document:
https://forums.ni.com/t5/NI-Package-Management/Introduction-to-NIPM-Packages/ta-p/3805984?profile.language=en

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
![SimpleDLL Properties](/Resources/SimpleDLL Properties.png)
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
  1. Package name: simplelibrary
  1. Destination directory: `C:\Users\<user>\Documents\NIPM Package Tutorial\builds\SharedLibrary\SimpleLibraryPackage\Package`
1. In **Destinations** create a new folder under the Program Files folder called 'SimpleLibrary'
1. Under **Source Files** select `BuildSpecification » SimpleDLL` on the left and select `Program Files\SimpleLibrary` on the right, then press the arrow pointing to the right. If you select this, you cannot add individual files from the build specification, and you have to ensure that the build specification has already been built.
  1. Alternatively, add the `SimpleLib.dll` from `ProgramFiles(x86)\SimpleLibrary` to your project and select that from `My Computer » SimpleLib.dll` instead of selecting `BuildSpecification » SimpleDLL`
1. Under Package fill out the following:
  1. Display Name: Simple Library
  1. Package Synopsis: Simple Lib for NIPM Tutorial
  1. Section: Infrastructure (hidden)
1. Look under Dependencies and note that *LabVIEW Runtime (32-bit)* has already been selected. This means that LabVIEW Runtime will be installed when this package is installed; if it isn't already present. You see the specific feed name in under Package name (ni-labview-2019-runtime-engine-x86) we will get to feeds later.
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

1. First, we are going to follow the directions here in the help file (http://www.ni.com/documentation/en/ni-package-manager/latest/manual/assemble-file-package/). These directions are repeated below with some more hand holding.
1. I created a directory in my NIPM Package Tutorial folder called `\CLI Package\SimpleLibPkg` to store all of the files I will use.
1. Inside the root directory (`SimpleLibPkg`), create a folder called control.
1. In the control subdirectory, create a control file. Name it control and delete the extension. Then open it in a text editor.
1.
