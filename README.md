# Atividade_do_Banco_de_Dados-3

# 🚗 Atividade III - SEDUC DEST2: Banco de Dados Precision Auto

Este repositório contém o projeto completo de planejamento, modelagem, normalização e implementação do banco de dados relacional para a empresa **Precision Auto**, desenvolvido como parte da avaliação da disciplina DEST2 (SEDUC).

---

## 📋 Sumário
1. [Domínio do Banco de Dados](#1-domínio-do-banco-de-dados)
2. [Contextualização](#2-contextualização)
3. [Proposta de Melhoria](#3-proposta-de-melhoria)
4. [Modelagem Lógica do Banco de Dados](#4-modelagem-lógica-do-banco-de-dados)
5. [Aplicação das Formas Normais](#5-aplicação-das-formas-normais)
6. [Instruções SQL](#6-instruções-sql)
7. [Passo a Passo do Dump](#7-passo-a-passo-do-dump)

---

## 1. Domínio do Banco de Dados
**Empresa / Domínio:** Oficina Mecânica — *Precision Auto*.  
O sistema é focado na gestão de clientes, veículos, equipe técnica (mecânicos), orçamento de peças e ordens de serviço (OS).

---

## 2. Contextualização
A **Precision Auto** é uma oficina mecânica de médio porte especializada em manutenção preventiva e corretiva de automóveis. 

### Problemas enfrentados sem um SGBD Relacional:
* **Redundância e Inconsistência:** Cadastros repetidos de clientes e veículos em planilhas avulsas e blocos de anotações.
* **Perda de Histórico de Serviços:** Dificuldade em rastrear reparos anteriores, peças trocadas e garantias ativas de um veículo.
* **Descontrole de Estoque e Preços:** Ausência de vínculo direto entre o uso de peças em ordens de serviço e o saldo em estoque.
* **Falta de Segurança e Concorrência:** Impossibilidade de múltiplos funcionários realizarem orçamentos simultâneos sem sobreposição de dados.

---

## 3. Proposta de Melhoria
Implementação de um **SGBD Relacional (MySQL / PostgreSQL)** para proporcionar:
* **Integridade Referencial:** Enfileiramento e relacionamento estrito entre Clientes, Veículos, Mecânicos e Peças.
* **Rastreabilidade e Histórico:** Registro centralizado de todas as Ordens de Serviço (OS) com detalhes de mão de obra e peças aplicadas.
* **Eficiência Operacional:** Redução de erros de digitação, busca rápida de dados e automação no controle de estoque.

---

## 4. Modelagem Lógica do Banco de Dados

### Estrutura das Tabelas

1. **`cliente`**
   * `id_cliente` (INT, PK, Auto Increment)
   * `nome` (VARCHAR(100), NOT NULL)
   * `cpf_cnpj` (VARCHAR(18), UNIQUE, NOT NULL)
   * `telefone` (VARCHAR(15))
   * `email` (VARCHAR(100))

2. **`mecanico`**
   * `id_mecanico` (INT, PK, Auto Increment)
   * `nome` (VARCHAR(100), NOT NULL)
   * `especialidade` (VARCHAR(50))
   * `comissao_porcentagem` (DECIMAL(5,2))

3. **`veiculo`**
   * `placa` (VARCHAR(7), PK)
   * `modelo` (VARCHAR(50), NOT NULL)
   * `marca` (VARCHAR(50), NOT NULL)
   * `ano` (INT)
   * `id_cliente` (INT, FK -> cliente.id_cliente)

4. **`peca`**
   * `id_peca` (INT, PK, Auto Increment)
   * `descricao` (VARCHAR(100), NOT NULL)
   * `valor_unitario` (DECIMAL(10,2), NOT NULL)
   * `estoque_qtd` (INT, NOT NULL)

5. **`ordem_servico`**
   * `id_os` (INT, PK, Auto Increment)
   * `data_emissao` (DATETIME, NOT NULL)
   * `valor_mao_obra` (DECIMAL(10,2))
   * `status` (VARCHAR(20), NOT NULL)
   * `placa_veiculo` (VARCHAR(7), FK -> veiculo.placa)
   * `id_mecanico` (INT, FK -> mecanico.id_mecanico)

6. **`item_os`** (Tabela Intermediária - Relacionamento N:M)
   * `id_os` (INT, PK/FK -> ordem_servico.id_os)
   * `id_peca` (INT, PK/FK -> peca.id_peca)
   * `quantidade` (INT, NOT NULL)
   * `preco_aplicado` (DECIMAL(10,2), NOT NULL)

### Relacionamentos e Cardinalidades
* **`cliente` (1) ------ (N) `veiculo`**: Um cliente pode possuir um ou vários veículos. Um veículo pertence obrigatoriamente a um único cliente.
* **`mecanico` (1) ------ (N) `ordem_servico`**: Um mecânico pode ser responsável por várias ordens de serviço.
* **`veiculo` (1) ------ (N) `ordem_servico`**: Um veículo pode acumular várias ordens de serviço ao longo do tempo.
* **`ordem_servico` (N) ------ (M) `peca`**: Uma ordem de serviço pode conter diversas peças e uma mesma peça pode ser utilizada em várias ordens de serviço. Resolvido via tabela intermediária `item_os`.

---

## 5. Aplicação das Formas Normais

* **1FN (Primeira Forma Normal):** 
  * Todos os atributos possuem valores atômicos e indivisíveis. 
  * Foi criada a tabela `item_os` para eliminar o problema de ter colunas multivaloradas de peças dentro da tabela `ordem_servico`.
* **2FN (Segunda Forma Normal):** 
  * A tabela `item_os` possui uma Chave Primária Composta (`id_os`, `id_peca`). Os atributos não-chave (`quantidade` e `preco_aplicado`) possuem dependência funcional total da chave composta.
* **3FN (Terceira Forma Normal):** 
  * Eliminação de dependências transitivas. Atributos do cliente não constam na tabela `veiculo` ou `ordem_servico`, dependendo unicamente da tabela `cliente` via `id_cliente`.
* **4FN (Quarta Forma Normal):** 
  * Não existem dependências multivaloradas independentes na mesma relação (por exemplo, informações de telefones ou especialidades separadas não causam combinações cartesianas na tabela principal).

---

## 6. Instruções SQL

```sql
-- =======================================================
-- 1. CRIAÇÃO DO BANCO DE DADOS E TABELAS (DDL)
-- =======================================================
CREATE DATABASE IF NOT EXISTS oficina_precision;
USE oficina_precision;

CREATE TABLE cliente (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf_cnpj VARCHAR(18) UNIQUE NOT NULL,
    telefone VARCHAR(15),
    email VARCHAR(100)
);

CREATE TABLE mecanico (
    id_mecanico INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    especialidade VARCHAR(50),
    comissao_porcentagem DECIMAL(5,2) DEFAULT 5.00
);

CREATE TABLE veiculo (
    placa VARCHAR(7) PRIMARY KEY,
    modelo VARCHAR(50) NOT NULL,
    marca VARCHAR(50) NOT NULL,
    ano INT,
    id_cliente INT NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES cliente(id_cliente) ON DELETE CASCADE
);

CREATE TABLE peca (
    id_peca INT AUTO_INCREMENT PRIMARY KEY,
    descricao VARCHAR(100) NOT NULL,
    valor_unitario DECIMAL(10,2) NOT NULL,
    estoque_qtd INT NOT NULL
);

CREATE TABLE ordem_servico (
    id_os INT AUTO_INCREMENT PRIMARY KEY,
    data_emissao DATETIME NOT NULL,
    valor_mao_obra DECIMAL(10,2) DEFAULT 0.00,
    status VARCHAR(20) NOT NULL,
    placa_veiculo VARCHAR(7) NOT NULL,
    id_mecanico INT NOT NULL,
    FOREIGN KEY (placa_veiculo) REFERENCES veiculo(placa),
    FOREIGN KEY (id_mecanico) REFERENCES mecanico(id_mecanico)
);

CREATE TABLE item_os (
    id_os INT NOT NULL,
    id_peca INT NOT NULL,
    quantidade INT NOT NULL,
    preco_aplicado DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (id_os, id_peca),
    FOREIGN KEY (id_os) REFERENCES ordem_servico(id_os) ON DELETE CASCADE,
    FOREIGN KEY (id_peca) REFERENCES peca(id_peca)
);

-- =======================================================
-- 2. INSERÇÃO DE DADOS (DML - MÍNIMO 5 REGISTROS CADA)
-- =======================================================
INSERT INTO cliente (nome, cpf_cnpj, telefone, email) VALUES
('Carlos Silva', '111.222.333-44', '(11) 98765-4321', 'carlos@email.com'),
('Ana Maria', '222.333.444-55', '(11) 97654-3210', 'ana@email.com'),
('João Pedro', '333.444.555-66', '(11) 96543-2109', 'joao@email.com'),
('Empresa XYZ Ltda', '12.345.678/0001-90', '(11) 3333-4444', 'contato@xyz.com'),
('Mariana Costa', '555.666.777-88', '(11) 95432-1098', 'mariana@email.com');

INSERT INTO mecanico (nome, especialidade, comissao_porcentagem) VALUES
('Roberto Souza', 'Motor', 7.50),
('Marcos Lima', 'Suspensão e Freios', 6.00),
('Lucas Alves', 'Injeção Eletrônica', 8.00),
('Felipe Rocha', 'Eletricista', 5.50),
('Gabriel Nunes', 'Alinhamento/Balanceamento', 5.00);

INSERT INTO veiculo (placa, modelo, marca, ano, id_cliente) VALUES
('ABC1D23', 'Civic', 'Honda', 2020, 1),
('XYZ9K87', 'Onix', 'Chevrolet', 2022, 2),
('DEF4G56', 'Corolla', 'Toyota', 2019, 3),
('GHI7H89', 'HB20', 'Hyundai', 2021, 4),
('JKL0I12', 'Golf', 'Volkswagen', 2018, 5);

INSERT INTO peca (descricao, valor_unitario, estoque_qtd) VALUES
('Filtro de Óleo', 35.00, 50),
('Pastilha de Freio', 120.00, 30),
('Vela de Ignição', 45.00, 40),
('Amortecedor Dianteiro', 350.00, 12),
('Óleo do Motor 5W30', 40.00, 100);

INSERT INTO ordem_servico (data_emissao, valor_mao_obra, status, placa_veiculo, id_mecanico) VALUES
('2024-10-01 09:00:00', 150.00, 'Concluída', 'ABC1D23', 1),
('2024-10-05 10:30:00', 200.00, 'Concluída', 'XYZ9K87', 2),
('2024-10-10 14:00:00', 100.00, 'Em Andamento', 'DEF4G56', 3),
('2024-10-15 11:15:00', 250.00, 'Concluída', 'GHI7H89', 4),
('2024-10-20 16:45:00', 80.00, 'Cancelada', 'JKL0I12', 5);

INSERT INTO item_os (id_os, id_peca, quantidade, preco_aplicado) VALUES
(1, 1, 1, 35.00),
(1, 5, 4, 40.00),
(2, 2, 2, 120.00),
(3, 3, 4, 45.00),
(4, 4, 2, 350.00);

-- =======================================================
-- 3. CONSULTAS SQL CONTEXTUALIZADAS (DQL)
-- =======================================================

-- Consulta 1: Listar todos os clientes em ordem alfabética
SELECT * FROM cliente ORDER BY nome ASC;
-- Resposta: Retorna os 5 clientes cadastrados ordenados de A a Z.

-- Consulta 2: Buscar veículos fabricados a partir de 2020
SELECT * FROM veiculo WHERE ano >= 2020;
-- Resposta: Retorna Civic (2020), Onix (2022) e HB20 (2021).

-- Consulta 3: Buscar clientes cujo nome contém "Silva" ou "Costa"
SELECT * FROM cliente WHERE nome LIKE '%Silva%' OR nome LIKE '%Costa%';
-- Resposta: Retorna "Carlos Silva" e "Mariana Costa".

-- Consulta 4: Listar apenas ordens de serviço concluídas
SELECT * FROM ordem_servico WHERE status = 'Concluída';
-- Resposta: Retorna as OSs de ID 1, 2 e 4.

-- Consulta 5: Peças com valor unitário entre R$ 40,00 e R$ 200,00
SELECT * FROM peca WHERE valor_unitario BETWEEN 40.00 AND 200.00;
-- Resposta: Retorna Pastilha de Freio (120.00), Vela de Ignição (45.00) e Óleo do Motor (40.00).

-- Consulta 6: Nome do cliente, modelo e placa do veículo (INNER JOIN)
SELECT c.nome AS cliente, v.modelo, v.placa 
FROM cliente c 
INNER JOIN veiculo v ON c.id_cliente = v.id_cliente;
-- Resposta: Mapeamento de todos os clientes com seus veículos vinculados.

-- Consulta 7: OS, nome do mecânico e valor da mão de obra
SELECT os.id_os, m.nome AS mecanico, os.valor_mao_obra 
FROM ordem_servico os 
INNER JOIN mecanico m ON os.id_mecanico = m.id_mecanico;
-- Resposta: Exibe a lista de OS com seus mecânicos responsáveis.

-- Consulta 8: Soma total do valor arrecadado em mão de obra de OSs concluídas
SELECT SUM(valor_mao_obra) AS total_mao_obra 
FROM ordem_servico 
WHERE status = 'Concluída';
-- Resposta: 600.00 (150.00 + 200.00 + 250.00).

-- Consulta 9: Média do preço unitário das peças cadastradas
SELECT AVG(valor_unitario) AS media_preco_peca FROM peca;
-- Resposta: 118.00.

-- Consulta 10: Quantidade de veículos cadastrados por cliente (GROUP BY)
SELECT c.nome, COUNT(v.placa) AS qtd_veiculos 
FROM cliente c 
LEFT JOIN veiculo v ON c.id_cliente = v.id_cliente 
GROUP BY c.id_cliente, c.nome;
-- Resposta: Retorna o nome de cada cliente com a contagem correspondente.

-- Consulta 11: Peças utilizadas na Ordem de Serviço número 1
SELECT p.descricao, i.quantidade, i.preco_aplicado 
FROM item_os i 
INNER JOIN peca p ON i.id_peca = p.id_peca 
WHERE i.id_os = 1;
-- Resposta: Filtro de Óleo (1x, 35.00) e Óleo do Motor (4x, 40.00).

-- Consulta 12: Valor total de peças aplicadas na OS número 4
SELECT id_os, SUM(quantidade * preco_aplicado) AS total_pecas 
FROM item_os 
WHERE id_os = 4 
GROUP BY id_os;
-- Resposta: OS 4 -> 700.00 (2x Amortecedores a 350.00).

-- Consulta 13: Mecânicos com comissão superior a 6%
SELECT nome, especialidade, comissao_porcentagem 
FROM mecanico 
WHERE comissao_porcentagem > 6.00 
ORDER BY comissao_porcentagem DESC;
-- Resposta: Lucas Alves (8.00%) e Roberto Souza (7.50%).

-- Consulta 14: OSs emitidas nos primeiros 10 dias de Outubro de 2024
SELECT * FROM ordem_servico 
WHERE data_emissao BETWEEN '2024-10-01' AND '2024-10-10 23:59:59';
-- Resposta: Retorna as OSs com IDs 1, 2 e 3.

-- Consulta 15: Relatório Geral (Cliente, Veículo, Mecânico, Mão de Obra e Status)
SELECT os.id_os, c.nome AS cliente, v.modelo, m.nome AS mecanico, os.valor_mao_obra, os.status 
FROM ordem_servico os 
INNER JOIN veiculo v ON os.placa_veiculo = v.placa 
INNER JOIN cliente c ON v.id_cliente = c.id_cliente 
INNER JOIN mecanico m ON os.id_mecanico = m.id_mecanico;
-- Resposta: Tabela consolidada contendo o resumo executivo dos atendimentos.
```

---

## 7. Passo a Passo do Dump

### Método 1: Linha de Comando (Terminal / CLI)
1. Abra o Terminal ou CMD no ambiente em que o MySQL está instalado.
2. Execute o comando `mysqldump` passando as credenciais e o nome da base:
   ```bash
   mysqldump -u root -p oficina_precision > dump_oficina_precision.sql
   ```
3. Digite a senha do usuário `root` e pressione Enter. O arquivo `.sql` gerado incluirá os scripts DDL e DML completos.

### Método 2: Interface Gráfica (MySQL Workbench)
1. Conecte-se ao servidor através do **MySQL Workbench**.
2. Na barra lateral, navegue até a aba **Management** e clique em **Data Export**.
3. Selecione o banco de dados `oficina_precision`.
4. Marque a opção **Export to Self-Contained File** e escolha o caminho de destino.
5. Certifique-se de marcar a caixa **Include Create Schema**.
6. Clique no botão **Start Export**.
