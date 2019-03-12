# 网页静态服务器-2显示需要的页面

```
import socket
from multiprocessing import Process
import re

# 这里配置服务器
documentRoot = './html'


def handle_client(client_sock):
    '''新的进程来处理客户端的数据收发'''
    # 1. 接收请求
    recv_data = client_sock.recv(4096)
    print(recv_data.decode())
    # 2. 构造响应数据
    http_request_header_lines = recv_data.splitlines()
    http_request_method_line = http_request_header_lines[0]
    request_file_name = re.match("[^/]+(/[^ ]*)", http_request_method_line.decode()).group(1)
    print("file name is ===>", request_file_name)  # for test only
    if request_file_name == '/':
        file_name = document_root + "/index.html"
    else:
        file_name = documentRoot + request_file_name
    print("file name is ===>", request_file_name)  # for test only
    try:
        f = open(file_name, 'rb')
    except IOError:
        response_headers = b'HTTP/1.1 404 not found\r\n'
        response_headers += b'\r\n'
        response_body = b'Sorry, file not found!'
    else:
        response_headers = b'HTTP/1.1 200 OK\r\n'
        response_headers += b'\r\n'
        response_body = f.read().encode()
        f.close()
    finally:
        response = response_headers + response_body
        # 3. 发送响应
        client_sock.send(response)
        # 4. 关闭 client_sock 套接字
        client_sock.close()


def main():
    '''程序的主体函数'''
    # 1. 创建套接字
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    ## 设置socket可复用
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 2. 绑定
    server_socket.bind(("", 7788))
    # 3. 监听
    server_socket.listen(10)
    while True:
        # 4. accept
        client_sock, client_addr = server_socket.accept()
        # 5. 用 client_sock 进行数据收发
        ## 创建进程
        client_p = Process(target=handle_client, args=(client_sock,))
        ## 启动进程
        client_p.start()


if __name__ == '__main__':
    main()
```

## 服务器端

![](/assets/Snip20161117_7.png)

## 客户端

![](/assets/Snip20161117_5.png)
