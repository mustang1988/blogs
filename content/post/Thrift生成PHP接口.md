---
date: 2017-04-27
title: "Thrift生成PHP接口"
tags:
    - Thrift
    - RPC
    - PHP
categories:
    - Thrift
    - RPC
    - PHP
gitment: true
---

本文所使用thrift版本([0.10.0](https://thrift.apache.org/download))

***

  1. 支持[PSR4](http://www.php-fig.org/psr/psr-4/)自动加载标准

    `` $ thrift --out {OUTPUT_DIR} --gen php:psr4 {THRIFT_FILE} ``
  
    * {OUTPUT_DIR}为php接口文件的输出目录

    * {THRIFT_FILE}为需要生成接口的thrift协议文件

  2. 支持指定namespace
  
    ``$ thrift --out {OUTPUT_DIR} --gen php:psr4,nsglobal={NAMESPACE} {THRIFT_FILE} ``
  
    * 如果NAMESPACE有多层级,注意使用引号引起来

