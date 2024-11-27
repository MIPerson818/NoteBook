# 工作准备阶段11.25--

## 安装环境



```
absl-py==2.1.0
addict==2.4.0
antlr4-python3-runtime==4.9.3
argcomplete==3.5.0
asttokens==2.4.1
av==12.3.0
av2==0.2.1
backcall==0.2.0
black==24.8.0
blinker==1.8.2
boto3==1.35.37
botocore==1.35.37
cachetools==5.5.0
certifi==2024.7.4
charset-normalizer==3.3.2
click==8.1.7
cloudpickle==3.0.0
colorlog==6.8.2
contourpy==1.1.1
cycler==0.12.1
Cython==3.0.11
dataset-management-sdk==0.4.46
debugpy==1.8.6
decorator==5.1.1
distlib==0.3.8
et-xmlfile==1.1.0
exceptiongroup==1.2.2
executing==2.0.1
filelock==3.15.4
fire==0.6.0
flake8==7.1.1
Flask==3.0.3
Flask-JWT-Extended==4.6.0
fonttools==4.53.1
fsspec==2024.6.1
fvcore==0.1.5.post20221221
GeometricKernelAttention==1.0
google-auth==2.34.0
google-auth-oauthlib==1.0.0
grpcio==1.65.5
h5py==3.11.0
huggingface-hub==0.24.6
hydra-core==1.3.2
idna==3.7
imageio==2.35.1
importlib_metadata==8.4.0
importlib_resources==6.4.3
iniconfig==2.0.0
iopath==0.1.9
ipython==8.12.3
itsdangerous==2.2.0
jedi==0.19.1
Jinja2==3.1.4
jmespath==1.0.1
joblib==1.4.2
kiwisolver==1.4.5
llvmlite==0.31.0
lyft-dataset-sdk==0.0.8
Markdown==3.7
markdown-it-py==3.0.0
MarkupSafe==2.1.5
matplotlib==3.5.3
matplotlib-inline==0.1.7
mccabe==0.7.0
mdurl==0.1.2
mmsegmentation==0.14.1
MultiScaleDeformableAttention==1.0
mypy-extensions==1.0.0
networkx==2.2
niofs==1.6.6
nox==2024.4.15
numba==0.48.0
numpy==1.23.0
nuscenes-devkit==1.1.11
oauthlib==3.2.2
omegaconf==2.3.0
opencv-python==4.10.0.84
openpyxl==3.1.5
packaging==24.1
pandas==2.0.3
parso==0.8.4
pathspec==0.12.1
pexpect==4.9.0
pickleshare==0.7.5
pillow==10.4.0
platformdirs==4.2.2
plotly==5.23.0
pluggy==1.5.0
plyfile==1.0.3
portalocker==2.10.1
prettytable==3.11.0
prompt_toolkit==3.0.47
protobuf==3.19.6
ptyprocess==0.7.0
pure_eval==0.2.3
pyarrow==17.0.0
pyasn1==0.6.0
pyasn1_modules==0.4.0
pychecktype==1.4.1
pycocotools==2.0.7
pycodestyle==2.12.1
pyflakes==3.2.0
Pygments==2.18.0
PyJWT==2.9.0
pyparsing==3.1.2
pyproj==3.5.0
pyquaternion==0.9.9
pytest==8.3.2
python-dateutil==2.9.0.post0
pytz==2024.1
PyWavelets==1.4.1
PyYAML==6.0.2
rdp==0.8
requests==2.32.3
requests-oauthlib==2.0.0
rich==13.7.1
rsa==4.9
s3transfer==0.10.3
safetensors==0.4.4
scikit-image==0.19.3
scikit-learn==1.3.2
scipy==1.10.1
Shapely==1.8.5.post1
similaritymeasures==1.1.0
six==1.16.0
stack-data==0.6.3
submitit==1.5.1
tabulate==0.9.0
tenacity==9.0.0
tensorboard==2.14.0
tensorboard-data-server==0.7.2
termcolor==2.4.0
terminaltables==3.1.10
threadpoolctl==3.5.0
tifffile==2023.7.10
timm==1.0.8
tomli==2.0.1
tqdm==4.66.5
traitlets==5.14.3
trimesh==2.35.39
typing_extensions==4.12.2
tzdata==2024.1
urllib3==1.26.20
virtualenv==20.26.3
wcwidth==0.2.13
Werkzeug==3.0.3
yacs==0.1.8
yapf==0.40.1
zipp==3.20.0
zstandard==0.23.0
descartes==1.1.0 -e git+https://github.com/facebookresearch/detectron2.git@2a420edb307c9bdf640f036d3b196bed474b8593#egg=detectron2
mmdet==2.14.0 -e git+ssh://git@ad-gitlab.nioint.com/yu.fang1/vma.git@2afb8ceed33ac16b7df3470601ea20fc9198ebaf#egg=mmdet3d&subdirectory=mmdetection3d
torch 
torchvision 
mmcv-full

```

## 配置终端bash

### 安装 `ncurses-term` 包

```bash
dpkg -l | grep ncurses-term
```

如果该包已安装，你会看到类似以下的输出信息，显示包的名称、版本等详细信息：

```plaintext
ii  ncurses-term  6.2-0ubuntu2  amd64        Terminal-related functions (GNU ncurses)
```

如果没有看到任何输出，则表示该包未安装。

```bash
sudo apt-get update
sudo apt-get install ncurses-term
```

安装完成后，编辑 `.bashrc` 文件

```bash
vim ~/.bashrc
```

在文件中添加或修改以下内容来启用彩色 `bash`：
查找文件中是否存在 `PS1` 变量的设置部分，如果存在，可以在其现有设置基础上进行修改；如果不存在，则在文件末尾添加以下几行代码：

```bash
# 设置彩色终端
if [ -t 1 ]; then
    export TERM=xterm-256color
fi

# 设置命令提示符颜色（示例）
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
```

### dircolors 配置

https://zhuanlan.zhihu.com/p/594067154

```bash
# 代码下载:
git clone https://github.com/seebi/dircolors-solarized
# 配色db文件拷贝,这里使用的是 color256
cp dircolors-solarized/dircolors.256dark ~/.dircolors
```

同样在~/.bashrc后加入：

```bash
# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    # echo 'dircolors exist'
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi
```

保存并关闭 `.bashrc` 文件后（在 `vim` 中按 `:wq` 保存并退出），执行以下命令使新的配置生效：

```bash
source ~/.bashrc
```

### 修改 `~/.bash_profile` 文件

加入以下代码：

```bash
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

在 Debian 系的 Linux 系统中，当通过 `ssh` 等方式登录时，系统默认会执行 `~/.bash_profile` 文件（如果存在），所以通过在这个文件中添加上述代码，就能确保每次登录后自动执行 `source ~/.bashrc` 操作了。



## `virtualenv`虚拟环境

```
sudo apt install virtualenv
```

```
# 创建新环境
virtualenv (-p /usr/bin/python3.8） myenv
# 复制一个环境
virtualenv-clone source_env newEnv
```

```
# 激活环境
source myenv/bin/activate
# 取消环境
deactivate
```

```
pip install -r requirements.txt
pip install --ignore-installed --no-deps -r requirements.txt
```



```bash
which *** # 找软件路径 如python pip conda
find / -type d -name "myfolder"
-type f(file) # 可以使用通配符* 此外 b c p s l等
```

#### 

```
pip freeze > requirements.txt
```

```
pip install -r requirements.txt
```



# MMLAB框架学习

https://v9999.blog.csdn.net/article/details/128486362







# 标注数据质检

专有名词记录：

```
internalRoad 内部路？
intersection 路口

```



python /home/jack.leey/vma-road_centerline_network/tools/custom/generate_centerline_from_pn_anno_new.py -img_dir /map/jack.leey/test/dataset_send2label -json_dir /map/jack.leey/test/anno_json -valid_map_json /map/yu.fang/pn_env_render_dataset/validation_map_ids.json  -traj_dir none -out_dir /map/jack.leey/test/output_send2label







```bash
source ~/.bashrc
vim ~/.bashrc
source ~/ENVS/VMA/bin/activate
/home/jack.leey/vma-road_centerline_network/tools/custom/generate_centerline_from_pn_anno_new.py 
```

```python
import torch
print(torch.version.cuda)  # 应该输出 11.1
print(torch.cuda.is_available())  # 检查是否能使用 GPU
python -c "import torch; print(torch.version.cuda) ; print(torch.cuda.is_available()) "
```

![image-20241126161827584](/home/nio/.config/Typora/typora-user-images/image-20241126161827584.png)

##### 

![image-20241126161719090](/home/nio/.config/Typora/typora-user-images/image-20241126161719090.png)





'/map/jack.leey/test/anno_json/../bev_img_range_info/PK-Downtown-4df0464a-1a585d94_aip_instance_1035331612_bev_img_range_info.json'

PK-Downtown-4df0464a-1a585d94_aip_instance_1035331612_vec_road_obstacle_0.jpg





# 数据标签可视化

annotationClass:有：

RoadBoundaryArea、RoadSplit、intersectionPoint、one_wayStreet、two_wayStreet、internalPath、start、end、





颜色标注

draw_polygon：蓝色(255, 0, 0)

```python
    if obj["annotationClass"] == "internalPath":
        color = (255, 0, 0)
    cv2.polylines(img, [pixel_points], isClosed=True, color=color, thickness=10)
```

draw_point：默认红色(0, 0, 255)，若intersectionPoint则深红色，给出的指示点为绿色

```
color = (0, 0, 255)
    if obj["annotationClass"] == "intersectionPoint":
        color = (255, 255, 255)
    # CHANGE: 改成填充圆点
    cv2.circle(img, tuple(pixel_points.tolist()), color=color, radius=50, thickness=-1)
```

draw_curve2:没找到





python /map/jack.leey/test/generate.py -img_dir /map/jack.leey/test/dataset_send2label -json_dir /map/jack.leey/test/anno_json -valid_map_json /map/jack.leey/test/validation_map_ids.json -traj_dir none -out_dir /map/jack.leey/test/output_send2label 
