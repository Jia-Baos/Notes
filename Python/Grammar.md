# Git Bash下的python交互

1. `python test.py  （执行python脚本,此种方式进入的是主python环境，因为你没有激活虚拟环境呀）`
2. `python  （进入交互界面，在此界面中可以进行类似于jupyter notebook的操作）`
    1. `python -i   （正常执行完python脚本后，还会进入python交互模式）`
    2. `python test.py  （执行python脚本就结束，不会进入交互模式）`
    3. `winpty python test.py   （执行完python脚本就结束，不会进入交互模式）`
3. `exit()  （退出界面）`
4. `pip list    （查看该python环境下安装了那些第三方库）`

## 新的文件夹下新建虚拟环境（在Pycharm中使用）

1. `python -m venv venv （创建虚拟环境）`
2. `source venv/Scripts/activate    （激活虚拟环境）`
3. `deactivate  （关闭虚拟环境，pycharm terminal也采用此种方法关闭虚拟环境）`
4. `ctrl + c    (退出visdom远程服务)`
