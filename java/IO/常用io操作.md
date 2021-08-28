https://www.cnblogs.com/rollenholt/archive/2011/09/11/2173787.html





#### PushBackInputStream回退流 ：

```java
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.PushbackInputStream;
 
/**
 * 回退流操作
 * */
public class PushBackInputStreamDemo{
    public static void main(String[] args) throws IOException{
        String str = "hello,rollenholt";
        PushbackInputStream push = null;
        ByteArrayInputStream bat = null;
        bat = new ByteArrayInputStream(str.getBytes());
        push = new PushbackInputStream(bat);
        int temp = 0;
        while((temp = push.read()) != -1){
            if(temp == ','){
                push.unread(temp);   // 将数据压回流中
                temp = push.read();  // 重新读取该数据
                System.out.print("(回退" + (char) temp + ") ");
            }else{
                System.out.print((char) temp);
            }
        }
    }
}
```

