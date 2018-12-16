# Windows实验代码

## 1. 文本文件合并

1. 选择文件目录对话框：`System.Windows.Forms.FolderBrowserDialog`

```c#
using System.Windows.Forms;
// 选择文件目录
private void button1_Click(object sender, RoutedEventArgs e)
{
 	FolderBrowserDialog folderBrowerDialog1 = new FolderBrowserDialog();
    folderBrowerDialog1.RootFolder = Environment.SpecialFolder.MyComputer;
    if (folderBrowerDialog1.ShowDialog() == System.Windows.Forms.DialogResult.OK)
    {
    	folder_path = folderBrowerDialog1.SelectedPath;
    }

}
```

2. 在所有文件中查找：`System.IO.Directory`

```c#
if (Directory.Exists(folder_path)) //检查文件目录是否存在
{   //搜索给定字符串的文件
	folder_files = Directory.GetFiles(folder_path, textBox1.Text,   
                                      SearchOption.AllDirectories);
	...
}
```

3. 选择目标文件：`System.Windows.Forms.SaveFileDialog`，同理还有`OpenFileDialog`，用法与此类似。

```c#
//选择目标文件
using System.Windows.Forms;

public static string dest_file;
private void button7_Click(object sender, RoutedEventArgs e)
{
	SaveFileDialog saveFileDialog1 = new SaveFileDialog();
    saveFileDialog1.Title = "选择要合并后的文件";
    saveFileDialog1.InitialDirectory = 
        System.Environment.SpecialFolder.DesktopDirectory.ToString();
    saveFileDialog1.OverwritePrompt = false;
    if (saveFileDialog1.ShowDialog() == System.Windows.Forms.DialogResult.OK)
    {
    	dest_file = saveFileDialog1.FileName;
        label3.Content = dest_file;
    }
}
```

4. 打开并写入文件：`System.IO.File`、`System.IO.FileStream`

```c#
private void button11_Click(object sender, RoutedEventArgs e)
{
	if (File.Exists(dest_file)) { // 文件若存在先删除
    	File.Delete(dest_file);
    }

	FileStream fs_dest = new FileStream(dest_file, FileMode.CreateNew, 
                                        FileAccess.Write);
    byte[]DataBuffer =  new byte[100000];
    byte[] file_name_buf;
    FileStream fs_source = null;
    int read_len;
    FileInfo fi_a = null;

    //逐个文件写入
   	for (int i= 0; i< listBox2.Items.Count; i++){
        fi_a = new FileInfo(listBox2.Items[i].ToString());
    	file_name_buf = Encoding.Default.GetBytes(fi_a.Name);

    	//写入文件名
        if (checkBox2.IsChecked == true)
        {
        	fs_dest.Write(file_name_buf, 0, file_name_buf.Length);
        }
                
                
        //换行
        if (checkBox1.IsChecked == true)
        {
        	fs_dest.WriteByte((byte)13);
            fs_dest.WriteByte((byte)10);
         }
         
         fs_source = new FileStream(fi_a.FullName, FileMode.Open, FileAccess.Read);
         read_len = fs_source.Read(DataBuffer, 0, 100000);
         while (read_len > 0)
         {
         	fs_dest.Write(DataBuffer, 0, read_len); ;
            read_len = fs_source.Read(DataBuffer, 0, 100000);
         }

         // 换行
         if (checkBox1.IsChecked == true)
         {
          	fs_dest.WriteByte((byte)13);
           	fs_dest.WriteByte((byte)10);
         }

            fs_source.Close();
    }

    //关闭文件流并打开目标文件
	fs_source.Dispose();
    fs_dest.Flush();
    fs_dest.Close();
    fs_dest.Dispose();
    if (checkBox3.IsChecked == true)
    {
        Process.Start(dest_file);
    }
           
}
```

## 2. C++编写DLL，C#调用

1. 使⽤用C++创建类库(DLL):Visual C++——> Win32项⽬目——> 应⽤用程序类型:DLL。
2. 编写头文件CppDll.h，声明导出函数，注意声明格式：

```c++
#pragma once

#ifndef CPPDLL_H
#define CPPDLL_H 

extern "C" _declspec(dllexport) int _stdcall add(int a, int b);
extern "C" _declspec(dllexport) int _stdcall minus(int a, int b);
extern "C" _declspec(dllexport) int _stdcall times(int a, int b);
extern "C" _declspec(dllexport) int _stdcall divide(int a, int b);

#endif // CPPDLL_H
```

2. 编写CppDll.cpp文件，导出函数实现在这里：

```c++
// CppDll.cpp : 定义 DLL 应用程序的导出函数。
#include "CppDll.h"

int _stdcall add(int a, int b) {
	return a + b;
}
...
}
```

3. 编写模块定义文件CppDll.def，将要在dll中显示的函数声明在这里：

```c++
Library "CppDll"

EXPORTS

add @ 1
minus @ 2
```

4. 在Release模式下生成，即可得到对应的dll文件。
5. 在C#中调用，无需using，而是用DllImport。

```c#
[DllImport(@"../../../Release/CppDll.dll", EntryPoint = "times", SetLastError = true, CharSet = CharSet.Ansi, ExactSpelling = false, CallingConvention = CallingConvention.StdCall)]
extern static int times(int a, int b);
```

## 3. C#编写DLL，C#调用

1. 新建C#项目时选择【类库】，然后在类中正常写函数即可，假设对应的命名空间为CSharpDll
2. 生成项目的时候会在bin/Debug中生成对应的dll文件。
3. 在另一个C#项目的引用中添加该dll：点击【添加引用】---》【浏览】—》找到该dll
4. 使用前`using CSharpDll;`然后就可以正常使用该命名空间下的类及其方法。

## 4. C#编写并调用COM

> 参考：https://blog.csdn.net/kingmax54212008/article/details/73604195?utm_source=blogxgwz4

1. 创建C#项目，选择【类库】。

2. 修改Properties目录下面的AssemblyInfo.cs，ComVisible属性设置为true。

   ```c#
   // 将 ComVisible 设置为 false 会使此程序集中的类型
   //对 COM 组件不可见。如果需要从 COM 访问此程序集中的类型
   //请将此类型的 ComVisible 特性设置为 true。
   [assembly: ComVisible(true)]
   
   // 如果此项目向 COM 公开，则下列 GUID 用于类型库的 ID
   [assembly: Guid("5709cddf-f127-41b7-80c6-6ccedf7d9938")]
   ```

3. 打开【项目】---》【属性】，切换到“生成”选项卡，在底部位置，勾选“为COM互操作注册”。

4. 打开【项目】---》【属性】，切换到“签名”选项卡勾选“为程序集签名”，在下面的下拉框里面选择“<新建...>”，在弹出的对话框里面，输入密钥文件名称，例如MyKey，去掉“使用密码保护文件(P)”的选项，会在项目中生成MyKey.snk。

5. 定义接口，注意生成GUID——GUID属性里面的那个字符串，在“工具”菜单下面，“创建 GUID”

     选择 [GUID(“xxxxxxxxxx-xxxx….xxxxxx”)]，然后复制。

```c#
[Guid("5459E45E-3AA9-4C09-9C1D-B77263690E28")]
[ComVisible(true)]
public interface IOperate
{
    int add(int a, int b);
}
```

6. 定义类实现接口，注意生成GUID。

```c#
[Guid("EE1C5250-7C64-4DDB-9207-3EBE24CF8BC6")]
[ComVisible(true)]
[ClassInterface(ClassInterfaceType.None)]
[Description("进行算术操作")]
public class myOperate : IOperate
{
	public int add(int a, int b)
	{
		return a + b;
	}

}
```

7. 编译：生成->生成解决方案（F6）。注意要生成.tlb文件，如果没有生成**.tlb**文件，需要用管理员身份打开Visual Studio，然后重新编译。

8. 注册：开始→所有程序→Microsoft Visual Sutdio 2017→Visual Studio Tools→Visual Studio命令提示符（2017）〖注：以管理员身份运行〗

   1. 在命令提示符下面，进入Dll所在的目录

      C:\Windows\system32>cd/d E:\MyLib\MyLib\bin\Debug

   2. 用 gacutil /i MyLib.dll 将这个DLL加入的全局缓存里

      E:\MyLib\MyLib\bin\Debug>gacutil /i mylib.dll

   3. 然后用 regasm MyLib.dll 注册这个dll

      E:\MyLib\MyLib\bin\Debug>regasm mylib.dll

9. 在另一个C#项目中调用COM。

## 6. 进程间通信——重定向

```c#
class Program
{
    static void ExecuteCmd(string cmd)
    {
        Process process = new Process();
        process.StartInfo.FileName = "cmd.exe";
        // 是否使用外壳程序   
        process.StartInfo.UseShellExecute = false;
        // 重定向输入流  
        process.StartInfo.RedirectStandardInput = true;
        // 重定向输出流
        process.StartInfo.RedirectStandardOutput = true;
        //打开程序
        process.Start();
        // 输入到程序输入流
        process.StandardInput.WriteLine(cmd);
        process.StandardInput.WriteLine("exit");
        // 获取输出信息
        string result = process.StandardOutput.ReadToEnd();
        Console.WriteLine(result);
        process.WaitForExit();
        process.Close();
    }

    static void Main(string[] args)
    {
        ExecuteCmd("getmac");
        Console.ReadLine();
        ExecuteCmd("shutdown /s /t 3600"); // shuwdown computer in 3600 seconds
        ExecuteCmd("shutdown /a");  // cancel shutdown plan
    }
}
```

## 7. 进程间通信——命名管道

1. 客户端：

   ```C#
   using System.IO.Pipes;
   
   public partial class Form1 : Form
   {
   	// 定义命名管道客户端流（用于读写）
   	NamedPipeClientStream pipeClient = new NamedPipeClientStream("localhost", 
   	"testpipe", PipeDirection.InOut, PipeOptions.Asynchronous, 
   	TokenImpersonationLevel.None);
   	// 声明写入流对象
   	StreamWriter sw = null;
   
   	//连接命名管道客户端
   	private void Form1_Load(object sender, EventArgs e)
   	{
   		try
   		{
   			// 连接命令管道客户端
   			pipeClient.Connect(5000);
   			// 创建写入流对象，参数指明写入的位置
   			sw = new StreamWriter(pipeClient);
   			sw.AutoFlush = true;
   		}
   		catch (Exception ex)
   		{
   			MessageBox.Show("连接建立失败，请确保服务端程序已经被打开。");
   			this.Close();
   		}
   
   	}
   	
   	private void button1_Click(object sender, EventArgs e)
   	{
   		if (sw != null)
   		{
   			sw.WriteLine(this.textBox1.Text);
   		}
   		else
   		{
   			MessageBox.Show("未建立连接，不能发送消息。");
   		}
   
   	}
   }
   ```

2. 服务器端：

   ```C#
   using System.IO.Pipes;
   
   public partial class Form1 : Form
   {
   	// 定义命名管道服务器端流（用于读写）
   	NamedPipeServerStream pipeServer = new NamedPipeServerStream("testpipe", 
   	PipeDirection.InOut, 1, PipeTransmissionMode.Message, 
   	PipeOptions.Asynchronous);
   	
   	private void Form1_Load(object sender, EventArgs e)
   	{
   		ThreadPool.QueueUserWorkItem(delegate
   		{
   			// 等待连接
   			pipeServer.BeginWaitForConnection((o) =>
   			{
   				NamedPipeServerStream pServer = 
   				(NamedPipeServerStream)o.AsyncState;
   				pServer.EndWaitForConnection(o);
   				StreamReader sr = new StreamReader(pServer);
   				while (true)
   				{
   					this.Invoke((MethodInvoker)delegate {
   					listBox1.Items.Add(sr.ReadLine()); });
   				}
   			}, pipeServer);
   		});
   
   	}
   }
   ```

## 8. 生产者消费者

```c#
// 信号量
static Semaphore fillCount = new Semaphore(0, BUFFER_SIZE);
fillCount.WaitOne();
fillCount.Release();

// 互斥量
static Mutex bufferMutex = new Mutex();
mutex.WaitOne();
mutex.ReleaseMutex();

//线程操作：创建、开始、等待一起结束
threads[0] = new Thread(new ThreadStart(producer)); // producer为线程执行函数
threads[1] = new Thread(new ThreadStart(consumer)); // consumer为线程执行函数
threads[0].Start();
threads[1].Start();
threads[0].Join();
threads[1].Join();
```

## 9. 消息机制

1. 发消息：

```c#
#region 定义常量消息值
public const int WM_COPYDATA1 = 0x005B;
#endregion

#region 定义结构体
public struct COPYDATASTRUCT
{
    public IntPtr dwData;
    public int cbData;
    [MarshalAs(UnmanagedType.LPStr)]
    public string lpData;
}
#endregion


#region 动态链接库引入
[DllImport("User32.dll", EntryPoint = "SendMessage")]
private static extern int SendMessage(
    IntPtr hWnd, // handle to destination window 
    int Msg, // message 
    int wParam, // first message parameter 
    ref COPYDATASTRUCT lParam // second message parameter 
);
#endregion

private void sendMessage1(string msg)
{
    // 或者通过枚举进程的方式找到目标进程句柄，然后发送消息
    Process[] procs = Process.GetProcesses();
    foreach (Process p in procs)
    {
        if (p.ProcessName.Equals("WPFSendMsg"))
        {
                    // 获取目标进程句柄
            IntPtr hWnd = p.MainWindowHandle;

                    // 封装消息
            byte[] sarr = System.Text.Encoding.Unicode.GetBytes(msg);
            int len = sarr.Length;
            COPYDATASTRUCT cds2;
            cds2.dwData = (IntPtr)0;
            cds2.cbData = len + 1;
            cds2.lpData = msg;

                    // 发送消息
            SendMessage(hWnd, WM_COPYDATA1, 0, ref cds2);
        }
    }
}
```

2. 收消息：

```C#
// 添加事件处理
public MainWindow()
{
    InitializeComponent();
    SourceInitialized += HandleInitialized;
}

//添加钩子程序
public void HandleInitialized(object o, EventArgs e)
{
    HwndSource hWndSource;
    WindowInteropHelper wih = new WindowInteropHelper(this);
    hWndSource = HwndSource.FromHwnd(wih.Handle);
    //添加处理程序 
    hWndSource.AddHook(MainWindowProc);
}

//钩子函数，处理所收到的消息
private IntPtr MainWindowProc(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
{
    switch (msg)
    {
        case WM_COPYDATA1:
            try
            {
                COPYDATASTRUCT mystr = new COPYDATASTRUCT();
                Type mytype = mystr.GetType();

                COPYDATASTRUCT MyKeyboardHookStruct = 
                    (COPYDATASTRUCT)Marshal.PtrToStructure(lParam, 
                                                           typeof(COPYDATASTRUCT));
                showComment1(MyKeyboardHookStruct.lpData);
                break;
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
                break;
            }
        default:
        {
            break;
        }
    }
    return hwnd;
}
```

## 9. 事件机制

```c#
//事件参数类
public class SwimEventArgs : EventArgs
{
    public SwimEventArgs(string river, string people)
    {
        this.river = river;
        this.people = people;
    }
        public string river; //河流
        public string people; //落水人
}

//事件源（发起者）类定义
public class SwimAlarm
{
    //将落水处理定义为SwimEventHandler 代理(delegate) 类型，这个代理声明的事件的参数列表
    public delegate void SwimEventHandler(object sender, SwimEventArgs se);
    //定义FireEvent 为FireEventHandler delegate 事件(event) 类型.
    public event SwimEventHandler SwimEvent;

    //激活事件的方法，创建了SwimEventArgs 对象，发起事件，并将事件参数对象传递过去
    public void ActivateSwimAlarm(string river, string people)
    {
        SwimEventArgs swimArgs = new SwimEventArgs(river, people);
        //执行对象事件处理函数指针，必须保证处理函数要和声明代理时的参数列表相同
        SwimEvent(this, swimArgs);
    }
}

    //用于处理事件的类：SwimHandlerClass，这个类定义了实际事件处理代码
class SwimHandlerClass
{
    private MainWindow parent;
    //事件处理类的构造函数使用事件源类作为参数
    public SwimHandlerClass(MainWindow _parent, SwimAlarm swimAlarm)
    {
        parent = _parent;
        //将事件处理的代理(函数指针) 添加到SwimAlarm 类的SwimEvent 事件中，当事件发生时，
        //就会执行指定的函数；
        swimAlarm.SwimEvent += new SwimAlarm.SwimEventHandler(HelpPeople);
    }
    
    //当落水事件发生时，用于处理落水的事件
    void HelpPeople(object sender, SwimEventArgs se)
    {
        parent.showComment2(sender.ToString() + " 对象调用，求人事件HelpPeople 函数.");
        //根据落水状况，输出不同的信息.
        if (se.river == "长江")
        {
            parent.showComment2(" 落水人是"+se.people+"，落水发生在" + se.river + 
                                "，必须派船只救人！");
        }
        else
        {
            parent.showComment2(" 落水人是" + se.people + "，落水发生在" + se.river +
                                "，可以由热心人士救助！");
        }
    }
}

// 事件产生和处理
private void event_button_Click(object sender, RoutedEventArgs e)
{
    clearComments2();
    //定义一个落水发生源类对象；
    SwimAlarm mySwimAlarm = new SwimAlarm();
    //定义一个落水处理类对象，并将源类对象作为参数传递给这个对象
    SwimHandlerClass mySwimeHandler = new SwimHandlerClass(this, mySwimAlarm);
    //发生落水情况，以事件机制执行
    mySwimAlarm.ActivateSwimAlarm("东湖", "陈女士");
    mySwimAlarm.ActivateSwimAlarm("长江", "小女孩");
}

```

## 10. 数据库操作

```c#
//数据库连接字符串
String strConnect = "Server=" + server + ";User ID=" + user + ";password=" + psw + ";data source=orcl;";

//测试连接
private void connect_Click(object sender, RoutedEventArgs e)
{
   	...
    //测试连接
    bool con_sucess = true;
    OracleConnection hCon = new OracleConnection(strConnect);
    try
    {
        hCon.Open();
        hCon.Close();
    }
    catch (Exception ee)
    {
        con_sucess = false;
        MessageBox.Show(handleException(ee));
    }
    finally
    {
        if (con_sucess) //连接成功
        {
            MessageBox.Show("数据库连接测试成功");
            button_query.IsEnabled = true;
        }
    }
}

// 加载数据
private void load_Data(string sql="")
{
    ...
    OracleConnection hCon = null;
    try
    {
        hCon = new OracleConnection(strConnect);
        hCon.Open();
        if (StringUtil.isEmpty(sql)) sql = "select * from student";
        DataSet ds = new DataSet();
        OracleDataAdapter da = new OracleDataAdapter(sql, hCon);
        da.Fill(ds);
        if (ds != null)
        {
            foreach (DataRow mDr in ds.Tables[0].Rows)
            {
                Student student = new Student()
                {
                    StudentNum = (mDr["studentNum"].Equals(DBNull.Value)) ? 
                        "" : (String)mDr["studentNum"],
                    OldStuNum = (mDr["studentNum"].Equals(DBNull.Value)) ? 
                        "" : (String)mDr["studentNum"],
                    Name = (mDr["studentName"].Equals(DBNull.Value)) ? 
                        "" : (String)mDr["studentName"],
                    Sex = (mDr["sex"].Equals(DBNull.Value)) ? 
                        "" : (String)mDr["sex"],
                    IDCard = (mDr["idCard"].Equals(DBNull.Value)) ? 
                        "" : (String)mDr["idCard"],
                    Password = (mDr["password"].Equals(DBNull.Value)) ? 
                        "" : (String)mDr["password"],
                    ClassNum = (mDr["class"].Equals(DBNull.Value)) ? 
                        0 : Decimal.ToInt32((Decimal)mDr["class"]),
                    entityState = EntityState.NONE
                };

                stuList.Add(student);
            }
        }

        dataGrid.ItemsSource = null;
        dataGrid.ItemsSource = stuList;
        ...
    }
    catch (Exception ee)
    {
        MessageBox.Show(handleException(ee));
        querySuccess = false; // 标志位设为查询失败
    }
    finally
    {
        if (hCon != null && hCon.State == ConnectionState.Open)
        {
            hCon.Close();
        }
    }
}

// 删除数据
private int delete(Student student)
{
    ...
    OracleConnection hCon = null;
    try
    {
        hCon = new OracleConnection(strConnect);
        hCon.Open();
        // studentNum: student id
        string sql = "delete from student where studentNum='" + 
            student.StudentNum + "'";
        OracleCommand command = new OracleCommand();
        command.Connection = hCon;
        command.CommandText = sql;
        int ret = command.ExecuteNonQuery();
        return ret;
    }
    catch (Exception ee)
    {
        MessageBox.Show(handleException(ee));
        return -1;
    }
    finally
    {
        if (hCon != null && hCon.State == ConnectionState.Open)
        {
            hCon.Close();
        }
    }
}

// 更新和添加
private void save_Click(object sender, RoutedEventArgs e)
{
    OracleConnection hCon = null;
	try
	{
        hCon = new OracleConnection(strConnect);
        hCon.Open();
        OracleCommand updateCommand = new OracleCommand("UPDATE student SET password = :password ,class = :class, studentNum = :studentNum , studentName = :studentName, IDCard = :IDCard , sex = :sex  " + " WHERE studentNum = :oldStuNum", hCon);
		OracleCommand insertCommand = new OracleCommand("insert into student (studentNum, studentName, class, password, sex, IDCard) values (:studentNum, :studentName, :class, :password, :sex, :IDCard)", hCon);
        int updatedCount = 0;
        int indertedCount = 0;

        foreach (Student student in stuList)
        {
            ....
            if (student.entityState == EntityState.MODIFIED)
            {
            	...
            	updateCommand.Parameters.AddWithValue("studentNum",
                                                      student.StudentNum);
                ...      
               	if (StringUtil.isEmpty(student.Password)) 
                    updateCommand.Parameters.AddWithValue("password", 
                                                          DBNull.Value);
                else updateCommand.Parameters.AddWithValue("password", 
                                                           student.Password);
				updatedCount = updatedCount + updateCommand.ExecuteNonQuery();
           	}
            if (student.entityState == EntityState.NEW)
            {
                ...
				insertCommand.Parameters.AddWithValue("studentNum",
                                                      student.StudentNum);
                ...
                indertedCount = indertedCount + insertCommand.ExecuteNonQuery();
            }
        }
        MessageBox.Show("更新了" + updatedCount + "条数据，新增了" + indertedCount +
                        "条数据");
    }
    catch (Exception ee)
    {
        MessageBox.Show(handleException(ee));
    }
    finally
    {
        if (hCon != null && hCon.State == ConnectionState.Open)
        {
            hCon.Close();
        }
        load_Data();
    }
}
```

