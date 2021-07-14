## 1) Configuring GIT timeline views （配置 git 的时间线视图）

``` shell

# 若使用全局配置

[format]
	pretty = %ad %Cred%h%Creset %C(yellow)%d%Creset %s <%Cgreen%an%Creset>
[log]
	decorate = short
  
# 或在终端执行

git log --oneline --graph

```

参考自：https://alvincrespo.com/blog/til-episode-4/
