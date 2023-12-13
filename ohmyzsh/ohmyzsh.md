# [ohmyzsh](https://ohmyz.sh/)

终端驱动框架，用于管理zsh配置和主题。




## 安装


### 1、[Install ZSH](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)

1. There are two main ways to install Zsh:
   - With the package manager of your choice, *e.g.* `sudo apt install zsh` (see [below for more examples](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH#how-to-install-zsh-on-many-platforms))
   - From [source](https://zsh.sourceforge.io/Arc/source.html), following [the instructions from the Zsh FAQ](https://zsh.sourceforge.io/FAQ/zshfaq01.html#l7).
2. Verify installation by running `zsh --version`. Expected result: `zsh 5.0.8` or more recent.
3. Make it your default shell: `chsh -s $(which zsh)` or use `sudo lchsh $USER` if you are on Fedora.
   - Note that this will not work if Zsh is not in your authorized shells list (`/etc/shells`) or if you don't have permission to use `chsh`. If that's the case [you'll need to use a different procedure](https://www.google.com/search?q=zsh+default+without+chsh).
   - If you use `lchsh` you need to type `/bin/zsh` to make it your default shell.
4. Log out and log back in again to use your new default shell.
5. Test that it worked with `echo $SHELL`. Expected result: `/bin/zsh` or similar.
6. Test with `$SHELL --version`. Expected result: 'zsh 5.8' or similar



### 2、[Install Oh My Zsh!](https://github.com/ohmyzsh/ohmyzsh/wiki)

- Once you have zsh, you can install Oh My Zsh by simply running one of these commands:

  | Method    | Command                                                      |
  | --------- | ------------------------------------------------------------ |
  | **curl**  | `sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` |
  | **wget**  | `sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` |
  | **fetch** | `sh -c "$(fetch -o - https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` |

  **NOTE: the installer will rename an existing `.zshrc` file to `.zshrc.pre-oh-my-zsh`.**

- Alternatively, the installer is also mirrored outside GitHub. **Using this URL may be required if you're in a country like India or China, that blocks `raw.githubusercontent.com`**:

  | Method    | Command                                          |
  | --------- | ------------------------------------------------ |
  | **curl**  | `sh -c "$(curl -fsSL https://install.ohmyz.sh)"` |
  | **wget**  | `sh -c "$(wget -O- https://install.ohmyz.sh)"`   |
  | **fetch** | `sh -c "$(fetch -o - https://install.ohmyz.sh)"` |



### 3、[Settings](https://github.com/ohmyzsh/ohmyzsh/wiki/Settings)

#### 修改主题

```bash
# 打开zsh配置文件
$ vim ~/.zshrc

# 将ZSH_THEME改成ys
ZSH_THEME="ys"
# 还可以设置随机主题
# ZSH_THEME="random"

# history 命令时间格式
HIST_STAMPS="yyyy-mm-dd"

# 设置一些命令别名
alias gitgo="git-open" # git-open插件，在终端里打开当前项目的远程仓库地址
alias rm="trash" # 安装一个 trash 命令，替代 rm 命令，被删除的文件会放到垃圾桶
alias cp="cp -i # 防止 copy 的时候覆盖已存在的文件

# 更新配置：
$ source ~/.zshrc   
```



#### 配置插件

### 1. zsh-syntax-highlightin

这个插件直接在输入过程中就会提示你，当前命令是否正确，错误红色，正确绿色。

```cmd
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```



### 2. zsh-autosuggestions

会记录你之前输入过的所有命令，并且自动匹配你可能想要输入命令，然后按→补全。

```cmd
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```



### 3. Z

 内置插件，目录间快速跳转,不用再一直 `cd` 了



### 4. git-open

在终端里打开当前项目的远程仓库地址

```bash
git clone https://github.com/paulirish/git-open.git $ZSH_CUSTOM/plugins/git-open
```



### 5. [fzf](https://github.com/junegunn/fzf)

fzf是一款使用 GO 语言编写的交互式的 Unix 命令行工具。可以用来查找任何列表内容、文件、历史命令、 本机绑定的host、 进程、 Git 分支、进程等。所有的命令行工具可以生成列表输出的都可以再通过管道 pipe 到 fzf 上进行搜索和查找。

```bash
git clone --depth 1 <https://github.com/junegunn/fzf.git> ~/.fzf
~/.fzf/install
```



### 6. zsh-completions

命令补全插件，输入命令按Tab键后会提示可以使用的命令和说明。

```bash
#Clone the repository inside your oh-my-zsh repo:
git clone <https://github.com/zsh-users/zsh-completions> ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions

#Add it to FPATH in your .zshrc by adding the following line before source "$ZSH/oh-my-zsh.sh":
fpath+=${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions/src

#Start a new terminal session.
zsh

```





安装完成后，修改配置文件，然后更新配置：

```cmd
vim ~/.zshrc

# 以下是配置内容
ZSH_THEME="ys" # ZSH_THEME="robbyrussell"
plugins=(git zsh-syntax-highlighting zsh-autosuggestions Z git-open zsh-completions) # plugins=(git)
source ~/.bash_profile # 如果 .bash_profile 中有配置内容的话，还需增加一行加载.bash_profile

source .zshrc
```





---

## 以下内容为参考资料

---



### 4、Getting started

Once Oh My Zsh is installed:

- Take a look at the most common questions and gotchas in the [FAQ](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ).
- Get a quick summary of the built-in plugins: [Plugins Overview](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins-Overview).
- Take a look at our [Themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) and [Plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins) (read the READMEs first!).
- If you need more, you can look at [External themes](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes) and [External plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/External-plugins). **Take caution, we do not review these.**
- Have a look at the [Cheatsheet](https://github.com/ohmyzsh/ohmyzsh/wiki/Cheatsheet) for other Oh My Zsh tricks.

### 5、Advanced topics

- Having problems? Check out the [FAQ](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ) for common problems, or the [Troubleshooting](https://github.com/ohmyzsh/ohmyzsh/wiki/Troubleshooting) page for instructions on how to diagnose the issue.
- Want to change stuff about Oh My Zsh? Learn about [Customization](https://github.com/ohmyzsh/ohmyzsh/wiki/Customization).
- If you want to learn more, check out the [Resources](https://github.com/ohmyzsh/ohmyzsh/wiki/Resources) page for more information.