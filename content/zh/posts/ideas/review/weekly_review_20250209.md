---
title: "周报 Vol.19 - 消失的年味儿"
date: 2025-02-10T15:05:00+08:00
draft: false
tags: 
  - "review"
  - "life"
  - "chinese new year"
categories: 
  - "Ideas"
authors:
  - "gentlelucky"
---

## 前言

本篇是对  `2024-01-27`  到  `2025-02-09`  这周生活的记录和思考。

春节基本没有自己的时间，要么在路上，要么就等开饭，实则无趣；w2x 产品进入测试阶段，也遇到了技术难点，正在攻克；由于之前的开发环境使用的是我家里的服务器（就是一个台式机），不稳定，所以把开发环境迁移到了阿里云服务器上。

## 消失的年味儿

按时间线，梳理了一下过年的日子：

| 日期             | 事由                                                         |
| ---------------- | ------------------------------------------------------------ |
| 1 月 27 日，傍晚 | 合肥 -> 古城；给古城拜年；                                   |
| 1 月 28 日，中午 | 古城 -> 无为；回无为拜年；                                   |
| 1 月 29 日，下午 | 无为 -> 合肥；无为太「无趣」，回合肥；                       |
| 1 月 30 日，下午 | 合肥 -> 巢湖边；看日落；                                     |
| 1 月 31 日，早上 | 合肥 -> 大普 -> 古城；先去大舅家拜年，由于二婶邀请吃饭，赶回古城； |
| 2 月 1 日，傍晚  | 古城 -> 合肥；小舅子带对象来古城，中午一起吃饭，晚上小明哥哥请客吃饭； |
| 2 月 2 日，全天  | 合肥；林林上班，我休息居家；                                 |
| 2 月 3 日，下午  | 合肥 -> 无为；由于走的匆忙，再回无为看看；                   |
| 2 月 4 日，下午  | 无为 -> 合肥；返回合肥，准备上班；                           |

### 年味儿

随着年级的增长，感受过年的氛围越来越弱，所以过年并没有给自己内心有所冲击。仿佛过年最开心的是「小孩们」，他们可以穿新衣，可以放烟花，有压岁钱.....

现在的过年，越来越觉得是一种不得不去的「应酬」，是那种被架在道德利益上的、不得不去的「应酬」。大家聚到一起，吃饭、喝酒、打牌、娱乐，重复再重复。

原来条件不好的时候，过年比穿上新衣，但今年我发现，大部分人的衣服并不是新的。这样是不是也可以反映出大家对过年并没有那么多仪式感。

现在饭桌上依旧很多荤菜，但我发现很多人喜欢吃素菜（我就是其中之一）。也可能是经济条件好了，这些「大鱼大肉」不一定非等到过年才能吃到。即使发现荤菜剩很多，请客者也得做好多荤菜，要不然会被客人觉得你小气！我觉得应该跳出这种思维怪圈，做到「光盘行动」就好！

### 碎片时间

时间被打碎，基本上都是在前往古城、合肥、无为的路上。而到了家里，也就是同一批人吃吃喝喝；

我觉得一代人有一代人的圈子和社交，长辈们喜欢过年在一起吃吃喝喝、打打牌那挺好的，但不应该绑定小一辈也来。小一辈也有自己的圈子和社交，他们有自己的兴趣和爱好。

我们考虑过带上家人一起，但由于各自父母的成长环境和爱好不同，很多组织到一起，最终我们放弃了这个想法。之前看过张踩铃的[我们的泰国之旅之我妈vs Wendy](https://www.bilibili.com/video/BV1XU411U7Pt/?vd_source=5470b2ac24647c353a06fe1e5de58791)视频，视频中就鲜明的说明了中国式父母和国外父母生活价值观的不同。

最终我们决定打算今年过年前夕，先回古城和无为，把年给提前拜了，然后计划我们独自出去旅游。出去走走，让自己心情放松，这应该才是过年的正确打开方式。

## w2x 产品测试阶段

![CleanShot 2025-02-10 at 14.15.17](https://image.gentlelucky.com/CleanShot%202025-02-10%20at%2014.15.17.png)

w2x 已完成第一轮测试，基础流程没什么大问题。

现阶段遇到的技术难点是：如何兼容非标准文本格式导入！

## 服务迁移

由于开发环境搭建在家庭服务器上（EXSI 的台式主机），有时间出现阿里云「动态DNS」异常，服务慢、卡等问题。决定迁移到阿里云服务器上，提高开发效率。

### 数据库

![image-20250210143837670](https://image.gentlelucky.com/image-20250210143837670.png)



数据使用 `docker-compose` 部署 postgres 数据库

```yaml
version: '3'
services:
  postgres:
    image: postgres
    container_name: postgres-5432
    restart: always
    environment:
      TZ: Asia/Shanghai
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data:rw
```

### SSL 证书

使用 [Nginx Proxy Manager](https://nginxproxymanager.com/) 做反向代理，而且自动续签 SSL 证书。

![CleanShot 2025-02-10 at 14.52.37](https://image.gentlelucky.com/CleanShot%202025-02-10%20at%2014.52.37.png)

#### docker 内部网络访问

Nginx Proxy Manager 使用 docker 部署的，创建一个「npm」网络，后续部署的 docker 应用在「npm」网络中，就可以用「容器名称 + 容器内部端口」访问应用了。

Nginx Proxy Manager  的 `docker-compose.yml`

```yaml
services:
  app:
    image: 'docker.io/jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - npm

networks:
  npm:
    external: true
```

moneynote 的 `docker-compose.yml`

```yaml
version: '3'

services:
  moneynote:
    container_name: moneynote
    image: registry.cn-hangzhou.aliyuncs.com/moneynote/moneynote-all:latest
    restart: always
    environment:
      - DB_PASSWORD=${DB_PASSWORD:-78p7gkc1}
      - invite_code=${invite_code:-111111}
    volumes:
      - moneynote_mysql_data:/var/lib/mysql
    networks:
      - npm

networks:
  npm:
    external: true

```

moneynote 的代理配置

![CleanShot 2025-02-10 at 15.01.48](https://image.gentlelucky.com/CleanShot%202025-02-10%20at%2015.01.48.png)

由于「moneynote」 和「Nginx Proxy Manager」同属于「npm」网络中，所以访问「Nginx Proxy Manager」界面，在「Forward Hostname / IP」输入框中输入「moneynote」后，即可访问 moneynote ！
