`` 1、为什么同时多个设备连接时经常连接不成功？``

&nbsp;&nbsp;&nbsp;&nbsp;可能原因： 在使用 BluetoothDevice.connectGatt() 或者 BluetoothGatt.connect() 等建立 BluetoothGatt 连接的时候，在任何时刻都只能最多一个设备在尝试建立连接。如果同时对多个蓝牙设备发起建立 Gatt 连接请求。如果前面的设备连接失败了，后面的设备请求会被永远阻塞住，不会有任何连接回调。 建议： 如果要对多个设备发起连接请求，最好建立一个请求队列，前一个设备请求建立连接，后面请求在队列中等待 。如果连接成功了，就处理下一个连接请求。如果连接失败了（例如出错，或者连接超时失败），就马上调用 BluetoothGatt.disconnect()/close()来释放建立连接请求，然后处理下一个设备连接请求。

`` 2、为什么有时候连接成功了，但是发现不了服务及特征值，进而影响数据的接收和发送。``

&nbsp;&nbsp;&nbsp;&nbsp;连接成功后，会进行BluetoothGatt.discoverServices()去发现服务，进而设置特征值等，因为该方法是在主 线程中执行的，所以为了连接过程的可靠性，建议不要在该过程中，在主线程中不要处理太多的操作(尤其是频 繁绘制操作)。 

`` 3、为什么连接成功后，过不一会又断开了？``

 &nbsp;&nbsp;&nbsp;&nbsp;这个问题其实并不主要是客户端的问题，所以不要一味的在代码中找问题了，建议与硬件沟通，让其进行优化 ，如可以调整设备的连接参数（ConnectionInterval（连接间隔）、SlaveLatency（从设备延迟或者从设备时 延）、SupervisionTimeout（超时时间或者监控超时）），这三个参数是低功耗蓝牙中十分重要的连接参数， 一起决定了BLE的功耗，一般硬件设备会在APP连接成功时主动去更新一下这三个参数，以保证不同手机的差异 性得到一致，但是APP端是没办法控制这三个参数的。 

`` 4、最大连接数问题 ``

 Android BLE 设备的最大连接数是7，这是由蓝牙规范和 Android 系统的蓝牙堆栈实现决定的。

 >>> 1. 蓝牙规范

 蓝牙规范定义了两种类型的蓝牙设备：主设备和从设备。主设备可以连接多个从设备，但从设备只能连接一个主设备。

在 BLE 中，主设备和从设备之间的连接称为“逻辑链路”。每个逻辑链路都需要一个物理通道。蓝牙规范规定，每个主设备最多可以有7个物理通道。

>>> 2. Android 系统的蓝牙堆栈实现

Android 系统的蓝牙堆栈在内核空间和用户空间都有实现。内核空间的实现负责管理蓝牙硬件和底层协议，用户空间的实现负责提供应用程序接口。

在 Android 系统的蓝牙堆栈中，每个逻辑链路都由一个 BluetoothGatt 对象表示。BluetoothGatt 对象有一个 connect() 方法，用于建立连接。

```
public void connect(BluetoothDevice device, boolean autoConnect,
                   BluetoothGattCallback callback) {
    if (mBluetoothGatt != null) {
        mBluetoothGatt.close();
        mBluetoothGatt = null;
    }
    try {
        mBluetoothGatt = device.connectGatt(context, autoConnect, callback);
    } catch (IllegalArgumentException e) {
        Log.e(TAG, "Failed to connect to GATT", e);
        if (callback != null) {
            callback.onConnectionStateChange(device, STATE_DISCONNECTED,
                    STATUS_CONNECT_ERROR);
        }
    }
}
```

