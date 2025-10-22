# CRM

## 🧩 MÓDULO 1 — CRM (Clientes)

🎯 Objetivo

Gerenciar cadastro de clientes e histórico de compras (que futuramente virá das vendas).

📁 1️⃣ Estrutura de Pastas

Dentro de backend/src/modules/ crie:

modules/

└── crm/

    ├── crm.model.js

    ├── crm.controller.js

    └── crm.routes.js

    E no frontend/src/pages/ crie:

    pages/
└── CRM/

    ├── ClientesList.jsx

    ├── ClienteForm.jsx

    └── HistoricoCompras.jsx

    🧠 2️⃣ Backend — CRUD de Clientes

📌 crm.model.js

import db from '../../database/connection.js';
import { DataTypes } from 'sequelize';

const Cliente = db.define('Cliente', {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  nome: { type: DataTypes.STRING(100), allowNull: false },
  email: { type: DataTypes.STRING(100), allowNull: false, unique: true },
  telefone: { type: DataTypes.STRING(20) },
  endereco: { type: DataTypes.STRING(200) },
});

export default Cliente;

📌 crm.controller.js

import Cliente from './crm.model.js';

// Criar cliente
export async function criarCliente(req, res) {
  try {
    const cliente = await Cliente.create(req.body);
    res.status(201).json(cliente);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
}

// Listar clientes
export async function listarClientes(req, res) {
  const clientes = await Cliente.findAll();
  res.json(clientes);
}

// Buscar cliente por ID
export async function buscarCliente(req, res) {
  const cliente = await Cliente.findByPk(req.params.id);
  if (!cliente) return res.status(404).json({ error: 'Cliente não encontrado' });
  res.json(cliente);
}

// Atualizar cliente
export async function atualizarCliente(req, res) {
  const cliente = await Cliente.findByPk(req.params.id);
  if (!cliente) return res.status(404).json({ error: 'Cliente não encontrado' });
  await cliente.update(req.body);
  res.json(cliente);
}

// Excluir cliente
export async function excluirCliente(req, res) {
  const cliente = await Cliente.findByPk(req.params.id);
  if (!cliente) return res.status(404).json({ error: 'Cliente não encontrado' });
  await cliente.destroy();
  res.json({ message: 'Cliente removido com sucesso' });
}


📌 crm.routes.js

import express from 'express';
import {
  criarCliente,
  listarClientes,
  buscarCliente,
  atualizarCliente,
  excluirCliente
} from './crm.controller.js';

const router = express.Router();

router.post('/clientes', criarCliente);
router.get('/clientes', listarClientes);
router.get('/clientes/:id', buscarCliente);
router.put('/clientes/:id', atualizarCliente);
router.delete('/clientes/:id', excluirCliente);

export default router;

Depois, importe essa rota no seu server.ts ou routes.ts:

import crmRoutes from './modules/crm/crm.routes.js';
app.use('/api', crmRoutes);

💾 3️⃣ Banco de Dados

No seu schema.sql, adicione:

CREATE TABLE clientes (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL UNIQUE,
  telefone VARCHAR(20),
  endereco VARCHAR(200)
);

🧭 4️⃣ Frontend — React
📌 ClientesList.jsx

import { useEffect, useState } from 'react';
import axios from 'axios';

export default function ClientesList() {
  const [clientes, setClientes] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:3001/api/clientes')
      .then(res => setClientes(res.data))
      .catch(err => console.error(err));
  }, []);

  return (
    <div className="p-4">
      <h2 className="text-xl font-semibold mb-2">Clientes Cadastrados</h2>
      <table className="w-full border">
        <thead>
          <tr>
            <th>Nome</th>
            <th>Email</th>
            <th>Telefone</th>
          </tr>
        </thead>
        <tbody>
          {clientes.map(cli => (
            <tr key={cli.id}>
              <td>{cli.nome}</td>
              <td>{cli.email}</td>
              <td>{cli.telefone}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

📌 ClienteForm.jsx

import { useState } from 'react';
import axios from 'axios';

export default function ClienteForm() {
  const [form, setForm] = useState({ nome: '', email: '', telefone: '', endereco: '' });

  const handleChange = e => setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = async e => {
    e.preventDefault();
    await axios.post('http://localhost:3001/api/clientes', form);
    alert('Cliente cadastrado com sucesso!');
    setForm({ nome: '', email: '', telefone: '', endereco: '' });
  };

  return (
    <form onSubmit={handleSubmit} className="p-4 bg-gray-100 rounded">
      <h2 className="text-lg font-semibold mb-3">Cadastrar Cliente</h2>
      <input name="nome" placeholder="Nome" value={form.nome} onChange={handleChange} className="block mb-2 p-1 border" />
      <input name="email" placeholder="Email" value={form.email} onChange={handleChange} className="block mb-2 p-1 border" />
      <input name="telefone" placeholder="Telefone" value={form.telefone} onChange={handleChange} className="block mb-2 p-1 border" />
      <input name="endereco" placeholder="Endereço" value={form.endereco} onChange={handleChange} className="block mb-2 p-1 border" />
      <button className="bg-blue-500 text-white px-3 py-1 rounded">Salvar</button>
    </form>
  );
}


📌 HistoricoCompras.jsx (Exemplo visual)

Mais à frente, este componente exibirá as compras vindas do módulo Vendas, mas você pode deixar a estrutura pronta:

export default function HistoricoCompras() {
  return (
    <div className="p-4">
      <h2 className="text-lg font-semibold">Histórico de Compras</h2>
      <p>Em breve aqui estarão os pedidos realizados pelo cliente.</p>
    </div>
  );
}

🔗 5️⃣ Integração Futuras

Assim que você implementar o módulo Vendas, cada venda registrada fará um:

INSERT INTO historico_compras (cliente_id, venda_id, total, data)

🧠 Resumo

✅ Backend modularizado (crm.model, crm.controller, crm.routes)

✅ Banco com tabela clientes

✅ Frontend com listagem e cadastro de clientes

✅ Preparado para integrar com Vendas e Financeiro


