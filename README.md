
### 1. 准备环境
确保你的 Debian 系统安装了 Python 3.10 及以上版本。如果没有，请先安装。

#### 安装 Python 3.10

如果你的 Debian 系统没有 Python 3.10，你可以通过以下命令安装：

```bash
sudo apt update
sudo apt install python3.10 python3.10-venv python3.10-dev
```

#### 安装 `pip` 和 `venv`

确保 `pip` 和 `venv` 安装正确：

```bash
sudo apt install python3-pip python3-venv
```

### 2. 上传或传输包到 Debian 系统

将你的 `liliput_api-0.1.0.tar.gz` 包文件上传到你的 Debian 服务器或机器上。你可以使用 `scp`、`rsync` 或任何其他文件传输工具来完成此步骤。例如：

```bash
scp liliput_api-0.1.0.tar.gz user@your_debian_server:/path/to/directory
```

### 3. 创建虚拟环境

为了确保你的包在一个干净的环境中运行，建议创建一个 Python 虚拟环境。

```bash
# 创建一个新的虚拟环境目录
python3.10 -m venv liliput_api_env

# 激活虚拟环境
source liliput_api_env/bin/activate
```

### 4. 安装 Python 包

激活虚拟环境后，使用 `pip` 安装你的 `.tar.gz` 包：

```bash
pip install liliput_api-0.1.0.tar.gz
```

这个命令将解压并安装 `liliput_api` 包及其所有依赖。


####  确保你的包安装成功

首先，确保你的包 `liliput-api` 已经正确安装，并且可以通过命令行运行。

```bash
pip list | grep liliput-api
```

如果你的包安装成功，它会出现在输出列表中。


### 5. 启动 FastAPI 应用

根据你的项目结构，假设 `liliput_api` 安装包中的 `FastAPI` 应用位于 `app.py` 文件中，你可以使用 `uvicorn` 启动该应用。

首先，确保 `uvicorn` 已安装：

```bash
pip install uvicorn
```

然后运行 `uvicorn` 启动你的 `FastAPI` 应用：

```bash
uvicorn liliput_api.main:app --host 0.0.0.0 --port 8080 --reload
```

#### 解释参数：

- **`liliput_api.main:app`**: 这里假设 `liliput_api` 是安装后的包名，`main` 是模块名，`app` 是 FastAPI 实例。
- **`--host 0.0.0.0`**: 使应用程序对外部网络可见。
- **`--port 8080`**: 指定应用运行的端口。你可以更改为其他可用端口。
- **`--reload`**: 使服务器在代码更改时自动重新加载（可选，适用于开发环境）。






#### 5.1 创建一个可执行脚本(可选)

你需要一个启动脚本来运行你的 FastAPI 应用。因为你在 `main.py` 中使用了 `if __name__ == "__main__":` 这种模式，我们假设你的 `app` 对象是在 `liliput_api.main` 里面定义的。

在 Linux 中，你可以创建一个可执行的 shell 脚本：

```bash
nano start_liliput_api.sh
```

在这个脚本中，写入以下内容：

```bash
#!/bin/bash

# Activate your virtual environment if necessary
# source /path/to/your/venv/bin/activate

# Run the FastAPI app with Uvicorn
uvicorn liliput_api.main:app --host 0.0.0.0 --port 8080 --reload
```

保存并退出 (`Ctrl + O` 然后 `Ctrl + X`)。

#### 5.2 使脚本可执行

给脚本执行权限：

```bash
chmod +x start_liliput_api.sh
```

#### 5.3 后台运行脚本

你可以使用 `nohup` 或 `&` 来在后台运行脚本。以下是使用 `nohup` 的方式：

```bash
nohup ./start_liliput_api.sh &
```

这会在后台运行脚本，并且即使你退出终端，应用程序也会继续运行。输出和错误日志会被重定向到 `nohup.out` 文件。


### 6. 访问 FastAPI 应用

应用程序启动后，你可以通过浏览器或 `curl` 等工具访问应用：

```bash
http://your_debian_server_ip:8080/
```

### 7. 配置生产环境（可选）

对于生产环境，不建议使用 `--reload`，你还可以考虑将 `uvicorn` 服务化，使用 `systemd` 管理。 为了更好地管理和保持你的应用程序在后台运行，特别是在服务器重启时自动启动应用程序，
你可以使用 `systemd` 来创建一个服务文件。

首先，创建一个新的服务文件：

```bash
sudo nano /etc/systemd/system/liliput-api.service
```

在文件中写入以下内容：

```ini
[Unit]
Description=Liliput API FastAPI Service
After=network.target

[Service]
User=your_username  # 替换为运行服务的用户名
WorkingDirectory=/path/to/your/project  # 替换为你的项目路径
ExecStart=/path/to/your/venv/bin/uvicorn liliput_api.main:app --host 0.0.0.0 --port 8080
Restart=always

[Install]
WantedBy=multi-user.target
```

保存并退出。

#### 7.1 启用并启动服务

启用服务，使其在开机时启动：

```bash
sudo systemctl enable liliput-api.service
```

启动服务：

```bash
sudo systemctl start liliput-api.service
```

查看服务状态，确保它正在运行：

```bash
sudo systemctl status liliput-api.service
```

#### 7.2 检查和管理日志

你可以使用 `journalctl` 来查看服务的输出日志：

```bash
sudo journalctl -u liliput-api.service -f
```

这将会实时显示你的服务日志。


## endpoint验证

在使用 `nohup` 方式启动 FastAPI 应用后，你可以查看日志和测试该应用的端点是否正常工作。以下是具体步骤：

### 1. 查看日志

使用 `nohup` 启动后，默认会生成一个 `nohup.out` 文件来保存标准输出和标准错误日志。你可以使用以下命令查看日志：

```bash
tail -f nohup.out
```

这将实时显示日志内容，如果有任何错误或输出信息都会显示在终端上。你也可以使用 `cat` 或 `less` 查看完整日志文件：

```bash
cat nohup.out  # 查看完整日志
less nohup.out  # 分页查看日志，使用上下键滚动
```

### 2. 测试端点

你可以使用 `curl` 命令或者 HTTP 客户端（如 `Postman` 或 `Insomnia`）来测试 FastAPI 的端点。

#### 使用 `curl` 测试

假设你的 FastAPI 应用运行在本地的 `8080` 端口，你可以使用以下命令测试每个端点：

1. **GET `/` 路由**

```bash
curl -X GET "http://localhost:8080/"
```

2. **POST `/` 路由**

```bash
curl -X POST "http://localhost:8080/" -H "Content-Type: application/json" -d '{"r18": false, "tag": "", "num": 1, "uid": "", "keyword": ""}'
```

3. **GET `/direct` 路由**

```bash
curl -X GET "http://localhost:8080/direct"
```

4. **GET `/json` 路由**

```bash
curl -X GET "http://localhost:8080/json"
```

#### 检查返回结果

- 如果端点正常工作，你会看到相应的响应。
- 如果出现错误，`curl` 会显示错误信息，同时你可以检查 `nohup.out` 文件中的日志，寻找任何异常或错误消息。

### 3. 验证应用正常工作

你可以通过检查 `curl` 的输出或 HTTP 客户端的响应，来确定应用程序的端点是否按预期工作。

### 4. 停止服务

如果你需要停止通过 `nohup` 启动的服务，可以找到它的进程 ID (PID)，然后杀死该进程。

首先，找到进程 ID：

```bash
ps aux | grep uvicorn
```

找到 `uvicorn` 相关的进程，记下 PID，然后用以下命令停止它：

```bash
kill -9 <PID>
```

将 `<PID>` 替换为实际的进程 ID。

### 总结

- 使用 `tail -f nohup.out` 可以查看实时日志，帮助你诊断问题。
- 使用 `curl` 或 HTTP 客户端可以测试端点的功能是否正常。
- 使用 `kill` 命令停止正在运行的 FastAPI 应用。
