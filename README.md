# experimento-api-express
Um teste unitário que utiliza o dummyJson para chamadas emulando um ambiente de sistema de software real

---

# Comandos iniciais
´npm init -y´ Cria o projeto Node.js (adiciona o arquivo package.json ao repositorio).
´npm install express´ Instala o api framework Expressjs (adição do node modules e um novo arquivo package-lock.json ao repositorio).
´npm install --save-dev jest supertest´ Instala as ferramentas de teste para javascript no repositorio.

Nota: O Jest pode ser instalado como dependência de desenvolvimento e executado por meio de um script
npm test. jestjs.io O Supertest permite enviar requisições HTTP diretamente para uma aplicação
Express, sem a necessidade de iniciar o servidor em uma porta real. qaskills.sh

---

# Etapas fundamentais
Abra o arquivo ´package.json´ e altere a seção de scripts:
## Antes:
´´´
{
  "name": "experimento-api-express",
  "version": "1.0.0",
  "description": "Um teste unitário que utiliza o dummyJson para chamadas emulando um ambiente de sistema de software real",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/TiNerl/experimento-api-express.git"
  },
  "keywords": [],
  "type": "commonjs",
  "bugs": {
    "url": "https://github.com/TiNerl/experimento-api-express/issues"
  },
  "homepage": "https://github.com/TiNerl/experimento-api-express#readme",
  "dependencies": {
    "express": "^5.2.1"
  },
  "devDependencies": {
    "jest": "^30.5.2",
    "supertest": "^7.3.0"
  }
}
´´´
## Depois:
´´´
{
  "name": "experimento-api-express",
  "version": "1.0.0",
  "description": "Um teste unitário que utiliza o dummyJson para chamadas emulando um ambiente de sistema de software real",
  "main": "/src/server.js",
  "scripts": {
  "start": "node src/server.js",
  "test": "jest --runInBand",
  "test:watch": "jest --watch",
  "test:coverage": "jest --coverage"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/TiNerl/experimento-api-express.git"
  },
  "keywords": [],
  "type": "commonjs",
  "bugs": {
    "url": "https://github.com/TiNerl/experimento-api-express/issues"
  },
  "homepage": "https://github.com/TiNerl/experimento-api-express#readme",
  "dependencies": {
    "express": "^5.2.1"
  },
  "devDependencies": {
    "jest": "^30.5.2",
    "supertest": "^7.3.0"
  }
}
´´´
Nota: As versões podem ser diferentes dependendo da data em que o comando for executado. O importante
é instalar os pacotes com npm e criar os scripts correspondentes.

---

# Configurar o encaminhamento da porta:

Criar a pasta ´.devcontainer´ e o arquivo `devcontainer.json`:
´´´
{
"name": "Node.js API",
"image": "mcr.microsoft.com/devcontainers/javascript-node:1-22-bookworm",
"forwardPorts": [3000],
"postCreateCommand": "npm install"
}
´´´

---

# Estrutura do projeto

Crie a seguinte estrutura:
experimento-api-express-testes/
├── .devcontainer/
│ └── devcontainer.json
├── src/
│ ├── app.js
│ └── server.js
├── tests/
│ └── products.test.js
├── package.json
└── package-lock.json

Execute: `mkdir -p src tests` e `touch src/app.js src/server.js tests/products.test.js`

Nota: A aplicação Express será exportada separadamente do arquivo que executa listen(). Essa separação
facilita os testes com Supertest, pois os testes podem importar a aplicação sem iniciar um servidor
permanente. qaskills.sh

---

# Implementação da aplicação Express
Dentro do arquivo `src/app.js`: 
´´´
const express = require("express");
const app = express();
app.use(express.json());
const DUMMY_JSON_URL = "https://dummyjson.com";
// GET /api/products
// Seleciona produtos da API externa.
app.get("/api/products", async (req, res) => {
  try {
    const limit = Number(req.query.limit) || 10;
    const response = await fetch(
      `${DUMMY_JSON_URL}/products?limit=${limit}`
    );

    if (!response.ok) {
      return res.status(502).json({
        error: "Não foi possível consultar a API externa"
      });
    }
    const data = await response.json();
return res.status(200).json(data);
} catch (error) {
return res.status(500).json({
error: "Erro interno ao consultar produtos"
});
}
});
// GET /api/products/:id
// Seleciona um produto específico.
app.get("/api/products/:id", async (req, res) => {
try {
const { id } = req.params;
const response = await fetch(`${DUMMY_JSON_URL}/products/${id}`);
if (response.status === 404) {
return res.status(404).json({
error: "Produto não encontrado"
});
}
if (!response.ok) {
return res.status(502).json({
error: "Erro na API externa"
});
}
const product = await response.json();
return res.status(200).json(product);
} catch (error) {
return res.status(500).json({
error: "Erro interno ao consultar produto"
});
}
});
// POST /api/products
// Simula o salvamento de um produto.
app.post("/api/products", async (req, res) => {
try {
const { title, price, category } = req.body;
if (!title || price === undefined) {
return res.status(400).json({
error: "Os campos title e price são obrigatórios"
});
}
const response = await fetch(`${DUMMY_JSON_URL}/products/add`, {
method: "POST",
headers: {CommentHighlight
"Content-Type": "application/json"
},
body: JSON.stringify({
title,
price,
category
})
});
if (!response.ok) {
return res.status(502).json({
error: "Não foi possível salvar o produto"
});
}
const product = await response.json();
return res.status(201).json(product);
} catch (error) {
return res.status(500).json({
error: "Erro interno ao salvar produto"
});
}
});
// DELETE /api/products/:id
// Simula a exclusão de um produto.
app.delete("/api/products/:id", async (req, res) => {
try {
const { id } = req.params;
const response = await fetch(`${DUMMY_JSON_URL}/products/${id}`, {
method: "DELETE"
});
if (response.status === 404) {
return res.status(404).json({
error: "Produto não encontrado"
});
}
if (!response.ok) {
return res.status(502).json({
error: "Não foi possível excluir o produto"
});
}
const product = await response.json();
return res.status(200).json(product);
} catch (error) {
return res.status(500).json({
error: "Erro interno ao excluir produto"
});
}
});
module.exports = app;
´´´ 
Nota: O Express utiliza métodos como app.get(), app.post() e app.delete() para associar verbos HTTP a
rotas específicas. O middleware express.json() interpreta corpos de requisição no formato JSON.

---

## No arquivo `src/serveer.js`: 
```const app = require("./app");
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
console.log(`API executando na porta ${PORT}`);
});
```
Nota: Observe que app.js contém a configuração da aplicação, enquanto server.js é responsável apenas por
iniciar o servidor.

---

# Executando a aplicação: 
Inicie a aplicação com `npm start`.

Quando a aplicação iniciar, a aplicação deverá detectarr a porta 3000, sendo assim também é possível abrir a aplicação manualmente: Ports > 3000 > Open in browser.

Teste no navegador ou com outra ferramenta HTTP: 
http://localhost:3000/api/products

Para limitar a quantidade de produtos:
http://localhost:3000/api/products?limit=3

Para obter um produto específico:
http://localhost:3000/api/products/1

Nota: O DummyJSON oferece suporte a parâmetros de paginação e limite, como limit e skip.
dummyjson.com

---
# Testes automatizados: 
Crie o arquivo `tests/products.test.js`:
```const request = require("supertest");
const app = require("../src/app");
describe("API de produtos", () => {
describe("GET /api/products", () => {
test("deve retornar uma lista de produtos", async () => {
const response = await request(app)
.get("/api/products?limit=3")
.expect("Content-Type", /json/)
.expect(200);
expect(response.body).toHaveProperty("products");CommentHighlight
expect(Array.isArray(response.body.products)).toBe(true);
expect(response.body.products.length).toBeGreaterThan(0);
});
});
describe("GET /api/products/:id", () => {
test("deve retornar um produto existente", async () => {
const response = await request(app)
.get("/api/products/1")
.expect("Content-Type", /json/)
.expect(200);
expect(response.body).toHaveProperty("id");
expect(response.body.id).toBe(1);
expect(response.body).toHaveProperty("title");
});
test("deve retornar 404 para um produto inexistente", async () => {
const response = await request(app)
.get("/api/products/999999")
.expect("Content-Type", /json/)
.expect(404);
expect(response.body).toEqual({
error: "Produto não encontrado"
});
});
});
describe("POST /api/products", () => {
test("deve simular o salvamento de um produto", async () => {
const newProduct = {
title: "Produto de teste",
price: 49.9,
category: "testes"
};
const response = await request(app)
.post("/api/products")
.send(newProduct)
.expect("Content-Type", /json/)
.expect(201);
expect(response.body).toHaveProperty("id");
expect(response.body.title).toBe(newProduct.title);
expect(response.body.price).toBe(newProduct.price);
});
test("deve rejeitar produto sem título", async () => {
const response = await request(app)
.post("/api/products")
.send({
price: 20
})
.expect("Content-Type", /json/)
.expect(400);
expect(response.body).toEqual({
error: "Os campos title e price são obrigatórios"
});
});
});
describe("DELETE /api/products/:id", () => {
test("deve simular a exclusão de um produto", async () => {
const response = await request(app)
.delete("/api/products/1")
.expect("Content-Type", /json/)
.expect(200);
expect(response.body).toHaveProperty("id");
expect(response.body.id).toBe(1);
expect(response.body.isDeleted).toBe(true);
expect(response.body).toHaveProperty("deletedOn");
});
});
});```

## Execute os testes:

npm test

### A saída esperada será algo semelhante a: 

```PASS tests/products.test.js
API de produtos
GET /api/products
✓ deve retornar uma lista de produtos
GET /api/products/:id
✓ deve retornar um produto existente
✓ deve retornar 404 para um produto inexistente
POST /api/products
✓ deve simular o salvamento de um produto
✓ deve rejeitar produto sem título
DELETE /api/products/:id
✓ deve simular a exclusão de um produto
Test Suites: 1 passed
Tests: 6 passed``` 

Nota: para executar os testes continuamente enquanto o código sofre alterações: `npm run test:coverage`.

Para gerar um relatório de cobertura: `npm run test:coverage`

---
# Conclusão e discussão: São testes unitários?

Os testes apresentados usam requisições HTTP contra rotas Express e, portanto, são principalmente
testes de integração da API. Eles verificam conjuntamente:
• Roteamento;
• Middleware JSON;
• Códigos de status HTTP;
• Formato das respostas;
• Validação dos dados;
• Comunicação com o DummyJSON.
Um teste unitário normalmente verifica uma função isolada, sem depender de uma API externa.
Assim, o experimento pode ser dividido em:
Tipo de teste O que verifica Ferramenta
Unitário Uma função isolada Jest
Integração Rotas e middleware da API Jest + Supertest
Externo Comunicação real com DummyJSON Jest + Supertest + internet
8. Atividade prática de continuidade
Após executar a versão inicial, implemente as seguintes melhorias:
1. Criar uma rota para pesquisar produtos:
GET /api/products/search?q=phone
2. Adicionar validação para impedir preços negativos.
3. Criar uma rota para listar apenas produtos de uma categoria.
4. Adicionar testes para:
• Preço negativo;
• Categoria inexistente;
• Parâmetro limit inválido;
• Erro de comunicação com a API externa.
5. Criar um middleware que registre:
• Método HTTP;
• URL;
• Horário da requisição.
6. Criar um relatório de cobertura e buscar pelo menos:
• 80% de cobertura de linhas;
• 80% de cobertura de funções;
• 80% de cobertura de branches.
Desafio adicional: mock da API externa
Os testes atuais dependem da internet. Para tornar os testes mais rápidos e previsíveis, altere o código
para receber uma função de consulta externa como dependência. Depois, substitua essa função por
um mock nos testes.
Essa alteração permite testar a aplicação mesmo quando o DummyJSON estiver indisponível e
aproxima o experimento de uma arquitetura testável em produção.



---

# Dicas:
Visite o repositório htttps://github.com/KAYOKG/BibliotecaDev/tree/main/LivrosDev