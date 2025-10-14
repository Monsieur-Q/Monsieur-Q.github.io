# 给armbian服务器nginx->boa静态网页添加登录密码认证

**一、生成密码文件（只需做一次）**
1. 安装工具（Debian/Ubuntu/Armbian 都一样）
`sudo apt update && sudo apt install apache2-utils -y`

**2. 建一个目录专门放密码**
`sudo mkdir -p /etc/nginx/passwd`

**3. 依次写入 3 个账号（每行一个，-c 只有第一次用）**
```
sudo htpasswd -bc /etc/nginx/passwd/boa_users alice 123456
sudo htpasswd -b /etc/nginx/passwd/boa_users bob 654321
sudo htpasswd -b /etc/nginx/passwd/boa_users carol abcdef
```

**4. 确认文件权限**
```
sudo chmod 640 /etc/nginx/passwd/boa_users
sudo chown root:www-data /etc/nginx/passwd/boa_users
```

**二、修改 Nginx 配置**
假设你现在的反向代理片段长这样（http 段，或者 server 段）：

```
location / {
proxy_pass http://127.0.0.1:8080; # Boa 监听的端口
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
}

```
把它改成：
```

location / {
# 1. 先要求基本认证
auth_basic “Armbian Boa — Login Required”;
auth_basic_user_file /etc/nginx/passwd/boa_users;

# 2. 认证通过后再反向代理到 Boa
proxy_pass http://127.0.0.1:8080;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
}
```

保存后检查语法并重载：

```
sudo nginx -t
sudo systemctl reload nginx
```

**三、验证**
内网用浏览器访问 http://<内网IP>/index-time.html
会弹出一个“输入用户名密码”的框，输入 alice/123456 才能看到页面。
外网通过穿透域名访问，同样会先弹登录框；输错 3 次直接 401。
一旦浏览器缓存了认证头，关闭标签再开会自动带过去；清缓存或换浏览器会重新要求登录。

**四、可选增强**
只保护部分目录
如果你只想保护 /www/web/secret/ 下的文件，而根目录可以匿名看，
就把上面的 location / 改成 location /secret/ { … } 即可。

用 HTTPS
既然已经暴露到外网，强烈建议给 Nginx 配个免费证书（certbot 一行命令），
否则 Basic Auth 的用户名密码是 Base64 明文传输，容易被中间人截获。

退出登录
Basic Auth 本身没有“退出”按钮，让用户关浏览器或清缓存即可；
如果一定要“点一下就退出”，只能换成 Cookie/Session 方案（Lua 或者写个简单后端），
那就超出“静态页 + Nginx”的范畴了。

**一句话总结**
“让 Nginx 先挡一道 Basic Auth 的墙，过了再把请求交给 Boa”，
不需要改 Boa、不需要改 HTML，3 组账号提前写在 /etc/nginx/passwd/boa_users 里，
内外网访问都会先弹登录框，认证失败就永远看不到真正的 index-time.html。

**关于密码**
可以不用 apache2-utils，只要你能“自己造”出和 htpasswd 生成的 完全一样的文件格式 即可。
Basic Auth 对 Nginx 来说只认“明文文件”里每一行的格式：

username:{加密方法}加密后的密码目前最常用的是 bcrypt（$2y$...）或 apr1（MD5，以 $apr1$ 开头）。
只要你能用任意工具/语言生成这三行文本，Nginx 就能读，无需安装 apache2-utils。

下面给你 3 种“零依赖”做法，挑一个顺手的即可。

用在线生成器（最懒）
打开
`https://hostingcanada.org/htpasswd-generator`
输入账号、密码，选 bcrypt 或 apr1，把结果复制下来，粘进文件：

```
alice:$2y$10$N1f8T4z5V9n4aB7cD1eF2uL6mQ9sR8wX5yA3bC7dE0gH4jK6lN8pQ1w
bob:$2y$10$K2g9U5y6V8pQ3wR7tY4A1xB6cD9eF0gH2jK5lM8nQ1oR4sU7vX0y
carol:$2y$10$M3h0V6z7W9qR4sT8uY5B2xC7dE1fG3hI4kL6mN9oQ2pS5tV8wX1y
```
保存成 /etc/nginx/passwd/boa_users，后面配置照用。

用 Python（系统自带）
Armbian 默认带 Python3，复制下面脚本一次性生成：

```
python3 - <<'PY'
import bcrypt, getpass, pathlib, sys

out = pathlib.Path('/etc/nginx/passwd/boa_users')
out.parent.mkdir(exist_ok=True)

users = [('alice','123456'),
         ('bob','654321'),
         ('carol','abcdef')]

with out.open('w') as f:
    for u, p in users:
        hashed = bcrypt.hashpw(p.encode(), bcrypt.gensalt(rounds=10)).decode()
        f.write(f'{u}:{hashed}\n')
PY
```
用 OpenSSL 纯命令（最小依赖）
系统一定有 openssl，但它只能做 apr1（MD5）——足够安全，且 Nginx 支持：

**生成单条**
`openssl passwd -apr1 123456`
输出示例：$apr1$8XJLwKBT$VmrLBBdRVWw8qTAv9NHLv0

** 手工拼文件**
```
cat >/etc/nginx/passwd/boa_users <<EOF
alice:$apr1$8XJLwKBT$VmrLBBdRVWw8qTAv9NHLv0
bob:$(openssl passwd -apr1 654321)
carol:$(openssl passwd -apr1 abcdef)
EOF
```

