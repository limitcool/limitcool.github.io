---
title: "Go语言解析Xml"
slug: "go-xml"
date: 2022-05-20T14:38:05+08:00
draft: false
description: "使用Go简简单单的解析Xml！"
tags:
  - Go
  - Xml
categories:
  - Go
---

# 开始之前 

```go
import "encoding/xml"
```

## 简单的`Xml`解析

### 1.假设我们解析的`Xml`内容如下:

```xml
<feed>
	<person name="initcool" id="1" age=18 />
</feed>
```

<!--more-->

### 2.接着我们构造对应的结构体

```go
type Feed struct {
    XMLName xml.Name    `xml:"feed"`
    Person struct{
        Name string 	`xml:"name"`
        Id string       `xml:"id"`
        Age int	        `xml:"age"`
    }					`xml:"person"`
}
```

### 3.对`Xml`数据进行反序列化

```go
var feed Feed

// 读取Xml文件，并返回字节流
content,err := ioutil.ReadFile(XmlFilename)
if err != nil {
    log.Fatal(err)
}

// 将读取到的内容反序列化到feed
xml.Unmarshal(content,&feed)
```

## 带有命名空间的`Xml`解析

部分`xml`文件会带有`命名空间`(`Namespace`),也就是冒号左侧的内容,此时我们需要在`go`结构体的`tag` 中加入`命名空间`。

### 1.带有命名空间(Namespace)的`Xml`文件

```xml
<feed xmlns:yt="http://www.youtube.com/xml/schemas/2015" xmlns:media="http://search.yahoo.com/mrss/" xmlns="http://www.w3.org/2005/Atom">
<!--  yt即是命名空间   -->
<yt:videoId>XXXXXXX</yt:videoId> 
<!--  media是另一个命名空间   -->
<media:community></media:community>
</feed>
```

### 2.针对命名空间构造结构体

```go
type Feed struct {
	XMLName   xml.Name  `xml:"feed"` // 指定最外层的标签为feed
	VideoId   string    `xml:"http://www.youtube.com/xml/schemas/2015 videoId"`
    Community string    `xml:"http://search.yahoo.com/mrss/ community"`
}
```

### 3.对`Xml`数据进行反序列化

```go
var feed Feed

// 读取Xml文件，并返回字节流
content,err := ioutil.ReadFile(XmlFilename)
if err != nil {
    log.Fatal(err)
}

// 将读取到的内容反序列化到feed
xml.Unmarshal(content,&feed)
```
