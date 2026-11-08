# Trabalho Final – Docker Compose

Projeto desenvolvido para a disciplina de Arquitetura de Serviços em Nuvem, ministrada pelo professor Andrews Egas.

**Integrantes:**
- João Del Manto Netto – RA 235126
- Rafael Alvarenga – RA 219262

## Apresentação do projeto

Neste trabalho, desenvolvemos uma aplicação composta por dois serviços executados simultaneamente por meio do Docker Compose.

A solução foi dividida em duas partes:

- **Site:** página HTML estática hospedada em um servidor Nginx, utilizando uma imagem Docker personalizada e disponibilizada na porta 8080.
- **API:** aplicação criada em Go, responsável por processar as requisições e retornar informações no formato JSON, acessível pela porta 8081.

A comunicação entre os componentes acontece por meio de uma requisição `fetch` realizada pelo JavaScript da página. Dessa maneira, o conteúdo retornado pela API é apresentado diretamente na interface do site.

## Desenvolvimento da API em Go

Optamos pela linguagem Go para implementar a API, aproveitando a oportunidade para desenvolver uma aplicação prática e conhecer melhor seus recursos.

Como o objetivo era construir um serviço simples, não houve necessidade de utilizar frameworks externos. A própria biblioteca padrão da linguagem, por meio do pacote `net/http`, disponibiliza os recursos necessários para receber requisições HTTP e retornar respostas.

A implementação foi concentrada no arquivo `api/main.go`. Para executar a aplicação no container, o Dockerfile realiza a compilação do código com `go build`, gerando um executável que é utilizado para iniciar o serviço.

## Organização dos arquivos

O projeto foi organizado da seguinte maneira:

```text
tp-final-docker/
├── api/
│   ├── Dockerfile
│   └── main.go
├── docs/
├── Dockerfile
├── docker-compose.yml
└── index.html
```

A pasta `api` concentra os arquivos responsáveis pelo funcionamento do serviço desenvolvido em Go. Já os arquivos localizados na raiz são utilizados para configurar e disponibilizar a página web e coordenar a execução dos containers.

## Executando a aplicação

Para iniciar o projeto, é necessário possuir o Docker instalado e configurado no computador.

Na pasta principal do repositório, basta executar:

```bash
docker compose up -d --build
```

Esse comando realiza a construção das imagens necessárias e inicia os serviços em segundo plano.

Após a inicialização, os componentes podem ser acessados pelos seguintes endereços:

- Página web: http://localhost:8080
- Serviço da API: http://localhost:8081

Caso seja necessário encerrar a aplicação, utilizamos:

```bash
docker compose down
```

## Etapas de implementação e validação

### 1. Verificação da estrutura

Inicialmente, conferimos a disposição dos diretórios e arquivos utilizando o comando `ls -R`.

Essa verificação permitiu identificar os arquivos responsáveis pela página HTML, pela configuração do Docker Compose e pela implementação da API.

### 2. Criação da imagem do site

Para gerar uma imagem própria para o serviço web, executamos:

```bash
docker build --no-cache -t site-tp:1.0 .
```

O processo utiliza o `nginx:alpine` como imagem de origem e adiciona o arquivo `index.html` ao diretório padrão de publicação do Nginx.

Após a conclusão, utilizamos o comando `docker images` para verificar se a imagem `site-tp:1.0` estava disponível localmente.

### 3. Inicialização dos serviços

Com as configurações finalizadas, iniciamos os dois componentes através do Docker Compose:

```bash
docker compose up -d --build
```

Durante essa operação, o Docker realiza o build das imagens, estabelece uma rede para a comunicação interna e inicializa os containers `app-api` e `site-nginx`.

Para acompanhar a situação dos serviços, utilizamos:

```bash
docker compose ps
```

O comando permite conferir se os containers estão em execução e verificar o mapeamento das portas 8080 e 8081.

### 4. Realização dos testes

Para verificar se os serviços estavam respondendo corretamente, realizamos requisições utilizando o `curl`:

```bash
curl localhost:8080 && curl localhost:8081
```

Com esse teste, foi possível validar o retorno HTML fornecido pelo Nginx e a resposta JSON produzida pela API em Go.

Também realizamos a conferência pelo navegador. Ao acessar a porta 8080, a página apresenta os dados recebidos da API. Já na porta 8081, é possível visualizar diretamente o conteúdo JSON retornado pelo serviço.

### 5. Inspeção dos containers

Outro procedimento realizado foi a consulta aos registros de execução dos serviços:

```bash
docker compose logs
```

Por meio desse comando, conseguimos acompanhar as mensagens geradas pelo Nginx e pela API, incluindo a informação de inicialização do serviço na porta 3000 dentro do container.

Eventuais mensagens relacionadas ao favicon não representam falhas na aplicação, pois o navegador tenta localizar automaticamente um ícone para a página, que não foi incluído no projeto.

Também acessamos o ambiente interno do container do site utilizando:

```bash
docker compose exec site sh
```

Dentro do container, verificamos o diretório `/usr/share/nginx/html`, responsável por armazenar os arquivos disponibilizados pelo servidor Nginx, confirmando a presença do `index.html`.

### 6. Encerramento e limpeza

Ao finalizar os testes, interrompemos os serviços e removemos os recursos criados pelo Docker Compose:

```bash
docker compose down
```

Em seguida, executamos novamente:

```bash
docker compose ps
```

A consulta permitiu confirmar que os containers e a rede associados à aplicação haviam sido removidos.

## Considerações técnicas

Durante a implementação, alguns detalhes foram importantes para garantir a comunicação entre os serviços:

- **Comunicação com a API:** o endereço utilizado no `fetch` é `localhost:8081`, em vez do nome do serviço Docker (`app`). Isso acontece porque a requisição é executada pelo navegador do usuário, fora da rede interna dos containers.
- **Configuração de CORS:** adicionamos o cabeçalho HTTP `Access-Control-Allow-Origin` na API para permitir as requisições originadas pelo site. Essa configuração foi necessária porque os dois serviços são acessados por portas diferentes, caracterizando origens distintas.

Com a conclusão dos testes, conseguimos validar a execução conjunta dos serviços, a disponibilização da página pelo Nginx e a integração entre o frontend e a API desenvolvida em Go.
