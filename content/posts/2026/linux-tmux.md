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
