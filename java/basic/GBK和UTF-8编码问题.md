## 总结：java  锟斤拷锟斤拷锟斤 的由来



**中文   GBK --> UTF-8 --> GBK       就会产生 锟斤拷锟斤**

**中文  UTF-8 --> GBK ---> UTF-8    正常中文**



我们知道，计算机存储数据都是2进制，就是0和1，那么这么多的字符就都需要有自己对应的0和1组成的序列，计算机将需要存储的字符转换成它们对应的01序列，然后就可以储存在电脑里了。

 

比如我们可以定义用8位2进制表示一个字符，“00000000”表示小写字母“a”，“00000001”表示小写字母“b”，那么计算机要存储“ab”的时候，其实在计算机里的存储的是“0000000000000001”，读取的时候先读取前8位，根据对应关系，可以解码出“a”，再读取后8位，又可以解码出“b”，这样就读出了当时写入的“ab”了。而我们定义的这种字符和二进制序列的对应关系，就可以称之为编码。我们如果需要将“ab”发送给别人，因为网络也是基于二进制，所以只要先约定好编码规则，就可以发送“0000000000000001”，然后对方根据约定的编码解码，就可以得到“ab”。现在是互联网的时代，我们经常需要和其他的计算机进行交互，一套编码系统还是比较复杂的，所以大家就需要约定通用的编码，这样的编码是大家都约定好的，通信时就不用再去约定编码规则了~然而，为了满足各种不同的需求，人们还是制定了很多种编码，没有哪一种能全面替代其他编码，所以现在多种编码并存。通常这些编码都被大家所接受和熟知，所以现在不用再通信前商量编码的对应规则和细节，只需要告诉对方，我采用的是什么通用编码，彼此就能愉快地通信了。

 

那么其实乱码的本质就是：读取二进制的时候采用的编码和最初将字符转换成二进制时的编码不一致。

 

ps：编码有动词含义也有名词含义，名词含义就是一套字符和二进制序列之间的转换规则，动词含义是使用这种规则将字符转换成二进制序列。

 

好了，废话不多，直接上一段代码：

 

```java
import java.io.UnsupportedEncodingException;

public class EncodingTest {
	public static void main(String[] args) throws UnsupportedEncodingException {
		String srcString = "我们是中国人";
		String utf2GbkString = new String(srcString.getBytes("UTF-8"),"GBK");
		System.out.println("UTF-8转换成GBK："+utf2GbkString);
		String utf2Gbk2UtfString = new String(utf2GbkString.getBytes("GBK"),"UTF-8");
		System.out.println("UTF-8转换成GBK再转成UTF-8："+utf2Gbk2UtfString);
	}
}
```




因为UTF-8和GBK是两套中文支持较好的编码，所以经常会进行它们之间的转换，这里就以它们举例。

以上代码运行打印出以下内容：

 

UTF-8转换成GBK：鎴戜滑鏄腑鍥戒汉
UTF-8转换成GBK再转成UTF-8：我们是中国人

 

我们看到，将"我们是中国人"以UTF-8编码转换成byte数组（byte数组其实就相当于二进制序列了，此过程即编码），再以GBK编码和byte数组创建新的字符串（此过程即以GBK编码去解码byte数组，得到字符串），就产生乱码了。

因为编码采用的UTF-8和解码采用的GBK不是同一种编码，所以最后结果乱码了。

之后再对乱码使用GBK编码，还原到解码前的byte数组，再使用和最初编码时使用的一致的编码UTF-8进行解码，就可得到最初的“我们是中国人”。

这种多余的转换有时候还是很有用的，比如ftp协议只支持ISO-8859-1编码，这个时候如果要传中文，只能先换成ISO-8859-1的乱码，ftp完成后，再转回UTF-8就又可以得到正常的中文了。

 

怎么样？编码转换是不是so easy？那该来点正经的了：

 ```java
 import java.io.UnsupportedEncodingException;
 
 public class EncodingTest {
 	public static void main(String[] args) throws UnsupportedEncodingException {
 		String srcString = "我们是中国人";
 		String gbk2UtfString = new String(srcString.getBytes("GBK"), "UTF-8");
 		System.out.println("GBK转换成UTF-8：" + gbk2UtfString);
 		String gbk2Utf2GbkString = new String(gbk2UtfString.getBytes("UTF-8"), "GBK");
 		System.out.println("GBK转换成UTF-8再转成GBK：" + gbk2Utf2GbkString);
 	}
 }
 ```



这次我们反过来，先将字符串以GBK编码再以UTF-8解码，再以UTF-8编码，再以GBK解码。

 

这次的运行结果是：

 

GBK转换成UTF-8：��������й��
GBK转换成UTF-8再转成GBK：锟斤拷锟斤拷锟斤拷锟叫癸拷锟斤拷

 

WTF？？万恶的“锟斤拷”，相信不少人都见过。这里GBK转成UTF-8乱码好理解，但是再转回来怎么变成了“锟斤拷锟斤拷锟斤拷锟叫癸拷锟斤拷”，这似乎不科学。

这其实和UTF-8独特的编码方式有关，由于UTF-8需要对unicode字符进行编码，unicode字符集是一个几乎支持所有字符的字符集，为了表示这么庞大的字符集，UTF-8可能需要更多的二进制位来表示一个字符，同时为了不致使UTF-8编码太占存储空间，根据二八定律，UTF-8采用了一种可变长的编码方式，即将常用的字符编码成较短的序列，而不常用的字符用较长的序列表示，这样让编码占用更少存储空间的同时也保证了对庞大字符集的支持。

正是由于UTF-8采用的这种特别的变长编码方式，这一点和其他的编码很不一样。比如GBK固定用两个字节来表示汉字，一个字节来表示英文和其他符号。

来测试一下：

import java.io.UnsupportedEncodingException;

public class EncodingTest {
	public static void main(String[] args) throws UnsupportedEncodingException {
		String srcString = "我们是中国人";
		byte[] GbkBytes = srcString.getBytes("GBK");
		System.out.println("GbkBytes.length:" + GbkBytes.length);
		byte[] UtfBytes = srcString.getBytes("UTF-8");
		System.out.println("UtfBytes.length:" + UtfBytes.length);
		String s;
		for (int i = 0; i < srcString.length(); i++) {
			s = Character.valueOf(srcString.charAt(i)).toString();
			System.out.println(s + ":" + s.getBytes().length);
		}
	}
}
运行结果为：

 

GbkBytes.length:12
UtfBytes.length:18
我:3
们:3
是:3
中:3
国:3
人:3

 

可以看到使用GBK进行编码，“我们是中国人”6个汉字占12个字节，而是用UTF-8进行编码则占了18个字节，其中每个汉字占3个字节（由于是常用汉字，只占3个字节，有的稀有汉字会占四个字节。）

UTF-8编码的读取方式也比较不同，需要先读取第一个字节，然后根据这个字节的值才能判断这个字节之后还有几个字节共同参与一个字符的表示。

对于某一个字符的UTF-8编码，如果只有一个字节则其最高二进制位为0；如果是多字节，其第一个字节从最高位开始，连续的二进制位值为1的个数决定了其编码的位数，其余各字节均以10开头。UTF-8最多可用到6个字节。 

如表： 
1字节 0xxxxxxx 
2字节 110xxxxx 10xxxxxx 
3字节 1110xxxx 10xxxxxx 10xxxxxx 
4字节 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx 
5字节 111110xx 10xxxxxx 10xxxxxx 10xxxxxx 10xxxxxx 
6字节 1111110x 10xxxxxx 10xxxxxx 10xxxxxx 10xxxxxx 10xxxxxx 
因此UTF-8中可以用来表示字符编码的实际位数最多有31位，即上表中x所表示的位。除去那些控制位（每字节开头的10等），这些x表示的位与UNICODE编码是一一对应的，位高低顺序也相同。 
实际将UNICODE转换为UTF-8编码时应先去除高位0，然后根据所剩编码的位数决定所需最小的UTF-8编码位数。 
因此那些基本ASCII字符集中的字符（UNICODE兼容ASCII）只需要一个字节的UTF-8编码（7个二进制位）便可以表示。 

上面一随便看看就好，只要知道“由于UTF-8的特殊编码方式，所以有些序列是不可能出现在UTF-8编码中的”就可以了。

 

所以当我们将由GBK编码的12个字节试图用UTF-8解码时会出现错误，由于GBK编码出了不可能出现在UTF-8编码中出现的序列，所以当我们试图用UTF-8去解码时，经常会遇到这种不可能序列，对于这种不可能序列，UTF-8把它们转换成某种不可言喻的字符“�”，当这种不可言喻的字符再次以UTF-8进行编码时，他们已经无法回到最初的样子了，因为那些是UTF-8编码不可能编出的序列。然后这个神秘字符再转换成GBK编码时就变成了“锟斤拷”。当然，还有很多其他的巧合，可能正好碰到UTF-8中存在的序列，甚至原本不是一个字符的字节，可能是某个字的第二个字节和下一个字的两个字节，正好被识别成一个UTF-8序列，于是解码出一个汉字，当然这些在我们看来都是乱码了，只不过不是“锟斤拷”的样子。因为不可能序列更普遍存在，所以GBK转UTF-8再转GBK时，最常见的便是“锟斤拷”！

**� \xef\xbf\xbd**

**锟 \xef\xbf**

**斤 \xbd\xef** 

**拷 \xbf\xbd**

可以看到UTF-8编码下的两个“��”和GBK编码下的“锟斤拷”的字节编码相同，都是 \xef\xbf\xbd\xef\xbf\xbd

所以：以非UTF-8编码编码出的字节数组，一旦以UTF-8进行解码，通常这是一条不归路，再尝试将解码出的字符以UTF-8进行编码，也无法还原之前的字节数组。

相反地，其他的固定长度编码几乎都可以顺利还原。

乱码解决方式：

出现“锟斤拷”说明在字节和字符的转换（编码和解码）过程中使用了不同的编码，找出编解码的代码，修改成使用同一种编码方式即可。

=====================补充==========================

上文中其实有一个东西一直在回避，就是既然所有字符在保存时都需要转换成二进制，那么java是使用什么编码来保存字符的呢？这个问题其实我们可以不必深究，因为这对我们是透明的，我们只要假设java使用某种编码可以表示所有字符。得益于这种透明，我们可以当作java是直接保存字符本身的，就如上文所做的这样。

java虚拟机中使用unicode字符集。

java虚拟机中使用UTF-16编码方式。