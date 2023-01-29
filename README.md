# Entwicklung der Android-Präzisionsschrittzählung

 <div  align="center">    

</div>





# for thanks

 * [xbase](http://www.jianshu.com/p/5d57f7fd84fa)

 * [finnfu](https://github.com/finnfu/stepcount/tree/master/demo%E4%BB%A5%E5%8F%8A%E7%AE%97%E6%B3%95%E6%96%87%E6%A1%A3)

# 1. Sie müssen Berechtigungen .xml AndroidManifest hinzufügen

```xml
    <!-- need Permission-->
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
    <uses-feature android:name="android.hardware.sensor.accelerometer" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
    <uses-feature
        android:name="android.hardware.sensor.stepcounter"
        android:required="true" />
    <uses-feature
        android:name="android.hardware.sensor.stepdetector"
        android:required="true" />

```
# 2. test

```java
 /**
     *  ob es unterstuzen
     *
     * @param context
     * @return
     */
    @TargetApi(Build.VERSION_CODES.KITKAT)
    public static boolean isSupportStepCountSensor(Context context) {
        // 获取传感器管理器的实例 sensor
        SensorManager sensorManager = (SensorManager) context
                .getSystemService(context.SENSOR_SERVICE);
        Sensor countSensor = sensorManager.getDefaultSensor(Sensor.TYPE_STEP_COUNTER);
        Sensor detectorSensor = sensorManager.getDefaultSensor(Sensor.TYPE_STEP_DETECTOR);
        return countSensor != null || detectorSensor != null;
    }
```

# 3.Funktion

```java
   
    private boolean isBind = false;
    private Messenger mGetReplyMessenger = new Messenger(new Handler(this));
    private Messenger messenger;

    /**
     * Aktivieren Sie den Schrittzählerdienst
     */
    private void setupService() {
        Intent intent = new Intent(this, StepService.class);
        isBind = bindService(intent, conn, Context.BIND_AUTO_CREATE);
        startService(intent);


    }
    /**
     * brufen der Anzahl der Schritte vom Servicedienst
     
     *
     * @param msg
     * @return
     */
    @Override
    public boolean handleMessage(Message msg) {
        switch (msg.what) {
            case Constant.MSG_FROM_SERVER:
                cc.setCurrentCount(10000, msg.getData().getInt("step"));
                break;
        }
        return false;
    }


    /**
     * 用于查询应用服务（application Service）的状态的一种interface，
     * 更详细的信息可以参考Service 和 context.bindService()中的描述，
     * 和许多来自系统的回调方式一样，ServiceConnection的方法都是进程的主线程中调用的。
     */
    ServiceConnection conn = new ServiceConnection() {
        /**
         * 在建立起于Service的连接时会调用该方法，目前Android是通过IBind机制实现与服务的连接。
         * @param name 实际所连接到的Service组件名称
         * @param service 服务的通信信道的IBind，可以通过Service访问对应服务
        Diese Methode wird aufgerufen, wenn eine Verbindung zu einem Dienst hergestellt wird und Android derzeit die Verbindung zum Dienst über den IBind-Mechanismus implementiert.
         * @param Name Der Name der Dienstkomponente, mit der sie tatsächlich verbunden ist
         * IBind des Kommunikationskanals des @param Service-Dienstes, und der entsprechende Dienst kann über den Dienst aufgerufen werden
         
         */
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            try {
                messenger = new Messenger(service);
                Message msg = Message.obtain(null, Constant.MSG_FROM_CLIENT);
                msg.replyTo = mGetReplyMessenger;
                messenger.send(msg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        /**
         * 当与Service之间的连接丢失的时候会调用该方法，
         * 这种情况经常发生在Service所在的进程崩溃或者被Kill的时候调用，
         * 此方法不会移除与Service的连接，当服务重新启动的时候仍然会调用 onServiceConnected()。
         * @param name 丢失连接的组件名称
        Diese Methode wird aufgerufen, wenn die Verbindung zum Dienst unterbrochen wird.
         * Dies geschieht häufig, wenn der Prozess, in dem sich der Dienst befindet, abstürzt oder von Kill aufgerufen wird.
         * Diese Methode entfernt die Verbindung zum Dienst nicht und ruft weiterhin ServiceConnected() auf, wenn der Dienst neu gestartet wird.
         * @param Name Der Name der Komponente, die die Verbindung verloren hat
         */
        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

```

