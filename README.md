# API de Filmes UniFECAF Flix

API REST em Node.js (Express) com MySQL, organizada no padrão MVC.

## Requisitos
- Node.js 18 ou superior e npm
- MySQL 8.0.16 ou superior

## Como executar
1. Criar o banco e carregar os dados:
   ```bash
   sudo mysql < database/script.sql
   ```
2. Criar um usuário para a API (substitua a senha):
   ```sql
   CREATE USER 'flix'@'localhost' IDENTIFIED BY 'SuaSenhaAqui';
   GRANT ALL PRIVILEGES ON unifecaf_flix.* TO 'flix'@'localhost';
   FLUSH PRIVILEGES;
   ```
3. Configurar as variáveis de ambiente:
   ```bash
   cp .env.example .env    # edite DB_PASSWORD com a senha criada
   ```
4. Instalar as dependências e iniciar:
   ```bash
   npm install
   npm start
   ```
   A API sobe em `http://localhost:3000`.

## Endpoints

| Método | Rota | Parâmetro | Sucesso | Erros |
|---|---|---|---|---|
| GET | `/v1/controle-filmes/filme` | `limite` (1 a 100, padrão 50) e `apos_id` (opcionais) | 200 | 400, 500 |
| GET | `/v1/controle-filmes/filme/:id` | `id` (inteiro positivo) | 200 | 400, 404, 500 |
| GET | `/v1/controle-filmes/filtro/filme?nome=xxx` | `nome` (2 a 100 caracteres) | 200 | 400, 404, 500 |

Respostas de erro seguem o formato `{ "erro": "mensagem" }`.

Endpoints auxiliares (fora do prefixo `/v1/controle-filmes`):

| Método | Rota | Descrição |
|---|---|---|
| GET | `/health` | Verifica a API e o banco: 200 se respondem, 503 se o banco não responde |
| GET | `/docs` | Documentação interativa (Swagger UI), gerada de `docs/openapi.yaml` |

## Estrutura de pastas
```
src/
  config/db.js                    conexão com o MySQL (pool)
  models/filmeModel.js            consultas ao banco
  controllers/filmeController.js  validação de entrada e status HTTP
  routes/filmeRoutes.js           rotas /v1/controle-filmes
  routes/sistemaRoutes.js         rota /health
  controllers/sistemaController.js  verificação de saúde
  models/sistemaModel.js          consulta mínima ao banco
  app.js                          inicialização do Express
database/script.sql               criação do banco, tabela e dados
docs/openapi.yaml                 documentação OpenAPI 3.0 (exibida em /docs)
.github/workflows/ci.yml          integração contínua (GitHub Actions)
tests/api.test.js                 testes automatizados (npm test)
postman/                          coleção de testes do Postman
prints/                           capturas de tela dos testes (ver prints/LEIAME.md)
```

## Testes
- **Postman:** importe `postman/UniFECAF-Flix.postman_collection.json` (10 cenários, com verificações automáticas).
- **Automatizados (Node.js):** `npm test` executa `tests/api.test.js`, com um teste por endpoint. Usa apenas recursos nativos do Node.js (18 ou superior), inicia a API em uma porta própria (3999) e exige o MySQL ligado e o script carregado.
- **Integração contínua:** a cada push ou pull request no GitHub, o workflow `.github/workflows/ci.yml` sobe um MySQL 8, carrega `database/script.sql` e executa `npm test`.

## Decisões técnicas
- **MVC:** o Model acessa o banco, o Controller valida e escolhe o status, e as Routes ligam o caminho ao Controller. A "view" é a resposta JSON.
- **Busca:** índice FULLTEXT (parser ngram) em `nome` e `sinopse`, que encontra trechos no meio das palavras, sem diferenciar maiúsculas e acentos.
- **Segurança:** consultas parametrizadas, validação de todas as entradas, credenciais em variáveis de ambiente e erros internos sem detalhes técnicos na resposta.
- **Escalabilidade:** listagem paginada por id com limite máximo e pool de conexões.

## Licença
MIT.
