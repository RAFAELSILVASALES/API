
## Criando um server com socket

Da mesma maneira que criamos um client com socket podemos também abrir um socket do lado servidor

`server.py`
```python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("localhost", 9000))
server.listen()

try:
    while True:
        client, address = server.accept()
        data = client.recv(5000).decode()
        print(data)

        client.sendall(
            "HTTP/1.0 200 OK\r\n<html><body>Hello World</body></html>\r\n\r\n".encode()
        )
        client.shutdown(socket.SHUT_WR)
except:
    server.close()
```
```
python server.py
```


Agora em outro terminal podemos rodar nosso client para se conectar a este server:


`http_client.py`
```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("localhost", 9000))
cmd = "GET http://example.com/index.html HTTP/1.0\r\n\r\n".encode()
client.send(cmd)

while True:
    data = client.recv(512)
    if len(data) < 1:
        break
    print(data.decode(), end="")

client.close()
```
```
python http_client.py
```
