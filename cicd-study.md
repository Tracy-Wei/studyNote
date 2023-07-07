在 ci 中如果要在一个项目中分两个子项目的话在对应项目的 editproject 中添加 Subprojects（子项目），如例子中 busybee 分前台和后台两个页面项目时候，开一个总的叫 busybee.web，然后在这个基础上开两个子项目，也就是 user 和 managerment

配 ci 的一些配置，需要配好一些：
1.VCS Roots（绑定版本控制工具）

保存前可以 test connection 一下是否有问题再保存

2.配置 parameters（参数）
其实就是配置一些 docker 的登陆用户名、密码和注册表，后续会在 build 的 step 中用到。

3.配置 connetction（连接）
配置的是支持的连接类型，需要写上一些用户 id 和 secret

4.配好 build 的配置

5.定义构造步骤

这如果在本地执行打包的话，可以按照以下步骤执行（通常用于自查自己配置有没有什么问题，一般本地能过，ci 就能过。前提需要下载一个 Docker）：
1.yarn install/npm install
2.yarn run build
3.docker build -f Dockerfile -t xxx . (这里 xxx 的命名用横线-或者下划线\_连接，比如：busybee-web，如果用带大写字母的如 busybeeWeb 是不行的)
4.docker run -p 8188:80 -t xxx 5.浏览器访问 localhost:8188

这里涉及到一些 docker 的写法
docker build -f Dockerfile -t xxx . =》这是用于使用 Dockerfile 创建镜像的命令，
-f :指定要使用的 Dockerfile 路径
而此处写的路径是以我们通常 Dockerfile 是放在 web 文件底下的，而我们运行是以 web 路径运行的，所以直接写 Dockerfile 就好了。
-t: 镜像的标签

docker run -p 8188:80 -t xxx=》创建一个新的容器并运行一个命令
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
-t: 为容器重新分配一个伪输入终端。但此处感觉是找到对应标签在端口容器执行
