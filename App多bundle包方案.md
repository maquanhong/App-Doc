# 一、全量包方案

## 一、打bundle包命令

### Android版本bundle包命令

`react-native bundle --platform android --dev false --entry-file index.js --bundle-output bundle/android/logistics.bundle --assets-dest bundle/android/assets`

### iOS版本bundle包命令

`react-native bundle --platform ios --dev false --entry-file index.js --bundle-output bundle/ios/logistics.jsbundle --assets-dest bundle/ios/assets`

**需要在工程根目录下事先创建bundle/android和bundle/ios文件夹**

## 二、加载离线包

### 一、Android加载离线包

1、把bundle包添加到工程的assets目录或者通过网络下载，在assets中的bundle包需要拷贝到App的外部存储中，拷贝代码如下

```java
InputStream is = mReactContext.getAssets().open("logistics.bundle");
File newFile = new File(mReactContext.getExternalFilesDir(null), "logistics.bundle");
FileOutputStream fos = new FileOutputStream(newFile);

int len = -1;
byte[] buffer = new byte[1024];
while ((len = is.read(buffer)) != -1) {
    fos.write(buffer, 0, len);
}
fos.close();
is.close();
```

 2、加载bundle包要创建独立的Application、ReactActivityDelegate和ReactActivity

（1）创建ReactActivity

```java
public class MyActivity  extends ReactActivity {
    @Nullable
    @Override
    protected String getMainComponentName() {
        String componetName = "logistics";
        return componetName;
    }

    @Override
    protected ReactActivityDelegate createReactActivityDelegate() {
        return new MyReactActivityDelegate(this,getMainComponentName());
    }
}
```

​    getMainComponentName方法中返回的componentName就是AppRegistry.registerComponent注册的名字。

（2）创建ReactActivityDelegate

```java
public class MyReactActivityDelegate extends ReactActivityDelegate {


    public MyReactActivityDelegate(ReactActivity activity, @Nullable String mainComponentName) {
        super(activity, mainComponentName);
    }

    @Override
    public ReactNativeHost getReactNativeHost() {
        return MyReactApplication.getInstance().getReactNativeHost();
    }
}
```

（3）创建独立的Application

```java
public class MyReactApplication extends Application implements ReactApplication {
    public static MyReactApplication mInstance;
    private WeakReference<Application> appReference;
    public static  ReactNativeHost mReactNativeHost;

    private MyReactApplication() {

    }

    public static MyReactApplication getInstance() {
        if (mInstance == null) {
            synchronized (MyReactApplication.class) {
                if (mInstance == null) {
                    mInstance = new MyReactApplication();
                }
            }
        }
        return mInstance;
    }

    public void init(Application application) {
        appReference = new WeakReference<>(application);
        mReactNativeHost = new ReactNativeHost(application) {
            @Override
            public boolean getUseDeveloperSupport() {
                return false;
            }

            @Override
            protected List<ReactPackage> getPackages() {
                List<ReactPackage> packages = new PackageList(this).getPackages();
                // Packages that cannot be autolinked yet can be added manually here, for example:
                // packages.add(new MyReactNativePackage());
                packages.add(new NativeReactPackage());
                return packages;
            }

            @Nullable
            @Override
            protected String getBundleAssetName() {
                return "logistics.bundle";
            }

            @Nullable
            @Override
            protected String getJSBundleFile() {
                File bundleFile = application.getExternalFilesDir(null);
                File dataFile=new File(bundleFile,"logistics.bundle");
                Log.d("TAG", "getJSBundleFile: " + dataFile.getAbsolutePath());
                return dataFile.getAbsolutePath();
            }

        };
    }
    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }
}
```

（4）在MainApplication的onCreate方法中初始化独立的Application

```java
public void onCreate() {
  super.onCreate();
  SoLoader.init(this, /* native exopackage */ false);
  initializeFlipper(this, getReactNativeHost().getReactInstanceManager());
  MyReactApplication.getInstance().init(this);
}
```

### 二、加载iOS bundle包

1.把bundle包拷贝到工程中或者通过网络下载

2.加载离线包

```objective-c
AppDelegate *delegate = (AppDelegate *)([UIApplication sharedApplication].delegate);
  UINavigationController *rootNav = delegate.navController;
  RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:self launchOptions:delegate.launchOptions];
  NSArray *moduleArray = bridge.moduleClasses;
  RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                           moduleName:@"AwesomeProject"
                       initialProperties:nil];
  DemoViewController *nativeVC = [[DemoViewController alloc] init];
  nativeVC.view = rootView;
  [rootNav pushViewController:nativeVC animated:YES];

- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
{
//  return [[NSBundle mainBundle] URLForResource:@"logistics" withExtension:@"jsbundle"];
  return [[NSBundle mainBundle] URLForResource:@"awesome" withExtension:@"jsbundle"];
}
```

以上就是加载离线包的整个方案。

# 二、差量包





















