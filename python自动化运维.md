# python devops  
https://github.com/py3study/wdpy websocket+Django+python+paramiko实现web页面执行命令并实时输出

https://blog.csdn.net/enweitech/article/details/89316979 各种框架
1. dockerfile(build, upload), 动静分离, 自动化部署
2. 后端Django
3. 前端框架： 
* web框架：Bootstrap, LayUI, ElementUI(VUE配套)
* js框架：Node.Js  Angular.js  JqueryMobile  React.js -> Vue.js
* UI组件：Echarts
1. ：

功能设计：
1. 登录LDAP认证，权限管理
2. CMDB管理
3. 任务管理： 执行shell,python,ansible脚本，显示实时日志
* 服务器初始化
* 添加cybeark
* jira查询等
* 对接device42 ?
4. 日志审计
5. 并发
6. 后续扩展：对接k8s

## python virtualenv和venv  
### virtualenv需单独安装  
pip3 install virtualenv
virtualenv --no-site-packages venv
source ./venv/bin/activate, source ./venv/Scripts/activate
deactive
### venv python3.3以上自带的
python3 -m venv ./venv
source ./venv/bin/active

## django基本命令  
django-admin startproject xxx
python manage.py startapp app-name 或者 django-admin startapp app-admin

python manage.py makemigrations //创建更改的文件
python manage.py migrate //将生成的py文件应用到数据库
python manage.py flush //清空数据库

python manage.py runserver 0.0.0.0:port_xxx

python manage.py createsuperuser
python manage.py changepassword username

//导入导出
python mamange.py dumpdata appname > appname.json
python manage.py loaddata appname.json

django shell:
python manage.py shell //可直接调用model.py中的api
python manage.py dbhell

## xadmin 权限管理  
https://www.cnblogs.com/haiton/p/11224987.html
https://blog.csdn.net/weixin_30751947/article/details/99949122






# HTML5 & JS  
