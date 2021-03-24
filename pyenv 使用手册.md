# pyenv 使用手册

## 安装

```shell
brew install pyenv
```

## 激活

在.bashrc或者.bash_profile中添加以下三行

```shell
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

## 系统环境

### 查看所有环境

```shell
~ pyenv versions
  system
```

## 安装python

### 查看Python环境列表

```shell
pyenv install -l
```

通过上述命令可以看到pyenv所有支持的环境列表,选择指定的版本进行安装:

```shell
pyenv install 3.x.x
```

