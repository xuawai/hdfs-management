# hdfs-manage-system #
基于Vue.js 2.x系列 + Element UI +hadoop的HDFS可视化管理系统。

## 前言 ##
本系统基于WebHDFS REST API，以可视化的形式操作HDFS的文件系统接口，实现文件的分布式管理。

## 功能 ##
- [x] 新建文件
- [x] 展示文件列表
- [x] 查看文件
- [x] 更新文件
- [x] 删除文件
- [x] 下载文件


## 安装步骤 ##

	git clone https://github.com/xuawai/hdfs-management/     // 把模板下载到本地
	cd hdfs-management   // 进入模板目录
	npm install         // 安装项目依赖，等待安装完成之后

	hosts文件里添加hadoop-master至ip 100.64.208.218的映射

## 本地开发 ##

	// 开启服务器，浏览器访问 http://localhost:8080
	npm run dev


