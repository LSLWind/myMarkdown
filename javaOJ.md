### 输入输出

格式1：Scanner sc = new Scanner (new BufferedInputStream(System.in));

格式2：Scanner sc = new Scanner (System.in);

在读入数据量大的情况下，格式1的速度会快些。

读一个整数： int n = sc.nextInt(); 相当于 scanf("%d", &n); 或 cin >> n; 

读一个字符串：String s = sc.next(); 相当于 scanf("%s", s); 或 cin >> s; 

读一个浮点数：double t = sc.nextDouble(); 相当于 scanf("%lf", &t); 或 cin >> t; 

读一整行： String s = sc.nextLine(); 相当于 gets(s); 或 cin.getline(...); 

判断是否有下一个输入可以用sc.hasNext()或sc.hasNextInt()或sc.hasNextDouble()或sc.hasNextLine()
https://blog.csdn.net/shijiebei2009/article/details/17305223

**整形后面的字符串有个换行，必须先读一行（换行符），才能再读一次字符串**

封装数据拆分

```java
static class InputReader {
    public BufferedReader reader;
    public StringTokenizer tokenizer;
 
    public InputReader(InputStream stream) {
        reader = new BufferedReader(new InputStreamReader(stream), 32768);
        tokenizer = null;
    }
 
    public String next() {
        while (tokenizer == null || !tokenizer.hasMoreTokens()) {
            try {
                tokenizer = new StringTokenizer(reader.readLine());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        return tokenizer.nextToken();
    }
 
    public int nextInt() {
        return Integer.parseInt(next());
    }
 
}
```

如果只有数字：

```java
import java.io.*;

public class Main {
	public static StreamTokenizer in = new StreamTokenizer(new BufferedReader(new InputStreamReader(System.in),32768));
	public static PrintWriter out = new PrintWriter(new OutputStreamWriter(System.out));

	public static double nextDouble() throws IOException{ in.nextToken(); return in.nval; }
	public static float nextFloat() throws IOException{ in.nextToken(); return (float)in.nval; }
	public static int nextInt() throws IOException{ in.nextToken(); return (int)in.nval; }
	public static String next() throws IOException{ in.nextToken(); return in.sval;}

	public static void main(String[] args) throws IOException{
//		获取输入
		while(in.nextToken()!=StreamTokenizer.TT_EOF){
			break;
		}
		int x = (int)in.nextToken();  //第一个数据应当通过nextToken()获取
		
		int y = nextInt();
		float f = nextFloat();
		double d = nextDouble();
		String str = next();

//		输出
		out.println("abc");
		out.flush();
		out.close();
	}
}
```



#### 格式化输出

System.out.printf("%",..);

### 模板

#### 全排列

```java
import java.util.Arrays;
import java.util.Stack;

public class Main {

    public static void main(String[] args) {
        DataContainer data = new DataContainer(new Character[]{'a', 'b', 'c', 'd'});//参数为Object数组
        Stack<Object[]> res = new Stack<>();//全排列结果
        permu(data, res, new Stack<Object>());//全排列调用

        //打印
        for(Object[] objs: res){
            for(Object o: objs){
                System.out.print(o);
            }
            System.out.println();
        }
    }
    
     //求全排列的函数
    static void permu(DataContainer data, Stack<Object[]> res, Stack<Object> objs) {
        if (data.effectSize == 0) {
            res.push(objs.toArray());
        }

        for (int i = 0; i < data.actualSize; i++) {
            if(data.state(i) == true){
                objs.add(data.getElement(i));
                permu(data, res, objs);

                //回溯
                data.recovery(i);
                objs.pop();
            }
        }
    }
}
class DataContainer {
    Inner[] inners;
    int effectSize;// 有效元素的个数
    int actualSize;// 实际元素的个数

    DataContainer(Object[] data) {
        int len = data.length;
        this.actualSize = len;
        this.effectSize = len;
        inners = new Inner[len];
        for (int i = 0; i < len; i++) {
            inners[i] = new Inner(data[i], true);
        }
    }
    Object getElement(int index){
        if(inners[index].state == true){
            inners[index].state = false;//取过就无效了
            effectSize--;
            return inners[index].element;
        }
        return null;
    }

    boolean state(int index){
        return inners[index].state;
    }
    void recovery(int index){
        if(inners[index].state == false){
            effectSize++;
            inners[index].state = true;
        }
    }

    class Inner {
        Object element;// 元素
        boolean state;// 状态：有效——true，无效——false

        Inner(Object element, boolean state) {
            this.element = element;
            this.state = state;
        }
    }
}
```

### 数学

#### 三角形面积公式-海伦公式

 S=√p(p-a)(p-b)(p-c) 

p=(a+b+c)/2

### 其它

