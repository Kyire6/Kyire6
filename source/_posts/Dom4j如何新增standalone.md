---
title: Dom4j如何新增standalone？
tags:
  - 笔记
  - 技巧
categories: Java
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220403000608.png
abbrlink: 2e39132d
date: 2021-12-16 15:26:07
updated: 2021-12-16 15:26:07
---

# Dom4j如何新增standalone属性？

## 前言

工作中调用一个第三方接口，需要上传 xml 文件。Java 操作 xml 文件的 api 很多，有 Dom、SAX 、JDom、Dom4j。我一般常用的是 Dom4j，但是对接此接口上传的 xml 文件需要添加 `standalone="no"` 属性。查阅相关资料，发现 `Dom4j -1.6.1` 版本并没有提供相应的方法设置。

> 解决方案

重写 `XMLWriter` 类中的 `writeDeclaration` 方法，具体代码如下：

```java
import java.io.FileOutputStream;
import java.io.FileWriter;
import java.io.IOException;
import java.io.UnsupportedEncodingException;

import org.dom4j.io.OutputFormat;
import org.dom4j.io.XMLWriter;

public class StandaloneWriter extends XMLWriter {

    public StandaloneWriter(FileOutputStream fileOutputStream, OutputFormat format)
    throws UnsupportedEncodingException {
        super(fileOutputStream, format);
    }

    public StandaloneWriter(FileWriter fileWriter, OutputFormat format)
    throws UnsupportedEncodingException {
        super(fileWriter, format);
    }

    @Override
    protected void writeDeclaration() throws IOException {
        OutputFormat format = getOutputFormat();

        String encoding = format.getEncoding();

        if (!format.isSuppressDeclaration()) {
            if (encoding.equals("UTF8")) {
                writer.write("<?xml version=\"1.0\"");

                if (!format.isOmitEncoding()) {
                    writer.write(" encoding=\"UTF-8\"");
                }

                writer.write(" standalone=\"yes\"");
                writer.write("?>");
            } else {
                writer.write("<?xml version=\"1.0\"");

                if (!format.isOmitEncoding()) {
                    writer.write(" encoding=\"" + encoding + "\"");
                }

                writer.write(" standalone=\"no\"");
                writer.write("?>");
            }

            if (format.isNewLineAfterDeclaration()) {
                println();
            }
        }
    }
}
```

在使用过程中，用 `StandaloneWriter` 替换掉 `XMLWriter` 即可。
