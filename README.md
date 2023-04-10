# SQLServer

```
-- 3. Utilizando o Microsoft SQL Server, execute o código SQL a seguir para criar o banco de dados 
-- LojaAULA12, dentro do diretório E:\LojaAULA12. Caso necessário, utilize outra letra para 
-- representar o seu diretório. Observe que está sendo utilizado o recurso de FILEGROUPS, para 
-- a criação do banco de dados em um diretório específico.

USE master
GO


EXECUTE xp_create_subdir 'C:\LojaAULA12\'
GO


CREATE DATABASE LojaAULA12
ON PRIMARY (
NAME = 'Arquivo_Principal',
FILENAME = 'C:\LojaAULA12\Arquivo_Principal.mdf',
SIZE = 5 MB,
FILEGROWTH = 10%
),
FILEGROUP FG1 (
NAME = 'Arquivo_Dados',
FILENAME = 'C:\LojaAULA12\Arquivo_Dados.ndf',
SIZE = 3 MB,
FILEGROWTH = 10%
)
LOG ON (
NAME = 'Arquivo_Log',
FILENAME = 'C:\LojaAULA12\Arquivo_Log.ldf',
SIZE = 1 MB,
FILEGROWTH = 10%
)
GO


EXECUTE xp_dirtree 'C:\LojaAULA12', 10, 1
GO


-- 4. Após criar o banco de dados LojaAULA12, habilite seu contexto e crie as tabelas 
-- correspondentes ao DER descrito nos exercícios 1 e 2. Verifique se as tabelas foram criadas.

CREATE TABLE CLIENTES (
	Codigo_Cliente		INT	PRIMARY KEY,
	Nome_Cliente		VARCHAR(50),
	DDD_Cliente			CHAR(3),
	Telefone_Cliente	CHAR(9),
	Saldo_Cliente		DECIMAL(10,2)
)
GO


CREATE TABLE FATURAS (
	Numero_Fatura		INT	PRIMARY KEY,
	Codigo_Cliente		INT FOREIGN KEY REFERENCES CLIENTES (Codigo_Cliente),
	Data_Fatura			DATE,
	Subtotal_Fatura		DECIMAL(10,2),
	Imposto_Fatura		DECIMAL(10,2),
	Total_Fatura		DECIMAL(10,2)
)
GO

CREATE TABLE LINHAS (
	Numero_Fatura				INT,
	Numero_Linha				INT,
	Codigo_Produto				VARCHAR(10) FOREIGN KEY REFERENCES PRODUTOS (Codigo_Produto),
	Quantidade_Produto_Linha	INT,
	Valor_Linha					DECIMAL(10,2),
	Total_Linha					DECIMAL(10,2),
	PRIMARY KEY (Numero_Fatura, Numero_Linha)
)
GO

CREATE TABLE PRODUTOS (
	Codigo_Produto				VARCHAR(10)	PRIMARY KEY,
	Descricao_Produto			VARCHAR(35),
	Data_Estocagem_Produto		DATE,
	Unidade_Disponiveis_Produto	INT,
	Valor_Unitario_Produto		DECIMAL(9,2),
	Taxa_Desconto_Produto		DECIMAL(5,2),
	Codigo_Fornecedor			INT FOREIGN KEY REFERENCES FORNECEDORES (Codigo_Fornecedor)
)
GO

CREATE TABLE FORNECEDORES (
	Codigo_Fornecedor			INT	PRIMARY KEY,
	Nome_Fornecedor				VARCHAR(35),
	Contato_Fornecedor			VARCHAR(15),
	DDD_Fornecedor				CHAR(3),
	Telefone_Fornecedor			CHAR(9),
	Estado_Fornecedor			CHAR(2)
)
GO


-- 5. Altere a estrutura da tabela CLIENTES, inserindo um novo campo do tipo DATE, denominado 
-- de Data_Nascimento, para armazenar os valores referentes à data de nascimento de cada 
-- cliente.

ALTER TABLE CLIENTES 
	ADD Data_Nascimento DATE;

GO


-- 6. Utilize os dados contidos no material fornecido pelo professor e faça o importe em massa para 
-- popular cada uma das tabelas do banco de dados LojaAULA12. Em seguida, liste o conteúdo de 
-- cada uma delas.

SET DATEFORMAT DMY


BULK INSERT CLIENTES
	FROM 'C:\LojaAULA12\Lista 2\Dados\clientes.csv'
WITH (
	FIRSTROW = 2,
	DATAFILETYPE = 'char',
	FIELDTERMINATOR = ';',
	CODEPAGE = 'ACP'
)
GO

BULK INSERT FATURAS
	FROM 'C:\LojaAULA12\Lista 2\Dados\faturas.csv'
WITH (
	FIRSTROW = 2,
	DATAFILETYPE = 'char',
	FIELDTERMINATOR = ';',
	CODEPAGE = 'ACP'
)
GO

BULK INSERT LINHAS
	FROM 'C:\LojaAULA12\Lista 2\Dados\linhas.csv'
WITH (
	FIRSTROW = 2,
	DATAFILETYPE = 'char',
	FIELDTERMINATOR = ';',
	CODEPAGE = 'ACP'
)
GO


BULK INSERT PRODUTOS
	FROM 'C:\LojaAULA12\Lista 2\Dados\produtos.csv'
WITH (
	FIRSTROW = 2,
	DATAFILETYPE = 'char',
	FIELDTERMINATOR = ';',
	CODEPAGE = 'ACP'
)
GO


BULK INSERT FORNECEDORES
	FROM 'C:\LojaAULA12\Lista 2\Dados\fornecedores.csv'
WITH (
	FIRSTROW = 2,
	DATAFILETYPE = 'char',
	FIELDTERMINATOR = ';',
	CODEPAGE = 'ACP'
)
GO


-- 9. Liste os dados de todos os fornecedores do estado de São Paulo.
-- Ordene o resultado de acordo com o nome do fornecedor.
SELECT * FROM FORNECEDORES
WHERE Estado_Fornecedor = 'SP'
ORDER BY Nome_Fornecedor
GO

-- 10. Liste o total de fornecedores de cad região. Utilize aliases para nomear corretamente 
-- as colunas retornadas pela consulta
SELECT Estado_Fornecedor AS 'Região'
	, count(*) AS 'Total de Fornecedores'
FROM FORNECEDORES
GROUP BY Estado_Fornecedor
ORDER BY Estado_Fornecedor
GO

-- 12. Liste as informações de todas as faturas onde o total esteja entre 10 e 30 mil reais.
-- Exiba somente as linhas onde o imposto da fatura esteja entre 20% e 30%.
SELECT * 
FROM FATURAS
WHERE Total_Fatura BETWEEN '10000.00' AND '30000.00'
AND Imposto_Fatura BETWEEN '0.20' AND '0.30'
ORDER BY Numero_Fatura
GO

-- 13. Liste as informações dos clientes que nasceram a partir de 1990. Exiba somente o nome e a 
-- data de nascimento, juntamente com a concatenação dos campos DDD e telefone, no formato 
-- '(ddd) telefone'.
SELECT Nome_Cliente AS 'Nome do cliente',
	Data_Nascimento AS 'Data de Nascimento', 
	CONCAT('(', DDD_Cliente ,')',' ',Telefone_Cliente) AS 'Telefone'
FROM CLIENTES
WHERE YEAR(Data_Nascimento) > '1990'
GO


-- 14. Liste as informações dos clientes que nasceram a partir de 1990. Exiba somente o nome e a 
-- data de nascimento, juntamente com a concatenação dos campos DDD e telefone, no formato 
-- '(ddd) xxxx-xxxx'.
SELECT Nome_Cliente AS 'Nome do cliente',
	Data_Nascimento AS 'Data de Nascimento', 
	CONCAT('(', DDD_Cliente ,')',' ',FORMAT(CAST(Telefone_Cliente AS NUMERIC(10)), '####-####')) AS 'Telefone'
FROM CLIENTES
WHERE YEAR(Data_Nascimento) > 1990
GO


-- 16. Execute a consulta anterior, exibindo somente duas casas decimais para os valores da coluna 
-- com o valor unitário do produto após a aplicação da taxa de desconto.
-- (15. Liste o código, descrição, valor unitário e taxa de desconto dos produtos. Acrescente uma 
-- coluna no seu relatório, para exibir o valor unitário do produto após a aplicação da taxa de 
-- desconto. Utilize aliases para nomear corretamente as colunas retornadas pela consulta.)



```
