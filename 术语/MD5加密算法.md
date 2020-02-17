MD5：Message-Digest Algorithm 5（信息-摘要算法），属于哈希散列算法一类，对于MD5而言，有两个特性是很重要的
　　第一：明文数据经过散列以后的值是定长的；
　　第二：任意一段明文数据，经过散列以后，其结果必须永远是不变的。
前者的意思是可能存在有两段明文散列以后得到相同长度的结果，后者的意思是如果我们散列特定的数据，得到的结果一定是相同的。
MD5的作用是让大容量信息在用数字签名软件签署私人密钥前被”压缩”成一种保密的格式（就是把一个任意长度的字节串变换成一定长的（32位）十六进制数字串）。

**算法原理**：对MD5算法简要的叙述可以为：MD5以512位分组来处理输入的信息，且每一分组又被划分为16个32位子分组，经过了一系列的处理后，算法的输出由四个32位分组组成，将这四个32位分组级联后将生成一个128位散列值。
　　在MD5算法中，首先需要对信息进行填充，使其位长对512求余的结果等于448。因此，信息的位长（Bits Length）将被扩展至N\*512+448，N为一个非负整数，N可以是零。填充的方法如下，在信息的后面填充一个1和无数个0，直到满足上面的条件时才停止用0对信息的填充。然后，在这个结果后面附加一个以64位二进制表示的填充前信息长度。经过这两步的处理，信息的位长=N*512+448+64=(N+1）\*512，即长度恰好是512的整数倍。这样做的原因是为满足后面处理中对信息长度的要求。

　　Java实现MD5过程：
　　1、通过单例的构造方法获取MessageDigest实例，并指定加密算法类型；
　　2、将需要加密的字符串加盐后转换成byte数组后进行随机哈希过程；
　　3、MessageDigest对字节数组进行摘要,得到摘要字节数组；
　　4、循环遍历生成的byte类型数组，让其生成32位字符串，然后拼接字符串得到MD5值；

```java
public class MD5Util {

    /**
     * 对指定的字符串进行MD5加密处理
     * @param password   待加密的原始密码值
     * @return MD5加盐加密后的密码值
     */
    public static String encodePassword(String password) {
        try {
            // 密码加盐处理,确保密码更加安全
            password = password + "neuyimi";
            // 获取MessageDigest实例，并指定加密算法类型
            MessageDigest digest = MessageDigest.getInstance("MD5");
            // 将需要加密的字符串转换成byte数组后进行随机哈希过程
            byte[] byteArray = password.getBytes();
            // 信息摘要对象对字节数组进行摘要,得到摘要字节数组
            byte[] md5Byte = digest.digest(byteArray);
            StringBuffer buffer = new StringBuffer();
            // 循环遍历byte类型数组，让其生成32位字符串
            for (byte b : md5Byte) {
                int i = b & 0xff;
                String str = Integer.toHexString(i);
                if (str.length() < 2) {
                    str = "0" + str;
                }
                buffer.append(str);
            }
            return buffer.toString();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        return "";
    }

}
```

MD5是不可逆的，也就是没有对应的算法，把转换后的MD5值逆向得到原始密码数据，但可以用穷举法，把可能出现的明文，用**MD5**算法散列之后，把得到的散列值和原始的数据形成一个一对一的映射表，通过匹配从映射表中找出破解密码所对应的原始明文。