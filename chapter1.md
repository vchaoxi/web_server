
# 网页静态服务器-1-显示固定的页面

```
import socket
from multiprocessing import Process

def handle_client(client_sock):
    '''新的进程来处理客户端的数据收发'''
    # 1. 接收请求
    recv_data = client_sock.recv(4096)
    print(recv_data.decode())
    # 2. 构造响应数据
    response_headers = 'HTTP/1.1 200 OK\r\n'.encode()
    response_headers += '\r\n'.encode()
    response_body = 'Hello, world\r\n'.encode()
    response_body += 'Hello, vchaoxi'.encode()
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
        # 6. 由于创建的新进程中,会对client_sock进行+1，所以,主进程中需要-1,即调用一次close
        client_sock.close()


if __name__ == '__main__':
    main()
```

 ## 服务器端

![](/assets/Snip20161117_2.png)

## 客户端
![](/assets/Snip20161117_3.png)
