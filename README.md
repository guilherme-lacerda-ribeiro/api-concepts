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

### CORS - OPTIONS
Cross-origin resource sharing. O método OPTIONS é utilizado pelo navegador exatamente para obter informações sobre acesso a recursos a partir de origens diferentes.

[![API Bloqueada](https://img.youtube.com/vi/Fha6Il-5RYE/0.jpg)](https://www.youtube.com/watch?v=Fha6Il-5RYE).

## Ferramentas
- Postman
- https://jsonplaceholder.typicode.com/
- https://my-json-server.typicode.com/
- https://mockapi.io/