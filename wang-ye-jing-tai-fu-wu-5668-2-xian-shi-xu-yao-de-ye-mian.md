# 网页静态服务器-2显示需要的页面

```
#coding=utf-8
import socket
from multiprocessing import Process
import re


def handleClient(clientSocket):
    '用一个新的进程，为一个客户端进行服务'
    recvData = clientSocket.recv(2014)
    requestHeaderLines = recvData.splitlines()
    for line in requestHeaderLines:
        print(line)

    httpRequestMethodLine = requestHeaderLines[0]
    getFileName = re.match("[^/]+(/[^ ]*)", httpRequestMethodLine).group(1)
    print("file name is ===>%s"%getFileName) #for test

    if getFileName == '/':
        getFileName = documentRoot + "/index.html"
    else:
        getFileName = documentRoot + getFileName

    print("file name is ===2>%s"%getFileName) #for test

    try:
        f = open(getFileName)
    except IOError:
        responseHeaderLines = "HTTP/1.1 404 not found\r\n"
        responseHeaderLines += "\r\n"
        responseBody = "====sorry ,file not found===="
    else:
        responseHeaderLines = "HTTP/1.1 200 OK\r\n"
        responseHeaderLines += "\r\n"
        responseBody = f.read()
        f.close()
    finally:
        response = responseHeaderLines + responseBody
        clientSocket.send(response)
        clientSocket.close()


def main():
    '作为程序的主控制入口'

    serverSocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serverSocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    serverSocket.bind(("", 7788))
    serverSocket.listen(10)
    while True:
        clientSocket,clientAddr = serverSocket.accept()
        clientP = Process(target = handleClient, args = (clientSocket,))
        clientP.start()
        clientSocket.close()


#这里配置服务器
documentRoot = './html'

if __name__ == '__main__':
    main()
```

## 服务器端

![](/assets/Snip20161117_7.png)

## 客户端

![](/assets/Snip20161117_5.png)
