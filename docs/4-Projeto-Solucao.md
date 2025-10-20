## 4. Projeto da Solução

<span style="color:red">Pré-requisitos: [Modelagem do Processo de Negocio](03-Modelagem%20do%20Processo%20de%20Negocio.md)</span>

## 4.1. Arquitetura da solução

A solução será desenvolvida seguindo uma arquitetura de microsserviços ou uma API REST monolítica, hospedada em nuvem (AWS, Azure ou GCP) para garantir a escalabilidade exigida pelo modelo SaaS. O front-end será desacoplado do back-end, consumindo os dados via API.

O back-end será responsável por toda a lógica de negócio, gerenciamento de concessionárias (multi-tenant), cadastro de veículos, clientes e processamento de propostas e vendas. O front-end será responsável pela interface do cliente (catálogo público) e pela interface administrativa (dashboard da concessionária), garantindo um design responsivo.

**Diagrama de Arquitetura:**

### 4.2. Protótipos de telas

Os protótipos de tela foram desenvolvidos na ferramenta Figma, focando em atender aos requisitos funcionais (como cadastro de veículos, visualização de detalhes, filtros) e não funcionais (como design responsivo e linguagem acessível). As telas seguem as jornadas definidas para as personas, como a consulta de histórico (João Silva) e a simulação de financiamento (Maria Oliveira e Pedro Souza).

Abaixo estão os links para o protótipo navegável (fluxo de telas) e para o design system (visão geral das telas).

* **CanutoMotors - Telas (Visão Geral):** [https://www.figma.com/design/ElgdeX2X3XEjZDIbl1wLoj/GesCar?node-id=40-1498&p=f&t=p8iFUPr8NXsFtJ7v-0](https://www.figma.com/design/ElgdeX2X3XEjZDIbl1wLoj/GesCar?node-id=40-1498&p=f&t=p8iFUPr8NXsFtJ7v-0)
* **CanutoMotors - Fluxo de Telas (Protótipo Interativo):** [https://www.figma.com/design/vfNuHZDU0Iu8mXS8uv40p1/GesCar---Fluxo-de-telas?node-id=40-1498&p=f&t=H4MYa5lO8XOohh55-0](https://www.figma.com/design/vfNuHZDU0Iu8mXS8uv40p1/GesCar---Fluxo-de-telas?node-id=40-1498&p=f&t=H4MYa5lO8XOohh55-0)

## Diagrama de Classes

O diagrama de classes abaixo ilustra a estrutura do software, detalhando as classes necessárias (como `Veiculo`, `Cliente`, `Venda`, `Funcionario`), seus atributos, métodos e os relacionamentos entre elas, servindo de base para a implementação do back-end.

![Diagrama de classes UML](images/CanutoMotorsUML.png)

## Modelo ER

O Modelo ER representa através de um diagrama como as entidades (coisas, objetos) se relacionam entre si na aplicação interativa.

### 4.3. Modelo de dados

O modelo de dados foi desenvolvido para suportar todos os processos da aplicação, garantindo a integridade e o armazenamento de concessionárias, veículos, clientes e usuários, conforme definido nos requisitos.

#### 4.3.1 Modelo ER

O Modelo Entidade-Relacionamento (Conceitual) abaixo demonstra as principais entidades do sistema e como elas se relacionam em alto nível.

![Modelo Entidade-Relacional (conceitual)](images/CanutoMotors_ModeloConceitual.jpeg)

#### 4.3.2 Esquema Relacional

O Esquema Relacional (Modelo Lógico) abaixo detalha o Modelo ER, traduzindo as entidades em tabelas, definindo os atributos com seus tipos, e especificando as chaves primárias (PK) e estrangeiras (FK) que implementam os relacionamentos.

![Modelo Lógico (Relacional)](images/CanutoMotors_ModeloLogico.png)

---

#### 4.3.3 Modelo Físico

Abaixo está o script SQL para criação da estrutura do banco de dados (Modelo Físico), baseado no esquema relacional.

```sql
-- Criação das tabelas principais (sem chaves estrangeiras)

CREATE TABLE CLIENTE (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cpf CHAR(11) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    telefone VARCHAR(20)
);

CREATE TABLE FUNCIONARIO (
    id_funcionario INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cpf CHAR(11) NOT NULL UNIQUE,
    cargo VARCHAR(50)
);

CREATE TABLE VEICULO (
    id_veiculo INT PRIMARY KEY AUTO_INCREMENT,
    placa CHAR(7) NOT NULL UNIQUE,
    marca VARCHAR(50) NOT NULL,
    modelo VARCHAR(50) NOT NULL,
    ano_fab INT,
    preco_compra DECIMAL(10, 2),
    preco_venda DECIMAL(10, 2),
    status VARCHAR(30) NOT NULL,
    historico TEXT
);

-- Criação das tabelas dependentes (com chaves estrangeiras)

CREATE TABLE ENDERECO (
    id_endereco INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    rua VARCHAR(100),
    numero VARCHAR(10),
    bairro VARCHAR(50),
    cidade VARCHAR(50),
    cep CHAR(8),
    FOREIGN KEY (id_cliente) REFERENCES CLIENTE(id_cliente)
);

CREATE TABLE VENDA (
    id_venda INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    id_funcionario INT NOT NULL,
    id_veiculo INT NOT NULL,
    data_venda DATE NOT NULL,
    valor_total DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES CLIENTE(id_cliente),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario),
    FOREIGN KEY (id_veiculo) REFERENCES VEICULO(id_veiculo)
);

CREATE TABLE PAGAMENTO (
    id_pagamento INT PRIMARY KEY AUTO_INCREMENT,
    id_venda INT NOT NULL,
    link_pagamento VARCHAR(255),
    status VARCHAR(30) NOT NULL,
    metodo_pagamento VARCHAR(50),
    valor_pagamento DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (id_venda) REFERENCES VENDA(id_venda)
);

CREATE TABLE PROPOSTA (
    id_proposta INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    id_funcionario INT NOT NULL,
    id_veiculo INT NOT NULL,
    data_proposta DATE NOT NULL,
    valor_ofertado DECIMAL(10, 2) NOT NULL,
    status VARCHAR(30) NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES CLIENTE(id_cliente),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario),
    FOREIGN KEY (id_veiculo) REFERENCES VEICULO(id_veiculo)
);

CREATE TABLE SIMULACAO_FINANC (
    id_simulacao INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT,
    id_veiculo INT NOT NULL,
    id_funcionario INT,
    data_simulacao DATE NOT NULL,
    valor_entrada DECIMAL(10, 2),
    numero_parcelas INT,
    taxa_juros DECIMAL(5, 2),
    valor_parcela DECIMAL(10, 2),
    FOREIGN KEY (id_cliente) REFERENCES CLIENTE(id_cliente),
    FOREIGN KEY (id_veiculo) REFERENCES VEICULO(id_veiculo),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario)
);

CREATE TABLE REPARO (
    id_reparo INT PRIMARY KEY AUTO_INCREMENT,
    id_veiculo INT NOT NULL,
    id_funcionario INT,
    data_reparo DATE NOT NULL,
    descricao TEXT,
    custo DECIMAL(10, 2),
    tipo VARCHAR(50),
    FOREIGN KEY (id_veiculo) REFERENCES VEICULO(id_veiculo),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario)
);
```
## 4.4. Tecnologias

A solução será implementada utilizing as seguintes tecnologias, conforme definido nas restrições do projeto:

* **Back-end:** Uma API REST será construída, sendo as opções de tecnologia Node.js, Django ou ASP.NET.
* **Front-end:** A interface será desenvolvida em React com o framework Next.js, visando responsividade e performance.
* **Banco de Dados:** Será utilizado um SGBD relacional, PostgreSQL ou MySQL, para garantir a persistência e integridade dos dados.
* **Infraestrutura:** A solução será hospedada em nuvem (AWS, Azure ou GCP) para escalabilidade.


| **Dimensão**   | **Tecnologia**             |
| ---            | ---                        |
| SGBD           | MySQL ou PostgreSQL        |
| Front end      | React com Next.js          |
| Back end       | Node.js, Django ou ASP.NET |
| Deploy         | AWS, Azure ou GCP          |
