##docker push 到harbor失败
原因:docker push 需要交互式输入密码，jenkin为非交互式运行,而使用docker login -u -p 明文指定密码不安全也会导致发送get
请求却携带参数。

解决方案:cat my_password.txt | docker login --username “ ” --password-stdin
密码存储在一个文件中，docker login可以从文件中获取密码。
在有公网IP的环境中可以使用expect来进行交互。