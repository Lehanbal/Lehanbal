# 博客使用步骤

**记得先把ssh配置到github上**

以ubuntu为例

1. 安装npm

   ```bash
   sudo apt install npm
   ```

2. 安装hexo

   ```bash
   npm install -g hexo
   
   # 嫌慢的话可以先执行npm切换国内淘宝镜像源
   # npm config set registry http://registry.npm.taobao.org/
   ```

3. 进入到项目目录进行相关插件的安装

   ```bash
   sudo npm install
   ```

4. 在项目目录下执行该相关hexo命令即可

   ```bash
   # 生成静态文件
   hexo g
   
   # 清除当前生成的静态文件
   hexo clean
   
   # 本地化浏览 可-p指定端口 -i指定ip
   hexo s
   
   # 部署
   hexo d
   ```