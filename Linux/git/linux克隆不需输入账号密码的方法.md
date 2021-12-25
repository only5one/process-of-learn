linux下每次git clone不需输入账号密码的方法

在~/下，
````
touch .git-credentials
vim .git-credentials
````

````
https://{username}:{password}@github.com
````
2. 在终端下执行
````
git config --global credential.helper store
````
3. 可以看到~/.gitconfig文件，会多了一项：
````
[credential]
helper = store
````
这个时候输入命令git clone http://username@url 时不需要输入密码，即可完成代码的git