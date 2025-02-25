# Manipulação HTTP

## 1 - Pacote Net/HTTP
O pacote net/http do Go é uma das bibliotecas mais poderosas e amplamente utilizadas para trabalhar com HTTP  (HyperText Transfer Protocol). Ele fornece funcionalidades para criar servidores HTTP, clientes HTTP e manipular requisições e respostas de forma eficiente.

## 2 - Principais Componentes do Pacote net/http  
#### 2.1 - Servidor HTTP  

O pacote net/http permite criar servidores HTTP que podem responder a requisições de clientes (como navegadores ou APIs).
A função principal para criar um servidor é http.ListenAndServe.
     

#### 2.2 - Cliente HTTP  

O pacote também fornece ferramentas para criar clientes HTTP que podem fazer requisições a servidores externos.
A função principal para fazer requisições é http.Get, http.Post, ou o uso de http.Client.
     

#### 2.3 - Manipulação de Rotas  

Você pode definir rotas e manipuladores (handlers) para processar diferentes URLs e métodos HTTP.
     

#### 2.4 - Requisições e Respostas  

O pacote define estruturas como http.Request e http.ResponseWriter para representar as requisições recebidas e as respostas enviadas.

## 3 - Exemplos Práticos  

#### Exemplo 3.1: Criando um Servidor HTTP Simples
```go
package main

import (
    "fmt"
    "net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    // Escreve uma resposta no cliente
    fmt.Fprintf(w, "Olá, mundo!")
}

func main() {
    // Define a rota "/hello" para o manipulador helloHandler
    http.HandleFunc("/hello", helloHandler)

    // Inicia o servidor na porta 8080
    fmt.Println("Servidor rodando em http://localhost:8080/hello")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("Erro ao iniciar o servidor:", err)
    }
}
```
Uso: http://localhost:8080/hello
Retorno:
> Olá, mundo!

#### Exemplo 3.2: Lendo Parâmetros da URL  
Você pode acessar parâmetros da URL usando r.URL.Query(). 

```go
package main

import (
    "fmt"
    "net/http"
)

func greetHandler(w http.ResponseWriter, r *http.Request) {
    // Lê o parâmetro "name" da URL
    name := r.URL.Query().Get("name")
    if name == "" {
        name = "mundo"
    }

    // Escreve uma resposta personalizada
    fmt.Fprintf(w, "Olá, %s!", name)
}

func main() {
    http.HandleFunc("/greet", greetHandler)

    fmt.Println("Servidor rodando em http://localhost:8080/greet")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("Erro ao iniciar o servidor:", err)
    }
}
```
Uso: http://localhost:8080/greet?name=João
Retorno:
> Olá, João!

Uso: http://localhost:8080/greet
Retorno:
> Olá, mundo!

#### Exemplo 3.3: Lidando com Métodos HTTP Diferentes  
Você pode verificar o método HTTP usado na requisição (GET, POST, etc.) e responder de acordo. 

```go
package main

import (
    "fmt"
    "net/http"
)

func methodHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        fmt.Fprintf(w, "Requisição GET recebida")
    case http.MethodPost:
        fmt.Fprintf(w, "Requisição POST recebida")
    default:
        http.Error(w, "Método não suportado", http.StatusMethodNotAllowed)
    }
}

func main() {
    http.HandleFunc("/method", methodHandler)

    fmt.Println("Servidor rodando em http://localhost:8080/method")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("Erro ao iniciar o servidor:", err)
    }
}
```
Uso: http://localhost:8080/method
Retorno:
> Requisição GET recebida

Uso: http://localhost:8080/method (com método POST)
Retorno:
> Requisição POST recebida

Uso: http://localhost:8080/method (com método PUT)
Retorno:
> Método não suportado

#### Exemplo 3.4: Fazendo Requisições HTTP com um Cliente  

Você pode usar o pacote net/http para fazer requisições a servidores externos. 

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {
    // Faz uma requisição GET para um servidor externo
    resp, err := http.Get("https://jsonplaceholder.typicode.com/posts/1")
    if err != nil {
        fmt.Println("Erro ao fazer a requisição:", err)
        return
    }
    defer resp.Body.Close()

    // Lê o corpo da resposta
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("Erro ao ler a resposta:", err)
        return
    }

    // Imprime o corpo da resposta
    fmt.Println(string(body))
}
```

Retorno:
```json
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
  ```

####  Exemplo 3.5: Criando um Cliente HTTP Personalizado  

Você pode configurar um cliente HTTP personalizado com timeouts e outros parâmetros. 

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
    "time"
)

func main() {
    // Cria um cliente HTTP personalizado
    client := &http.Client{
        Timeout: 10 * time.Second,
    }

    // Cria uma requisição GET
    req, err := http.NewRequest("GET", "https://jsonplaceholder.typicode.com/posts/1", nil)
    if err != nil {
        fmt.Println("Erro ao criar a requisição:", err)
        return
    }

    // Adiciona cabeçalhos personalizados
    req.Header.Add("Accept", "application/json")

    // Faz a requisição
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println("Erro ao fazer a requisição:", err)
        return
    }
    defer resp.Body.Close()

    // Lê o corpo da resposta
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("Erro ao ler a resposta:", err)
        return
    }

    // Imprime o corpo da resposta
    fmt.Println(string(body))
}
```

Retorno:
```json
{
    "userId": 1,
    "id": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```

## 4 - Considerações Importantes  

- Rotas Dinâmicas : 
    Use bibliotecas como gorilla/mux ou httprouter para lidar com rotas dinâmicas e parâmetros de URL de forma mais avançada.
        

- Segurança : 
    Sempre valide e sanitize entradas de usuários para evitar vulnerabilidades como injeção de código ou XSS.
        

- Timeouts : 
    Configure timeouts para evitar que requisições fiquem pendentes indefinidamente.
        

- Middleware : 
    Use middlewares para adicionar funcionalidades como logging, autenticação e compressão.

## 5 - Conclusão

O pacote net/http é uma ferramenta poderosa e versátil para trabalhar com HTTP em Go. Ele permite criar servidores e clientes HTTP de forma simples e eficiente, além de oferecer controle granular sobre requisições e respostas. Combinado com boas práticas de segurança e design, ele é ideal para construir APIs RESTful, microsserviços e muito mais. 

