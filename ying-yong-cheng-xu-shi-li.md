# 应用程序示例


```
import time

def app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-Type', 'text/plain')]
    start_response(status, response_headers)
    return [str(environ)+'==Hello world from a simple WSGI application!--->%s\n' % time.ctime()]
```
