## WSL 下 Ubuntu 下载 Redis

https://phoenixnap.com/kb/install-redis-on-ubuntu-20-04

其中执行 `sudo systemctl restart redis.servic` 会报错，因为 wsl 没有 systemctl （参考自 https://github.com/MicrosoftDocs/WSL/issues/457）
