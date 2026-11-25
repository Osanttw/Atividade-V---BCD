# Relatório de Queries, Stored Procedures e Views – TechStore
Por Eduardo Dos Santos Lima

Neste relatório mostramos as 10 consultas (DQL), as 5 stored procedures e as 5 views que fizemos no banco `loja_tech` para responder às necessidades da loja de tecnologia TechStore. Tudo foi testado no MySQL Workbench com os dados de teste da aula (5 categorias, 11 produtos, 5 clientes, 5 pedidos e 7 itens de pedido).

## Arquivos do repositório

| Arquivo | Conteúdo |
|---|---|
| `01_criar_banco.sql` | Criação do banco, tabelas, relacionamentos e carga de dados (DDL + DML) |
| `02_consultas.sql` | As 10 consultas (DQL) |
| `03_procedures.sql` | As 5 stored procedures |
| `04_views.sql` | As 5 views |
| `README.md` | Este relatório |

## Como executar

Execute os scripts nesta ordem:

```bash
mysql -u root -p < 01_criar_banco.sql
mysql -u root -p < 02_consultas.sql
mysql -u root -p < 03_procedures.sql
mysql -u root -p < 04_views.sql
```

No MySQL Workbench, abra cada arquivo e execute com `Ctrl+Shift+Enter`.

## Banco utilizado

Usamos 6 tabelas ligadas por chaves estrangeiras:

| Tabela | O que guarda | Relacionamento |
|---|---|---|
| `categorias` | nome e descrição do grupo de produtos | 1 categoria tem vários produtos |
| `produtos` | nome, preço, estoque e estoque mínimo | liga-se a `categorias`; 1 produto aparece em vários itens |
| `clientes` | nome, e-mail, CPF e telefone | 1 cliente faz vários pedidos |
| `pedidos` | cliente, data, status e valor total | 1 pedido tem vários itens |
| `itens_pedido` | produto, quantidade, preço e subtotal | liga `pedidos` e `produtos` (N:N) |
| `log_operacoes` | tabela afetada, operação e descrição | auditoria das operações |

---

## Parte 1 – Consultas 1 a 10

**1) Marketing:** nome e e-mail de todos os clientes.

```sql
SELECT nome, email FROM clientes;
```
Resultado: 5 clientes. Exemplo: Ana Silva, ana.silva@email.com.

**2) Vitrine promocional:** produtos que custam menos de R$ 500,00.

```sql
SELECT nome, preco FROM produtos WHERE preco < 500.00;
```
Resultado: 5 produtos, do SSD Kingston 480GB (R$ 219,90) ao Headset HyperX Cloud II (R$ 499,90).

**3) Atendimento:** pedidos aguardando pagamento.

```sql
SELECT * FROM pedidos WHERE status = 'pendente';
```
Resultado: 1 pedido, o nº 3 da cliente Carla Santos, no valor de R$ 1.099,70.

**4) Catálogo:** produtos do mais caro para o mais barato.

```sql
SELECT nome, preco FROM produtos ORDER BY preco DESC;
```
Resultado: 11 produtos. O mais caro é o iPhone 15 (R$ 6.499,00) e o mais barato é o SSD Kingston 480GB (R$ 219,90).

**5) Busca de telas:** produtos com "Monitor" no nome.

```sql
SELECT id_produto, nome, preco FROM produtos WHERE nome LIKE '%Monitor%';
```
Resultado: 2 produtos: Monitor LG UltraWide 29" (R$ 1.599,00) e Monitor Samsung Curvo 27" (R$ 1.899,00).

**6) Logística:** periféricos com mais de 40 unidades em estoque.

```sql
SELECT nome, estoque FROM produtos WHERE id_categoria = 3 AND estoque > 40;
```
Resultado: 2 produtos: Teclado Mecânico Redragon (60) e Mouse Logitech G502 (45). O Headset HyperX tem 30 e fica de fora.

**7) Total de clientes cadastrados.**

```sql
SELECT COUNT(*) AS total_clientes FROM clientes;
```
Resultado: 5 clientes.

**8) Faturamento bruto:** soma de todos os pedidos.

```sql
SELECT SUM(valor_total) AS faturamento_bruto FROM pedidos;
```
Resultado: R$ 15.897,40. Esse valor inclui o pedido cancelado.

**9) Catálogo com categoria:** produto e nome da categoria.

```sql
SELECT p.nome AS produto, c.nome AS categoria
FROM produtos p
INNER JOIN categorias c ON c.id_categoria = p.id_categoria;
```
Resultado: 11 linhas. Exemplo: Notebook Dell Inspiron 15 pertence a Notebooks.

**10) Pedidos da cliente Ana Silva.**

```sql
SELECT pe.id_pedido, pe.status, pe.valor_total
FROM pedidos pe
INNER JOIN clientes c ON c.id_cliente = pe.id_cliente
WHERE c.nome = 'Ana Silva';
```

| Pedido | Status | Valor total |
|---|---|---|
| 1 | entregue | R$ 3.749,80 |
| 4 | processando | R$ 6.499,00 |

---

## Parte 2 – Stored Procedures 1 a 5

**1) `listar_categorias_ativas` (sem parâmetros):** categorias ativas em ordem alfabética.

```sql
DROP PROCEDURE IF EXISTS listar_categorias_ativas;
DELIMITER //
CREATE PROCEDURE listar_categorias_ativas()
BEGIN
    SELECT id_categoria, nome, descricao
    FROM categorias
    WHERE ativo = TRUE
    ORDER BY nome;
END //
DELIMITER ;

CALL listar_categorias_ativas();
```
Resultado: 5 categorias, de Armazenamento até Smartphones.

**2) `buscar_pedidos_por_status` (parâmetro IN):** pedidos de um status informado.

```sql
DROP PROCEDURE IF EXISTS buscar_pedidos_por_status;
DELIMITER //
CREATE PROCEDURE buscar_pedidos_por_status(IN p_status VARCHAR(20))
BEGIN
    SELECT id_pedido, id_cliente, data_pedido, valor_total
    FROM pedidos
    WHERE status = p_status;
END //
DELIMITER ;

CALL buscar_pedidos_por_status('pendente');
```
Resultado: 1 pedido (nº 3, cliente 3, R$ 1.099,70). Com `'entregue'` retorna o pedido nº 1.

**3) `inserir_categoria` (INSERT com parâmetros IN):** cadastra uma nova categoria.

```sql
DROP PROCEDURE IF EXISTS inserir_categoria;
DELIMITER //
CREATE PROCEDURE inserir_categoria(
    IN p_nome VARCHAR(100),
    IN p_descricao TEXT
)
BEGIN
    INSERT INTO categorias (nome, descricao) VALUES (p_nome, p_descricao);
END //
DELIMITER ;

CALL inserir_categoria('Acessórios', 'Cabos, capas e carregadores');
```
Resultado: a categoria "Acessórios" entra com o id 6.

**4) `atualizar_telefone_cliente` (UPDATE com parâmetros IN):** troca o telefone de um cliente.

```sql
DROP PROCEDURE IF EXISTS atualizar_telefone_cliente;
DELIMITER //
CREATE PROCEDURE atualizar_telefone_cliente(
    IN p_id_cliente INT,
    IN p_telefone VARCHAR(15)
)
BEGIN
    UPDATE clientes SET telefone = p_telefone WHERE id_cliente = p_id_cliente;
END //
DELIMITER ;

CALL atualizar_telefone_cliente(1, '11999998888');
```
Resultado: o telefone da Ana Silva passa de 11987654321 para 11999998888.

**5) `contar_produtos_categoria` (parâmetros IN e OUT):** quantidade de produtos de uma categoria.

```sql
DROP PROCEDURE IF EXISTS contar_produtos_categoria;
DELIMITER //
CREATE PROCEDURE contar_produtos_categoria(
    IN p_id_categoria INT,
    OUT p_total INT
)
BEGIN
    SELECT COUNT(*) INTO p_total
    FROM produtos
    WHERE id_categoria = p_id_categoria;
END //
DELIMITER ;

CALL contar_produtos_categoria(3, @total);
SELECT @total AS total_produtos;
```
Resultado: 3 produtos em Periféricos.

---

## Parte 3 – Views (quando usar no lugar de procedures)

Usamos view quando a tarefa é só leitura e quem consulta precisa filtrar ou ordenar do próprio jeito. Procedure fica para quando há parâmetro de entrada, escrita de dados, transação ou validação.

**View 1 – `vw_catalogo_produtos`:** vitrine de produtos ativos com categoria. O site e os vendedores aplicam filtros diferentes a cada consulta.

```sql
CREATE OR REPLACE VIEW vw_catalogo_produtos AS
SELECT p.id_produto, p.nome AS produto, c.nome AS categoria, p.preco, p.estoque
FROM produtos p
INNER JOIN categorias c ON c.id_categoria = p.id_categoria
WHERE p.ativo = TRUE;

SELECT * FROM vw_catalogo_produtos WHERE categoria = 'Periféricos';
```
Resultado: 3 periféricos (Teclado, Mouse e Headset).

**View 2 – `vw_pedidos_clientes`:** pedido e cliente juntos, para o atendimento não repetir o JOIN.

```sql
CREATE OR REPLACE VIEW vw_pedidos_clientes AS
SELECT pe.id_pedido, c.nome AS cliente, c.email, pe.data_pedido, pe.status, pe.valor_total
FROM pedidos pe
INNER JOIN clientes c ON c.id_cliente = pe.id_cliente;

SELECT * FROM vw_pedidos_clientes WHERE cliente = 'Ana Silva';
```
Resultado: 2 pedidos (nº 1 e nº 4).

**View 3 – `vw_estoque_baixo`:** lista de reposição para o setor de compras, com regra fixa (`estoque <= estoque_minimo`).

```sql
CREATE OR REPLACE VIEW vw_estoque_baixo AS
SELECT id_produto, nome, estoque, estoque_minimo,
       (estoque_minimo - estoque) AS faltam_para_minimo
FROM produtos
WHERE ativo = TRUE AND estoque <= estoque_minimo;

SELECT * FROM vw_estoque_baixo;
```
Resultado: vazia com os dados iniciais, porque nenhum produto está abaixo do mínimo. Ao reduzir o estoque do iPhone 15 para 3, ele passa a aparecer na lista.

**View 4 – `vw_faturamento_categoria`:** faturamento por categoria para dashboards de BI, desconsiderando pedidos cancelados.

```sql
CREATE OR REPLACE VIEW vw_faturamento_categoria AS
SELECT c.nome AS categoria,
       SUM(ip.quantidade) AS unidades_vendidas,
       SUM(ip.subtotal) AS faturamento
FROM itens_pedido ip
INNER JOIN pedidos pe ON pe.id_pedido = ip.id_pedido
INNER JOIN produtos pr ON pr.id_produto = ip.id_produto
INNER JOIN categorias c ON c.id_categoria = pr.id_categoria
WHERE pe.status <> 'cancelado'
GROUP BY c.id_categoria, c.nome;

SELECT * FROM vw_faturamento_categoria ORDER BY faturamento DESC;
```

| Categoria | Unidades vendidas | Faturamento |
|---|---|---|
| Smartphones | 2 | R$ 10.798,00 |
| Notebooks | 1 | R$ 3.499,90 |
| Periféricos | 5 | R$ 1.349,50 |

**View 5 – `vw_clientes_publico`:** dados de clientes sem CPF e data de nascimento, para o time comercial.

```sql
CREATE OR REPLACE VIEW vw_clientes_publico AS
SELECT id_cliente, nome, email, telefone
FROM clientes
WHERE ativo = TRUE;

SELECT * FROM vw_clientes_publico WHERE nome LIKE 'A%';
```
Resultado: 1 cliente (Ana Silva). O usuário recebe permissão só de `SELECT` na view e nunca enxerga as colunas sensíveis.

---

## Conclusão

Com JOINs, GROUP BY, LIKE e funções como COUNT e SUM, o banco `loja_tech` respondeu às 10 consultas pedidas. As 5 stored procedures padronizaram as operações de consulta, inserção, atualização e contagem, e as 5 views deixaram as consultas de leitura mais simples e seguras. A regra de escolha ficou clara: view para ler dados, procedure para executar ações com parâmetros e validações.
