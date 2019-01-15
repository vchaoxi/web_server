# 网络动态服务器-1
```
#coding=utf-8
import socket
import sys
from multiprocessing import Process
import re

class WSGIServer(object):

    addressFamily = socket.AF_INET
    socketType = socket.SOCK_STREAM
    requestQueueSize = 5

    def __init__(self, serverAddress):
        #创建一个tcp套接字
        self.listenSocket = socket.socket(self.addressFamily,self.socketType)
        #允许重复使用上次的套接字绑定的port
        self.listenSocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        #绑定
        self.listenSocket.bind(serverAddress)
        #变为被动，并制定队列的长度
        self.listenSocket.listen(self.requestQueueSize)

        self.servrName = "localhost"
        self.serverPort = serverAddress[1]

    def serveForever(self):
        '循环运行web服务器，等待客户端的链接并为客户端服务'
        while True:
            #等待新客户端到来
            self.clientSocket, client_address = self.listenSocket.accept()

            #方法2，多进程服务器，并发服务器于多个客户端
            newClientProcess = Process(target = self.handleRequest)
            newClientProcess.start()

            #因为创建的新进程中，会对这个套接字+1，所以需要在主进程中减去依次，即调用一次close
            self.clientSocket.close()

    def setApp(self, application):
        '设置此WSGI服务器调用的应用程序入口函数'
        self.application = application

    def handleRequest(self):
        '用一个新的进程，为一个客户端进行服务'
        self.recvData = self.clientSocket.recv(2014)
        requestHeaderLines = self.recvData.splitlines()
        for line in requestHeaderLines:
            print(line)

        httpRequestMethodLine = requestHeaderLines[0]
        getFileName = re.match("[^/]+(/[^ ]*)", httpRequestMethodLine).group(1)
        print("file name is ===>%s"%getFileName) #for test

        if getFileName[-3:] != ".py":

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
                self.clientSocket.send(response)
                self.clientSocket.close()
        else:

            #根据接收到的请求头构造环境变量字典
            env = {}

            #调用应用的相应方法，完成动态数据的获取
            bodyContent = self.application(env, self.startResponse)

            #组织数据发送给客户端
            self.finishResponse(bodyContent)


    def startResponse(self, status, response_headers):
        serverHeaders = [
            ('Date', 'Tue, 31 Mar 2016 10:11:12 GMT'),
            ('Server', 'WSGIServer 0.2'),
        ]
        self.headers_set = [status, response_headers + serverHeaders]

    def finishResponse(self, bodyContent):
        try:
            status, response_headers = self.headers_set
            #response的第一行
            response = 'HTTP/1.1 {status}\r\n'.format(status=status)
            #response的其他头信息
            for header in response_headers:
                response += '{0}: {1}\r\n'.format(*header)
            #添加一个换行，用来和body进行分开
            response += '\r\n'
            #添加发送的数据
            for data in bodyContent:
                response += data

            self.clientSocket.send(response)
        finally:
            self.clientSocket.close()

#设定服务器的端口
serverAddr = (HOST, PORT) = '', 8888
#设置服务器静态资源的路径
documentRoot = './html'
#设置服务器动态资源的路径
pythonRoot = './wsgiPy'

def makeServer(serverAddr, application):
    server = WSGIServer(serverAddr)
    server.setApp(application)
    return server

def main():

    if len(sys.argv) < 2:
        sys.exit('请按照要求，指定模块名称:应用名称,例如 module:callable')

    #获取module:callable
    appPath = sys.argv[1]
    #根据冒号切割为module和callable
    module, application = appPath.split(':')
    #添加路径套sys.path
    sys.path.insert(0, pythonRoot)
    #动态导入module变量中指定的模块
    module = __import__(module)
    #获取module变量中指定的模块的，application变量指定的属性
    application = getattr(module, application)
    httpd = makeServer(serverAddr, application)
    print('WSGIServer: Serving HTTP on port %d ...\n'%PORT)
    httpd.serveForever()

if __name__ == '__main__':
    main()
```