---
title: protobuf入门安装使用-Windows版本
date: 2021-08-24 00:50:57
tags: protbuf
---

[Java安装文档]: https://github.com/protocolbuffers/protobuf/blob/master/src/README.md
[github仓库]: https://github.com/protocolbuffers/protobuf
[protobuf使用文档]: https://developers.google.com/protocol-buffers/docs/overview

#### 一、Windows环境安装

- 下载对应安装包

  [各版本安装包列表]: https://github.com/protocolbuffers/protobuf/releases/tag/v3.17.3

- 解压文件到文件夹 

  <!--注意：WinRAR解压到指定文件夹时解压后目录无bin目录，建议拷贝压缩包到目标文件夹下后执行解压到当前文件夹，或解压到当前文件夹后再拷贝至目标文件夹-->

- 配置环境变量

- 验证环境

  cmd或者gitbash下

  ```shell
  $ protoc --version
  libprotoc 3.17.3
  ```

至此protobuff环境安装完成。

