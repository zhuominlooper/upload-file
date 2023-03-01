# upload-file

实现文件的 BASE64 和 FROMDATA 的格式上传，以及大文件的分片和断点续传功能,该项目包含前端后 node 后端，所以需要安装前后端的依赖

![RUNOOB 图标](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/13/1720b816a2303d87~tplv-t2oaga2asx-zoom-crop-mark:1304:1304:1304:734.awebp)

## 客户端

1. cd ./client
2. npm i or yarn 安装前端依赖
3. npm run serve 启动项目

## 服务端

1. cd ./server
2. npm i or yarn 安装前端依赖
3. node server.js 启动服务器

## 上传的功能

| 上传功能实现       | 细节                                        |
| ------------------ | ------------------------------------------- |
| FormData           | 请求头为'multipart/form-data'               |
| Base64             | 请求头为'application/x-www-form-urlencoded' |
| 上传进度           | 利用 axios 的配置项的进度回调函数实现       |
| 多文件上传         | 利用 input 实现可以多选在上传               |
| 大文件分片断点续传 | 利用 Blob 进行分片和续传                    |

## 大文件上传

### 注意点

1. 分片的策略，控制分片数量和分片大小
2. 分片的上传并发需要控制，不然会造成服务器和浏览器压力过大
3. 分片的文件名很重要，提供了 smark-md5 根据内容生成 hash 值
4. 判断切片上传完成的时机，进行分片的合并
