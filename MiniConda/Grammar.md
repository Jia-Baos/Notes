# MiniConda常用命令

1. `conda create -n nlp python=3.6  （创建python=3.6的环境，名字叫做nlp）`
2. `conda remove -n nlp --all   （删除虚拟环境）`
3. `conda activate nlp  （激活环境）`
4. `source activate nlp （如果前面一步操作进不去环境，那执行此命令后再执行前面的命令）`
5. `conda deactivate    （退出当前虚拟环境）`
6. `conda-env list  （linux下查看已有的虚拟环境）`
7. `python --version    （查看python版本）`
8. `conda list  （查看环境下面的安装包）`
9. `conda -V    （查看conda版本）`
10. `conda info -e  （查看环境名）`
11. `conda info --envs  （查看已有的虚拟环境）`
12. `conda create --prefix=D:/Conda/envs/envs_name python=3.8   （将虚拟环境安装到指定路径，默认路径为C盘）`
13. `conda update -n base -c defaults conda conda更新`

## jupyter notebook的使用

1. `jupyter notebook, then enter    （开启jupyter notebook）`
2. `Ctrl + C    （关闭jupyter notebook）`