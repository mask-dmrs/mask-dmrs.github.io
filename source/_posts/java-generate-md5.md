---
title: java生成md5
date: 2017-12-06 23:45:02
permalink: java-generate-md5
categories: Gists
tags:
 - java
---

```java
public static String md5(String source){
    try {
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        return bytesConvertToHexString(md5.digest(source.getBytes()));
    }catch (Exception e){
    }
    return null;
}

/**
 * 把字节数组转化成字符串返回
 * @param bytes
 * @return
 */
public static String bytesConvertToHexString(byte [] bytes) {
    StringBuffer sb = new StringBuffer();
    for (byte aByte : bytes) {
        String s= Integer.toHexString(0xff & aByte);
        if(s.length()==1){
            sb.append("0"+s);
        }else{
            sb.append(s);
        }
    }
    return sb.toString();
}
```
