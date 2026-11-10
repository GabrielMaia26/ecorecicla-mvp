ecorecicla-mvp/
├── public/
│   ├── index.html       # Telas da aplicação (Login, Pesagem, Resultados)
│   ├── style.css        # Estilos com design de alto contraste e touch targets de 44x44px
│   └── app.js           # Lógica de navegação e simulação offline/online
├── src/
│   └── server.js        # API backend com Express e rotas de pesagem
├── tests/
│   └── api.test.js      # Testes automatizados unitários/integração
├── package.json         # Dependências do projeto
├── .gitignore           # Arquivos ignorados pelo Git (ex: node_modules)
└── README.md            # Documentação técnica do projeto

2. Código do Servidor Backend (src/server.js)
JavaScript
const express = require('express');
const app = express();
app.use(express.json());

// Banco de dados em memória para o MVP
let pesagens = [];

// Rota de Autenticação simulada
app.post('/api/login', (req, res) => {
    const { email, senha } = req.body;
    if (email === 'operador@acappr.org.br' && senha === 'password123') {
        return res.status(200).json({ success: true, message: 'Login bem-sucedido' });
    }
    return res.status(401).json({ success: false, message: 'Credenciais inválidas' });
});

// Rota para Registrar Pesagem (Funcionalidade Central)
app.post('/api/pesagem', (req, res) => {
    const { material, peso } = req.body;
    if (!material || !peso || peso <= 0) {
        return res.status(400).json({ success: false, message: 'Dados de pesagem inválidos' });
    }
    
    const valorCalculado = parseFloat(peso) * 2.50; // Ex: R$ 2,50 por kg
    const novaPesagem = {
        id: pesagens.length + 1,
        material,
        peso: parseFloat(peso),
        valor: valorCalculado,
        status: 'Salvo em Nuvem',
        data: new Date().toISOString()
    };
    
    pesagens.push(novaPesagem);
    return res.status(201).json({ success: true, data: novaPesagem });
});

// Rota para Listar Resultados
app.get('/api/pesagens', (req, res) => {
    return res.status(200).json({ success: true, data: pesagens });
});

// Exporta para testes e inicialização
const PORT = process.env.PORT || 3000;
if (process.env.NODE_ENV !== 'test') {
    app.listen(PORT, () => console.log(`Servidor EcoRecicla rodando na porta ${PORT}`));
}

module.exports = app;

3. Testes Automatizados (tests/api.test.js)
(Utilizando Supertest e Jest para validação de endpoints)

JavaScript
const request = require('supertest');
const app = require('../src/server');

describe('Testes da API EcoRecicla MVP', () => {
    it('Deve realizar login com sucesso com credenciais válidas', async () => {
        const res = await request(app)
            .post('/api/login')
            .send({ email: 'operador@acappr.org.br', senha: 'password123' });
        expect(res.statusCode).toEqual(200);
        expect(res.body.success).toBe(true);
    });

    it('Deve rejeitar login com credenciais inválidas', async () => {
        const res = await request(app)
            .post('/api/login')
            .send({ email: 'errado@email.com', senha: 'wrong' });
        expect(res.statusCode).toEqual(401);
    });

    it('Deve registrar uma nova pesagem corretamente', async () => {
        const res = await request(app)
            .post('/api/pesagem')
            .send({ material: 'Plástico PET', peso: 15.4 });
        expect(res.statusCode).toEqual(201);
        expect(res.body.data.valor).toEqual(38.5);
        expect(res.body.data.status).toEqual('Salvo em Nuvem');
    });
});

4. Arquivo de Configuração do Node (package.json)
JSON
{
  "name": "ecorecicla-mvp",
  "version": "1.0.0",
  "description": "MVP do EcoRecicla para gestão de pesagem de materiais recicláveis - ACAPPR",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "test": "jest --detectOpenHandles"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {   
    "jest": "^29.5.0",
    "supertest": "^6.3.3"
  }
}
