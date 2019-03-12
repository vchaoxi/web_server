## 网页静态服务器-3-使用类

```
import socket
from multiprocessing import Process
import re


class WSGIServer(object):

    addressFamily = socket.AF_INET
    socketType = socket.SOCK_STREAM
    requestQueueSize = 5

    def __init__(self, server_address):
        # 创建一个tcp套接字
        self.listenSocket = socket.socket(self.addressFamily, self.socketType)
        # 允许重复使用上次的套接字绑定的port
        self.listenSocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        # 绑定
        self.listenSocket.bind(server_address)
        # 开始监听，并指定队列的长度
        self.listenSocket.listen(self.requestQueueSize)

    def serveForever(self):
        '循环运行web服务器，等待客户端的链接并为客户端服务'
        while True:
            # 等待新客户端到来
            self.clientSocket, client_address = self.listenSocket.accept()

            # 方法2，多进程服务器，并发服务器于多个客户端
            newClientProcess = Process(target=self.handleRequest)
            newClientProcess.start()

            # 因为创建的新进程中，会对这个套接字+1，所以需要在主进程中减去一次，即调用一次close
            self.clientSocket.close()

    def handleRequest(self):
        '用一个新的进程，为一个客户端进行服务'
        recvData = self.clientSocket.recv(2048)
        requestHeaderLines = recvData.splitlines()
        for line in requestHeaderLines:
            print(line.decode())

        httpRequestMethodLine = requestHeaderLines[0]
        getFileName = re.match("[^/]+(/[^ ]*)", httpRequestMethodLine.decode()).group(1)
        print("file name is ===>%s" % getFileName)  # for test

        if getFileName == '/':
            getFileName = documentRoot + "/index.html"
        else:
            getFileName = documentRoot + getFileName

        print("file name is ===2>%s" % getFileName)  # for test

        try:
            f = open(getFileName, 'rb')
        except IOError:
            responseHeaderLines = b"HTTP/1.1 404 not found\r\n"
            responseHeaderLines += b"\r\n"
            responseBody = b"====sorry ,file not found===="
        else:
            responseHeaderLines = b"HTTP/1.1 200 OK\r\n"
            responseHeaderLines += b"\r\n"
            responseBody = f.read()
            f.close()
        finally:
            response = responseHeaderLines + responseBody
            self.clientSocket.send(response)
            self.clientSocket.close()


# 设定服务器的端口
serverAddr = (HOST, PORT) = '', 8888
# 设置服务器服务静态资源时的路径
documentRoot = './html'


def makeServer(serverAddr):
    server = WSGIServer(serverAddr)
    return server


def main():
    httpd = makeServer(serverAddr)
    print('web Server: Serving HTTP on port %d ...\n' % PORT)
    httpd.serveForever()


if __name__ == '__main__':
    main()
```
