# argparse  
https://www.jianshu.com/p/a41fbd4919f8
parser = argparse.ArgumentParser(description='xxxx') // 创建解析对象
parser.add_argument()   //添加命令行参数和选项
parser.parse_args()  //进行解析
parser.add_argument('--debug','-d',help='Enable debug mode',action='store_true')  //action="store_true" 不需要指定参数值 
add_argument支持的参数： type=int (数据类型), choices=[](限定范围), help='xxx'(帮助) default=xxx(默认值)

group1=parser.add_mutually_exclusive_group() //互斥参数
group1.add_argument(xxx)

# LDAP   
LDAP Host: 
Port: 389
Base DN: 
Search attribute: sAMAccountName
Bind DN: 
Bind password: xxx


# pyxl 设置格式  
https://www.codeleading.com/article/9359278171/
