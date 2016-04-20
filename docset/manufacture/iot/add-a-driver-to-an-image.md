---
Description: 'We''ll create some test files and registry keys to the image, again packaging them up so that they can be serviced after they reach your customers.'
title: 'Lab 2a: Add a driver to an image'
---

# Lab 2a: Add a driver to an image




We'll show you how to add drivers to a Windows 10 IoT Core image. 

When you're using a pre-built Board Support Package (BSP), you'll want to know whether there's already similar drivers on the board. If not, you can usually just add the new driver. 

If there is, then you'll need to replace the driver from the BSP, which we'll cover in Lab 2b.  



In our lab, we'll use the sample driver: [Hello, Blinky!](https://ms-iot.github.io/content/en-US/win10/samples/DriverLab.htm), package it up, and deploy it to the device.


## <span id="Prerequisites"></span><span id="prerequisites"></span><span id="PREREQUISITES"></span>Prerequisites

-  Complete Lab 1. [Lab 1a: Create a basic image](create-a-basic-image.md).



## <span id="Create_your_test_files"></span><span id="create_your_test_files"></span><span id="CREATE_YOUR_TEST_FILES"></span>Create your test files


-  Complete the exercises in [Installing The Sample Driver](https://ms-iot.github.io/content/en-US/win10/samples/DriverLab.htm) to build the Hello, Blinky app. You'll create three files: ACPITABL.dat, gpiokmdfdemo.inf, and gpiokmdfdemo.sys, which you'll use to install the driver.

   (You can also use your own IoT Core driver, so long as it doesn't conflict with the existing Board Support Package (BSP).


## <span id="Build_a_package_for_your_driver"></span><span id="build_a_package_for_your_driver"></span><span id="BUILD_A_PACKAGE_FOR_YOUR_DRIVER"></span>Build a package for your driver


1.  Run **C:\\IoT-ADK-AddonKit\\Tools\\IoTCoreShell** as an administrator.
2.  Create a working folder for the app, for example:

    ``` syntax
    newpkg pkgDrv Drivers HelloBlinky
    ```

    The new folder at **C:\\IoT-ADK-AddonKit\\Source-&lt;arch&gt;\\Packages\\Drivers.HelloBlinky\\**.


**Add sample files to the package**

1.  Update the app's package definition file, **C:\\IoT-ADK-AddonKit\\Source-&lt;arch&gt;\\Packages\\Drivers.HelloBlinky\\Drivers.HelloBlinky.pkg.xml**.

    The default package definition file includes sample XML that you can modify to add your own driver files.

    Update the values of Driver InfSource to point to your .INF file.

    Update the value of Reference to point to your .SYS file.
	
	Update the value of File Source to point to your ACPITABL.dat file.
    
    ``` syntax
    <?xml version="1.0" encoding="utf-8"?>
      <Package xmlns="urn:Microsoft.WindowsPhone/PackageSchema.v8.00"
        Owner="$(OEMNAME)"
        OwnerType="OEM"    
        ReleaseType="Production"
		Platform="arm"
		Component="Drivers"
		SubComponent="HelloBlinky">
        <Components>
		  <Driver
		    InfSource="$(PRJDIR)\Packages\Drivers.HelloBlinky\gpiokmdfdemo.inf">
			<Reference Source="$(PRJDIR)\Packages\Drivers.HelloBlinky\gpiokmdfdemo.sys" />
			<Files>
			  <File Source="$(PRJDIR)\Packages\Drivers.HelloBlinky\ACPITABL.dat" />
            </Files>
          </Driver>
        </Components>
      </Package> 
    ```


2.  From the IoT Core Shell, build the package. (The BuildAllPackages tool builds everything in the source folders.)

    ``` syntax
    createpkg c:\IoT-ADK-AddonKit\Source-<arch>\Packages\Drivers.HelloBlinky\Drivers.HelloBlinky.pkg.xml
    ```

    The package is built, appearing as **C:\\IoT-ADK-AddonKit\\Build\\&lt;arch&gt;\\pkgs\\&lt;your OEM name&gt;.Drivers.HelloBlinky.cab**.

    
## <span id="Update_your_feature_manifest"></span><span id="update_your_feature_manifest"></span><span id="UPDATE_YOUR_FEATURE_MANIFEST"></span>Update your feature manifest


**Add your driver package to the feature manifest**

1.  Open the architecture-specific feature manifest file, **C:\\IoT-ADK-AddonKit\\Source-<arch>\\Packages\\OEMFM.xml**
2.  Create a new PackageFile section in the XML, with your package file listed, and give it a new FeatureID, such as "OEM\_DriverHelloBlinky".

    ``` syntax      
          <PackageFile Path="%PKGBLD_DIR%" Name="%OEM_NAME%.File.TestFileAndRegKey.cab">
            <FeatureIDs>
              <FeatureID>OEM_DriverHelloBlinky</FeatureID>
            </FeatureIDs>
          </PackageFile>
    ```

    You'll now be able to add your driver to your product by adding a reference to this feature manifest.

## <span id="Update_the_project_s_configuration_files"></span><span id="update_the_project_s_configuration_files"></span><span id="UPDATE_THE_PROJECT_S_CONFIGURATION_FILES"></span>Update the project's configuration files


## <span id="Update_the_project_s_configuration_files"></span><span id="update_the_project_s_configuration_files"></span><span id="UPDATE_THE_PROJECT_S_CONFIGURATION_FILES"></span>Update the project's configuration files

1.  Open your product's test configuration file: **C:\\IoT-ADK-AddonKit\\Source-arm\\Products\\ProductA\\TestOEMInput.xml**.
2.  Make sure your feature manifest, OEMFM.xml, is in the list of AdditionalFMs. Add it if it isn't there already there:

    ``` syntax
      <AdditionalFMs>
        <AdditionalFM>%AKROOT%\FMFiles\arm\IoTUAPNonProductionPartnerShareFM.xml</AdditionalFM>
        <AdditionalFM>%AKROOT%\FMFiles\arm\IoTUAPRPi2FM.xml</AdditionalFM>
        <AdditionalFM>%AKROOT%\FMFiles\arm\RPi2FM.xml</AdditionalFM>
        <AdditionalFM>%SRC_DIR%\Packages\OEMFM.xml</AdditionalFM>
        <AdditionalFM>%COMMON_DIR%\Packages\OEMCommonFM.xml</AdditionalFM>
      </AdditionalFMs>
    ```

3.  Add the FeatureID for your file and reg key:

    ``` syntax
    <OEM> 
    <Feature>RPI2_DRIVERS</Feature> 
    <Feature>RPI2_DEVICE_TARGETINGINFO</Feature> 
    <Feature>PRODUCTION</Feature> 
    <Feature>OEM_CustomCmd</Feature> 
    <Feature>OEM_AppxHelloWorld</Feature> 
    <Feature>OEM_FileAndRegKey</Feature> 
    <Feature>OEM_DriverHelloBlinky</Feature> 
    </OEM>
    ```

## <span id="Build_and_test_the_image"></span><span id="build_and_test_the_image"></span><span id="BUILD_AND_TEST_THE_IMAGE"></span>Build and test the image


**Build the image**

1.  From the IoT Core Shell, create the image:

    ``` syntax
    createimage ProductA Test
    ```

    This creates the product binaries at C:\\IoT-ADK-AddonKit\\Build\\&lt;arch&gt;\\ProductA\\Flash.FFU.

2.  Start **Windows IoT Core Dashboard** &gt; **Setup a new device** &gt; **Custom**, and browse to your image. Put the Micro SD card in the device, select it, accept the license terms, and click **Install**. This replaces the previous image with our new image.
3.  Put the card into the IoT device and start it up.

    After a short while, the device should start automatically, and you should see your app.

**Check to see if your driver works**

1.  Use the procedures in the [Hello, Blinky! lab](https://ms-iot.github.io/content/en-US/win10/samples/DriverLab3.htm) to test your driver.



## <span id="Next_steps"></span><span id="next_steps"></span><span id="NEXT_STEPS"></span>Next steps

That's it for now, check back later for info on replacing a driver.

<!-- 
[Lab 2b: Add a provisioning package to an image](add-a-provisioning-package-to-an-image.md)
-->

 

 


