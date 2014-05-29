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

    jumpstart@local:mab> api-generate android
    
This command would generate the mobile API in the following directory: `~/MABProjects/jumpstart/mobile/apis/assets/android`
    
#### Copy the Mobile APIs and Configuration files
You can copy the generated mobile assets to your Android project directory by running the following command on the Mobile App Builder tool:
    
    jumpstart@local:mab> exec cp -R ~/MABProjects/jumpstart/mobile/apis/assets/android/com/magnetapi/* </path/to/MyProject/src/com/magnetapi>
    
    jumpstart@local:mab> exec cp ~/MABProjects/jumpstart/mobile/apis/assets/android/*beans*.jar </path/to/MyProject/libs>
    
    jumpstart@local:mab> exec cp ~/MABProjects/jumpstart/mobile/apis/assets/android/magnet_type_mapper.xml </path/to/MyProject/assets>

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
    
    // Retrieve connection configuration for the backend from
    // assets/connection_configs.xml
    // TODO If no configuration, create one
    ConnectionConfig connConfig = cm.getConnectionConfig("jumpstart");

###### Initialize the HelloWorldController controller

      try {
        // instantiate controller factory
		HelloWorldControllerFactory cf = new HelloWorldControllerFactory(magnetClient);
		  
		// get an instance of the controller
        HelloWorldController hwController = cf.obtainInstance("jumpstart");
      } catch (SchemaException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
    
###### Call the HelloWorldController controller

      Call<String> call = hwController.postHello(input, new AsyncCallOptions());
	  // blocks until response is returned
	  String response = call.get();

#### Call the SimpleEntity controller API

The SimpleEntity controller API provides basic operations like create, read, update and delete on a sample Entity.

To call the SimpleEntity controller API, follow these steps:

###### Import the SimpleEntityController and SimpleEntityControllerFactory classes
    
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.SimpleEntityController;
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.SimpleEntityControllerFactory;
    
###### Initialize the SimpleEntityController controller

      try {
        // instantiate controller factory
		SimpleEntityControllerFactory ef = new SimpleEntityControllerFactory(magnetClient);
		  
		// get an instance of the controller
        SimpleEntityController entityController = ef.obtainInstance("jumpstart");
      } catch (SchemaException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
    
###### Call the SimpleEntityController controller

    // Initialize a SimpleEntityBean using the generated SimpleEntityBeanBuilder class
    SimpleEntityBeanBuilder builder = new SimpleEntityBeanBuilder();
    builder.name("John Appleseed").customerId(100);
    
    // Initialize a SimpleValueBean using the generated SimpleValueBeanBuilder class
    SimpleValueBeanBuilder valueBuilder = new SimpleValueBeanBuilder();
    valueBuilder.bigDecimal(BigDecimal.TEN);  // workaround since this column is not nullable
    valueBuilder.character("c");
    valueBuilder._boolean(false);
    simpleEntityBean.value = simpleValueBean;
  
    // set value of SimpleValueBean
    builder.value(valueBuilder.build());
    
    // Call the controller to create the SimpleEntityBean using null as async options
    Call<Integer> call = entityController.create(builder.build(), null);
    Integer id = call.get();

#### Putting it together
You can call the HelloWorld and SimpleEntity controller APIs by adding the code below to `HelloWorldActivity`:

Note: this code is strictly for demonstration purpose. Controller calls should not be invoked from onCreate() method as it incurrs a network request.

	import com.magnet.android.mms.MagnetMobileClient;
	import com.magnet.android.mms.async.AsyncCallOptions;
	import com.magnet.android.mms.async.Call;
	import com.magnet.android.mms.connection.ConnectionConfigManager;
	import com.magnet.android.mms.connection.ConnectionConfigManager.ConnectionConfig;
	import com.magnet.android.mms.exception.SchemaException;
	import com.magnetapi.apps.jumpstart.controllers.helloworld.api.HelloWorldController;
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.HelloWorldControllerFactory
	import com.magnetapi.apps.jumpstart.controllers.helloworld.api.SimpleEntityController;
    import com.magnetapi.apps.jumpstart.controllers.helloworld.api.SimpleEntityControllerFactory;
    public class HelloWorldActivity extends Activity {
    private static final String LOG_TAG = HelloWorldActivity.class.getSimpleName();
    private MagnetMobileClient magnetClient;
    private HelloWorldController hwController;  // for Hello World controller
    
    protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);

    	setContentView(R.layout.hello_world_activity);

    	// initialize MagnetMobileClient
    	magnetClient = MagnetMobileClient.getInstance(getApplicationContext());

      	HelloWorldControllerFactory cf = new HelloWorldControllerFactory(magnetClient);
      	ConnectionConfigManager cm = magnetClient.getManager(ConnectionConfigManager.class, this);
      	ConnectionConfig connConfig = cm.getConnectionConfig("jumpstart");
      	try {
        	if (connConfig != null) {
          		hwController = cf.obtainInstance("jumpstart");
     			Call<String> call = hwController.postHello(input, new AsyncCallOptions());
	  			// blocks until response is returned
	  			// *** Strictly for demonstration purpose and should not be in onCreate() method as it waits for network request to complete
	  			String response = call.get();

        		// instantiate controller factory
				SimpleEntityControllerFactory ef = new SimpleEntityControllerFactory(magnetClient);
		  
				// get an instance of the controller
        		SimpleEntityController entityController = ef.obtainInstance("jumpstart");
    			// Initialize a SimpleEntityBean
    			SimpleEntityBeanBuilder builder = new SimpleEntityBeanBuilder();
    			builder.name("John Appleseed").customerId(100);
    
    			// Initialize a SimpleValueBean
    			SimpleValueBeanBuilder valueBuilder = new SimpleValueBeanBuilder();
    			valueBuilder.bigDecimal(BigDecimal.TEN);  // workaround since this column is not nullable
    			valueBuilder.character("c");
    			valueBuilder._boolean(false);
    			simpleEntityBean.value = simpleValueBean;
  
    			// set value of SimpleValueBean
    			builder.value(valueBuilder.build());
    
    			// Call the controller to create the SimpleEntityBean using null as async options
    			Call<Integer> call = entityController.create(builder.build(), null);
	  			// blocks until response is returned
	  			// *** Strictly for demonstration purpose and should not be in onCreate() method as it waits for network request to complete

    			Integer id = call.get();
        	}
      	} catch (SchemaException e) {
        	// TODO Auto-generated catch block
        	Log.e(LOG_TAG, "failed to get instance of the controller");
        }
      }
    }


###6. Deploy the Mobile Backend
You can deploy the Mobile Backend server for the Jumpstart app to your local machine by running the following command on the Mobile App Builder tool:

    jumpstart@local:mab> server-start
    
###7. Run the app
You are now ready to run the Jumpstart app on the iOS simulator using Xcode!

###8. Where To Go From Here?
You can download the completed project from [here](https://someurl.git) or clone this project using git:

    $ git clone https://someurl.git
    
    


