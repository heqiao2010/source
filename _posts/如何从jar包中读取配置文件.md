---
layout:     post
title:      "如何在jar包中读取配置文件 "
category:   服务端
date:       2018-08-02 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Java
---
![](https://img.hacpai.com/bing/20180215.jpg?imageView2/1/w/960/h/520/interlace/1/q/100)

>今天开发的时候遇到一个问题——当程序以jar包运行的时候,有个txt配置文件无法获取到,但是本地测试无法复现.后来发现是因为以Jar包形式运行，文件无法访问到，这里记录一下。

### 1. 如何判断当前进程是否以jar包形式运行的？
```
/**
 * 是否以Jar包运行
 * 
 * @return
 */
public static boolean isRunningInJar() {
    try {
        String className = GenericUtils.class.getName().replace('.', '/');
        String classJar = GenericUtils.class.getResource("/" + className + ".class").toString();
        logger.info("classJar: " + classJar);
        return classJar.startsWith("jar:");
    } catch (Exception e) {
        logger.warn("get Running status failed.");
        return false;
    }
}
```

### 2.从Jar包中读取文件内容
```
public static String txt2String(String fileName) {
    StringBuilder result = new StringBuilder();
    BufferedReader br = null;
    try {
        Reader r = null;
        if (isRunningInJar()) {
            InputStream in = GenericUtils.class.getResourceAsStream(File.separator + fileName);
            r = new InputStreamReader(in);
        } else {
            String path = GenericUtils.class.getClassLoader().getResource(fileName).getPath();
            File file = new File(path);
            r = new FileReader(file);
        }
        br = new BufferedReader(r);// 构造一个BufferedReader类来读取文件
        String s = null;
        while ((s = br.readLine()) != null) {// 使用readLine方法，一次读一行
            result.append(System.lineSeparator() + s);
        }
    } catch (Exception e) {
        logger.error("error when function:getTxtFromFile!", e);
    } finally {
        try {
            if (br != null) {
                br.close();
            }
        } catch (final IOException ioe) {
            // ignore
        }
    }
    return result.toString();
}

```