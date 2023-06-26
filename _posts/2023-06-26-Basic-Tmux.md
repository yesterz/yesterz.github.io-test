---
title: "使用Jekyll的时候配置的记录"
categories:
  - Linux
tags:
  - tmux
  - tools
toc: true
---

### tmux使用学习文档
1. Tmux 使用手册 http://louiszhai.github.io/2017/09/30/tmux/
2. 基础教程 https://www.ruanyifeng.com/blog/2019/10/tmux.html
3. tmux之道？Tao of Tmux https://tao-of-tmux.readthedocs.io/zh_CN/latest/

### 基本使用流程

    # step 1: 新建会话
    tmux new -s <my_session>
    
    # step 2: 在 Tmux 窗口运行所需的程序
    
    # step 3: 按下快捷键将会话分离
    Ctrl+b d
    
    # step 4: 下次使用时，重新连接到会话
    tmux attach-session -t <my_session>

### 建立新的 tmux 窗口

    # 默认编号命名
    tmux
    
    # 推荐使用自定义窗口名称(窗口外使用)
    tmux new -s <session-name>
    

### 退出当前 tmux 窗口

    # 显式输入(窗口内使用)
    tmux dettach
    
    # 快捷键(linux)(窗口内使用)
    Ctrl+b　d

### 查看所有 tmux 窗口

    # 在窗口外显式输入
    tmux ls
    OR
    tmux list-session
    
    # 在窗口内显式输入
    tmux detach
    
    # 在窗口内快捷键(linux)
    Ctrl+b s

### 接入窗口

    # 使用会话编号(窗口外使用)
    tmux attach -t 0
    OR
    tmux at -t 0
    
    # 使用会话名称(窗口外使用)
    tmux attach -t <session-name>
    OR
    tmux at -t <session-name>

### 杀死、切换、重命名会话

    # 使用会话编号(窗口内外均可)
    tmux kill-session -t 0
    
    # 使用会话名称(窗口内外均可)
    tmux kill-session -t <session-name>
    
    # 在窗口内显式输入
    exit
    # 使用会话编号
    $ tmux switch -t 0
    
    # 使用会话名称
    $ tmux switch -t <session-name>
    # 显式输入(窗口内外均可使用)
    tmux rename-session -t <old-name> <new-name>
    
    # 快捷键(linux)
    Ctrl+b $
    

### 划分窗格

    # 划分上下两个窗格
    tmux split-window　# 显式输入
    Ctrl+b " # 快捷键 
    
    # 划分左右两个窗格
    tmux split-window -h # 显式输入
    Ctrl+b % # 快捷键
    
    # 光标切换到上方窗格
    tmux select-pane -U # 显式输入
    Ctrl+b <up arrow key> # 快捷键
    Ctrl+b ; # 快捷键
    
    # 光标切换到下方窗格
    tmux select-pane -D # 显式输入
    Ctrl+b <down arrow key> # 快捷键
    Ctrl+b o # 快捷键
    
    # 光标切换到左边窗格
    tmux select-pane -L # 显式输入
    Ctrl+b <left arrow key> # 快捷键
    
    # 光标切换到右边窗格
    tmux select-pane -R # 显式输入
    Ctrl+b <right arrow key> # 快捷键
    
    # 当前窗格上移
    tmux swap-pane -U # 显式输入
    Ctrl+b { # 快捷键
    
    # 当前窗格下移
    tmux swap-pane -D # 显式输入
    Ctrl+b } # 快捷键
    
    # 关闭当前窗格
    Ctrl+b x # 快捷键



### 常用配置文件

配置文件位于 ~/.tmux.conf

    set -g prefix ^b
    set -g prefix2 F9
    bind ^b send-prefix
    
    bind h select-pane -L
    bind j select-pane -D
    bind k select-pane -U
    bind l select-pane -R
    bind C-l select-window -l
    setw -g mode-keys vi
    setw -g utf8 on
    bind -t vi-copy v begin-selection
    bind -t vi-copy y copy-selection
    
    
    # List of plugins
    set -g @plugin 'tmux-plugins/tpm'
    set -g @plugin 'tmux-plugins/tmux-sensible'
    
    # Other examples:
    # set -g @plugin 'github_username/plugin_name'
    # set -g @plugin 'git@github.com/user/plugin'
    # set -g @plugin 'git@bitbucket.com/user/plugin'
    
    # Initialize TMUX plugin manager (keep this line at the very bottom of
    # tmux.conf)
    run '~/.tmux/plugins/tpm/tpm'
    
    
    set -s escape-time 0
    
    
    bind c new-window -c "#{pane_current_path}"
    bind % split-window -h -c "#{pane_current_path}"
    bind '"' split-window -c "#{pane_current_path}"
    
    
    run-shell ~/.tmux/tmux-resurrect/resurrect.tmux
    
    
    set-window-option -g utf8 on
    set -g default-terminal "screen-256color"
    

### 常见快捷键

    tmux分屏工具的操作：
    “     将当前面板上下分屏
    %     将当前面板左右分屏
    x     关闭当前分屏
    !     将当前面板置于新窗口,即新建一个窗口,其中仅包含当前面板
    Ctrl+方向键    以1个单元格为单位移动边缘以调整当前面板大小
    Alt+方向键     以5个单元格为单位移动边缘以调整当前面板大小
    空格键   可以在默认面板布局中切换，试试就知道了
    q     显示面板编号
    o     选择当前窗口中下一个面板
    方向键   移动光标选择对应面板
    {     向前置换当前面板
    }     向后置换当前面板
    Alt+o 逆时针旋转当前窗口的面板
    Ctrl+o      顺时针旋转当前窗口的面板
    z     tmux 1.8新特性，最大化当前所在面板
    ,     重命名当前窗口
    
    基本操作：
    ?     列出所有快捷键；按q返回
    d     脱离当前会话,可暂时返回Shell界面，输入tmux attach能够重新进入之前会话
    s     选择并切换会话；在同时开启了多个会话时使用
    D     选择要脱离的会话；在同时开启了多个会话时使用
    :     进入命令行模式；此时可输入支持的命令，例如kill-server所有tmux会话
    [     复制模式，光标移动到复制内容位置，空格键开始，方向键选择复制，回车确认，q/Esc退出
    ]     进入粘贴模式，粘贴之前复制的内容，按q/Esc退出
    ~     列出提示信息缓存；其中包含了之前tmux返回的各种提示信息
    t     显示当前的时间
    Ctrl+z      挂起当前会话
    
    窗口操作:
    c     创建新窗口
    &     关闭当前窗口
    数字键   切换到指定窗口
    p     切换至上一窗口
    n     切换至下一窗口
    l     前后窗口间互相切换
    w     通过窗口列表切换窗口
    ,     重命名当前窗口，便于识别
    .     修改当前窗口编号，相当于重新排序
    f     在所有窗口中查找关键词，便于窗口多了切换
    
    面板操作:
    “     将当前面板上下分屏
    %     将当前面板左右分屏
    x     关闭当前分屏
    !     将当前面板置于新窗口,即新建一个窗口,其中仅包含当前面板
    Ctrl+方向键    以1个单元格为单位移动边缘以调整当前面板大小
    Alt+方向键     以5个单元格为单位移动边缘以调整当前面板大小
    空格键   可以在默认面板布局中切换，试试就知道了
    q     显示面板编号
    o     选择当前窗口中下一个面板
    方向键   移动光标选择对应面板
    {     向前交换当前面板
    }     向后交换当前面板
    Alt+o 逆时针旋转当前窗口的面板
    Ctrl+o      顺时针旋转当前窗口的面板
    z     tmux 1.8新特性，最大化当前所在面板
    
    
    新建session：
    ctrl+b new -s <name-of-my-new-session>
    tmux new -s <name-of-my_new_seesion>
    
    切换session：
    ctrl+b+s
    
    重命名session：
    ctrl+b+$
    
    上下分屏与左右分屏切换： ctrl + b  再按空格键
    
    退出tmux的方式：
    Ctrl+b + d
    
    删除session
    prefix+kill-session -t index
    
    -t选择session、window和pane:
    tmux select-window -t ${tmux_session}:${tmux_window}.${tmux_pane}