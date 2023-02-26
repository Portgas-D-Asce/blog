# 示例

```bash
# 拷贝本地文件到远程
scp local_file remote_username@remote_ip:remote_file

scp -r local_folder remote_username@remote_ip:remote_folder

# 拷贝远程文件到本地
scp remote_username@remote_ip:remote_file local_file

scp -r remote_username@remote_ip:remote_folder local_folder

# 使用密钥
scp -i id_rsa local_file remote_username@remote_ip:remote_file
```

