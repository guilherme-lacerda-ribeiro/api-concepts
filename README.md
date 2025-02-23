# API
## Conceitos
- Resource: recurso que se trata. `example.com/api/recurso`.
- Payload: corpo da requisição (body request).

### Métodos HTTP
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

## Ferramentas
- Postman
- https://jsonplaceholder.typicode.com/
- https://my-json-server.typicode.com/
- https://mockapi.io/