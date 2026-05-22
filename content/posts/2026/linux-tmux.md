## 
```
$ tmux ls
$ tmux attach -t 
```

### 新建会话
```
$ tmux new -s  <session-name>
```

### 分离会话 
```
$ tmux detach 
```
### 访问会话
```
# 使用会话编号
$ tmux attach -t 0

# 使用会话名称
$ tmux attach -t <session-name>
```
### 鼠标操作
tmux 默认没有启用鼠标，导致无法使用鼠标滚动历史记录。启用鼠标后，还可以直接点击切换不同窗格，以及拖动选择文字复制。

set-option -g mouse on # open mouse scroll

