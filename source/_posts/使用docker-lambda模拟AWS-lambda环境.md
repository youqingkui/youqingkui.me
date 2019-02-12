---
title: 使用docker-lambda模拟AWS lambda环境
date: 2019-01-20 19:28:51
tags:
---
# 使用docker-lambda模拟AWS lambda环境
有时候要在 AWS Lambda 上运行程序，但是要安装一些本地的依赖，如 pip install package。如果是在本地机器 pip install package，一些 package 会在本地编译，但是打包上传到 Lambda 上运行则会提示安装的一些 package 出错。查了下资料，因为 AWS Lambda 是 Ubuntu 环境运行的，如果本地机器是 Ubuntu 那应该问题不大，但可能也会有 Ubuntu 系统版本的区别。

发现了一个docker项目，可以模拟 AWS Lambda 的运行环境。
https://github.com/lambci/docker-lambda

## 获取 docker image
根据文档选择对应的 docker images，我用的是 Python 环境。去本地项目目录所在地方，运行`docker run --rm -v "$PWD":/var/task lambci/lambda:python3.7 lambda_function.lambda_handler`

> `lambda_function.lambda_handler` 是默认执行的主函数，执行后如果本地没有对应 docker image 会自动下载，也可以先拉去对应 images `docker pull lambci/lambda:python3.7`
> 
> 没有带build的 image 是 进入不了 bash，如果要安装对应pip，需要进行下面步骤的 image 运行

## 安装 pip 依赖
可以先用 bash 进入运行环境，`docker run -it lambci/lambda:build-python3.6 bash`，然后去对项目目录执行 `pip install -t . Package`，这样会把 package 安装到当前目录。这样安装的 package 是用当前 docker image 进行编译的。

## 测试运行
去对应项目执行运行命令，此命令运行后会立即删除此次运行的docker image。
`docker run --rm -v "$PWD":/var/task lambci/lambda:python3.7 lambda_function.lambda_handler`

![169E473A-2178-424D-AAEA-C0601A09771E](/images/169E473A-2178-424D-AAEA-C0601A09771E.png)

如图，看到的是因为参数没有导致的运行错误，不是 package 的问题。可以在运行命令后面加上对应参数来传提  `’{“some": "event"}' `。

## 打包为zip上传
如果没有问题就可以打包上传 AWS Lambda，在当前项目目录执行 `zip -r lambda.zip .` 会生成一个 lambda.zip 文件，上传此文件就可以。

> 注意不要使用 `zip -r lambda.zip *`，这样会一些东西没有打包进入压缩文件



`docker run -it -v "$PWD":/var/task lambci/lambda:build-python3.6 bash`
