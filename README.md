# 🏠 Sistema de Análise de Hospedagens – PostgreSQL (Data Warehouse)

Projeto de Banco de Dados Relacional + Data Warehouse para análise de performance de hospedagens, desenvolvido em PostgreSQL,
com foco em portfólio profissional, estudo e avaliação técnica (Júnior / Estágio).

---

## 📌 Sobre o Projeto

Este projeto implementa uma solução completa de dados:

📋 Base operacional (OLTP)  
📝 Análise de avaliações  
📍 Segmentação geográfica  
💰 Cálculo de receita estimada  
📊 Consultas analíticas  
🏗️ Data Warehouse (modelo estrela)  
🔄 Processo ETL completo  

✅ Integridade e regras de negócio garantidas via banco de dados.

---

## 🧠 Regras de Negócio

1. Cada imóvel pertence a um único bairro  
2. Um imóvel pode ter várias avaliações  
3. Cada avaliação pertence a um imóvel  
4. Receita estimada
5. Alta disponibilidade = baixa demanda  
6. Baixa receita = baixo desempenho  

---

## 📂 Estrutura do Repositório

📦 datawarehouse-hospedagens  
├── 📁 sql  
│   ├── 01_criacao_banco.sql  
│   ├── 02_modelagem_oltp.sql  
│   ├── 03_carga_dados.sql  
│   ├── 04_consultas.sql  
│   ├── 05_datawarehouse.sql  
│
├── 📁 docs  
│   ├── DER.png  
│   ├── dicionario_dados.md  
│
├── README.md  
└── .gitignore  

---

## 🛠️ Tecnologias Utilizadas

- PostgreSQL  
- SQL (DDL + DML)  
- Modelagem Relacional  
- Modelagem Dimensional (Star Schema)  
- ETL  

---

---

## 🧩 Modelagem de Dados

Este projeto foi estruturado em três níveis de modelagem:

- 🔹 Modelo Conceitual  
- 🔸 Modelo Lógico  
- ⭐ Modelo Físico (implementado em SQL)  

---

### 🧠 Modelo Conceitual

Representa a visão de negócio do sistema, sem preocupação com tecnologia.

**Entidades principais:**

- 🏠 Imóvel (Listings)  
- 👤 Anfitrião (Host)  
- 📝 Avaliação (Reviews)  
- 📍 Localização (Neighbourhoods)  

**Relacionamentos:**

- Um **Anfitrião** possui vários **Imóveis** (1:N)  
- Um **Imóvel** possui várias **Avaliações** (1:N)  
- Um **Imóvel** pertence a um **Bairro** (N:1)  

📌 Este modelo descreve **como o negócio funciona**, sem detalhes técnicos.

 
![modelo conceitual ](https://github.com/user-attachments/assets/200f11b5-38a8-4e64-ba66-a2ee378afed2)


---

### 🧱 Modelo Lógico

Representa a estrutura já organizada para banco de dados relacional.

**Tabelas e relacionamentos:**

- `neighbourhoods (neighbourhood PK)`  
- `listings (id PK, neighbourhood FK)`  
- `reviews (id PK, listing_id FK)`  

**Relacionamentos:**

- neighbourhoods 1:N listings  
- listings 1:N reviews  

📌 Aqui já temos:
- Chaves primárias  
- Chaves estrangeiras  
- Estrutura relacional  

![conceito logico](https://github.com/user-attachments/assets/7186b390-a0a2-4344-850a-1712f183e481)


# 🗄️ Scripts SQL

---

## 📄 01_criacao_banco.sql

```sql
-- Cria o banco principal do projeto
CREATE DATABASE datawarehouse_hospedagens;

-- Conecta ao banco criado
\c datawarehouse_hospedagens;

📄 02_modelagem_oltp.sql
-- ================================
-- TABELA: BAIRROS
-- ================================

CREATE TABLE neighbourhoods (
    neighbourhood VARCHAR(100) PRIMARY KEY -- Identificador único do bairro
);

-- ================================
-- TABELA: IMÓVEIS (LISTINGS)
-- ================================

CREATE TABLE listings (
    id INT PRIMARY KEY, -- Identificador do imóvel

    name VARCHAR(255), -- Nome do anúncio

    host_id INT, -- ID do anfitrião
    host_name VARCHAR(255), -- Nome do anfitrião

    neighbourhood VARCHAR(100), -- Bairro do imóvel

    room_type VARCHAR(100), -- Tipo de acomodação

    price NUMERIC(10,2), -- Valor da diária

    minimum_nights INT, -- Mínimo de noites

    number_of_reviews INT, -- Total de avaliações

    last_review DATE, -- Data da última avaliação

    availability_365 INT, -- Dias disponíveis no ano

    CONSTRAINT fk_neighbourhood
    FOREIGN KEY (neighbourhood)
    REFERENCES neighbourhoods(neighbourhood) -- Garante integridade com bairros
);

-- ================================
-- TABELA: REVIEWS
-- ================================

CREATE TABLE reviews (
    id SERIAL PRIMARY KEY, -- ID da avaliação

    listing_id INT, -- Imóvel avaliado

    date DATE, -- Data da avaliação

    CONSTRAINT fk_listing
    FOREIGN KEY (listing_id)
    REFERENCES listings(id) -- Relaciona review ao imóvel
    ON DELETE CASCADE -- Remove avaliações ao excluir imóvel
);


📄 03_carga_dados.sql
-- Importa dados externos para as tabelas

COPY neighbourhoods FROM 'caminho/neighbourhoods.csv' CSV HEADER; -- Carrega bairros

COPY listings FROM 'caminho/listings.csv' CSV HEADER; -- Carrega imóveis

COPY reviews FROM 'caminho/reviews.csv' CSV HEADER; -- Carrega avaliações


📄 05_datawarehouse.sql
-- ================================
-- CRIAÇÃO DO DATA WAREHOUSE
-- ================================

CREATE SCHEMA data_warehouse; -- Cria área analítica separada

-- ================================
-- DIMENSÃO DATA
-- ================================

CREATE TABLE data_warehouse.dim_data (
    id_data DATE PRIMARY KEY, -- Data completa
    ano INT, -- Ano
    mes INT, -- Mês
    dia INT, -- Dia
    dia_semana INT -- Dia da semana
);

-- ================================
-- DIMENSÃO ANFITRIÃO
-- ================================

CREATE TABLE data_warehouse.dim_anfitriao (
    host_id BIGINT PRIMARY KEY, -- ID do host
    host_name VARCHAR(255) -- Nome do host
);

-- ================================
-- DIMENSÃO LISTINGS
-- ================================

CREATE TABLE data_warehouse.dim_listings (
    listing_id BIGINT PRIMARY KEY, -- ID do imóvel
    nome VARCHAR(255), -- Nome do anúncio
    host_id BIGINT, -- Host relacionado
    tipo_quarto VARCHAR(100), -- Tipo do quarto
    preco_base NUMERIC -- Preço base
);

-- ================================
-- DIMENSÃO REVIEWS
-- ================================

CREATE TABLE data_warehouse.dim_reviews (
    review_id BIGINT PRIMARY KEY, -- ID da review
    listing_id BIGINT, -- Imóvel
    data_review DATE -- Data da avaliação
);

-- ================================
-- TABELA FATO
-- ================================

CREATE TABLE data_warehouse.fato_anuncios (
    id_fato BIGSERIAL PRIMARY KEY, -- ID da linha fato

    iddim_listings BIGINT, -- FK para listings
    preco NUMERIC, -- Preço da diária
    disponibilidade INT, -- Dias disponíveis
    receita_estimativa NUMERIC, -- Receita calculada

    iddim_reviews BIGINT, -- FK para reviews
    iddim_anfitriao BIGINT, -- FK para anfitrião
    iddim_data DATE -- FK para data
);

-- ================================
-- RELACIONAMENTOS
-- ================================

ALTER TABLE data_warehouse.fato_anuncios
ADD FOREIGN KEY (iddim_listings)
REFERENCES data_warehouse.dim_listings(listing_id);

ALTER TABLE data_warehouse.fato_anuncios
ADD FOREIGN KEY (iddim_reviews)
REFERENCES data_warehouse.dim_reviews(review_id);

ALTER TABLE data_warehouse.fato_anuncios
ADD FOREIGN KEY (iddim_anfitriao)
REFERENCES data_warehouse.dim_anfitriao(host_id);

ALTER TABLE data_warehouse.fato_anuncios
ADD FOREIGN KEY (iddim_data)
REFERENCES data_warehouse.dim_data(id_data);

-- ================================
-- ETL (CARGA DAS DIMENSÕES)
-- ================================

INSERT INTO data_warehouse.dim_listings
SELECT DISTINCT id, name, host_id, room_type, price
FROM listings; -- Carrega imóveis

INSERT INTO data_warehouse.dim_anfitriao
SELECT DISTINCT host_id, host_name
FROM listings; -- Carrega hosts

INSERT INTO data_warehouse.dim_reviews
SELECT id, listing_id, date
FROM reviews; -- Carrega reviews

INSERT INTO data_warehouse.dim_data
SELECT 
    d::DATE,
    EXTRACT(YEAR FROM d),
    EXTRACT(MONTH FROM d),
    EXTRACT(DAY FROM d),
    EXTRACT(DOW FROM d)
FROM generate_series('2010-01-01','2025-12-31','1 day') d; -- Cria calendário

-- ================================
-- CARGA DA FATO
-- ================================

INSERT INTO data_warehouse.fato_anuncios
SELECT 
    l.id,
    l.price,
    l.availability_365,
    (l.price * (365 - l.availability_365)), -- Receita estimada
    r.id,
    l.host_id,
    l.last_review
FROM listings l
LEFT JOIN reviews r ON r.listing_id = l.id;

📊 INSIGHTS DE NEGÓCIO

🔻 Hosts com pior desempenho comercial

Problema de negócio:
Anfitriões com portfólio pouco eficiente

Indicadores:

Alta disponibilidade
Baixa receita
SELECT
    da.host_id,
    da.host_name,
    COUNT(*) AS total_anuncios,
    ROUND(AVG(fa.disponibilidade), 2) AS media_disponibilidade,
    ROUND(AVG(fa.receita_estimativa), 2) AS media_receita_estimativa
FROM data_warehouse.fato_anuncios fa
INNER JOIN data_warehouse.dim_anfitriao da
    ON da.host_id = fa.iddim_anfitriao
GROUP BY da.host_id, da.host_name
HAVING COUNT(*) >= 5
ORDER BY media_receita_estimativa ASC, media_disponibilidade DESC
LIMIT 100;
```
![Anfitriões com portfólio pouco eficiente](https://github.com/user-attachments/assets/bd95fe06-6ed5-4d03-8f9f-6b67540780d1)
```
🔻 Bairros com baixa atratividade

Problema de negócio:
Regiões com baixa demanda e baixa geração de receita

Indicadores:

Alta disponibilidade
Baixa receita
Muitos anúncios
SELECT
    dl.nome_bairro,
    dl.nome_cidade,
    COUNT(*) AS total_anuncios,
    ROUND(AVG(fa.disponibilidade), 2) AS media_disponibilidade,
    ROUND(AVG(fa.receita_estimativa), 2) AS media_receita_estimativa
FROM data_warehouse.fato_anuncios fa
INNER JOIN data_warehouse.dim_listings dl
    ON dl.listing_id = fa.iddim_listings
GROUP BY dl.nome_bairro, dl.nome_cidade
HAVING COUNT(*) >= 5
ORDER BY media_disponibilidade DESC, media_receita_estimativa ASC;
```
![Regiões onde os imóveis ficam mais tempo disponíveis e geram menos receita](https://github.com/user-attachments/assets/230bf91f-11d8-4024-8271-c2640a5e816b)



_______________________________________________________________________________________________________________________________

📖 Storytelling do Projeto

Este projeto nasceu a partir de uma pergunta simples, mas extremamente relevante no contexto de negócios digitais:

> **Por que alguns imóveis performam bem enquanto outros permanecem ociosos?**

Ao analisar dados de hospedagens, ficou evidente que não bastava apenas armazenar informações — era necessário **transformar dados brutos em decisões estratégicas**.



🔍 O Problema

Plataformas de hospedagem possuem milhares de imóveis, mas nem todos geram o retorno esperado.

Alguns sinais de alerta incluem:

- Imóveis com alta disponibilidade ao longo do ano  
- Baixa geração de receita  
- Regiões com muitos anúncios, mas pouca demanda  

Esses padrões indicam **ineficiência comercial**, impactando diretamente anfitriões e a própria plataforma.



🧠 A Solução

Para resolver esse problema, foi construída uma arquitetura de dados em duas camadas:

🔹 **Camada Operacional (OLTP)**  
Responsável por armazenar dados brutos de imóveis, avaliações e localizações.

🔸 **Camada Analítica (Data Warehouse)**  
Responsável por organizar os dados em um modelo dimensional (Star Schema), permitindo análises rápidas e eficientes.

Além disso, foi implementado um processo de **ETL em SQL**, garantindo:

- Consistência dos dados  
- Cálculo de métricas de negócio  
- Integração entre diferentes entidades  


📊 Os Insights

Com essa estrutura, foi possível identificar:

✔ Hosts com baixo desempenho comercial  
✔ Bairros com sinais de baixa atratividade  
✔ Relação entre disponibilidade e receita  

Esses insights permitem ações práticas, como:

- Ajuste de preços  
- Melhorias nos anúncios  
- Estratégias de marketing regional  


🚀 O Impacto

Este projeto demonstra, na prática:

- Como estruturar um banco de dados do zero  
- Como evoluir para um Data Warehouse  
- Como transformar dados em valor de negócio  

Mais do que um exercício técnico, este projeto representa a transição de:

  “armazenar dados” → “gerar inteligência de negócio”


💡 Conclusão

Em um cenário orientado por dados, a diferença não está em quem possui mais informações, mas sim em quem consegue **interpretá-las melhor**.

Este projeto é um passo nessa direção.

---

👤 Autor
Carla Antunes

Estudante de Dados | SQL | Data Warehouse
