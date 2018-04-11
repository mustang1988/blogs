---
date: 2017-11-27
title: "MongoDB 数据库数据迁移"
tags:
    - MongoDB
categories:
    - MongoDB
gitment: true
---

1. 导出旧数据

    > mongodump -h 服务器ip  -d 需要导出的数据库 -o 导出文件所在目录
    
    导出完成后会在指定目录生成对应数据库名称的目录,其中有.json 和 .bson格式的数据
    
2. 拷贝导出的数据文件到新数据库所在服务器

3. 导入旧数据

    > mongorestore -h 服务器ip --port 端口 [-u 用户名 -p 密码] -d 数据库名称 [--drop] 数据文件路径 
    
 
