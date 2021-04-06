# Python

## 开发环境

### pyenv 在国内的使用

以安装 **python 3.6.12** 为例，`pyenv install`默认使用 python 官方源安装 python，在不使用或者不方便使用梯子时，用起来特别难受，这里对使用已有包进行安装的方法进行记录：

1. 按照[官方文档](https://github.com/pyenv/pyenv)提供的方式安装`pyenv`
2. 从其它地方拷贝或者从国内镜像源下载 python 安装包到任意目录：

    - [淘宝](https://npm.taobao.org/mirrors/python/)

3. 安装:

    ```bash
    PYTHON_BUILD_BUILD_PATH=`pwd` pyenv install 3.6.12
    ```

4. 直接修改 mirror_url 安装：

    ```bash
    PYTHON_BUILD_MIRROR_URL=https://npm.taobao.org/mirrors pyenv install 3.6.12
    ```
