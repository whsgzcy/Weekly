### 第三周 2020年3月23日

前言

永不背叛，王者背负，王者审判，王者不可阻挡

1、有这么一个场景，手机屏幕竖直方向倒置，普通的方法

```
ActivityInfo.SCREEN_xxx

等，只能设置 横屏与 正向竖屏

其他设置均不生效，就不要白费功夫了，

生效的方案有，

xxx.setScaleX(-1)

还有 封装一个Animation将 动画后的效果保存

建议使用setScaleX
```

2、DrawerLayout 与 fragment 滑动控件冲突问题

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <RelativeLayout
        android:id="@+id/menu_layout"
        android:layout_width="match_parent"
        android:layout_height="90dp"
        android:layout_alignParentBottom="true">

      xxx
    </RelativeLayout>

    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@+id/menu_layout" />

    <androidx.drawerlayout.widget.DrawerLayout
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

      xxxx

    </androidx.drawerlayout.widget.DrawerLayout>

</RelativeLayout>

这种情况 fragment的滑动控件将不生效，将DrawerLayout 与FrameLayout 互换位置，fragment是可以滑动的，但是，侧滑栏会被遮盖

解决方案如下

<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

      xxxxx

        <FrameLayout
            android:id="@+id/fragment_container"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_above="@+id/menu_layout" />

    </RelativeLayout>

    <RelativeLayout
        android:layout_width="160dp"
        android:layout_height="match_parent"
        android:layout_gravity="right">

        <LinearLayout
            android:id="@+id/menu_container"
            android:layout_width="160dp"
            android:layout_height="match_parent"
            android:layout_gravity="right"
            android:background="#eeeeee"
            android:orientation="vertical">

        </LinearLayout>

    </RelativeLayout>


</androidx.drawerlayout.widget.DrawerLayout>

```


### 第二周 2020年3月16日

前言

做坏蛋要有结局会悲剧的觉悟

1、手机插入USB HID HOST设备，读取数据时，以鼠标为例，如果usb上接入的为蓝牙，则此代码不生效

```
private final BroadcastReceiver mUsbDiskReceiver = new BroadcastReceiver() {
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (action.equals(UsbManager.ACTION_USB_DEVICE_ATTACHED)) {
                Toast.makeText(MainActivity.this, "发现新设备", Toast.LENGTH_SHORT).show();
            } else if (action.equals(UsbManager.ACTION_USB_DEVICE_DETACHED)) {
                mConnection = null;
                FLAG_ = -1;
                Toast.makeText(MainActivity.this, "拔出设备", Toast.LENGTH_SHORT).show();
            }
        }
    };
```

若插入有线usb鼠标，则上述代码生效，在android应用层 需要检查是否有读写的权限，

```
private final BroadcastReceiver mUsbPermissionActionReceiver = new BroadcastReceiver() {
			public void onReceive(Context context, Intent intent) {
					String action = intent.getAction();
					if (ACTION_USB_PERMISSION.equals(action)) {
							synchronized (this) {
									UsbDevice usbDevice = (UsbDevice) intent.getParcelableExtra(UsbManager.EXTRA_DEVICE);
									if (intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)) {
											if (null != usbDevice) {
													afterGetUsbPermission(usbDevice);
											}
									} else {
											Toast.makeText(context, String.valueOf("Permission denied for device" + usbDevice), Toast.LENGTH_LONG).show();
									}
							}
					}
			}
	};

	然后才能执行

	private void doYourOpenUsbDevice(UsbDevice usbDevice) {
        //now follow line will NOT show: User has not given permission to device UsbDevice
        mConnection = mUsbManager.openDevice(usbDevice);

        Toast.makeText(MainActivity.this, "open device success1", Toast.LENGTH_SHORT).show();
        mConnection.claimInterface(mInterface, true);
        if (mInterface != null) {
            for (int i = 0; i < mInterface.getEndpointCount(); i++) {

                UsbEndpoint ep = mInterface.getEndpoint(i);

                if (ep.getType() == UsbConstants.USB_ENDPOINT_XFER_INT) {
                    if (ep.getDirection() == UsbConstants.USB_DIR_IN) {
                        mInEndpoint = ep;
                    }
                }
            }
        }

        FLAG_ = 1;
    }

		读取数据的方式有两种，

		byte[] byte2 = new byte[16];
	    mConnection.bulkTransfer(mInEndpoint, byte2, byte2.length, 3000)

此方式读取的数据，有相当的延时，所以 不推荐使用，

			int inMax = mInEndpoint.getMaxPacketSize();
            ByteBuffer byteBuffer = ByteBuffer.allocate(inMax);
       	 UsbRequest usbRequest = new UsbRequest();
				 usbRequest.initialize(mConnection, mInEndpoint);
				 usbRequest.queue(byteBuffer, inMax);
				 if (mConnection.requestWait() == usbRequest) {
					 byte[] retData = byteBuffer.array();
					 StringBuffer sb = new StringBuffer();
					 for (Byte byte1 : retData) {
						 System.err.println(byte1);
						 sb.append(byte1);
					 }

```

对于读取设备数据的方式，我想 设备数据都是通过jni反射到java层，那么，在现有的api里面有没有这样的方式，不需要通过循环，

### 第一周 2020年3月8日

前言

小本本上的八卦羞答答，人生太复杂

1、我把嵌入式开发思路结合android启动流程梳理了一下

![git clone](https://raw.githubusercontent.com/whsgzcy/Weekly/master/images/kernel_android.png)

```
位于2位置，kernel层，主要是加载驱动和使能设备

3位置 c++ framework层 我认为是linux 应用开发层，调用各个驱动的代码，调试各个函数，最后以反射的形式 到1位置，所以，我认为，在java framework层，例如 WindowManagerService 点亮屏幕等操作，在其源码的作用处 肯定有个调用jni的方法，
往里面填数据，然后向下 作用硬件，所以，向下，肯定有个地方调用到屏幕的驱动代码

我觉得我对这个图 理解的比之前要深刻 我看到的不是图 我看到的是代码
```

2、今天把单片机的 gpio用例 调试起来，下一步应该就是 把温度传感器 调试起来，读到他的温度的数值

3、本周突然要解决一个问题，要用到文件的筛选，也就是从一个A文件中 剔除 B文件中的数据，其中A的范围要大于B，B也差不多含与A，各个情况都要考虑到，绝对不能有错，有错的话设备就不能用了，只有半个小时完成， 我立马想到的是 Beyond Compare 4 和 diff，简单的用了一下，发现只是比较对应行的变化，反馈的结果不可靠，所以还是乖乖写代码执行，以后再遇到这样的情况 直接copy就行了，

```
public class FileUtils {

	/**
	 * 读取
	 * @param filePath_ 文件路径
	 * @return
	 */
	public static List<String> read(String filePath_) {

		if (filePath_ == null) {
			System.out.println("filePath_ is null");
			return null;
		}

		List<String> lists = new ArrayList<>();

		/**
		 * 读取方法
		 */
		// 定义文件对象,
		// 可以指定路径,这里用的时相对路径
		File file = new File(filePath_);

		if (!file.exists()) {
			System.out.println(filePath_ + " is not exit, please check check it,");
			return null;
		}

		// 判断文件是否存在
		System.err.println(filePath_);

		try {
			// 1 创建文件输入流(字节流)
			FileInputStream fis = new FileInputStream(file);
			// 2 读取输入流(字符流)
			InputStreamReader isr = new InputStreamReader(fis, "UTF-8");
			// 3 缓冲读取InputStreamReader字节流
			BufferedReader br = new BufferedReader(isr);
			// 临时存放读取到的一行数据
			String line;
			// 开始读取
			// line等于读取到的一行数据,如果line读到的值不是空,说明还没有读完
			while ((line = br.readLine()) != null) {
				// 输出一行数据
//				System.out.println(line);
				lists.add(line);
			}

			// 关闭流,先打开流后关闭
			br.close();
			isr.close();
			fis.close();

		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}

		return lists;
	}

	/**
	 * 写入
	 * @param fileName 文件名称
	 * @param filePath 文件路径
	 * @param contents 传入写入lists
	 * @param msg 描述
	 */
	public static void write(String fileName, String filePath, List<String> contents, String msg) {
		/**
		 * 写入方法
		 */
		try {
			// 文件输出流
			// 新建文件对象
			// arg:新文件名,
			File newfile = new File(filePath + File.separator + fileName);
//			if (newfile.exists()) {
//				newfile.delete();
//			}
//
//			newfile.mkdirs()

			// 文件输出流,如果文件不存在,会自动创建这个文件(字节流),如果文件存在会被完全覆盖,
			// FileOutputStream(arg1:字符串或者文件名, arg2:true追加写入,false覆盖写入)
			FileOutputStream fos = new FileOutputStream(newfile, true);
			// 写入流(字符流)
			OutputStreamWriter osw = new OutputStreamWriter(fos, "UTF-8");
			// 缓冲写入
			BufferedWriter bw = new BufferedWriter(osw);

			if (contents == null || contents.size() == 0) {
				bw.write(msg + "\n");
				// 任务完成关闭流,先创建的后关闭
				bw.close();
				osw.close();
				fos.close();
				System.out.println("写入完成");
				return;
			}

			bw.write(msg + "\n");
			bw.write("total " + contents.size() + "\n");
			for (int i = 0; i < contents.size(); i++) {
				// 开始一行一行写入
				bw.write(contents.get(i) + "\n");
			}

			bw.close();
			osw.close();
			fos.close();
			System.out.println("写入完成");
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}

public class ComparedHandleService {

	public static String PATH_A = "/Users/super_yu/Desktop/xxx/compared/A.txt";
	public static String PATH_B = "/Users/super_yu/Desktop/xxx/compared/B.txt";

	public static String A1 = "A1.txt";

	public static String PATH = "/Users/super_yu/Desktop/xxx/compared/";


	// step 1 检查 文件中是否有重复的数据 并打印

	public List<String> checkIsHasRepeatingData(String path) {

		List<String> lists = FileUtils.read(path);

		if (lists == null || lists.isEmpty()) {
			System.out.println(path + " is null or empty");
			return null;
		}
		System.out.println(path + " counts = " + lists.size());

		boolean isRepeatingData = false;
		for (int i = 0; i < lists.size() - 1; i++) {

			String a = lists.get(i).trim();

			if (a == null || a.isEmpty()) {
				System.out.println(path + " has a blank space");
				return null;
			}

			for (int j = i + 1; j < lists.size(); j++) {

				String b = lists.get(j).trim();

				if (a.equals(b)) {
					isRepeatingData = true;
					System.out.println(path + " repeatingdata is " + a);
				}

			}
		}

		if (isRepeatingData) {
			System.out.println(path + " has repeating data");
			return null;
		}

		return lists;
	}

	// step 2比较两个数组 A 与 B 与之对应
	// 1、处理 将A中B部分删除 写入新的A1文件
	// 2、处理 打印 A 与 B共有文件的log 并打印个数
	// 3、处理 打印 A 与 B差异文件 即A中删除的对应行内容 并输出个数

	public void handleCompared(List<String> listA, List<String> listB) {

		if (listA == null || listA.isEmpty() || listB == null || listB.isEmpty())
			return;

		// 记录相同的部分 A 与 B 重合的部分
		List<String> list_a = new ArrayList<String>();
		List<String> listA1 = new ArrayList<String>();
		// 重合部分与B的差异 差异部分打印出来
		List<String> listB1 = new ArrayList<String>();

		for (int i = 0; i < listB.size(); i++) {

			for (int j = 0; j < listA.size(); j++) {

				if (listB.get(i).equals(listA.get(j))) {
					list_a.add(listB.get(i));
				}
			}
		}

		System.out.println("the same data counts = " + list_a.size());

		if (list_a.isEmpty()) {
			System.out.println("no same data");
			return;
		}

		listA1 = listA;

		for (Iterator<String> itA1 = listA1.iterator(); itA1.hasNext();) {
			String temp = itA1.next();
			for (int i = 0; i < list_a.size(); i++) {
				if (temp.equals(list_a.get(i)))
				{
					itA1.remove();
				}
			}
		}

		System.out.println("handled new A1 counts  = " + listA1.size());

		// A 与 B共有数据中 重合数据中 与 B的差异 输出
		if(list_a.size() != listB.size()){
			System.out.println("共有部分数据 并未完全重合与A");
			listB1 = listB;

			for (Iterator<String> itB1 = listB1.iterator(); itB1.hasNext();) {
				String temp = itB1.next();
				for (int i = 0; i < list_a.size(); i++) {
					if (temp.equals(list_a.get(i)))
					{
						itB1.remove();
					}
				}
			}		

			System.out.println("共有部分数据 并未完全重合与A counts = " + listB1.size());
			System.out.println("共有部分数据 并未完全重合与A ");

			for(int i = 0; i < listB1.size(); i++){
				System.out.println(listB1.get(i));
			}
		}

		FileUtils.write(A1, PATH, listA1, "");
	}

	public static void main(String[] args) {

		List<String> listsA = new ComparedHandleService().checkIsHasRepeatingData(PATH_A);
		List<String> listsB = new ComparedHandleService().checkIsHasRepeatingData(PATH_B);

		if (listsA == null || listsB == null)
			return;

		new ComparedHandleService().handleCompared(listsA, listsB);

	}

}
```
