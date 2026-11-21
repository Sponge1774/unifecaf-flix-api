# UniFECAF Flix API

API REST de consulta ao acervo de filmes da plataforma de streaming UniFECAF Flix, desenvolvida em **Node.js (Express)** com **MySQL**, no padrão **MVC**.

Projeto da disciplina *Web Programming for Back End* (UniFECAF).

## Sumário
- [Funcionalidades](#funcionalidades)
- [Tecnologias](#tecnologias)
- [Arquitetura](#arquitetura)
- [Como executar](#como-executar)
- [Variáveis de ambiente](#variáveis-de-ambiente)
- [Endpoints](#endpoints)
- [Testes](#testes)
- [Estrutura de pastas](#estrutura-de-pastas)
- [Decisões técnicas](#decisões-técnicas)
- [Autor](#autor)
- [Licença](#licença)

## Funcionalidades
- Listagem de filmes com paginação por id.
- Busca de um filme pelo id.
- Filtro por nome, parte do nome ou sinopse, sem diferenciar maiúsculas e acentos.
- Validação das entradas e status HTTP adequados (200, 400, 404, 500).
- Verificação de saúde da API e do banco (`/health`).
- Documentação interativa OpenAPI/Swagger (`/docs`).
- Testes no Postman e testes automatizados, executados também por integração contínua.

## Tecnologias
| Tecnologia | Uso |
|---|---|
| Node.js 18+ | Ambiente de execução |
| Express 5 | Servidor HTTP e rotas |
| MySQL 8 | Banco de dados relacional |
| mysql2 | Driver do banco, com pool de conexões |
| dotenv | Variáveis de ambiente |
| cors | Acesso entre origens |
| swagger-ui-express e yaml | Documentação OpenAPI em `/docs` |
| Postman | Testes manuais dos endpoints |
| GitHub Actions | Integração contínua |

## Arquitetura
A API segue o padrão MVC. A "view" é a resposta JSON.

```
Cliente → Routes → Controller → Model → MySQL
```

| Camada | Pasta | Responsabilidade |
|---|---|---|
| Routes | `src/routes/` | Liga cada verbo e caminho ao método do Controller |
| Controller | `src/controllers/` | Valida a entrada, chama o Model e escolhe o status HTTP |
| Model | `src/models/` | Único ponto de acesso ao banco, com consultas parametrizadas |

## Como executar

### Pré-requisitos
- Node.js 18 ou superior e npm
- MySQL 8.0.16 ou superior

### Passo a passo
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
5. Conferir:
   ```bash
   curl http://localhost:3000/health
   ```
   A resposta esperada é `{"status":"ok","banco":"ok"}`. A documentação interativa fica em `http://localhost:3000/docs`.

## Variáveis de ambiente
Definidas no arquivo `.env` (modelo em `.env.example`). O `.env` não é versionado.

| Variável | Descrição | Padrão |
|---|---|---|
| `PORT` | Porta da API | `3000` |
| `DB_HOST` | Endereço do MySQL | `127.0.0.1` |
| `DB_PORT` | Porta do MySQL | `3306` |
| `DB_USER` | Usuário do banco | (obrigatória) |
| `DB_PASSWORD` | Senha do usuário | (obrigatória) |
| `DB_NAME` | Nome do banco | `unifecaf_flix` |

## Endpoints

Todos os endpoints de filmes usam o prefixo `/v1/controle-filmes`.

| Método | Rota | Parâmetro | Sucesso | Erros |
|---|---|---|---|---|
| GET | `/v1/controle-filmes/filme` | `limite` (1 a 100, padrão 50) e `apos_id` (opcionais) | 200 | 400, 500 |
| GET | `/v1/controle-filmes/filme/:id` | `id` (inteiro positivo) | 200 | 400, 404, 500 |
| GET | `/v1/controle-filmes/filtro/filme?nome=xxx` | `nome` (2 a 100 caracteres) | 200 | 400, 404, 500 |

Endpoints auxiliares, fora do prefixo:

| Método | Rota | Descrição |
|---|---|---|
| GET | `/health` | 200 se a API e o banco respondem, 503 se o banco não responde |
| GET | `/docs` | Documentação interativa (Swagger UI), gerada de `docs/openapi.yaml` |

### Exemplos

Listagem paginada. Quando `proximo_apos_id` não é `null`, envie-o em `apos_id` para obter a próxima página:

```bash
curl "http://localhost:3000/v1/controle-filmes/filme?limite=2"
```
```json
{
  "total": 2,
  "proximo_apos_id": 2,
  "filmes": [
    { "id": 1, "nome": "O Poderoso Chefão", "...": "..." },
    { "id": 2, "nome": "Forrest Gump", "...": "..." }
  ]
}
```

Busca pelo id:

```bash
curl http://localhost:3000/v1/controle-filmes/filme/3
```
```json
{
  "filme": {
    "id": 3,
    "nome": "Matrix",
    "sinopse": "Um programador descobre que a realidade é uma simulação e se junta à resistência contra as máquinas.",
    "duracao": "02:16:00",
    "data_lancamento": "1999-03-31",
    "valor_unitario": 16.9
  }
}
```

Filtro por nome, parte do nome ou sinopse (aqui, sem acento e em minúsculas):

```bash
curl "http://localhost:3000/v1/controle-filmes/filtro/filme?nome=chefao"
```

### Status HTTP e erros
As respostas de erro seguem o formato `{ "erro": "mensagem" }`.

| Status | Quando | Exemplo |
|---|---|---|
| 200 | Consulta bem-sucedida | `GET /filme/3` |
| 400 | Parâmetro ausente ou inválido | `GET /filme/abc` → `O id deve ser um número inteiro positivo.` |
| 404 | Filme inexistente, filtro sem resultados ou rota desconhecida | `GET /filme/999` → `Filme com id 999 não encontrado.` |
| 500 | Falha inesperada, como erro no banco | `Erro interno do servidor.` (sem detalhes internos) |

## Testes
- **Postman:** importe `postman/UniFECAF-Flix.postman_collection.json` (10 cenários, com verificações automáticas). A API precisa estar rodando. Os prints da execução ficam em `prints/` (nomes em `prints/LEIAME.md`).
- **Automatizados (Node.js):** `npm test` executa `tests/api.test.js`, com um teste por endpoint. Usa apenas recursos nativos do Node.js, inicia a API em uma porta própria (3999) e exige o MySQL ligado, o script carregado e o `.env` configurado.
- **Integração contínua:** a cada push ou pull request, o workflow `.github/workflows/ci.yml` sobe um MySQL 8, carrega `database/script.sql` e executa `npm test`.

## Estrutura de pastas
```
.github/workflows/ci.yml            integração contínua (GitHub Actions)
database/script.sql                 criação do banco, da tabela e dos dados
docs/openapi.yaml                   documentação OpenAPI 3.0 (exibida em /docs)
postman/                            coleção de testes do Postman
prints/                             capturas de tela dos testes
src/
  config/db.js                      conexão com o MySQL (pool)
  models/                           consultas ao banco
  controllers/                      validação de entrada e status HTTP
  routes/                           rotas da API e do /health
  app.js                            inicialização do Express
tests/api.test.js                   testes automatizados (npm test)
.env.example                        modelo das variáveis de ambiente
```

## Decisões técnicas
- **Busca:** índice FULLTEXT (parser ngram) em `nome` e `sinopse`, que encontra trechos no meio das palavras, sem diferenciar maiúsculas e acentos. As *stopwords* ficam desligadas para que termos curtos, como "de", sejam encontrados.
- **Paginação:** por id (`WHERE id > ? ORDER BY id LIMIT ?`), que se mantém rápida em catálogos grandes. O limite máximo é de 100 filmes por requisição.
- **Segurança:** consultas parametrizadas contra SQL injection, validação de todas as entradas, credenciais em variáveis de ambiente, usuário do banco com acesso apenas ao banco da API e erros internos sem detalhes técnicos na resposta.
- **Escalabilidade:** API sem estado, *pool* de conexões (até 10) e versionamento na rota (`/v1`).

## Autor
Eduardo Souza Mattos — Análise e Desenvolvimento de Sistemas, UniFECAF.

## Licença
Distribuído sob a licença MIT.
