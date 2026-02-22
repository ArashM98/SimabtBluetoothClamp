# SimabtBluetoothClamp
SDK for connecting Siamb Tashkhis Bluetooth Clamps

SimabtBluetoothClamp is an open source SDK, provided for Simab Tashkhis Co. Bluethooth Clamps
SimabtBluetoothClamp allows developers to find clamp and connect to it and get the data ! easy and super fast !

SimabtBluetoothClamp's primary focus is on easy to use




# Download

You can download a jar from GitHab's [release page](https://github.com/ArashM98/SimabtBluetoothClamp)

Or user Gradle :

Add it in your root settings.gradle at the end of repositories
```
dependencyResolutionManagement {
	repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
	repositories {
		mavenCentral()
		maven { url 'https://jitpack.io' }
	}
}
```
Add the dependency
```
dependencies {
        implementation 'com.github.ArashM98:SimabtBluetoothClamp:v2.0.1'
}

```

Or Maven : 
```
<dependency>
    <groupId>com.github.ArashM98</groupId>
    <artifactId>SimabtBluetoothClamp</artifactId>
    <version>v2.0.1</version>
</dependency>
```
# how do i use BluetoothClampLib ?

1 - Connect to SIMAB bluetooth clamp and get the device address

2 - Implement these classes in Activity that you want to use this SDK , and implement methods 
```
public class Main_Activity extends AppCompatActivity implements ServiceConnection, BlClampSerialListener {

    private BlClampSerialService service;
    private BlClampDTM0660Parser parser;
    private String packet = "";
    boolean flag;
    BluetoothDevice selectedDevice;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

		// bind the BlClampSerialService
        bindService(new Intent(this, BlClampSerialService.class), this, Context.BIND_AUTO_CREATE);

        parser = new BlClampDTM0660Parser(new BlClampDTM0660Parser.MultiListener() {
            @Override
            public void onVoltageReceived(String v) {
                Log.d("parse", "v:" + v);
            }

            @Override
            public void onCurrentReceived(String c) {
                Log.d("parse", "c:" + c);
            }

            @Override
            public void onFrequencyReceived(String f) {
                Log.d("parse", "f:" + f);
            }
        });
    }

    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        service = ((BlClampSerialService.SerialBinder) binder).getService();
        service.attach(this);
        try {
            BluetoothAdapter bluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
            BluetoothDevice simabDevice = bluetoothAdapter.getRemoteDevice(deviceAddress);
            BlClampSerialSocket socket = new BlClampSerialSocket(this.getApplicationContext(), simabDevice);
            service.connect(socket);
			Log.d("onServiceConnected", "simab device connected")

			************************************
			///// in this section after you see the log above press and hold "REL" button on the Simabt Clamp Device until you see read info in logs //////
			************************************

        } catch (Exception e) {
            onSerialConnectError(e);
        }
        Log.d("onServiceConnected", "connected : " + name);		
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
		service = null;
    }

    @Override
    public void onSerialConnect() {
        Log.d("onSerialConnect", "serial connected");
    }

    @Override
    public void onSerialConnectError(Exception e) {
		Log.d("onSerialConnectError", "error : " + e.getMessage());
    }

    @Override
    public void onSerialRead(byte[] data) {
        ArrayDeque<byte[]> datas = new ArrayDeque<>();
        datas.add(data);
        showInfo(data);
    }

    @Override
    public void onSerialRead(ArrayDeque<byte[]> data) {
        showInfo(data.getFirst());
    }

    @Override
    public void onSerialIoError(Exception e) {
        service.disconnect();
    }
}

```
in onServiceConnected method section after you see the log "simab device connected" , press and hold "REL" button on the Simabt Clamp Device until you see read info in logs

3 - showInfo method :

```
    private void showInfo(byte[] data) {
        try {
            if (data == null) {
                Log.d(TAG + "onSerialRead","no Data");
            } else {
                if (data.length > 0) {
                    String hex = BlClampHelper.toHexString(data, data.length);
                    for (int i = 0; i < hex.length(); i = i + 2) {

                        if (hex.length() >= i + 1) {
                            String OneByteHex = hex.substring(i, i + 2);
                            String oneByte = BlClampHelper.hexToBin(hex.substring(i, i + 2));

                            // check if reach end of packet
                            if (BlClampHelper.reachEnd(oneByte)) {
                                packet = OneByteHex;
                            } else if (BlClampHelper.reachEnd2(oneByte)) {
                                packet = packet + OneByteHex;
                                if (packet.length() == 30) {
                                    final String output = parser.parse(packet);
                                    runOnUiThread(() -> {
                                        Log.d(TAG + "onSerialRead", "output : " + output);
                                    });
                                }
                                packet = "";
                            } else
                                packet = packet + OneByteHex;
                        }
                    }
                }
            }
        } catch (Exception e) {
            Log.d("onSerialRead", "err : " + e.getMessage());
        }
    }

```

# Compatibility
* Minimum Android SDK: BluetoothClampLib requires a minimum API level of 21
* Compile Android SDK: BluetoothClampLib requires you to compile against Api 26 or later

# Getting help
To report a specific problem or feature request, open a new issue on Github. For questions, suggestions, or anything else, email Simab Tashkhis on [Website](https://simabt.com)

# Author
Arash Torkaman [@ArashM98](https://github.com/ArashM98) on Github

# License
Apache 2.0

# Disclaimer
This is not an official Google product
