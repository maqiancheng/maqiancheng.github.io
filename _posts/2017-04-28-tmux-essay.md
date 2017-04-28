---
layout: post
title: "tmux随笔"
date: 2017-04-28
tag: tmux
---   
## tmux快捷键

### 新建tmux会话
```ruby
tmux new -s session-name
```

### 列出所有会话
```ruby
tmux ls
```

### 接入之前的会话
```ruby
tmux a -t session-name
```

### 断开会话
```ruby
tmux detach
```

### 关闭会话
```ruby
tmux kill-session -t session-name
```

### 操作帮助
```ruby
ctl+b ?
```
