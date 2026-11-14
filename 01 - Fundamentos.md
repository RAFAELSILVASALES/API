
## Criando um server com socket


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


Usando apenas funcionalizades embutidas no Python de como podemos renderizar um template HTML.

No terminal Ipython

```py
dados = {"nome": "Bruno", "cidade": "Viana"}
template = """\
<html>
<body>
    <ul>
        <li>Nome: {dados[nome]}</li>
        <li>Cidade: {dados[cidade]}</li>
    </ul>
</body>
</html>
"""
print(template.format(dados=dados))
```

```
<html>
<body>
    <ul>
        <li>Nome: Bruno</li>
        <li>Cidade: Viana</li>
    </ul>
</body>
</html>
```



`list.template.html` 
```html
<html>
<body>
    <h1>Blog Posts:</h1>
    <ul>
        {post_list}
    </ul>
</body>
</html>
```

`post.template.html` 
<html>
<body>
    <h1>{post[title]}</h1>
    <hr />
    <p>
        {post[content]}
    </p>
    <hr />
    <small>{post[author]}</small>
</body>
</html>
```



`database.py`
```py
# 1 - conectamos ao banco de dados
from sqlite3 import connect
conn = connect("blog.db")
cursor = conn.cursor()

# 2 - Criamos a tabela caso não exista
conn.execute(
    """\
    CREATE TABLE if not exists post (
        id integer PRIMARY KEY AUTOINCREMENT,
        title varchar UNIQUE NOT NULL,
        content varchar NOT NULL,
        author varchar NOT NULL
    );
    """
)

# 3 - Criamos os posts iniciais para alimentar o banco de dados
posts = [
    {
        "title": "Python é eleita a linguagem mais popular",
        "content": """\
        A linguem Python foi eleita a linguagem mais popular pela revista
        tech masters e segue dominando o mundo.
        """,
        "author": "Satoshi Namamoto",
    },
    {
        "title": "Como criar um blog utilizando Python",
        "content": """\
        Neste tutorial você aprenderá como criar um blog utilizando Python.
        <pre> import make_a_blog </pre>
        """,
        "author": "Guido Van Rossum",
    },
]


# 4 - Inserimos os posts caso o banco de dados esteja vazio
count = cursor.execute("SELECT * FROM post;").fetchall()
if not count:
    cursor.executemany(
        """\
        INSERT INTO post (title, content, author)
        VALUES (:title, :content, :author);
        """,
        posts,
    )
    conn.commit()

# 5 - Verificamos que foi realmente inserido
posts = cursor.execute("SELECT * FROM post;").fetchall()
assert len(posts) >= 2
```

Agora vamos finalmente fazer a renderização das postagens que estão no banco de dados e preencher os templates HTML.

- Buscamos os posts no banco de dados
- Criamos a pasta `site` que será o destino do site final
- Renderizamos a página `index.html`
- Renderizamos uma página para cada post
- Fechamos a conexão

`render.py`
```py
from pathlib import Path
from database import conn

# 1 - Obtemos os posts do banco de dados e deserializamos em um dict
cursor = conn.cursor()
fields = ("id", "title", "content", "author")
results = cursor.execute(f"SELECT * FROM post;")
posts = [dict(zip(fields, post)) for post in results]

# 2 - Criamos a pasta de destino do site
site_dir = Path("site")
site_dir.mkdir(exist_ok=True)

# 3 - Criamos uma função capaz de gerar a url de um post
def get_post_url(post):
    slug = post["title"].lower().replace(" ", "-")
    return f"{slug}.html"


# 3 - Renderizamos o a página `index.html` a partir do template.
index_template = Path("list.template.html").read_text()
index_page = site_dir / Path("index.html")
post_list = [
    f"<li><a href='{get_post_url(post)}'>{post['title']}</a></li>"
    for post in posts
]
index_page.write_text(
    index_template.format(post_list="\n".join(post_list))
)

# 4 - Renderizamos todas as páginas para cada post  partir do template.
for post in posts:
    post_template = Path("post.template.html").read_text()
    post_page = site_dir / Path(get_post_url(post))
    post_page.write_text(post_template.format(post=post))

print("Site generated at", site_dir)

# 5 - fechamos a conexão
conn.close()
```



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Blog</title>
</head>
...
```


