**优化网页访问实现更多控制功能**

### Boa静态网页服务器

1. Boe直接下载的package.tar.gz解压缩后
2. 相应修改文件参数，执行./configure, 生成make文件
3. 再make出二进制boa文件
4. 将Boa拷贝进/bin目录
5. 将package下的boa.conf拷贝进/etc/boa文件夹并适当修改
6. 在/www文件夹下生成index.html，及/cgi-bin文件夹，以备后续静态网页按钮对应的脚本执行

（备份好相关文件及代码）

### 静态网页执行脚本

1. 脚本执行 - 根据deepseek/kimi生成的网页
2. 设置/etc/sudoers.d/里面的nobody-run-scr1设置允许nobody执行root的参数
3. 将index静态网页里的按钮参数与/data/local/tmp下的各个.sh对应

（*保存好相关html代码，/cgi-bin下面的.py文件 - python-CGI）

### NGINX

1. 尝试过用boa设置反向代理花生壳内网映射的80端口到rslsync的8888端口，最终由于无法实现认证登录所以采用了nginx
2. 将boa监听端口改为8081
3. 将nginx监听端口改为8080
4. 对应花生壳服务器80端口或443端口建立映射到内网127.0.0.1:8080
5. nginx在设置里将location分配端口转发/反向代理
6. 代理内容为除了/rslsync/和/gui/两个下级网址，其他都转到8081即Boa监听的端口，访问静态网页
7. 如此可通过nginx扩展其他类似rslsync的app其他端口通过花生壳一条映射访问