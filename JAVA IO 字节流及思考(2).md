# 常用字节流及其思考 #

----------

## File  ##
定义：更应该叫做一个路径
分类：绝对路径/相对路径

		 * File 的基本操作
		 *
		 * 构造函数
		 * File file =new File(string pathname)
		 * File file =new File(string parent ,string child) 父子可以动态设置。
		 * File file =new File(File parent ,string child) 父类可以用File的功能。
		 *
		 * 创建
		 * public boolean createNewFile()  创建返回True 没~False
		 * public boolean mkdir()
		 * public boolean mkdirs()         如果父级不存在，会帮忙创建。
		 *
		 * 重命名/删除
		 * public boolean renameTo(File) 路径相同 改名 ~不 改名并且剪切
		 * public boolean delete()       删除文件/文件夹 不走回收站  文件夹应是空
		 *
		 * 判断
		 * public boolean isDirectory()
		 * public boolean isFile()
		 * public boolean exists()
		 * public boolean setReadable(boolean) Windows中所有文件都可读s
		 * public boolean setWritable(boolean)
		 * public boolean canRead()
		 * public boolean canWrite()
		 * public boolean isHidden()
		 *
		 * 获取
		 * public string getAbsolutePath()
		 * public string getPath()
		 * public string getName()
		 * public long length()
		 * public long lastModified()
		 * public String[] list()           指定目录下的所有文件和文件夹的名字数组。
		 * public File[]  listFiles()		 ~文件数组。
		 *
		 * 文件名称过滤器
		 */
		File dir = new File("G:\\");
		String[] arr = dir.list(new FilenameFilter() {
			@Override
			public boolean accept(File dir, String name) {
				File file =new File(dir,name);
				return file.exists()&&file.getName().endsWith(".java");
			}
		});

		for (String string :arr){
			System.out.println(string);
		}

		File Ipfile =new File("TEST.txt");
		File Opfile =new File("copy.txt");

##File的练习
给一个文件夹路径，打印出该文件夹所有的.jav
	
	public class IOTest03 {
    public static void main(String[] args) {
    File file =GetDir();
    PrintJavaFile(file);
    }

    public static File GetDir() {
        Scanner sc = new Scanner(System.in);
        System.out.println("Please out a dir.");
        while (true) {
            String line = sc.nextLine();
            File file = new File(line);
            if (!file.exists()) {
                System.out.println("Not Exist");
            } else if (file.isFile()) {
                System.out.println("Is a file");
            } else {
                return file;
            }
        }
    }

    public static void PrintJavaFile(File file) {
        File[] files = file.listFiles();
        for (File file1 : files) {
            if (file1.isFile() && file1.getName().endsWith(".java")) {
                System.out.println(file1);
            }else if(file1.isDirectory()){
                PrintJavaFile(file1);
            }
        }
    }
}
        

----------
## 文件流 ##
IO程序的书写：导包，异常处理，释放资源。





        // 没有文件时候会抛出异常。

        FileInputStream fis = new FileInputStream(Ipfile);

        // fos 在没有文件时会自动创建，但是有文件是会先清空再添加，当第二参数为true时 ，就变成追加模式。
        // FileOutputStream fosapd = new FileOutputStream(Opfile, true);

        FileOutputStream fos = new FileOutputStream(Opfile);

        //read()每次返回一个字节，可以把a改变一下类型，进行缓存。
        //int a；每次一个字节的传输，不推荐。
        //
        //read(byte[]) 读到一个字节数组中,这个返回值是读了多少个字节，如果到末尾则返回-1.
        //byte[] a = new byte[fis.available()];一次把所有的都添加进来或者，输出出去。不推荐。文件的字节总数。
        //fis.read(a)
        //
        //小数组
        //在自定义数组上，优化一下数组大小。

        byte[] a = new byte[1024 * 8];
        int len;
        while ((len = fis.read(a)) != -1) {
            //如果忘了加a ，返回的不是字节个数，而是码表值。
            //要写的字节数组，数组的索引从0开始，要写入len个。
            fos.write(a, 0, len);
        }

        fis.close();
        fos.close();

----------


		 *         int read（）方法会读取一个字节，并返回此字节，如果读到最后则返回-1，所以-1是一个关键字。
		 * 
		 *         但是为什么返回的是Int而不是Byte ？ 
		 *         如果当我们传输 00000000 11111111 01010101 时
		 *         ，我们一次比较此字节是否和-1相等，如果相等就退出
		 *         如果不等，就继续传输，但是我们字节的运算是按照补码运算的，而-1的补码恰好就是11111111，所以后面字节的传输
		 *         被中断。这个时候read（）在读每一个字节时，都把他转换成Int类型，也就是在前面加24个0，这样的话原本的想要传输
		 *         的数据，就不会因为和关键字的补码相等而缺少一个。多加24个0会不会对原本的数值产生影响？所以在write（）写出时，
		 *         会自动把前24个0去除掉。这样就不用担心数值的变化了。


		 *         字节传输是最高位是否是符号位？
		 *         1.根据我的推测最高位不是符号位，因为如果是最高位是符号位，那传输时的范围是-128~127
		 *         ，其中包含-1这个值， 用一个包含的值当做关键字，会让范围少一，从256减少到255.
		 *         相比而言，如果最高位不是符号位，传输时一个字节的表示范围是0~255，范围256。
		 *         2.还有一种思路，原本要把byte转换成Int，其实就是因为缺少一个符号位，如果传输时最高位是符号位的话，就有两个符号位，矛盾。
		 *         3.在java文档中也有体现。
		 *         public abstract int read()throws IOException 从输入流中读取数据的下一个字节。返回 0 到 255 范围内的 int 字节值。
         *         如果因为已经到达流末尾而没有可用的字节，则返回值 -1。在输入数据可用、检测到流末尾或者抛出异常前，此方法一直阻塞。 
         *        
		 * 
		 *         为什么用Int类型，而不是用Short呢？
		 *         按理说多补一位就够用了，但是补了24位，这样会不会导致效率没有用short高呢？








----------
## 缓存流 ##

 
         * BufferedInputStream/~Out~
         * 装饰模式 使用时原本的fileinputstream就不用自己写个小数组了，bufferedinputstream已经写好了。

        BufferedInputStream bis=new BufferedInputStream(fis);
        BufferedOutputStream bos=new BufferedOutputStream(fos);

        int b;
        while ((b=bis.read())!=-1){
            bos.write(b);
        }

        //定义小数组和Buffered 的读取那个更快？
        //
        //定义小数组如果是8192个字节，会更快一点。因为小数组操作的是一个数组。
        //
        //而BufferedInputStream操作的是两个数组。buffered一次性从文件中读取8192个，存在缓存区，
        //从缓存中一个一个返回给程序，不用找文件，直到缓存中用完了，再从文件中读取8192个
        //因为这个缓存是在内存中，所以处理的速度比较快。
        //BufferedOutputStream也是，先都存到缓存区，再从缓存区到文件。
        //所以是操作了两个数组。





----------

##.close() 和 .flush() 的区别。##

close 自带flush 可以将缓冲区的字节刷新到文件上，然后关流，关流之后不能再写。
flush 将缓冲区的字节刷新到文件上 ，不关流之前可以一直写。


##字节流读写中文##
可能造成乱码，标点算一个字节，汉字有的码表是两个字节，用字节流读可能造成乱码。
字节流操作的都是字节，所以写出中文必须将字符串转换成字节数组。
fos.write("".getByte);

##流的标准处理异常代码##

     * 异常处理1.6之前
     */

    public static void A() throws IOException {
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            fis = new FileInputStream("xxx.txt");
            fos = new FileOutputStream("yyy.txt");
            int b;
            while ((b = fis.read()) != -1) {
                fos.write(b);
            }
        } finally {
            //finally 里面的代码一定会执行，但是如果抛出异常，
            //依旧会终止程序。

            try {
                fis.close();
            } finally {
                fos.close();
            }
        }
    }

    /**
     * 异常处理1.7
     */
    public static void AA() throws IOException {
        //try({}执行之后自动关闭){}
        //实现了AutoCloseable 的方法都可以自动关闭。
        try (
                FileInputStream fis = new FileInputStream("XXX.TXT");
                FileOutputStream fos = new FileOutputStream("yyy.txt");
        ) {
            int b;
            while ((b = fis.read()) != -1) {
                fos.write(b);
            }
        }
    }

##图片加密
 A 被 B 异或两次 = A  ，  B就是密钥。
 异或二进制运算。 
