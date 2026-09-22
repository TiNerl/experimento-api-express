# experimento-api-express

Este é um projeto de teste unitário que utiliza o dummyJson para simular chamadas em um ambiente de sistema de software real. O projeto foi desenvolvido utilizando Express.js e Jest com Supertest para testes automatizados.

## Comandos iniciais

1. **Criação do projeto Node.js:**
   ```bash
   npm init -y
   ```
   Este comando cria o arquivo `package.json` no repositório.

2. **Instalação do Express.js:**
   ```bash
   npm install express
   ```
   Este comando instala o framework Express.js e adiciona as dependências necessárias ao projeto.

3. **Instalação das ferramentas de teste Jest e Supertest:**
   ```bash
   npm install --save-dev jest supertest
   ```
   Este comando instala Jest e Supertest como dependências de desenvolvimento, permitindo a execução de testes via scripts no `package.json`.

## Estrutura do projeto

A estrutura do projeto deve ser organizada da seguinte maneira:
```
experimento-api-express/
├── src/
│ ├── app.js
│ └── server.js
├── tests/
│ └── products.test.js
├── package.json
└── package-lock.json
```

Para criar a estrutura, execute os seguintes comandos:
```bash
mkdir -p src tests
touch src/app.js src/server.js tests/products.test.js
```

### Implementação da aplicação Express

Dentro do arquivo `src/app.js`:

```javascript
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
      headers: {
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
```

### No arquivo `src/server.js`:

```javascript
const app = require("./app");
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`API executando na porta ${PORT}`);
});
```

## Executando a aplicação

Inicie a aplicação com:
```bash
npm start
```

Quando a aplicação iniciar, ela detectará a porta 3000 e também será possível abrir a aplicação manualmente: Ports > 3000 > Open in browser.

Teste no navegador ou com outra ferramenta HTTP:

- **Lista de produtos:** `http://localhost:3000/api/products`
- **Limitar a quantidade de produtos:** `http://localhost:3000/api/products?limit=3`
- **Obter um produto específico:** `http://localhost:3000/api/products/1`

## Testes automatizados

Crie o arquivo `tests/products.test.js`:

```javascript
const request = require("supertest");
const app = require("../src/app");

describe("API de produtos", () => {
  describe("GET /api/products", () => {
    test("deve retornar uma lista de produtos", async () => {
      const response = await request(app)
        .get("/api/products?limit=3")
        .expect("Content-Type", /json/)
        .expect(200);
      expect(response.body).toHaveProperty("products");
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
});
```

### Execução dos testes

Execute os testes com:
```bash
npm test
```

A saída esperada será algo semelhante a:

```
PASS tests/products.test.js
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

Test Suites: 1 passed, 1 total
Tests:       6 passed, 6 total
Snapshots:   0 total
Time:        2.34s
```

### Execução contínua dos testes

Para executar os testes continuamente enquanto o código sofre alterações:
```bash
npm run test:watch
```

### Geração de relatório de cobertura

Para gerar um relatório de cobertura:
```bash
npm run test:coverage
```

## Conclusão e discussão: São testes unitários?

Os testes apresentados usam requisições HTTP contra rotas Express e, portanto, são principalmente testes de integração da API. Eles verificam conjuntamente:

- Roteamento;
- Middleware JSON;
- Códigos de status HTTP;
- Formato das respostas;
- Validação dos dados;
- Comunicação com o DummyJSON.

Um teste unitário normalmente verifica uma função isolada, sem depender de uma API externa. Assim, o experimento pode ser dividido em:

| Tipo de teste | O que verifica | Ferramenta |
|---------------|---------------|-----------|
| Unitário      | Uma função isolada | Jest |
| Integração    | Rotas e middleware da API | Jest + Supertest |
| Externo       | Comunicação real com DummyJSON | Jest + Supertest + internet |

## Atividade prática de continuidade

Após executar a versão inicial, implemente as seguintes melhorias:

1. **Criar uma rota para pesquisar produtos:**
   ```bash
   GET /api/products/search?q=phone
   ```

2. **Adicionar validação para impedir preços negativos.**

3. **Criar uma rota para listar apenas produtos de uma categoria.**

4. **Adicionar testes para:**
   - Preço negativo;
   - Categoria inexistente;
   - Parâmetro limit inválido;
   - Erro de comunicação com a API externa.

5. **Criar um middleware que registre:**
   - Método HTTP;
   - URL;
   - Horário da requisição.

6. **Criar um relatório de cobertura e buscar pelo menos:**
   - 80% de cobertura de linhas;
   - 80% de cobertura de funções;
   - 80% de cobertura de branches.

### Desafio adicional: mock da API externa

Os testes atuais dependem da internet. Para tornar os testes mais rápidos e previsíveis, altere o código para receber uma função de consulta externa como dependência. Depois, substitua essa função por um mock nos testes.

Essa alteração permite testar a aplicação mesmo quando o DummyJSON estiver indisponível e aproxima o experimento de uma arquitetura testável em produção.

## Dicas

Visite o repositório [BibliotecaDev](https://github.com/KAYOKG/BibliotecaDev/tree/main/LivrosDev) para mais informações.
