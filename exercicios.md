# Momento — Exercícios SQL

Resolução dos níveis 1 a 6, conforme a base anexada.

**Critérios:** resultados dos níveis 1 a 5 após o cadastro de Adrielly em 1.1; funcionários de todos os status; salários separados por moeda; vendas apenas com status `CONCLUIDA`. Sem conversão cambial. No nível 5, pedidos cancelados e o status não esclarecido `1` não são tratados como vendas concluídas.

```sql
USE momento;
```

## Nível 1

**1.1 — Você acabou de ser contratado pela Momento! Vamos começar incluindo você no sistema, escolha um departamento, cargo e salário.**

R: Cadastro fictício de Adrielly em Tecnologia da Informação, no cargo Desenvolvedor Web, com salário de USD 4.500,00. Execute uma vez em uma base recém-carregada.

```sql
INSERT INTO funcionarios
(funcionario_id, primeiro_nome, sobrenome, email, data_contratacao,
 cargo_id, salario, moeda_salario, departamento_id)
VALUES
(1000, 'Adrielly', 'Soares Ferreira de Araujo',
 'adrielly.soares@momento.org', '2026-11-08', 9, 4500.00, 'USD', 6);
```

**1.2 — Agora que você faz parte da equipe, quantos funcionários temos ao total na empresa?**

R: 124.

```sql
SELECT COUNT(*) AS total_funcionarios FROM funcionarios;
```

**1.3 — Quantos funcionários trabalham especificamente no Departamento de Tecnologia?**

R: 10.

```sql
SELECT COUNT(*) AS total_funcionarios FROM funcionarios f
JOIN departamentos d ON d.departamento_id = f.departamento_id
WHERE d.departamento_nome = 'Tecnologia da Informação';
```

**1.4 — Liste todos os departamentos que existem na empresa. Quantos são?**

R: 30 departamentos. A consulta lista os nomes.

```sql
SELECT departamento_id, departamento_nome FROM departamentos ORDER BY departamento_id;
SELECT COUNT(*) AS total_departamentos FROM departamentos;
```

**1.5 — Quantos escritórios a Momento possui? Em quais países?**

R: 20 escritórios. Países: Alemanha, Argentina, Brasil, Brasil Bruxo, Canadá, Estados Unidos, França, França Mágica, Japão, Japão Místico, Peru, Portugal Bruxo, Reino Unido, Reino dos Mágos Unidos, Wakanda, África do Sul.

```sql
SELECT COUNT(*) AS total_escritorios FROM escritorios;
SELECT DISTINCT p.pais_nome FROM escritorios e
JOIN paises p ON p.pais_id = e.pais_id ORDER BY p.pais_nome;
```

## Nível 2

**2.1 — Quantos funcionários trabalham no Departamento de Vendas?**

R: 5.

```sql
SELECT COUNT(*) AS total_funcionarios FROM funcionarios f
JOIN departamentos d ON d.departamento_id = f.departamento_id
WHERE d.departamento_nome = 'Vendas';
```

**2.2 — Qual é o custo total com salários do Departamento de Vendas?**

R: USD — 59390.76.

```sql
SELECT f.moeda_salario, ROUND(SUM(f.salario), 2) AS custo_salarial
FROM funcionarios f JOIN departamentos d ON d.departamento_id = f.departamento_id
WHERE d.departamento_nome = 'Vendas' GROUP BY f.moeda_salario;
```

**2.3 — Qual é a média salarial da empresa, excluindo os cargos de CEO, CMO e CFO?**

R: BRL — 15537.5; EUR — 5670.0; GBP — 7586.67; USD — 10027.96.

Sem taxa de câmbio, a média da empresa é apresentada por moeda.

```sql
SELECT f.moeda_salario, ROUND(AVG(f.salario), 2) AS media_salarial
FROM funcionarios f JOIN cargos c ON c.cargo_id = f.cargo_id
WHERE c.cargo_nome NOT IN ('CEO', 'CMO', 'CFO') GROUP BY f.moeda_salario;
```

**2.4 — Qual é a média salarial do Departamento de Tecnologia?**

R: USD — 7186.06.

```sql
SELECT f.moeda_salario, ROUND(AVG(f.salario), 2) AS media_salarial
FROM funcionarios f JOIN departamentos d ON d.departamento_id = f.departamento_id
WHERE d.departamento_nome = 'Tecnologia da Informação' GROUP BY f.moeda_salario;
```

**2.5 — Qual departamento possui a maior média salarial?**

R: Poções — BRL — 18000.0; Criaturas Mágicas — EUR — 7350.0; Poções — GBP — 11100.0; Biotecnologia — USD — 29263.47.

Maior média dentro de cada moeda; não há uma comparação financeira única entre moedas sem conversão.

```sql
WITH medias AS (
 SELECT d.departamento_id, d.departamento_nome, f.moeda_salario, AVG(f.salario) AS media
 FROM departamentos d JOIN funcionarios f ON f.departamento_id = d.departamento_id
 GROUP BY d.departamento_id, d.departamento_nome, f.moeda_salario
)
SELECT departamento_nome, moeda_salario, ROUND(media, 2) AS media_salarial
FROM medias m WHERE media =
 (SELECT MAX(m2.media) FROM medias m2 WHERE m2.moeda_salario = m.moeda_salario)
ORDER BY moeda_salario;
```

**2.6 — Qual departamento possui o menor número de funcionários?**

R: Administração Executiva — 1; Contabilidade — 1; Recursos Humanos — 1.

Inclui departamentos sem funcionários e todos os empates.

```sql
WITH totais AS (
 SELECT d.departamento_id, d.departamento_nome, COUNT(f.funcionario_id) AS total
 FROM departamentos d LEFT JOIN funcionarios f ON f.departamento_id = d.departamento_id
 GROUP BY d.departamento_id, d.departamento_nome
)
SELECT departamento_nome, total FROM totais
WHERE total = (SELECT MIN(total) FROM totais) ORDER BY departamento_nome;
```

## Nível 3

**3.1 — Quantos funcionários da empresa Momento possuem cônjuges?**

R: 15.

```sql
SELECT COUNT(DISTINCT funcionario_id) AS total_funcionarios
FROM dependentes WHERE relacionamento = 'Cônjuge';
```

**3.2 — Quantos funcionários possuem filhos registrados?**

R: 61.

```sql
SELECT COUNT(DISTINCT funcionario_id) AS total_funcionarios
FROM dependentes WHERE relacionamento = 'Filha(o)';
```

**3.3 — Qual funcionário foi contratado há mais tempo na empresa?**

R: Madeleine — Prior — 1987-03-23.

```sql
SELECT primeiro_nome, sobrenome, data_contratacao FROM funcionarios
WHERE data_contratacao = (SELECT MIN(data_contratacao) FROM funcionarios);
```

**3.4 — Qual funcionário foi contratado há menos tempo na empresa?**

R: Adrielly — Soares Ferreira de Araujo — 2026-11-08.

```sql
SELECT primeiro_nome, sobrenome, data_contratacao FROM funcionarios
WHERE data_contratacao = (SELECT MAX(data_contratacao) FROM funcionarios);
```

**3.5 — Liste os 5 funcionários com mais tempo de casa, ordenados pela data de contratação.**

R: Madeleine — Prior — 1987-03-23; Noah — Kim — 1987-04-11; Yuki — Tanaka — 1987-07-31; Neena — Kochhar — 1987-08-17; Lex — T.Challa — 1988-02-08.

```sql
SELECT primeiro_nome, sobrenome, data_contratacao FROM funcionarios
ORDER BY data_contratacao, funcionario_id LIMIT 5;
```

**3.6 — Quantos funcionários foram contratados na década de 1990 (entre 1990-1999)?**

R: 28.

```sql
SELECT COUNT(*) AS total_funcionarios FROM funcionarios
WHERE data_contratacao >= '1990-01-01' AND data_contratacao < '2000-01-01';
```

**3.7 — Como a média salarial da Momento evoluiu ao longo dos anos? Agrupe por ano de contratação e calcule a média salarial.**

R: 72 resultados, listados pela consulta.

A consulta mostra salários atuais agrupados pelo ano de contratação, não o histórico dos reajustes da empresa.

```sql
SELECT YEAR(data_contratacao) AS ano, moeda_salario,
 ROUND(AVG(salario), 2) AS media_salarial FROM funcionarios
GROUP BY YEAR(data_contratacao), moeda_salario ORDER BY ano, moeda_salario;
```

## Nível 4

**4.1 — Qual é o custo total de suprimentos em cada escritório? Ordene do mais caro ao mais barato.**

R: 15 resultados, listados pela consulta.

```sql
SELECT e.escritorio_nome, s.moeda,
 ROUND(SUM(s.quantidade_estoque * s.custo_unitario), 2) AS custo_total
FROM escritorios e JOIN suprimentos s ON s.escritorio_id = e.escritorio_id
GROUP BY e.escritorio_id, e.escritorio_nome, s.moeda ORDER BY custo_total DESC;
```

**4.2 — Qual escritório possui a maior quantidade de diferentes tipos de suprimentos?**

R: 15 resultados, listados pela consulta.

Conta tipos pelo nome do suprimento, não pelo identificador da compra.

```sql
WITH tipos AS (
 SELECT e.escritorio_nome, COUNT(DISTINCT s.suprimento_nome) AS total_tipos
 FROM escritorios e LEFT JOIN suprimentos s ON s.escritorio_id = e.escritorio_id
 GROUP BY e.escritorio_id, e.escritorio_nome
)
SELECT escritorio_nome, total_tipos FROM tipos
WHERE total_tipos = (SELECT MAX(total_tipos) FROM tipos);
```

**4.3 — Qual é o suprimento mais caro (considerando preço unitário) em toda a empresa?**

R: Computadores — 909.5 — USD.

Todos os suprimentos desta base estão em USD.

```sql
SELECT suprimento_nome, custo_unitario, moeda FROM suprimentos
WHERE custo_unitario = (SELECT MAX(custo_unitario) FROM suprimentos);
```

**4.4 — Calcule o valor total do inventário de suprimentos da empresa (quantidade × preço unitário de todos os itens em todos os escritórios).**

R: USD — 700535.5.

```sql
SELECT moeda, ROUND(SUM(quantidade_estoque * custo_unitario), 2) AS valor_total
FROM suprimentos GROUP BY moeda;
```

## Nível 5

**5.1 — Quais produtos foram vendidos pela Momento? Liste todos os produtos únicos.**

R: 23 resultados, listados pela consulta.

```sql
SELECT DISTINCT p.produto_id, p.produto_nome FROM itens_venda i
JOIN vendas v ON v.venda_id = i.venda_id
JOIN status_venda s ON s.status_venda_id = v.status_venda_id
JOIN produtos p ON p.produto_id = i.produto_id
WHERE s.nome = 'CONCLUIDA' ORDER BY p.produto_nome;
```

**5.2 — Qual é o produto mais vendido (maior quantidade total)?**

R: Reator Arc decorativo — 48.

Considera produtos com ao menos uma venda concluída.

```sql
WITH totais AS (
 SELECT p.produto_id, p.produto_nome, SUM(i.quantidade) AS unidades FROM itens_venda i
JOIN vendas v ON v.venda_id = i.venda_id
JOIN status_venda s ON s.status_venda_id = v.status_venda_id
JOIN produtos p ON p.produto_id = i.produto_id
 WHERE s.nome = 'CONCLUIDA' GROUP BY p.produto_id, p.produto_nome
)
SELECT produto_nome, unidades FROM totais
WHERE unidades = (SELECT MAX(unidades) FROM totais);
```

**5.3 — Qual é o produto menos vendido?**

R: Chapéu Seletor — 1.

Considera produtos com ao menos uma venda concluída.

```sql
WITH totais AS (
 SELECT p.produto_id, p.produto_nome, SUM(i.quantidade) AS unidades FROM itens_venda i
JOIN vendas v ON v.venda_id = i.venda_id
JOIN status_venda s ON s.status_venda_id = v.status_venda_id
JOIN produtos p ON p.produto_id = i.produto_id
 WHERE s.nome = 'CONCLUIDA' GROUP BY p.produto_id, p.produto_nome
)
SELECT produto_nome, unidades FROM totais
WHERE unidades = (SELECT MIN(unidades) FROM totais);
```

**5.4 — Pensando na relação quantidade × valor unitário, qual produto gerou mais receita para a empresa?**

R: Tesseract decorativo — USD — 236788.65.

Usa o preço efetivo do item vendido, não o preço atual do catálogo.

```sql
WITH receitas AS (
 SELECT p.produto_id, p.produto_nome, i.moeda,
 SUM(i.quantidade * i.preco_unitario) AS receita FROM itens_venda i
JOIN vendas v ON v.venda_id = i.venda_id
JOIN status_venda s ON s.status_venda_id = v.status_venda_id
JOIN produtos p ON p.produto_id = i.produto_id
 WHERE s.nome = 'CONCLUIDA' GROUP BY p.produto_id, p.produto_nome, i.moeda
)
SELECT produto_nome, moeda, ROUND(receita, 2) AS receita FROM receitas r
WHERE receita = (SELECT MAX(r2.receita) FROM receitas r2 WHERE r2.moeda = r.moeda);
```

**5.5 — Qual é o produto mais caro (maior preço unitário) no catálogo?**

R: Tesseract decorativo — USD — 7000.

```sql
SELECT produto_nome, moeda, preco_catalogo FROM produtos p
WHERE preco_catalogo = (SELECT MAX(p2.preco_catalogo) FROM produtos p2 WHERE p2.moeda = p.moeda);
```

**5.6 — Qual foi o faturamento total da empresa em vendas?**

R: USD — 780061.94.

```sql
SELECT i.moeda, ROUND(SUM(i.quantidade * i.preco_unitario), 2) AS faturamento
FROM itens_venda i JOIN vendas v ON v.venda_id = i.venda_id
JOIN status_venda s ON s.status_venda_id = v.status_venda_id
WHERE s.nome = 'CONCLUIDA' GROUP BY i.moeda;
```

**5.7 — Quantas vendas foram realizadas no mês de junho de 2023?**

R: 2.

```sql
SELECT COUNT(*) AS total_vendas FROM vendas v
JOIN status_venda s ON s.status_venda_id = v.status_venda_id
WHERE s.nome = 'CONCLUIDA' AND v.data_venda >= '2023-06-01'
AND v.data_venda < '2023-07-01';
```

**5.8 — Qual vendedor realizou mais vendas (em quantidade de transações)?**

R: John — Russell — 10; Jack — Livingston — 10.

Conta pedidos, sem multiplicar a transação pelo número de itens.

```sql
WITH totais AS (
 SELECT f.funcionario_id, f.primeiro_nome, f.sobrenome, COUNT(*) AS transacoes
 FROM funcionarios f JOIN vendas v ON v.funcionario_id = f.funcionario_id
 JOIN status_venda s ON s.status_venda_id = v.status_venda_id
 WHERE s.nome = 'CONCLUIDA' GROUP BY f.funcionario_id, f.primeiro_nome, f.sobrenome
)
SELECT primeiro_nome, sobrenome, transacoes FROM totais
WHERE transacoes = (SELECT MAX(transacoes) FROM totais);
```

**5.9 — Qual vendedor gerou mais receita para a empresa?**

R: Jack — Livingston — USD — 187825.62.

```sql
WITH receitas AS (
 SELECT f.funcionario_id, f.primeiro_nome, f.sobrenome, i.moeda,
 SUM(i.quantidade * i.preco_unitario) AS receita
 FROM funcionarios f JOIN vendas v ON v.funcionario_id = f.funcionario_id
 JOIN itens_venda i ON i.venda_id = v.venda_id
 JOIN status_venda s ON s.status_venda_id = v.status_venda_id
 WHERE s.nome = 'CONCLUIDA' GROUP BY f.funcionario_id, f.primeiro_nome, f.sobrenome, i.moeda
)
SELECT primeiro_nome, sobrenome, moeda, ROUND(receita, 2) AS receita FROM receitas r
WHERE receita = (SELECT MAX(r2.receita) FROM receitas r2 WHERE r2.moeda = r.moeda);
```

## Nível 6

Execute os blocos 6.1 a 6.6 em ordem, na mesma conexão. A transação termina com `ROLLBACK`, para simular as mudanças sem gravá-las. Os identificadores 31 e 106 estão livres na base original; não execute os INSERTs novamente em uma base já alterada.

```sql
START TRANSACTION;
```

**6.1 — Um novo departamento foi criado: **Inovações**. Ele será alocado no escritório "Wayne Offices". Adicione-o ao banco de dados.**

R: Criar o departamento 31, Inovações, no escritório Wayne Offices.

```sql
INSERT INTO departamentos
(departamento_id, departamento_nome, educacional, escritorio_id)
SELECT 31, 'Inovações', 0, escritorio_id FROM escritorios
WHERE escritorio_nome = 'Wayne Offices';

SELECT departamento_id, departamento_nome, escritorio_id FROM departamentos WHERE departamento_id = 31;
```

**6.2 — O departamento de Inovações está sem funcionários. Transfira 2 funcionários do departamento de Tecnologia para Inovações.**

R: Transferir Valli Stark (106) e Diana Lorentz (107), ambas de Tecnologia, para Inovações. Escolha didática para manter Bruce no departamento até a promoção.

```sql
UPDATE funcionarios SET departamento_id = 31
WHERE funcionario_id IN (106, 107) AND departamento_id = 6;

SELECT funcionario_id, primeiro_nome, sobrenome, departamento_id FROM funcionarios WHERE funcionario_id IN (106, 107);
```

**6.3 — A empresa decidiu dar um aumento de 10% para todos os funcionários do departamento de Tecnologia. Atualize os salários.**

R: Reajustar em 10% os salários de quem permaneceu em Tecnologia.

```sql
UPDATE funcionarios SET salario = ROUND(salario * 1.10, 2)
WHERE departamento_id = 6;

SELECT funcionario_id, primeiro_nome, salario, moeda_salario FROM funcionarios WHERE departamento_id = 6;
```

**6.4 — O funcionário "Bruce Ernst" foi promovido a "Senior Web Developer" e recebeu um aumento para $5.000. Atualize suas informações.**

R: Atualizar Bruce para Senior Web Developer e USD 5.000,00, seguindo o valor pedido. Há uma inconsistência no enunciado: Bruce já recebe USD 6.000,00 antes do reajuste, e o cargo sênior tem mínimo cadastrado de USD 7.000,00; portanto, USD 5.000,00 representa redução e fica abaixo da faixa do cargo. O banco permite a alteração, pois não há restrição entre salário e faixa do cargo.

```sql
UPDATE funcionarios
SET cargo_id = (SELECT cargo_id FROM cargos WHERE cargo_nome = 'Senior Web Developer'),
 salario = 5000.00
WHERE funcionario_id = 104 AND primeiro_nome = 'Bruce' AND sobrenome = 'Ernst';

SELECT f.primeiro_nome, f.sobrenome, c.cargo_nome, f.salario FROM funcionarios f JOIN cargos c ON c.cargo_id = f.cargo_id WHERE f.funcionario_id = 104;
```

**6.5 — Adicione um novo suprimento ao escritório "Wayne Offices": 15 unidades de "Headsets" com preço unitário de $150 cada.**

R: Adicionar 15 headsets de USD 150,00: total USD 2.250,00. Referência e data são valores didáticos.

```sql
INSERT INTO suprimentos
(suprimento_id, referencia_compra, suprimento_nome, quantidade_estoque,
 custo_unitario, moeda, data_compra, escritorio_id)
SELECT 106, 'AULA-HEADSETS-20261108', 'Headsets', 15, 150.00,
 'USD', '2026-11-08', escritorio_id FROM escritorios
WHERE escritorio_nome = 'Wayne Offices';

SELECT suprimento_nome, quantidade_estoque, custo_unitario, moeda FROM suprimentos WHERE suprimento_id = 106;
```

**6.6 — Todos os funcionários contratados antes de 1990 estão aposentando. Remova-os do banco de dados (CUIDADO: execute um **SELECT** antes para ver quantos serão afetados!).**

R: 8 funcionários. A exclusão física é bloqueada por relacionamentos, como os dependentes desses funcionários. O próprio nível 12 orienta registrar a aposentadoria como `INATIVO`, preservando o histórico. Por isso, a solução executável abaixo faz a inativação; o DELETE literal é mostrado como comentário.

```sql
SELECT funcionario_id, primeiro_nome, sobrenome FROM funcionarios WHERE data_contratacao < '1990-01-01';

SELECT COUNT(*) AS afetados FROM funcionarios
WHERE data_contratacao < '1990-01-01';

-- Comando literal do nível 6.6, bloqueado por chaves estrangeiras:
-- DELETE FROM funcionarios WHERE data_contratacao < '1990-01-01';

UPDATE funcionarios SET status_funcionario_id =
 (SELECT status_funcionario_id FROM status_funcionario WHERE nome = 'INATIVO')
WHERE data_contratacao < '1990-01-01';

SELECT f.funcionario_id, f.primeiro_nome, s.nome AS status
FROM funcionarios f JOIN status_funcionario s
 ON s.status_funcionario_id = f.status_funcionario_id
WHERE f.data_contratacao < '1990-01-01';

ROLLBACK;
```
