# String.getBytes()方法

11100100 在转成


汉字 “中” 在UTF-8编码表中的编码是 E4B8AD，转成二进制是 0000 0000 1110 0100 1011 1000 1010 1101

当调用 

```
public static void main(String[] args) throws UnsupportedEncodingException {

        String str = "中";
        byte[] bytes = str.getBytes("utf-8");
        System.out.println(bytes.length);
        for (byte b : bytes){
            System.out.println(b);
            System.out.println(Integer.toBinaryString(b));
        }

    }
    
```

因为前8位全部是0，所以会被忽略，所以获得的字节数组为
[1110 0100] -100
[1011 1000] -56
[1010 1101] -45

