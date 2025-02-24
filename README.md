# API
## Conceitos
- Resource: recurso que se trata. `example.com/api/recurso`.
- Payload: corpo da requisição (body request).

## Ferramentas
- Postman
- https://jsonplaceholder.typicode.com/
- https://my-json-server.typicode.com/
- https://mockapi.io/

## Request
### [Métodos HTTP](https://datatracker.ietf.org/doc/html/rfc9110#name-methods)
[RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231#page-24), [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789) e [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110).
- GET: obter
  - `example.com/api/recurso` todos os recursos
  - `example.com/api/recurso/1` todos os dados do recurso 1
- HEAD: idêntico ao GET porém não retorna o corpo da resposta (response body), apenas os cabeçalhos
  - `curl -I`
- POST: grava um novo recurso, exige payload
- PUT: atualiza um recurso se o ID existir ou grava um novo se o ID não existir, exige payload
  - comumente não grava caso não exista o ID, apenas atualiza, gerando, erroneamente, um 'not found'
  - se fosse RESTFull (completa) deveria gerar o erro
- PATCH: atualiza um recurso com o ID informado, exige payload
  - se não existir, retorna erro
  - atualização parcial (apenas um campo por exemplo), apenas o que foi inserido no payload
- DELETE: remove o recurso
- OPTIONS: exibe informações dos métodos HTTP permitidos e do CORS.
  - traz apenas os cabeçalhos
  - quais as URLs podem acessar
  - quais métodos suporta
  - o navegador fazendo uma requisição javascript primeiro ele faz um OPTIONS para conferir se o URL aceita o que foi codificado.

#### Propriedades
- Safe methods: seguros, GET, HEAD, OPTIONS. Pode ser chamado indiscriminadamente sem preocupar com estar gravando coisas no lado do servidor. Motores de busca, por exemplo o Google, utilizam safe methods porque eles são readonly.
- Idempotent methods: todas as vezes que eu chamar trará sempre o mesmo resultado. Os métodos safe são também idempotentes, uma vez que se eu fizer GET uma vez ou 10 vezes o resultdao teria que ser o mesmo (claro, se ninguém alterar nada de modo concorrente).\
PUT e DELETE também. Se removi um registro, o registro está removido, se remover novamente, o registro estará removido, não causa efeito do lado do servidor. Se atualizo um recuso inteiro, todo o recurso está atualizado, se eu mando atualizar novamente, manterá atualizado o recurso inteiro.\
O retorno em si não precisa ser idêntico (sucesso ou erro), mas o resultado esperado (no lado do servidor, dos dados) precisa ser idêntico.
- Cacheable method: pode ser armazenado para resultados mais rápidos, pode realizar cache. GET e HEAD por exemplo podem, se a resposta tiver definido explicitamente informações de quão recente é a informação ([fressness](https://datatracker.ietf.org/doc/html/rfc9111#name-freshness)) e o 'Content-Location' no cabeçalho. Informações de freshness podem ser fornecidas através de cabeçalhos como Expires ou Cache-Control: max-age.

| Método   | Seguro | Idempotente | Cacheável |
|----------|--------|-------------|------------|
| GET      | Sim    | Sim         | Sim        |
| POST     | Não    | Não         | *          |
| PUT      | Não    | Sim         | Não        |
| DELETE   | Não    | Sim         | Não        |
| PATCH    | Não    | Não         | *          |
| HEAD     | Sim    | Sim         | Sim        |
| OPTIONS  | Sim    | Sim         | Não        |

### CORS - OPTIONS
Cross-origin resource sharing. O método OPTIONS é utilizado pelo navegador exatamente para obter informações sobre acesso a recursos a partir de origens diferentes. 

[![API Bloqueada](https://img.youtube.com/vi/Fha6Il-5RYE/0.jpg)](https://www.youtube.com/watch?v=Fha6Il-5RYE).

Com o navegador vou na aba de rede e vejo duas requisições, uma OPTIONS (preflight) response e a outra efetivamente a requisição. O javascript (por exemplo) faz a validação via OPTIONS se consegue acessar ou não o recurso e enviar parâmetros, cabeçalhos, etc.

Se não quiser receber o erro (mas não ter acesso à resposta), posso incluir o "no-cors" no front-end. Não aconselhável.
```js
fetch('http://app.back:8080/api.php', { mode: "no-cors" })
  .then(res => res.json())
  .then(res => document.getElementById('api').textContent = res.text)
  .catch(error => document.getElementById('api').textContent = error.message);
```

Para corrigir, na API eu devo inserir os headers permitindo o que foi solicitado (origem, cabeçalhos e métodos).
```php
<?php

declare(strict_types=1);

header('Access-Control-Allow-Origin: http://app.front:8080');
header('Access-Control-Allow-Headers: content-type, x-teste');
header('Access-Control-Allow-Methods: PUT, PATCH, DELETE');

echo json_encode([
    'text' => 'Resposta vindo da API',
]);
```

### Query parameters (params)
`server/api/endpoint?parameter=value` - filtrar os parametros (campos do json de resposta) em razão do value informado. Pode ser por comparação (campo like %valor%) ou pode ser por exatidão (campo=valor).

`server/api/endpoint?limit=10&page=1` - paginação - reduzir o custo/tempo para as respostas. 10 itens, estou na 1a página. Ordenação padrão por ID.

`server/api/endpoint?limit=10&page=1&orderBy=titulo` - poderia ser _sortBy_ também.

`server/api/endpoint?limit=10&page=1&orderBy=titulo&order=desc` - desc|asc - crescente ou decrescente.

[Outros exemplos](https://github.com/mockapi-io/docs/wiki/Code-examples#filtering).

### [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) - Hypermedia as the Engine of Application State
Ao final de cada response já vem os links para o que poder ser feito (próxima página, link pra remover, link para os relacionamentos filhos, etc).

```json
{
    "account": {
        "account_number": 12345,
        "balance": {
            "currency": "usd",
            "value": 100.00
        },
        "links": {
            "deposits": "/accounts/12345/deposits",
            "withdrawals": "/accounts/12345/withdrawals",
            "transfers": "/accounts/12345/transfers",
            "close-requests": "/accounts/12345/close-requests"
        }
    }
}
```

Se a conta estiver negativa, haverá por exemplo apenas o link para depósito e os outros não seriam mostrados.

O [Spring Framework](https://docs.spring.io/spring-hateoas/docs/current/reference/html/#fundamentals.link-relations) (Java) por exemplo usa o HATEOAS através de [Hypertext Application Language (HAL)](https://en.wikipedia.org/wiki/Hypertext_Application_Language), cuja [RFC](https://datatracker.ietf.org/doc/draft-kelly-json-hal/) ainda não está em vigor. [Outro exemplo](https://adidas.gitbook.io/api-guidelines/rest-api-guidelines/message/hal).


## Response
Toda resposta http tem um [código (status)](https://datatracker.ietf.org/doc/html/rfc7231#page-47), mais comum é 200, ok.

404 por exemplo é um problema do lado do cliente, que tentou fazer uma requisição em um recursos que não foi encontrado. Tentar excluir (method DELETE) um recurso que não existe por exemplo.

Ao criar uma API é muito importante utilizar os status corretamente. Por exemplo, jamais devolver um status da faixa 200 indicando que falta um parametro por exemplo.

- 1xx (Informational): The request was received, continuing process
- 2xx (Successful): The request was successfully received, understood, and accepted
- 3xx (Redirection): Further action needs to be taken in order to complete the request
- 4xx (Client Error): The request contains bad syntax or cannot be fulfilled

## Cabeçalhos
Usados para negociar como serão os detalhes das transações, qual o resultado esperado pela requisição (accept), qual o conteúdo da resposta (content-type), etc. Utilize OPTIONS para saber os tipos aceitos (cors, headers, etc).

Além dos de formato, os de cache também são os mais uitilizados. (etag também pode ser usado para cache). Nem todo cliente tem a capacidade de realizar cache, mas o servidor precisa prover as informações.

## Padrões de integração
- RPC - remote procedure call era um padrão de chamar as funções remotamente. `http://server/recuperarTodosOsCursos` por exemplo.

- SOAP foi um protocolo que utilizava este padrão RPC. Era um protocolo aberto até 2009, criado pela Microsoft. Via XML. Desde 2009 não é mais utilizado em novas implementações.

- REST tem por ideia a transferência de estados (Representational State Transfer). Quando utilizo REST não estou chamando uma função mas transferindo um estado. O GET é "pegar todos os estados do recurso indicado". O PATCH é "atualize o recurso tal com este estado agora".

- RESTful - API que segue os padrões REST.

- [GraphQL](https://graphql.org/): acesso mais flexível aos dados. Preciso de todos os telefones (um recurso), de todos os alunos (outro recurso) que fizeram o curso tal (outro recurso ainda), ou seja, 3 requisições pelo menos. O graphql permite que criemos uma query que traz isso de uma só vez por exemplo, traz flexibilidade. Não segue todos os padrões HTTP (todos os status são 200, mesmo com erro, tem que analisar o corpo da requisição e da resposta). Como se fosse um SQL direto ao banco de dados. O cliente precisa saber o que quer e implementar, o graphql fornece tipo uma interface).

- [gRPC](https://grpc.io/): baixa latência. SOAP era muito verboso. grpc é um framework para implementar RPC mais moderno. Como se estivesse chamando um método mesmo. `resposta = variavel.funcao('parametro')` variavel "aponta" para o servidor remoto. Utilizado quando precisamos de baixa latência. Serviços muito críticos. Comunicação entre serviços (e não entre navegador e servidor). Por padrão tem compressão.

- [Webhook](https://docs.github.com/pt/webhooks): exponho um endpoint, URL, para que outro serviço acessa para informar que algo aconteceu. Um push no github por exemplo. O que quero ouvir de eventos, preciso pesquisar pra saber como o provedor (github por exemplo) fornece esse dado.

## Versionamento de API
Se é necessário mudar alguma definição da API, o interessante é gerar uma nova versão e incluir no URL o número da versão. `endpoint/api/v1/recurso` e `endpoint/api/v2/recurso` por exemplo.

Facilitação na criação de contratos e fornecimento de prazos para descontinuação de funcionalidades.

## Comunicação em tempo real
### Long e short polling
Atualização em tempo real. Não quero mais uma resposta estática, mas, sim dinâmica. À medida que for atualizando os resultados devem ser informados.
- short polling: em intervalos curtos (a cada segundo) pergunto ao servidor se tem atualização. Gera sobrecarga.
- long polling: faço a requisição agora e o servidor não responde de imediato, ele só responde quando tiver alteração no dado. Cliente fica esperando por segundos, minutos ou até dar timeout. Seja porque eu recebi uma resposta do servidor ou porque deu timeout eu faço a requisição novamente para continuar recebendo as atualizações em tempo real. Implementação mais complexa no lado do servidor, exceto se tiver alguma biblioteca no lado servidor que auxilie.

### SSE - Server-sent events
Forneço uma fonte de eventos no servidor e o cliente codifica o evento. Server-sent events, em teoria, são uma forma de aplicar long polling, segundo alguns autores. Com SSE, a requisição não é encerrada após o envio de algum dado, mantendo a conexão aberta e ativa, permitindo o envio de diversas mensagens.

```html
<body>
  Número de matriculados(as): <output id="sse"></output>

  <script>
    const eventSource = new EventSource('http://localhost:8123/');
    const output = document.getElementById('sse');
    eventSource.addEventListener('matricula', function (e) {
        console.log(e);

        output.innerText = e.data;
    });
  </script>
</body>
```

EventSource é a API do servidor que permite a comunicação deste tipo com o servidor.

No devtools, em Network, aparece uma aba chamada EventStream para a requisição.

Caso o cliente queira enviar alguma mensagem para o servidor durante esse tempo, uma nova requisição precisa ser feita, normalmente em outro endpoint. Para cenários onde a comunicação bi-direcional é necessária, como chat ou jogos online, há uma outra técnica que resolve esse problema, porém com uma complexidade a mais: [WebSockets](https://youtu.be/QkhbQoajdCw) (HTML 5).
