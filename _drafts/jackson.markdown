---
layout: post
title:  "Jackson"
categories: json java
---

# 1. Json

## 1.1 Gson

## 1.2 Fastjson

## 1.3 Jackson

- `jackson-core`: 核心模块，低级流API，是对JSON specification的实现
- `jackson-annotations`: 标准Jackson注解包
- `jackson-databind`: 基于`jackson-core`和`jackson-annotations`实现数据绑定，对象序列化操作
- extensions: jackson通过`ObjectMapper.registerModule()`实现三方jackson模块的插件机制，添加java库各种数据类型的序列化和反序列化支持，这样databind包中`ObjectMapper`，`ObjectReader`，`ObjectWriter`可以读写这些类型

# 2. Serialize (Unmarshalling)

## 2.1 ObjectMapper

## 2.2 IgnoredProperties

## 2.3 

# 3. Deserialize (Marshalling)

# 4. Type Conversion