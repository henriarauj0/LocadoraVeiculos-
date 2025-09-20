-- Banco e esquema
CREATE DATABASE IF NOT EXISTS LocadoraVeiculos;
USE LocadoraVeiculos;

-- Cliente
CREATE TABLE IF NOT EXISTS cliente (
  idCliente INT AUTO_INCREMENT PRIMARY KEY,
  cpf CHAR(11) NOT NULL,
  nome VARCHAR(120) NOT NULL,
  telefone VARCHAR(20) NOT NULL,
  email VARCHAR(120) NOT NULL,
  endereco VARCHAR(255) NOT NULL,
  CONSTRAINT uq_cliente_cpf UNIQUE (cpf),
  CONSTRAINT uq_cliente_email UNIQUE (email)
) ENGINE=InnoDB;

-- Veículo
CREATE TABLE IF NOT EXISTS veiculo (
  idVeiculo INT AUTO_INCREMENT PRIMARY KEY,
  modelo VARCHAR(80) NOT NULL,
  marca VARCHAR(60) NOT NULL,
  ano SMALLINT NOT NULL,
  placa CHAR(7) NOT NULL,
  valorDiaria DECIMAL(10,2) NOT NULL,
  estado VARCHAR(20) NOT NULL,
  CONSTRAINT uq_veiculo_placa UNIQUE (placa)
) ENGINE=InnoDB;

-- Pagamento
CREATE TABLE IF NOT EXISTS pagamento (
  idPagamento INT AUTO_INCREMENT PRIMARY KEY,
  forma ENUM('cartao','pix','dinheiro') NOT NULL,
  dataPagamento DATE NOT NULL,
  valorTotal DECIMAL(10,2) NOT NULL,
  estado ENUM('pago','pendente') NOT NULL
) ENGINE=InnoDB;

-- Locação
CREATE TABLE IF NOT EXISTS locacao (
  idLocacao INT AUTO_INCREMENT PRIMARY KEY,
  idCliente INT NOT NULL,
  idPagamento INT NOT NULL UNIQUE,
  dataInicio DATE NOT NULL,
  dataFim DATE NOT NULL,
  CONSTRAINT fk_locacao_cliente
    FOREIGN KEY (idCliente) REFERENCES cliente(idCliente)
    ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_locacao_pagamento
    FOREIGN KEY (idPagamento) REFERENCES pagamento(idPagamento)
    ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB;

-- Locação x Veículo (associativa)
CREATE TABLE IF NOT EXISTS locacaoVeiculo (
  idLocacao INT NOT NULL,
  idVeiculo INT NOT NULL,
  PRIMARY KEY (idLocacao, idVeiculo),
  CONSTRAINT fk_lv_locacao
    FOREIGN KEY (idLocacao) REFERENCES locacao(idLocacao)
    ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_lv_veiculo
    FOREIGN KEY (idVeiculo) REFERENCES veiculo(idVeiculo)
    ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB;

-- Manutenção
CREATE TABLE IF NOT EXISTS manutencao (
  idManutencao INT AUTO_INCREMENT PRIMARY KEY,
  idVeiculo INT NOT NULL,
  descricao VARCHAR(255) NOT NULL,
  dataManutencao DATE NOT NULL,
  custo DECIMAL(10,2) NOT NULL,
  CONSTRAINT fk_manutencao_veiculo
    FOREIGN KEY (idVeiculo) REFERENCES veiculo(idVeiculo)
    ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB;

-- Consultas do trabalho

-- 1) Manutenções: descrição, data e custo
SELECT descricao, dataManutencao, custo
FROM manutencao;

-- 2) Valor total arrecadado (somente pagos)
SELECT COALESCE(SUM(valorTotal), 0) AS total_arrecadado
FROM pagamento
WHERE estado = 'pago';

-- 3) Veículos mais locados: modelo, marca e quantidade
SELECT v.modelo, v.marca, COUNT(lv.idLocacao) AS vezes_locado
FROM locacaoVeiculo lv
JOIN veiculo v ON v.idVeiculo = lv.idVeiculo
GROUP BY v.modelo, v.marca
ORDER BY vezes_locado DESC;

-- 4) Clientes com pagamento pendente e valor devido
SELECT c.nome, c.endereco, SUM(p.valorTotal) AS valor_devido
FROM cliente c
JOIN locacao l ON l.idCliente = c.idCliente
JOIN pagamento p ON p.idPagamento = l.idPagamento
WHERE p.estado = 'pendente'
GROUP BY c.nome, c.endereco
ORDER BY c.nome ASC;
