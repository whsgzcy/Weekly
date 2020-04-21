### 第七周 2020年4月21日

前言

观今夜星象 知天下大事

1、二分法的巧妙运用

```
在这里我只能分享思路，分享需求的话不合适，当我们从一堆sharedpreferencesd的xml文件中找出自己想要的，比如调试qq或者微信，那么可以用二分法来调试，到此就不能再具体了

有一串shell脚本，得在这里备注一下，因为网上都是乱七八糟的东西，自己都没调试过就放上去

find /xxx/xxx/xxx -type f -not -name 'xxx.xxx' -not -name 'xxx.xxx' | xargs rm
```

2、tcpip wifi

```
说是四次交互，但实际是五次，其实我不理解这个，但到kernel这层的时候，我是可以理解的

我们都知道 是否连接到外网 通过ping的方式，但ping实际等的是pong，他有一个超时时间，例如200ms，做递减，此时包的协议是逐步增加的，也就是在每一层长度都是增加的，然后包回来的时候匹配到对应的路径下去，得到延时时间
```

3、Ubuntu安装vbox(VM)

```
注意，这个是个巨坑！！！

现象，我们安装好vbox之后，再去安装虚拟机，Ubuntu或者Windows，会出现这样一个错，

***************** ******************** ********************* ***************
Kernel driver not installed (rc=-1908)

The VirtualBox Linux kernel driver (vboxdrv) is either not loaded or there is a permission problem with /dev/vboxdrv.
Please reinstall the kernel module by executing

'/sbin/vboxconfig'

as root

where:sulibOslnit what:3

xxxxx
xxxxx
**************** ******************** ********************** ***************

这个，网上乱七八糟的东西一大堆，我想诺诺的问一句，你们自己验证过吗？

我们从这个应用的log可以看出:

***************** ******************** ********************* ***************
vboxdrv:
Running module version sanity check.
 -Origin module
  -No origin module exists within this kernel
 -Installation
  -Installation to /lib/modules/4.15.0-45-generic/updates/dkms

vboxnetadp.ko:
Running module version sanity check.
 -Origin module
  -No origin module exists within this kernel
  -Installation
   -Installation to /lib/modules/4.15.0-45-generic/updates/dkms

vboxnetflt.ko:
Running module version sanity check.
 -Origin module
  -No origin module exists within this kernel
  -Installation
  -Installation to /lib/modules/4.15.0-45-generic/updates/dkms

vboxpci.ko:
Running module version sanity check.
 -Origin module
  -No origin module exists within this kernel
  -Installation
  -Installation to /lib/modules/4.15.0-45-generic/updates/dkms

  ...

depmod.....

DKMS: install completed.

他明显就是缺少驱动啊，在Ubuntu下处理这个问题的最好方式是：

sudo apt-get -y install virtualbox

***************** ******************** ********************* ***************

```

4、记录一串shell脚本，特别容易出错

```
result=`ssh $HOSTCHECK $IP 'xxx xxx xxx'`
#echo $result

// 此时的if 如果少了一个空格都会出错，切记
if [ -n "$result" ] && [ "null" != "$result" ]
then
        echo "$IP" ";yes;"  "$result"
  else
        echo "$IP" ";no"
fi
```

5、与代码无关的东西

```
三年前我爽在专业面前教过我

stay hungry stay foolish

解释：在专业面前保持敬畏

三年后，我爽在感情方面教我

一期一会
```

6、与代码有关的东西
```
绝不能容忍bug
```

### 第六周 2020年4月14日

前言

除了我,猴子是世界第一。 在猴哥面前,一点也不谦虚。除了我,猴子是世界第一,干脆说你就是...

1、ws、wss、http、https，我们在开发中总会遇到，它的原理是什么，可能也没必要花时间去了解，我也不想在这上面花时间，这周在开发上遇到的一个坑，我想把它记录下来，

```
当遇到用go语言写的server的时候，一定得注意

put Origin https与wss http与ws 尤其是遇到

org.java-websocket:Java-WebSocket:1.4.1

的时候，这个坑不能跳，钦此。
```

### 第五周 2020年4月7日

前言

给予破坏者正确的绝望

1、有这么一个场景，王者荣耀游戏进行热更，我们把新的文件提取出来，放置到其他设备对应的路径下，那其他设备就不需要再去热更，100台设备，可以这么操作，那么10000台呢？两天两夜，如果硬盘性能还不是很好的情况下，大批量的去mount的地址去拉数据，并发数再设置不当，就会出现卡死，而且，卡死的原因可能不止是软件层面的问题，还有硬件层面的问题

```
方案进展

1、王者荣耀热更时的检测

我尝试过以下方案

a、抓包
很可惜，大厂的游戏这块都是严格加密过的，pass

b、分析当前的进程，找到当前热更的子进程
没找到，pass

c、反编译
想多了，pass

d、抓取增量
我想找到这方面的增量文件，找了很多很多，对比也很多很多，pass

e、图像识别
我看可行，然，需求还没有到这地步，pass

那么，对于大文件，大量的热更处理是怎样处理的呢？

我只能分享思路，

a、拿一台设备具体分析当前热更的大小，时间，下载速度，以及分析当前带宽

b、拿50台设备，具体分析当前热更时间，已经那段时间的带框

c、上述如果都在预期之内，则设置并发，休眠时间

d、最终测试，10000台设备，3.3小时搞定

这种方式还是会产生大量的流量，只是过渡方案，或者是出现问题的保底方案

最终方案设想，准备一个非常非常非常牛逼的硬盘，10000个设备，1000个一组，将存档路径指向那个地址即可

```

2、游戏奔溃处理

```
我们都知道，sharedpreferences，只是对应目录下的xml，我们通过增量的方式将文件取出来，进行对比，其实这个问题是有难度的，但解决起来不难，比较难定位这个问题，因为游戏奔溃只能看到log，所以，我认为有些莫名其妙的问题肯定是细节问题，不能大刀阔斧，否则很累。

```

3、HextoStr

```
这个代码要记得，网上一大片都是垃圾

//16进制字符串转换为byte[]
    public static byte[] HexToByte(String hexString) {
        int len = hexString.length();
        byte[] b = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {
            // 两位一组，表示一个字节,把这样表示的16进制字符串，还原成一个字节
            b[i / 2] = (byte) ((Character.digit(hexString.charAt(i), 16) << 4) + Character
                    .digit(hexString.charAt(i + 1), 16));
        }
        return b;
    }
```

4、倒置时，滑动的处理

```
这是一段 向右滑动的代码，根据偏移量实时绘制托条位置，但现在的需求是，手机竖直方向倒置，手机放过来，在源码基础上，我找到这块代码，分析

从左向右，即自增，比如
0 1 2 3
倒置之后，即自减，比如
3 2 1 0

只修改了，
//mDistanceX = event.getRawX() - mDownX;
  mDistanceX = mDownX - event.getRawX();

...
@Override
        public boolean onTouchEvent(MotionEvent event) {
            if (mEnable) {
                if (event.getAction() == MotionEvent.ACTION_DOWN) {
                    // 记录手指按下时SlideIcon的X坐标
                    mDownX = event.getRawX();
                    return true;
                } else if (event.getAction() == MotionEvent.ACTION_UP) {
                    // 设置手指松开时SlideIcon的X坐标
                    mDownX = 0;
                    mX = mX + mDistanceX;
                    mDistanceX = 0;
                    // 触发松开回调并传入当前SlideIcon的X坐标
                    if (listener != null) {
                        listener.onActionUp((int) mX);
                    }
                    return true;
                } else if (event.getAction() == MotionEvent.ACTION_MOVE) {
                    // 记录SlideIcon在X轴上的拖动距离
//                    mDistanceX = event.getRawX() - mDownX;
                    mDistanceX = mDownX - event.getRawX();
                    // 触发拖动回调并传入当前SlideIcon的拖动距离
                    if (listener != null) {
                        listener.onActionMove((int) mDistanceX);
                    }
                    return true;
                }
                return false;
            } else {
                return true;
            }
        }
    }
...
```

5、计划

```
HID设备程序调试，本周一定得拿出10个小时时间，专注在这方面，

```

### 第四周 2020年3月31日

pass

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
