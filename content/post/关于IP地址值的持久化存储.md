---
date: 2018-04-11
title: "关于IP地址值的持久化存储"
tags:
    - 数据库
    - 持久化
    - IP地址
categories:
    - 数据库
    - 持久化
    - IP地址
gitment: true
---

前两天整理浏览器书签的时候翻出来这篇文章[《赶集mysql军规》](https://mp.weixin.qq.com/s/oQstfRFuGOvUVnElRqS5aw)文中"字段类军规"中写到

> 用int而不是char(15)存储ip

由此联想到,如何使用int来存储ip呢?因为我通常都会使用char(15)来存储ip地址信息的,这个int来存储是什么鬼?

后来仔细想了想,也不是完全没有道理,ip地址其本质就是一个32位的二进制数啊,我把这个二进制数转成十进制再持久化没毛病啊,于是开始了尝试,首先就是xxx.xxx.xxx.xxx格式的IP地址和int值的双向转换.

* IP字符串转int

  1. 将xxx.xxx.xxx.xxx格式的IP地址拆分成\[xxx,xxx,xxx,xxx\]的数组
  2. 将数组中每个十进制值转成二进制,不足8位的需要补0
  3. 将数组转回到字符串,得到一个32位的二进制值
  4. 将这个32位的二进制值转成十进制

  ```javascript
    function ipv4ToInt(ip) {
        const ip_add_arr = ip.split('.')
        if (ip_add_arr.length !== 4) {
            throw `invalid ip address "${ip}"`
        }
        for (let i = 0, length = ip_add_arr.length; i < length; i++) {
            ip_add_arr[ i ] = leftpad(parseInt(ip_add_arr[ i ]).toString(2), 8, '0')
        }
        const result = parseInt(ip_add_arr.join(''), 2)
        if (result >= 0 && result <= 4294967295) {
            return result
        }
        else {
            throw `invalid ip address "${ip}"`
        }
    }
  ```

* int 转IP字符串

  1. 将十进制数字转换为32位二进制值,不足32位的需要补0
  2. 以8位为一个单位,拆分这个二进制值,得到一个长度为4的二进制数组
  3. 将拆分后的各个部分,转换成十进制,得到一个长度4的十进制数组
  4. 将这个数组转换为字符串,用“.”隔开元素

  ```javascript
    function intToIPv4(int) {
        if (int < 0 || int > 4294967295) {
            throw `invalid int value "${int}" to parse`
        }
        // 10进制value转,ip地址字符串
        let ip_bin = leftpad(parseInt(int).toString(2), 32, '0')
        const reg = /\d{8}/g
        const rs = ip_bin.match(reg)
        for (let i = 0, length = rs.length; i < length; i++) {
            rs[ i ] = parseInt(rs[ i ], 2)
        }
        return rs.join('.')
    }
  ```
  
  * 测试
    
  ```javascript
    try {
        const ip = '127.0.0.1'
        const value = ipv4ToInt(ip)
        console.log(`${ip} => ${value}`)
        const int = value + Math.random() * 10
        console.log(`${int} => ${intToIPv4(int)}`)
    }
    catch (e) {
        console.log(e)
    }
  ```
  
  运行结果
  
  ```shell
    $ node index.js
      127.0.0.1 => 2130706433
      2130706440.9084392 => 127.0.0.8
  ```
  * 然而在使用中,并不需要这么麻烦的

  在数mysql数据库中,其实已经内见了字符串IP,int互转的内置函数inet_aton()和inet_ntoa(),在插入数据时我们只需要声明好字段的类型,然后插入的时候和查询的时候调用下就可以了
  
  ```sql
    CREATE TABLE `user` (
     `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
     `name` varchar(100) NOT NULL,
     `ip` int(10) unsigned NOT NULL,
     PRIMARY KEY (`id`)
    ) ENGINE=InnoDB;
    
    
    INSERT INTO `user` (`id`, `name`, `ip`) VALUES
    (2, 'Abby', inet_aton('192.168.1.1')),
    (3, 'Daisy', inet_aton('172.16.11.66')),
    (4, 'Christine', inet_aton('220.117.131.12'));
    
    SELECT `id`,`name`,inet_ntoa(`ip`) AS `ip` FROM `user`;
  ```
  
  运行结果
  
  ```shell
  mysql> CREATE TABLE `user` (
      ->      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      ->      `name` varchar(100) NOT NULL,
      ->      `ip` int(10) unsigned NOT NULL,
      ->      PRIMARY KEY (`id`)
      ->     ) ENGINE=InnoDB;
  Query OK, 0 rows affected (0.02 sec)

  mysql>
  mysql>
  mysql>     INSERT INTO `user` (`id`, `name`, `ip`) VALUES
      ->     (2, 'Abby', inet_aton('192.168.1.1')),
      ->     (3, 'Daisy', inet_aton('172.16.11.66')),
      ->     (4, 'Christine', inet_aton('220.117.131.12'));
  Query OK, 3 rows affected (0.00 sec)
  Records: 3  Duplicates: 0  Warnings: 0

  mysql>
  mysql>     SELECT `id`,`name`,inet_ntoa(`ip`) AS `ip` FROM `user`;
  +----+-----------+----------------+
  | id | name      | ip             |
  +----+-----------+----------------+
  |  2 | Abby      | 192.168.1.1    |
  |  3 | Daisy     | 172.16.11.66   |
  |  4 | Christine | 220.117.131.12 |
  +----+-----------+----------------+
  3 rows in set (0.00 sec)
  ```
  
  
