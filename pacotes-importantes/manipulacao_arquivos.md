# Manipulação de arquivos

## Principais Funções para manipulação de arquivos
- [**⚓ 1 - os.Create**](#create)
    Cria ou trunca um arquivo e o abre para leitura e escrita.
    É uma abstração simplificada de os.OpenFile com as flags O_RDWR|O_CREATE|O_TRUNC.
- [**⚓ 2 - os.Open**](#open)
    Abre um arquivo apenas para leitura (O_RDONLY). Não permite escrita.
    Exemplos:
    - Ler Todo o Conteúdo de um Arquivo
    - Ler Linha por Linha Usando bufio.Scanner
    - Ler em Blocos (Chunked Reading) "Ex.: Lendo em blocos de 10 bytes"
    - Ler um arquivo de forma incremental(linha por linha)
- [**⚓ 3 - os.OpenFile**](#openfile)
    Oferece mais controle sobre o modo de abertura do arquivo 
    Abre um arquivo para escrita, criando-o se ele não existir
    Trunca ou adiciona conteúdo ao final do arquivo se ele existir.
- [**⚓ 4 - os.Remove**](#remove)
    Remove um arquivo ou diretório vazio.
- [**⚓ 5 - os.RemoveAll**](#removeall)
    Remove recursivamente um diretório e seu conteúdo.
- [**⚓ 6 - os.Rename**](#rename)
    Renomeia ou move um arquivo ou diretório.
- [**⚓ 7 - os.ReadFile**](#readfile)
    Lê todo o conteúdo de um arquivo em uma única operação.
    Exemplos:
    - Lendo o conteúdo de um arquivo
    - Lendo o conteúdo de um arquivo json

<a id="create"></a>
## 1 - os.Create

#### 1.1 - Descrição da função:

É uma das maneiras mais simples e comuns de criar ou truncar um arquivo em Go. Ela é amplamente utilizada quando você precisa escrever dados em um arquivo, seja para criar um novo arquivo ou sobrescrever o conteúdo de um arquivo existente.

A função os.Create cria um novo arquivo ou trunca (apaga o conteúdo) de um arquivo existente. Ela retorna um ponteiro para um objeto do tipo *os.File, que pode ser usado para realizar operações de leitura e escrita no arquivo. 

#### 1.2 - Assinatura:  

```func Create(name string) (*os.File, error)```

###### Parâmetros : 
- name: O caminho (relativo ou absoluto) do arquivo a ser criado ou truncado.
- Retorno : 
    - Um ponteiro para um objeto `*os.File`, que representa o arquivo aberto.
    - Um erro (`error`), caso ocorra algum problema durante a criação do arquivo.
         
#### 1.3 - Comportamento da Função  

- Se o arquivo não existir : Ele será criado com as permissões padrão 0666 (leitura e escrita para todos, ajustadas pela umask do sistema).
- Se o arquivo já existir : O conteúdo do arquivo será truncado (apagado), e o arquivo será aberto no modo de escrita.
- Modo de Abertura : O arquivo é aberto no modo de leitura e escrita (O_RDWR|O_CREATE|O_TRUNC).

#### 1.4 - Exemplo Básico

###### 1.4.1 - Criando um novo arquivo e escrevendo nele:

    Se o arquivo exemplo.txt não existir, ele será criado.
    Se o arquivo já existir, seu conteúdo será apagado antes da escrita.
     

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Criando ou truncando o arquivo
    file, err := os.Create("exemplo.txt")
    if err != nil {
        fmt.Println("Erro ao criar o arquivo:", err)
        return
    }
    defer file.Close() // Garante que o arquivo será fechado ao final

    // Escrevendo no arquivo
    content := "Olá, mundo!\n"
    n, err := file.WriteString(content)
    if err != nil {
        fmt.Println("Erro ao escrever no arquivo:", err)
        return
    }

    fmt.Printf("Escritos %d bytes no arquivo.\n", n)
    fmt.Println("Arquivo criado com sucesso!")
}
```

###### 1.4.2 - Usos Comuns de os.Create

- Criar um Novo Arquivo  

    Se você precisa criar um arquivo vazio ou inicializar um arquivo com algum conteúdo, os.Create é a escolha ideal. 
    ```go
    file, err := os.Create("novo_arquivo.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close()
    ```
- Sobrescrever um Arquivo Existente  

    Se o arquivo já existir, os.Create automaticamente trunca o conteúdo do arquivo, permitindo que você escreva novos dados.
    ```go
    file, err := os.Create("existente.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close()

    file.WriteString("Novo conteúdo\n")
    ```
 - Criar Arquivos Temporários  
    Você pode usar os.Create em combinação com funções como os.TempDir() para criar arquivos temporários.
    ```go
    tempFile, err := os.CreateTemp("", "prefixo_*.txt")
    if err != nil {
        panic(err)
    }
    defer tempFile.Close()

    fmt.Println("Arquivo temporário criado:", tempFile.Name())
    ``` 

<a id="open"></a>
## 2 - os.Open

#### 2.1 - Descrição da função:
A função os.Open é uma das funções mais básicas e amplamente utilizadas no pacote **os** do Go para abrir arquivos. Ela é usada quando você precisa ler  um arquivo existente, sem modificar seu conteúdo. 

#### 2.2 - Assinatura:
```func Open(name string) (*os.File, error)```

- Parâmetros : 
    - name: O caminho (relativo ou absoluto) do arquivo a ser aberto.
    - Retorno : 
        Um ponteiro para um objeto *os.File, que representa o arquivo aberto.
        Um erro (error), caso ocorra algum problema durante a abertura do arquivo.
- Modo de Abertura : 
    O arquivo é aberto no modo somente leitura  (O_RDONLY).
    Não é possível escrever no arquivo usando o manipulador retornado por os.Open.
    
#### 2.3 - Utilização de os.Open  
A função os.Open é ideal quando você precisa apenas ler dados de um arquivo , como ler o conteúdo completo, processar linha por linha ou realizar operações de análise. 
- Passos Comuns:  
    - Abrir o arquivo com os.Open.
    - Ler o conteúdo do arquivo usando métodos como Read, ReadString, bufio.Scanner, etc.
    - Fechar o arquivo com defer file.Close() para liberar os recursos.
     
#### 2.4 - Exemplos Práticos
###### 2.4.1 - Ler Todo o Conteúdo de um Arquivo
Uso: Se você deseja carregar todo o conteúdo de um arquivo na memória.
```go
package main

import (
    "fmt"
    "io/ioutil"
    "os"
)

func main() {
    // Abrindo o arquivo
    file, err := os.Open("arquivo.txt")
    if err != nil {
        fmt.Println("Erro ao abrir o arquivo:", err)
        return
    }
    defer file.Close()

    // Lendo todo o conteúdo do arquivo
    content, err := ioutil.ReadAll(file)
    if err != nil {
        fmt.Println("Erro ao ler o arquivo:", err)
        return
    }

    fmt.Println(string(content)) // Convertendo []byte para string
}
```

###### 2.4.2 - Ler Linha por Linha Usando bufio.Scanner
Se o arquivo for grande ou você precisar processar o conteúdo linha por linha.
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    // Abrindo o arquivo
    file, err := os.Open("arquivo.txt")
    if err != nil {
        fmt.Println("Erro ao abrir o arquivo:", err)
        return
    }
    defer file.Close()

    // Criando um scanner para ler linha por linha
    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        fmt.Println(scanner.Text()) // Imprime cada linha
    }

    // Verificando erros durante a leitura
    if err := scanner.Err(); err != nil {
        fmt.Println("Erro ao ler o arquivo:", err)
    }
}
```

###### 2.4.3 - Ler em Blocos (Chunked Reading)
Se o arquivo for muito grande e você quiser ler em blocos menores para economizar memória.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Abrindo o arquivo
    file, err := os.Open("arquivo.txt")
    if err != nil {
        fmt.Println("Erro ao abrir o arquivo:", err)
        return
    }
    defer file.Close()

    // Lendo em blocos de 10 bytes
    buffer := make([]byte, 10)
    for {
        n, err := file.Read(buffer)
        if err != nil {
            break // Fim do arquivo ou erro
        }
        fmt.Printf("Lidos %d bytes: %s\n", n, buffer[:n])
    }
}
```

###### 2.4.4 - Ler um arquivo de forma incremental(linha por linha)
Para ler um arquivo linha por linha em Go, você pode combinar os.Open com bufio.NewReader. Essa abordagem é eficiente porque lê o arquivo de forma incremental (linha por linha), sem carregar todo o conteúdo na memória. Isso é especialmente útil para arquivos grandes.

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    // Abrindo o arquivo
    file, err := os.Open("arquivo.txt")
    if err != nil {
        fmt.Println("Erro ao abrir o arquivo:", err)
        return
    }
    defer file.Close()

    // Criando um leitor com buffer
    reader := bufio.NewReader(file)

    // Lendo linha por linha
    for {
        line, err := reader.ReadString('\n') // Lê até o próximo '\n'
        if err != nil {
            break // Fim do arquivo ou erro
        }
        fmt.Print(line) // Imprime a linha (já inclui o '\n')
    }

    // Verificando erros durante a leitura
    if err != nil && err.Error() != "EOF" {
        fmt.Println("Erro ao ler o arquivo:", err)
    }
}
```

<a id="openfile"></a>
## 3 - os.OpenFile
#### 3.1 - Descrição da função:
Se você precisar de mais controle sobre o modo de abertura do arquivo (por exemplo, adicionar conteúdo ao final do arquivo sem sobrescrever), pode usar os.OpenFile. 

#### 3.2 - Exemplos descritivos de uso:
- Abre um arquivo para escrita, criando-o se ele não existir e ou truncando-o se ele existir.
```f, err := os.OpenFile(pathFileName, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, 0644)```

- Abre um arquivo para escrita, criando-o se ele não existir e adicionando conteúdo ao final do arquivo se ele existir.
```f, err := os.OpenFile(pathFileName, os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0644)```

#### 3.3 - Parâmetros referentes aos modos de abertura e permissões:
##### 3.3.1 - Modos de Abertura com os.OpenFile:  

    os.O_WRONLY: Abre o arquivo apenas para escrita.
    os.O_CREATE: Cria o arquivo, se ele não existir.
    os.O_TRUNC: Trunca o arquivo (apaga o conteúdo existente).
    os.O_APPEND: Adiciona conteúdo ao final do arquivo (sem sobrescrever).

##### 3.3.2 - Formato das Permissões  

As permissões são representadas por três dígitos octais (base 8), onde cada dígito corresponde a um conjunto específico de permissões: 

- Primeiro dígito : Permissões para o proprietário  do arquivo.
- Segundo dígito : Permissões para o grupo  associado ao arquivo.
- Terceiro dígito : Permissões para outros usuários  (todos os demais).
     
Cada dígito pode ter um valor entre 0 e 7, que é uma combinação das seguintes permissões: 
- 4 Leitura (r) Permite ler o conteúdo do arquivo.
- 2 Escrita (w) Permite modificar o conteúdo do arquivo.
- 1 Execução (x) Permite executar o arquivo (para scripts ou programas).
 
Os valores são somados para criar uma combinação de permissões. Por exemplo: 
- 6 = 4 + 2 = Leitura (r) + Escrita (w)
- 7 = 4 + 2 + 1 = Leitura (r) + Escrita (w) + Execução (x)
- 5 = 4 + 1 = Leitura (r) + Execução (x)
     

#### 3.4 - Segue exemplo de implementação usando os.OpenFile:

```go
package main

import (
    "fmt"
    "os"
)

func writeFileString(pathFileName string, content string) int {
    // Abrindo o arquivo com permissão de escrita e criação, se necessário
    f, err := os.OpenFile(pathFileName, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, 0644)
    if err != nil {
        panic(err)
    }
    defer f.Close()

    // Escrevendo uma string no arquivo e pegando o tamanho
    fileSize, err := f.WriteString(content)
    if err != nil {
        panic(err)
    }

    fmt.Println("Escrita realizada com sucesso no arquivo", f.Name())
    return fileSize
}

func main() {
    // Testando a função
    fileSize := writeFileString("arquivo.txt", "Olá, mundo!\n")
    fmt.Printf("Tamanho do arquivo: %d bytes\n", fileSize)
}
```

<a id="remove"></a>
## 4 - os.Remove
#### 4.1 - Descrição da função:
A função os.Remove remove um arquivo ou um diretório vazio do sistema de arquivos.
Se o caminho especificado não existir ou ocorrer algum erro (como tentar remover um diretório não vazio), ela retorna um erro (error).


#### 4.2 - Assinatura da função:
```go
func Remove(name string) error
```
###### 4.2.1 - Parâmetros: 
- name: O caminho (relativo ou absoluto) do arquivo ou diretório a ser removido.
- Retorno : Um erro (error), caso ocorra algum problema durante a remoção. Se a operação for bem sucedida, o retorno será nil.
#### 4.3 - Comportamentos da função:
- Arquivos : 
    Se o caminho apontar para um arquivo, ele será excluído permanentemente.
- Diretórios : 
    Se o caminho apontar para um diretório vazio , ele será removido.
    Se o diretório contiver arquivos ou subdiretórios, a função retornará um erro (syscall.ENOTEMPTY).
- Caminhos Inválidos : 
    Se o caminho não existir, a função retornará um erro (os.ErrNotExist).
- Permissões : 
    O programa deve ter permissão suficiente para remover o arquivo ou diretório. Caso contrário, um erro de permissão será retornado.
#### 4.4 - Exemplo de uso:
###### 4.4.1 - Removendo um arquivo:
Se você deseja excluir um arquivo específico.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Nome do arquivo a ser removido
    fileName := "arquivo.txt"

    // Removendo o arquivo
    err := os.Remove(fileName)
    if err != nil {
        fmt.Println("Erro ao remover o arquivo:", err)
        return
    }

    fmt.Println("Arquivo removido com sucesso!")
}
```

###### 4.4.2 - Removendo um diretório vazio:
Se você deseja excluir um diretório vazio.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Nome do diretório a ser removido
    dirName := "diretorio_vazio"

    // Removendo o diretório
    err := os.Remove(dirName)
    if err != nil {
        fmt.Println("Erro ao remover o diretório:", err)
        return
    }

    fmt.Println("Diretório removido com sucesso!")
}
```

<a id="removeall"></a>
## 5 - os.RemoveAll
#### 5.1 - Descrição da função:
A função os.RemoveAll remove um arquivo ou diretório, incluindo todos os seus conteúdos, do sistema de arquivos, ou seja, remove recursivamente todos os arquivos e diretórios dentro do diretório especificado.

#### 5.2 - Assinatura da função:
```go
func RemoveAll(path string) error
```
###### 5.2.1 - Parâmetros:
- path: O caminho (relativo ou absoluto) do arquivo ou diretório a ser removido.
- Retorno : Um erro (error), caso ocorra algum problema durante a remoção. Se a operação for bem sucedida, o retorno será nil.

#### 5.3 - Exemplo de uso:
###### 5.3.1 - Removendo um diretório com conteúdo:
Se você deseja excluir um diretório com conteúdo.
***ATENÇÃO:*** *A função os.RemoveAll remove tudo recursivamente, incluindo arquivos e subdiretórios. Use-a com cuidado para evitar exclusões acidentais.*
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Nome do diretório a ser removido
    dirName := "diretorio_nao_vazio"

    // Removendo o diretório e todo o seu conteúdo
    err := os.RemoveAll(dirName)
    if err != nil {
        fmt.Println("Erro ao remover o diretório:", err)
        return
    }

    fmt.Println("Diretório e seu conteúdo removidos com sucesso!")
}
```

<a id="rename"></a>
## 6 - os.Rename
#### 6.1 - Descrição da função:
A função os.Rename renomeia um arquivo ou diretório no sistema de arquivos. Também pode ser usada para mover um arquivo ou diretório para um novo local.
A função os.Rename é uma ferramenta poderosa para renomear ou mover arquivos e diretórios no Go. Ela é simples de usar, mas requer atenção a detalhes como permissões, conflitos de nomes e comportamentos específicos do sistema operacional.
###### Comportamento:
Se o caminho de origem (oldPath) não existir, a função retornará um erro.
Se o caminho de destino (newPath) já existir, ele será sobrescrito (dependendo das permissões do sistema).
Se os caminhos estiverem em sistemas de arquivos diferentes, o comportamento pode variar dependendo do sistema operacional (alguns sistemas podem copiar o arquivo e depois excluí-lo da origem).

###### 6.2 - Assinatura da função:
```go
func Rename(oldPath string, newPath string) error
```
###### 6.2.1 - Parâmetros:
- oldPath: O caminho (relativo ou absoluto) do arquivo ou diretório a ser renomeado.
- newPath: O novo caminho (relativo ou absoluto) para o qual o arquivo ou diretório será renomeado ou movido.
- Retorno : Um erro (error), caso ocorra algum problema durante a renomeação ou movimentação. Se a operação for bem sucedida, o retorno será nil.
###### 6.3 - Exemplo de uso:
###### 6.3.1 - Renomeando um arquivo:
Se você deseja renomear um arquivo existente.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Nome atual do arquivo
    oldName := "arquivo_antigo.txt"

    // Novo nome do arquivo
    newName := "arquivo_novo.txt"

    // Renomeando o arquivo
    err := os.Rename(oldName, newName)
    if err != nil {
        fmt.Println("Erro ao renomear o arquivo:", err)
        return
    }

    fmt.Println("Arquivo renomeado com sucesso!")
}
```
###### 6.3.2 - Movendo um arquivo para um novo diretório:
Se você deseja mover um arquivo para um novo diretório.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Caminho atual do arquivo
    oldPath := "arquivo.txt"

    // Novo caminho (movendo para outro diretório)
    newPath := "nova_pasta/arquivo.txt"

    // Movendo o arquivo
    err := os.Rename(oldPath, newPath)
    if err != nil {
        fmt.Println("Erro ao mover o arquivo:", err)
        return
    }

    fmt.Println("Arquivo movido com sucesso!")
}
```
###### 6.3.3 - Renomeando um diretório:
Se você deseja renomear um diretório existente.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Nome atual do diretório
    oldDir := "diretorio_antigo"

    // Novo nome do diretório
    newDir := "diretorio_novo"

    // Renomeando o diretório
    err := os.Rename(oldDir, newDir)
    if err != nil {
        fmt.Println("Erro ao renomear o diretório:", err)
        return
    }

    fmt.Println("Diretório renomeado com sucesso!")
}
```
###### 6.3.4 - Movendo um diretório para um outro local:
Se você deseja mover um diretório para um novo local.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Caminho atual do diretório
    oldPath := "diretorio_origem"

    // Novo caminho (movendo para outro diretório)
    newPath := "nova_pasta/diretorio_destino"

    // Movendo o diretório
    err := os.Rename(oldPath, newPath)
    if err != nil {
        fmt.Println("Erro ao mover o diretório:", err)
        return
    }

    fmt.Println("Diretório movido com sucesso!")
}
```

<a id="readfile"></a>

## 7 - os.ReadFile

#### 7.1 - Descrição da função:
A função os.ReadFile é uma das maneiras mais simples e eficientes de ler o conteúdo completo de um arquivo em Go. Ela foi introduzida no Go 1.16 como uma alternativa conveniente para substituir padrões antigos, como o uso combinado de os.Open, ioutil.ReadAll e file.Close.

#### 7.2 - O Que Faz?  
A função os.ReadFile lê todo o conteúdo de um arquivo  e retorna os dados como um slice de bytes ([]byte).
Ela é uma função de alto nível que simplifica a leitura de arquivos pequenos ou médios, pois lida automaticamente com a abertura, leitura e fechamento do arquivo.

A função os.ReadFile é uma ferramenta poderosa e conveniente para ler arquivos pequenos ou médios em Go.
No entanto, para arquivos grandes ou cenários mais complexos, outras abordagens podem ser mais adequadas.
     

#### 7.3 - Comportamento:  
Se o arquivo não existir ou ocorrer algum erro durante a leitura, a função retornará um erro (error).
O arquivo é lido inteiramente na memória, então essa função não é recomendada para arquivos muito grandes.

#### 7.4 - Assinatura da função:
```go
func ReadFile(filename string) ([]byte, error)
```
#### 7.4.1 - Parâmetros:
- filename: O caminho (relativo ou absoluto) do arquivo a ser lido.
- Retorno : Um slice de bytes ([]byte) contendo o conteúdo do arquivo e um erro (error), caso ocorra algum problema durante a leitura.
#### 7.5 - Exemplo de uso:
###### 7.5.1 - Lendo o conteúdo de um arquivo:
Se você deseja ler todo o conteúdo de um arquivo e imprimi-lo.
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Nome do arquivo a ser lido
    fileName := "arquivo.txt"

    // Lendo o conteúdo do arquivo
    content, err := os.ReadFile(fileName)
    // Verificando se o arquivo existe
    // Verificando se ocorreu algum erro
    if err != nil {
        if os.IsNotExist(err) {
            fmt.Println("O arquivo não existe.")
        } else {
            fmt.Println("Erro ao ler o arquivo:", err)
        }
        return
    }
    // Convertendo o conteúdo para string e imprimindo
    fmt.Println(string(content))
}
```

###### 7.5.2 - Lendo o conteúdo de um arquivo json
Se você deseja ler o conteúdo de um arquivo JSON e analisar o conteúdo em uma estrutura de dados.
```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

type Config struct {
    Name  string `json:"name"`
    Value int    `json:"value"`
}

func main() {
    // Nome do arquivo JSON
    fileName := "config.json"

    // Lendo o conteúdo do arquivo
    content, err := os.ReadFile(fileName)
    if err != nil {
        fmt.Println("Erro ao ler o arquivo:", err)
        return
    }

    // Decodificando o JSON
    var config Config
    err = json.Unmarshal(content, &config)
    if err != nil {
        fmt.Println("Erro ao decodificar o JSON:", err)
        return
    }

    // Imprimindo os dados processados
    fmt.Printf("Nome: %s, Valor: %d\n", config.Name, config.Value)
}
```

