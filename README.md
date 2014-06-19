## Magnet Jumpstart App for Android

The Magnet Jumpstart App for Android is licensed under the terms of the [Magnet Software License Agreement](http://www.magnet.com/resources/tos.html).  Please see [LICENSE](LICENSE) file for full details.

In this tutorial, you will learn how to build a "Jumpstart" Android app that interacts with a "Jumpstart" Mobile Backend running locally.

###1. Prerequisites
1. Mobile App Builder tool (Installation instructions can be found [here](http://instructionsForMABInstall.magnet.com).
2. [Android SDK](http://developer.android.com/tools/index.html) with Eclipse, minimally Android 4.1.2, API Level 16.
3. [Magnet Mobile Server SDK for Android](https://github.com/magnetsystems/magnet-sdk-android)

###2. Build the Mobile Backend

#### Use the Mobile App Builder tool to first build a Mobile Backend server.
The following command will automatically build a sample Mobile Backend server for the Jumpstart app that contains two controller APIs: HelloWorld and basic operations like create, read, update and delete on a sample Entity:

    jumpstart@local:mab> run jumpstart.mab

You can also find detailed instructions for building the above Mobile Backend server from scratch [here](http://buildMobileBackendServer.magnet.com).

###3. Create Android App Project
Create a new Android Application called "Jumpstart" using Eclipse


###4. Import Mobile Server SDK and copy generated mobile API assets:

#### Generate the Mobile APIs
You can generate the mobile API by running the following command on the Mobile App Builder tool:

    jumpstart@local:mab> api-generate -wf android
    
This command would generate the mobile API in the following directory: `~/MABProjects/jumpstart/mobile/apis/assets/android`
    
#### Copy the Mobile APIs and Configuration files
You can copy the generated mobile assets to your Android project directory by running the following command on the Mobile App Builder tool:
    
    jumpstart@local:mab> exec cp -R ~/MABProjects/jumpstart/mobile/apis/assets/android/com/magnetapi/* </path/to/MyProject/src/com/magnetapi>
    
    jumpstart@local:mab> exec cp -R ~/MABProjects/jumpstart/mobile/apis/assets/android/com/magnet/* </path/to/MyProject/src/com/magnet>
    
    jumpstart@local:mab> exec cp ~/MABProjects/jumpstart/mobile/apis/assets/android/*beans*.jar </path/to/MyProject/libs>
    
    jumpstart@local:mab> exec cp ~/MABProjects/jumpstart/mobile/apis/assets/android/magnet_type_mapper.xml </path/to/MyProject/res/xml>

#### Import Mobile Server SDK as library project to Eclipse
Create the Magnet library as an "Android Library" project and include it in your Android app as a dependency:

1. File->New Project->Android->Android Project from Existing Code
2. Browse to the unzipped Magnet library project directory and select "libproject/2.3.0" as the "Root Directory".
A new Android project will be created with the name "magnetlib-2.3.0"
4. Select the newly created library project and right click Properties, under "Android", "is library" must be checked.
5. From your main Android application project, add the library project under Project Properties->"Android", select the "magnetlib-2.3.0" library project and add it.
6. Build your main Android application project using Eclipse.



###5. Use the Mobile APIs

#### Call the HelloWorld controller API

The HelloWorld controller API concatenates the string `Hello ` with the input string argument and returns it. For example, given the input string argument `Magnet` it returns the string `Hello Magnet`.

To call the HelloWorld controller API, follow these steps:

###### Import the HelloWorldController and HelloWorldControllerFactory classes
    
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.HelloWorldController;
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.HelloWorldControllerFactory;
    
###### Initialize the MagnetMobileClient instance and connection configuration
	// Initialize MagnetMobileCient for this Activity
	MagnetMobileClient magnetClient = MagnetMobileClient.getInstance(getApplicationContext());
	
    // Get instance of connection configuration manager
    ConnectionConfigManager cm = magnetClient.getManager(ConnectionConfigManager.class, this);
    
    // Retrieve connection configuration named "jumpstart" from
    // assets/connection_configs.xml
    // If no configuration, create one
    // hostUrl = URL to the jumpstart backend. If the backend is running locally,
    // set to "http://10.0.2.2:8080/rest" for app running on the Android emulator
    ConnectionConfig connConfig = cm.getConnectionConfig("jumpstart");
    if (connConfig == null) {
          String hostUrl = "http://10.0.2.2:8080/rest";
          connConfig = cm.addOrReplaceConnectionConfig("jumpstart", 
          Uri.parse(hostUrl),
          ConfigType.MAGNET_REST, MagnetRestAuthHandler.class, "magnet");
    }

###### Initialize the HelloWorldController controller

     try {
       // instantiate controller factory
	HelloWorldControllerFactory cf = new HelloWorldControllerFactory(magnetClient);
	  
	// get an instance of the controller
       HelloWorldController hwController = cf.obtainInstance("jumpstart");
     } catch (SchemaException e) {
	   Log.e(LOG_TAG, "can't get HelloWorldController", e);
     }
    
###### Call the HelloWorldController controller

     Call<String> call = hwController.postHello("Magnet", new AsyncCallOptions());
     // blocks until response is returned
     String response = call.get();

#### Call the SimpleEntity controller API

The SimpleEntity controller API provides basic operations like create, read, update and delete on a sample Entity.

To call the SimpleEntity controller API, follow these steps:

###### Import the SimpleEntityController and SimpleEntityBean related classes
    
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.SimpleEntityController;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.SimpleEntityControllerFactory;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.bean.SimpleEntityBean;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.bean.SimpleEntityBeanBuilder;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.bean.SimpleValueBeanBuilder;
    
###### Initialize the SimpleEntityController controller

     try {
       // instantiate controller factory
	SimpleEntityControllerFactory ef = new SimpleEntityControllerFactory(magnetClient);
	  
	// get an instance of the controller
       SimpleEntityController entityController = ef.obtainInstance("jumpstart");
     } catch (SchemaException e) {
   	Log.e(LOG_TAG, "can't get HelloWorldController", e);
     }
    
###### Call the SimpleEntityController controller

    // Initialize a SimpleEntityBean using the generated SimpleEntityBeanBuilder class
    SimpleEntityBeanBuilder builder = new SimpleEntityBeanBuilder();
    builder.name("John Smith")
           .customerId(100);
    
    // create a SimpleValueBean, set its string value
    SimpleValueBeanBuilder svb = new SimpleValueBeanBuilder();
    svb.string("simple value string");
    
    // set SimpleValueBean instance to SimpleEntityBean "value" field
    builder.value(svb.build());      

    // Call the controller to create the SimpleEntity using null as async options
    Call<Integer> call = entityController.create(builder.build(), null);
    Integer id = call.get();

#### Putting it together
You can call the HelloWorld and SimpleEntity controller APIs by adding the code below to `JumpstartActivity`:

Note: this code is strictly for demonstration purpose.


    package com.magnet.apps.jumpstart;
    
    import java.util.concurrent.ExecutionException;
    import android.app.Activity;
    import android.net.Uri;
    import android.os.Bundle;
    
    import com.magnet.android.mms.MagnetMobileClient;
    import com.magnet.android.mms.async.AsyncCallOptions;
    import com.magnet.android.mms.async.Call;
    import com.magnet.android.mms.connection.ConnectionConfigManager;
    import com.magnet.android.mms.connection.ConnectionConfigManager.ConnectionConfig;
    import com.magnet.android.mms.connection.ConnectionConfigManager.ConnectionConfig.ConfigType;
    import com.magnet.android.mms.connection.MagnetRestAuthHandler;
    import com.magnet.android.mms.exception.SchemaException;
    import com.magnet.android.mms.utils.logger.Log;
    
    // generated from "api-generate" mab command
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.HelloWorldController;
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.HelloWorldControllerFactory;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.SimpleEntityController;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.SimpleEntityControllerFactory;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.bean.SimpleEntityBeanBuilder;
    import com.magnetapi.apps.jumpstart.controllers.simplecontroller.api.bean.SimpleValueBeanBuilder;
    
    public class JumpstartActivity extends Activity {
    	private static final String LOG_TAG = HelloWorldActivity.class.getSimpleName();
    	private MagnetMobileClient magnetClient;
    	
    	protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);

    	// initialize MagnetMobileClient
    	magnetClient = MagnetMobileClient.getInstance(getApplicationContext());
    	// Get instance of connection configuration manager
    	ConnectionConfigManager cm = magnetClient.getManager(
        	ConnectionConfigManager.class, this);

    	// set connection configuration
    	ConnectionConfig connConfig = cm.getConnectionConfig("jumpstart");
    	if (connConfig == null) {
      		String hostUrl = "http://10.0.2.2:8080/rest";
      		connConfig = cm.addOrReplaceConnectionConfig("jumpstart",
          		Uri.parse(hostUrl), ConfigType.MAGNET_REST,
          	MagnetRestAuthHandler.class, "magnet");
    	}
    	try {
      		// instantiate controller factory
      		HelloWorldControllerFactory cf = new HelloWorldControllerFactory(
          		magnetClient);

	      // get an instance of the controller
	      HelloWorldController hwController = cf.obtainInstance("jumpstart");
	      Call<String> call = hwController.postHello("Magnet",
	          new AsyncCallOptions());
	      // blocks until response is returned
	      // *** Strictly for demonstration purpose and should not be in onCreate()
	      // method because 'get' is a blocking call, waiting for network request to
	      // complete
	      String response = call.get();
	    } catch (SchemaException e) {
	      Log.e(LOG_TAG, "can't get HelloWorldController", e);
	    } catch (ExecutionException e) {
	      Log.e(LOG_TAG, "failed to execute postHello", e);
	      e.printStackTrace();
	    } catch (InterruptedException e) {
	      Log.e(LOG_TAG, "postHello interrupted", e);
	    }

	    try {
	      // instantiate controller factory
	      SimpleEntityControllerFactory ef = new SimpleEntityControllerFactory(
	          magnetClient);
	
	      // get an instance of the controller
	      SimpleEntityController entityController = ef.obtainInstance("jumpstart");
	
	      // Initialize a SimpleEntityBean using the generated
	      // SimpleEntityBeanBuilder class
	      SimpleEntityBeanBuilder builder = new SimpleEntityBeanBuilder();
	      builder.name("John Smith").customerId(100);
	
	      // create a SimpleValueBean, set its string value
	      SimpleValueBeanBuilder svb = new SimpleValueBeanBuilder();
	      svb.string("simple value string");
	
	      // set SimpleValueBean instance to SimpleEntityBean "value" field
	      builder.value(svb.build());
	
	      // Call the controller to create the SimpleEntity using null as async
	      // options
	      Call<Integer> call = entityController.create(builder.build(), null);
	      // blocks until response is returned
	      Integer id = call.get();
	
	    } catch (SchemaException e) {
	      Log.e(LOG_TAG, "failed to get instance of the controller");
	    } catch (ExecutionException e) {
	      Log.e(LOG_TAG, "failed to execute create entity", e);
	    } catch (InterruptedException e) {
	      Log.e(LOG_TAG, "create entity interrupted", e);
	    }
	  }
	}



###6. Deploy the Mobile Backend
You can deploy the Mobile Backend server for the Jumpstart app to your local machine by running the following command on the Mobile App Builder tool:

    jumpstart@local:mab> server-start
    
###7. Run the app
You are now ready to run the Jumpstart app on the Android device!

###8. Where To Go From Here?
You can download the completed project from [here](https://someurl.git) or clone this project using git:

    $ git clone https://someurl.git
    
    


