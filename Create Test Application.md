<h1>Create a Simple Test Application</h1>
We want to create a simple test application. This application will consist of three main parts:

- A payload (In this case is an executable)
- Dependencies, which we will have two types:
    - NI Software (LabVIEW Run-Time Engine)
    - Third-party Software (A custom shared library (dll) that we are going to create a NIPM package to distribute)

<h2>Create the Shared Library</h2>
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

<h2>Create the Payload Executable</h2>
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
