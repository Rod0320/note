

```java
作者：Java3y
链接：https://www.zhihu.com/question/29005375/answer/667616386
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class SimpleFileTransferTest {

    private long transferFile(File source, File des) throws IOException {
        long startTime = System.currentTimeMillis();

        if (!des.exists())
            des.createNewFile();

        BufferedInputStream bis = new BufferedInputStream(new FileInputStream(source));
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(des));

        //将数据源读到的内容写入目的地--使用数组
        byte[] bytes = new byte[1024 * 1024];
        int len;
        while ((len = bis.read(bytes)) != -1) {
            bos.write(bytes, 0, len);
        }

        long endTime = System.currentTimeMillis();
        return endTime - startTime;
    }

    private long transferFileWithNIO(File source, File des) throws IOException {
        long startTime = System.currentTimeMillis();

        if (!des.exists())
            des.createNewFile();

        RandomAccessFile read = new RandomAccessFile(source, "rw");
        RandomAccessFile write = new RandomAccessFile(des, "rw");

        FileChannel readChannel = read.getChannel();
        FileChannel writeChannel = write.getChannel();


        ByteBuffer byteBuffer = ByteBuffer.allocate(1024 * 1024);//1M缓冲区

        while (readChannel.read(byteBuffer) > 0) {
            byteBuffer.flip();
            writeChannel.write(byteBuffer);
            byteBuffer.clear();
        }

        writeChannel.close();
        readChannel.close();
        long endTime = System.currentTimeMillis();
        return endTime - startTime;
    }

    public static void main(String[] args) throws IOException {
        SimpleFileTransferTest simpleFileTransferTest = new SimpleFileTransferTest();
        File sourse = new File("F:\\电影\\[电影天堂www.dygod.cn]猜火车-cd1.rmvb");
        File des = new File("X:\\Users\\ozc\\Desktop\\io.avi");
        File nio = new File("X:\\Users\\ozc\\Desktop\\nio.avi");

        long time = simpleFileTransferTest.transferFile(sourse, des);
        System.out.println(time + "：普通字节流时间");

        long timeNio = simpleFileTransferTest.transferFileWithNIO(sourse, nio);
        System.out.println(timeNio + "：NIO时间");


    }

}
```

作者：Java3y
链接：https://www.zhihu.com/question/29005375/answer/667616386
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



我分别测试了文件大小为13M，40M，200M的：



![img](../../../../图片保存\v2-102b00913a607b0c14ec5364a4eceec2_720w.jpg)![img](../../../../图片保存\v2-102b00913a607b0c14ec5364a4eceec2_720w.jpg)





![img](../../../../图片保存\v2-ec40d579decde3f9a655aacddbbdf0e5_720w.jpg)![img](../../../../图片保存\v2-ec40d579decde3f9a655aacddbbdf0e5_720w.jpg)

![img](../../../../图片保存\v2-44f892b938c070fd4e6b88c1e419309f_720w.jpg)![img](../../../../图片保存\v2-44f892b938c070fd4e6b88c1e419309f_720w.jpg)