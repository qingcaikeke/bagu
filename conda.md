- 对于 NumPy，你应该能够熟练地创建、操作数组，进行基本的数学运算，理解广播机制。
- 对于 Pandas，你应该能够独立地创建和操作 Series 和 DataFrame，进行简单的数据清洗和处理。

jupter用一个新的虚拟环境

conda create --name <虚拟环境名称> python=3.8

conda activate <虚拟环境名称>

pip install ipykernel

python -m ipykernel install --user --name=pytorch

常用，注意，jupte打开时默认用的是base，需要收到activate要的虚拟环境

conda env list

conda list 当前环境的所有包

conda env create -f environment.yml，创建一个 Conda 环境，环境的配置信息存储在一个名为 `environment.yml` 的文件中。（安装指定的软件包和它们的版本。）

**新clone一个项目**

1.创建虚拟环境 conda create -n myenv python=3.8 

 2.激活虚拟环境 conda activate myenv

3.cd 路径

np.arange(9.)这个`9.`是用来表示数字类型的。

