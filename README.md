# API
## Conceitos
- Resource: recurso que se trata. `example.com/api/recurso`.
- Payload: corpo da requisição (body request).

### Métodos HTTP
RFC xxxx e RFC xxxx
- GET: obter
  - `example.com/api/recurso` todos os recursos
  - `example.com/api/recurso/1` todos os dados do recurso 1
- POST: grava um novo recurso, exige payload
- PUT: atualiza um recurso se o ID existir ou grava um novo se o ID não existir, exige payload
  - comumente não grava, apenas atualiza, gerando, erroneamente, um 'not found'
- PATCH: atualiza um recurso com o ID informado, exige payload
  - se não existir, erro
- DELETE: remove o recurso
- HEAD: idêntico ao GET porém não retorna o corpo da resposta (response body), apenas os cabeçalhos
  - `curl -I`
- OPTIONS: exibe informações dos métodos HTTP permitidos e do CORS.

## Ferramentas
- Postman
- https://jsonplaceholder.typicode.com/
- https://my-json-server.typicode.com/
- https://mockapi.io/