# WSL-Windows Subsystem For Linux

# 1、安装WSL

```rust
// 用管理员身份打开Windows终端管理器
wsl --install 
```

# 2、查找 WSL 可安装的版本并安装

```rust
wsl -l -o
wsl --install -d <DistroName>
```

# 3、关闭 WSL

```rust
wsl --shutdown
```

# 4、卸载WSL子系统

```rust
wsl --unregister <DistroName>
```

# 5、导出子系统

```rust
wsl --shutdown <DistroName>
wsl --export <DistroName> <your_path\file_name> (eg:C:\Workspace\wsl-bak\VLN\Ubuntu-20.04.tar)
```

# 6、还原子系统

```rust
wsl --import <DistroName> <your_install_path> <your_Ubuntu_bak_path>
// 1. 但是注意，重新导入之后会将用户变为root
// 2. 改为默认用户
// 3. ls /home/   ———— 查看当前用户
// 4. su your_username ———— 尝试切换到用户权限
// 5. sudo su ———— 切换至root权限
// 6. 修改默认配置文件
		/*
			cp /etc/wsl.conf /etc/wsl.conf.bk
			vim /etc/wsl.conf
					在最前面增加以下内容
					[user]
					default=your_username
		*/
// 7. shutdown当前的子系统并重启

```

# 7、查看安装的子系统

```rust
wsl -l -v
```

# 8、启动WSL

```bash
wsl -d <DistroName>
```