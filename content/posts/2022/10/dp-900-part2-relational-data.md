---
title: DP-900：Azure 数据基础第二部分之 Azure 中的关系型数据
author: olzhy
type: post
date: 2022-10-27T16:07:00+08:00
url: /posts/dp-900-part2-relational-data.html
categories:
  - 计算机
tags:
  - Azure
keywords:
  - 数据库
description: Azure 中的关系型数据。包括基本的关系数据概念和 Azure 中的关系型数据库服务。
---

**本文依据文末 Azure 参考资料进行翻译及整理，作学习及知识总结之用。**

关系型数据是大多数业务应用程序的核心，也是构建许多企业数据解决方案的基础。 Azure 提供用于管理关系型数据库的服务，使您能够构建新应用程序或将现有应用程序迁移到云。

## 1 基本的关系型数据概念

在计算系统的早期，每个应用程序都以自己独特的结构存储数据。当开发人员想要构建应用程序来使用这些数据时，他们必须对特定的数据结构有很多了解才能找到他们需要的数据。这些数据结构效率低下，难以维护，并且难以优化以获得良好的应用程序性能。关系数据库模型旨在解决多个任意数据结构的问题。关系模型提供了一种表示和查询可供任何应用程序使用的数据的标准方法。关系数据库模型的主要优势之一是它使用表，这是一种直观、高效且灵活的存储和访问结构化信息的方式。

各种类型和规模的组织都使用简单而强大的关系模型来满足各种信息管理需求。关系型数据库用于跟踪库存、处理电子商务交易、管理大量关键任务客户信息等。关系型数据库可用于存储任何包含相关数据元素的信息，这些数据元素必须以基于规则的一致结构进行组织。

在本部分，您将了解关系型数据库的关键特征，并探索关系数据结构。

### 1.1 了解关系型数据

在关系型数据库中，您将来自现实世界的实体集合建模为表。实体可以是您要为其记录信息的任何事物；通常是重要的对象和事件。例如，在零售系统示例中，您可以为客户、产品、订单和订单中的行项目创建表。一个表包含多行，每一行代表一个实体的一个实例。

![关系表](https://olzhy.github.io/static/images/uploads/2022/10/relational-tables.png#center)

关系表是结构化数据的一种格式，表中的每一行都有相同的列；尽管在某些情况下，并非所有列都需要有值（如上图中的 MiddleName）。

每列存储特定数据类型的数据。如，客户表中的电子邮件列可能被定义为存储字符类型的数据（长度可能是固定的或可变的），产品表中的价格列可能被定义为存储十进制数字数据，Order 表中的 Quantity 列可能被限制为整数数值，而 Order 表中的 OrderDate 列用来存储日期或时间值。定义表时可以使用的数据类型取决于您使用的数据库系统，尽管大多数数据库系统都支持 ANSI（American National Standards Institute，美国国家标准协会）定义的标准数据类型。

### 1.2 了解规范化

规范化是数据库专业人员用于模式设计过程的一个术语，其将数据重复降至最低并加强了数据完整性。

虽然有许多复杂的规则定义了将数据重构为不同级别的规范化过程，但一个出于实际目的的简单定义为：

- 将每个实体分割到自己的表中；
- 将每个离散属性分成自己的列；
- 使用主键唯一标识每个实体实例（行）；
- 使用外键列链接相关实体。

要了解规范化的核心原则，假设下表代表公司用来跟踪其销售的电子表格。

![非规范化数据](https://olzhy.github.io/static/images/uploads/2022/10/unnormalized-data.png#center)

请注意，如上表中客户和产品详细信息对于所售出的每件商品都是重复的；并且客户姓名和邮政地址，以及产品名称和价格组合在同一个电子表格单元格中。

现在让我们看看规范化如何改变数据的存储方式。

![规范化后的数据](https://olzhy.github.io/static/images/uploads/2022/10/normalized-data.png#center)

数据中表示的每个实体（客户、产品、销售订单和行条目）都存储在自己的表中，这些实体的每个离散属性都在自己的列中。

将实体的每个实例记录为实体特定表中的一行可以消除数据重复。例如，要更改客户的地址，您只需修改单行中的值。

将属性分解为单独的列可确保将每个值限制为适当的数据类型。例如，产品价格必须是十进制值，而行条目数量必须是整数。此外，单个列的创建为查询数据提供了有用的粒度级别。例如，您可以轻松地将客户筛选为居住在特定城市的人。

每个实体的实例由一个 ID（或称为主键） 唯一标识，当一个实体引用另一个实体时（例如，订单有关联的客户），相关实体的主键被存储为外键。您可以通过引用 Customer 表中的相应记录来查找 Order 表中每条记录的客户地址（仅存储一次）。通常，关系数据库管理系统 (RDBMS) 可以强制执行参照完整性，以确保输入到外键字段中的值在相关表中具有存在的对应主键 —— 例如，阻止不存在的客户订单。

在某些情况下，可以将键（主键或外键）定义为基于多个列的唯一组合的组合键。例如，上例中的 LineItem 表使用 OrderNo 和 ItemNo 的唯一组合来识别单个订单中的行项目。

### 1.3 探索 SQL

SQL 代表结构化查询语言（Structured Query Language），用于与关系数据库进行通信。SQL 语句用于执行诸如更新或检索数据等任务。一些使用 SQL 的常见关系数据库管理系统包括 Microsoft SQL Server、MySQL、PostgreSQL、MariaDB 和 Oracle。

**_SQL 最初于 1986 年由美国国家标准协会 (ANSI，American National Standards Institute) 标准化，并于 1987 年由国际标准化组织 (ISO，International Organization for Standardization) 标准化。从那时起，随着关系数据库供应商向其系统中添加新功能，该标准已多次扩展.此外，大多数数据库供应商都包括他们自己的专有扩展，这些扩展不属于标准的一部分，这导致了各种 SQL 方言。_**

您可以使用 SELECT、INSERT、UPDATE、DELETE、CREATE 和 DROP 等 SQL 语句来完成几乎所有需要对数据库执行的操作。尽管这些 SQL 语句是 SQL 标准的一部分，但许多数据库管理系统也有自己的附加专有扩展来处理该数据库管理系统的细节。这些扩展提供 SQL 标准未涵盖的功能，并包括安全管理和可编程性等领域。

例如，基于 SQL Server 数据库引擎的 Microsoft SQL Server 和 Azure 数据库服务使用了 Transact-SQL。该实现包括用于编写存储过程、触发器（可以存储在数据库中的应用程序代码）以及管理用户帐户的专有扩展。 PostgreSQL 和 MySQL 也有自己的这些特性的版本。

一些流行的 SQL 方言包括：

- Transact-SQL (T-SQL)，此版本的 SQL 由 Microsoft SQL Server 和 Azure SQL 服务所使用。
- pgSQL，这是在 PostgreSQL 中实现了扩展的 SQL 方言。
- PL/SQL，这是甲骨文使用的方言。 PL/SQL 代表 Procedural Language/SQL。

计划专门使用单个数据库系统的用户应该了解他们首选的 SQL 方言和平台的复杂性。

**_除非另有说明，本模块中的 SQL 代码示例均基于 Transact-SQL 方言。其他方言的语法大体相似，但在某些细节上可能有所不同。_**

#### SQL 语句类型

SQL 语句主要分三种：

- 数据定义语言 (Data Definition Language，DDL)
- 数据控制语言 (Data Control Language，DCL)
- 数据操作语言 (Data Manipulation Language，DML)

**DDL 语句**

您可以使用 DDL 语句来创建、修改和删除数据库中的表和其他对象（表、存储过程、视图等）。

最常见的 DDL 语句是：

| 语句   | 描述                                     |
| ------ | ---------------------------------------- |
| CREATE | 在数据库中创建一个新对象，例如表或视图。 |
| ALTER  | 修改对象的结构。例如，更改表以添加新列。 |
| DROP   | 从数据库中删除一个对象。                 |
| RENAME | 重命名现有对象。                         |

**_DROP 语句非常强大。删除表时，该表中的所有行都将丢失。除非您有备份，否则您将无法检索此数据。_**

下面的示例创建了一个新的数据库表。括号之间内指定了每列的详细信息，包括名称、数据类型、值是否非空（NOT NULL）以及列中的数据是否用于唯一（PRIMARY KEY ）。每个表都应该有一个主键，尽管 SQL 不强制执行此规则。

标记为 NOT NULL 的列称为强制列。如果省略 NOT NULL 子句，则可以创建列中不包含值的行。行中的空列被称为含有 NULL 值。

```sql
CREATE TABLE Product
(
    ID INT PRIMARY KEY,
    Name VARCHAR(20) NOT NULL,
    Price DECIMAL NULL
);
```

表中列可用的数据类型因不同的数据库管理系统而异。但是，大多数数据库管理系统都支持诸如 INT 数值类型（整数）、DECIMAL（十进制数）和字符串（如 VARCHAR）类型。

**DCL 语句**

数据库管理员通常使用 DCL 语句来授予、拒绝或撤销特定用户或组的权限来管理对数据库中对象的访问。

三个主要的 DCL 语句是：

| 语句   | 描述                     |
| ------ | ------------------------ |
| GRANT  | 授予执行特定操作的权限。 |
| DENY   | 拒绝执行特定操作的权限。 |
| REVOKE | 删除之前授予的权限。     |

例如，以下 GRANT 语句允许名为 user1 的用户读取、插入和修改 Product 表中的数据。

```sql
GRANT SELECT, INSERT, UPDATE
ON Product
TO user1;
```

**DML 语句**

您使用 DML 语句来操作表中的行。这些语句使您能够检索（查询）数据、插入新行或修改现有行。如果不再需要某些行，也可以删除它们。

四个主要的 DML 语句是：

| 语句   | 描述                 |
| ------ | -------------------- |
| SELECT | 从表中读取行。       |
| INSERT | 向表中插入新行。     |
| UPDATE | 修改现有行中的数据。 |
| DELETE | 删除现有行。         |

INSERT 语句的基本形式是一次插入一行。默认情况下，SELECT、UPDATE 和 DELETE 语句应用于表中的每一行。通常在这些语句中应用 WHERE 子句来指定条件，只有符合这些条件的行才会被选择、更新或删除。

**_SQL 没有二次确认提示。因此在使用不带 WHERE 子句的 DELETE 或 UPDATE 时要小心，因为您可能会丢失或修改大量数据。_**

以下代码是一个 SQL 语句示例，该语句从 Customer 表中选择 City 列值为“Seattle”的所有列（用 \* 表示）：

```sql
SELECT *
FROM Customer
WHERE City = 'Seattle';
```

仅从表中检索特定的列，请在 SELECT 子句中列出它们，SQL 如下：

```sql
SELECT FirstName, LastName, Address, City
FROM Customer
WHERE City = 'Seattle';
```

如果查询返回许多行，它们不一定以任何特定顺序出现。如果要对数据进行排序，可以添加 ORDER BY 子句。数据将按指定列排序：

```sql
SELECT FirstName, LastName, Address, City
FROM Customer
WHERE City = 'Seattle'
ORDER BY LastName;
```

您还可以在 SELECT 语句中使用 JOIN 子句从多个表中检索数据。JOIN 指示一个表中的行如何与另一个表中的行连接以确定要返回的数据。典型的连接条件匹配一个表中的外键及其在另一个表中的关联主键。

以下查询显示了连接 Customer 和 Order 表的示例。当指定在 SELECT 子句中检索哪些列以及在 JOIN 子句中匹配哪些列时，查询使用表别名来缩写表名。

```sql
SELECT o.OrderNo, o.OrderDate, c.Address, c.City
FROM Order AS o
JOIN Customer AS c
ON o.Customer = c.ID
```

下一个示例显示如何使用 SQL 修改现有行。对于 ID 列中值为 1 的行，它会更改 Customer 表中 Address 列的值。所有其它行保持不变：

```sql
UPDATE Customer
SET Address = '123 High St.'
WHERE ID = 1;
```

**_如果省略 WHERE 子句，UPDATE 语句将修改表中的所有行。_**

使用 DELETE 语句删除行。指定表名，以及标识要删除的行的 WHERE 子句：

```sql
DELETE FROM Product
WHERE ID = 162;
```

**_如果省略 WHERE 子句，则 DELETE 语句将删除表中的所有行。_**

INSERT 语句的形式略有不同。在 INTO 子句中指定表和列，以及要存储在这些列中的值列表。标准 SQL 仅支持一次插入一行，如下例所示。某些方言允许您指定多个 VALUES 子句以一次添加多行：

```sql
INSERT INTO Product(ID, Name, Price)
VALUES (99, 'Drill', 4.99);
```

### 1.4 描述数据库对象

除了表之外，关系数据库还可以包含其他有助于优化数据组织、封装编程操作和提高访问速度的结构。在本单元中，您将更详细地了解其中三种结构：视图、存储过程和索引。

#### 什么是视图？

视图是基于 SELECT 查询结果的虚拟表。您可以将视图视为一个或多个基础表中指定行的窗口。例如，您可以在 Order 和 Customer 表上创建一个视图，该视图检索订单和客户数据以提供单个对象，以便轻松确定订单的交货地址：

```sql
CREATE VIEW Deliveries
AS
SELECT o.OrderNo, o.OrderDate,
       c.FirstName, c.LastName, c.Address, c.City
FROM Order AS o JOIN Customer AS c
ON o.Customer = c.ID;
```

您可以像查询表一样查询视图和过滤数据。以下查询查找居住在 Seattle 的客户的订单详细信息：

```sql
SELECT OrderNo, OrderDate, LastName, Address
FROM Deliveries
WHERE City = 'Seattle';
```

#### 什么是存储过程？

存储过程定义了可以在命令上运行的 SQL 语句。存储过程用于将程序逻辑封装在数据库中，以用于应用程序在处理数据时需要执行的操作。

您可以使用参数定义存储过程，为可能需要根据特定键或条件应用于数据的常见操作创建灵活的解决方案。例如，可以定义以下存储过程来根据指定的产品 ID 更改产品的名称。

```sql
CREATE PROCEDURE RenameProduct
	@ProductID INT,
	@NewName VARCHAR(20)
AS
UPDATE Product
SET Name = @NewName
WHERE ID = @ProductID;
```

当需要重命名产品时，您可以传递产品的 ID 和要分配的新名称来执行如下存储过程：

```sql
EXEC RenameProduct 201, 'Spanner';
```

#### 什么是索引？

索引可帮助您搜索表中的数据。将表格上的索引想象成书后的索引。书籍索引包含一组排序的参考文献，以及每个参考文献出现的页面。当你想在书中找到对某个项目的引用时，你可以通过索引来查找它。您可以使用索引中的页码直接转到书中正确的页面。如果没有索引，您可能必须通读整本书才能找到您要查找的参考资料。

在数据库中创建索引时，您从表中指定一列，并且索引包含该数据排序后的副本，这些副本带有指向表中相应行的指针。当用户运行在 WHERE 子句中指定该列的查询时，数据库管理系统可以使用该索引来更快地获取数据，而不是必须逐行扫描整个表。

例如，您可以使用以下代码在 Product 表的 Name 列上创建索引：

```sql
CREATE INDEX idx_ProductName
ON Product(Name);
```

该索引创建了一个基于树的结构，数据库系统的查询优化器可以使用该结构根据指定的名称在 Product 表中快速查找行。

![索引](https://olzhy.github.io/static/images/uploads/2022/10/index.png#center)

对于包含少量行的表，使用索引可能并不比简单地读取整个表并查找请求的行更有效（在这种情况下，查询优化器将忽略索引）。但是，当一个表有很多行时，索引可以显著提高查询的性能。

您可以在一个表上创建许多索引。因此，如果您还想根据价格查找产品，则在 Product 表的 Price 列上创建另一个索引可能会很有用。但是，索引不是免费的。索引会消耗存储空间，每次在表中插入、更新或删除数据时，都必须维护该表的索引。这种额外的工作会减慢插入、更新和删除操作。您必须在拥有加快查询速度的索引与执行其他操作的成本之间取得平衡。

## 2 Azure 中的关系型数据库服务

Azure 支持多种数据库服务，使您能够在云上运行流行的关系型数据库管理系统，例如 SQL Server、PostgreSQL 和 MySQL。

大多数 Azure 数据库服务都是完全托管的，从而节省了您原本用于管理数据库的宝贵时间。具有内置高可用性的企业级性能意味着您可以快速扩展并实现全球分布，而无需担心代价高昂的停机时间。开发人员可以利用行业领先的创新，例如具有自动监控和威胁检测功能的内置安全性、自动调整以提高性能。除了所有这些功能之外，您还可以保证可用性。

在本模块中，你将探索 Azure 中的关系型数据库服务的可用选项。

### 2.1 Azure SQL 服务和功能

Azure SQL 是 Azure 中一系列基于 Microsoft SQL Server 的数据库服务的统称。特定的 Azure SQL 服务包括：

- Azure 虚拟机上的 SQL Server - 在 Azure 虚拟机上运行的 SQL Server。使用 VM 使此选项成为基础架构即服务 (IaaS) 解决方案，可虚拟化用于 Azure 中的计算、存储和网络的硬件基础架构；使其成为将现有 On-Premise SQL Server 安装“直接迁移”到云的绝佳选择。

- Azure SQL 托管实例 - 一种平台即服务 (PaaS) 选项，可提供与本地 SQL Server 实例近 100% 的兼容性，同时抽象底层硬件和操作系统。该服务包括自动化软件更新管理、备份和其他维护任务，减少了支持数据库服务器实例的管理负担。

- Azure SQL Database - 一种完全托管、高度可扩展的 PaaS 数据库服务，专为云而设计。此服务包括 On-Premise SQL Server 的核心数据库级功能，当您需要在云上创建新应用程序时，它是一个不错的选择。

- Azure SQL Edge - 针对需要处理流时间序列数据的物联网 (IoT) 场景进行了优化的 SQL 引擎。

#### Azure SQL 服务比较

|                   | Azure 虚拟机上的 SQL Server                                                                             | Azure SQL 托管实例                                                                                      | Azure SQL Database                                                                                                 |
| ----------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| 云服务类型        | IaaS                                                                                                    | PaaS                                                                                                    | PaaS                                                                                                               |
| SQL Server 兼容性 | 与 On-Premise 物理和虚拟化安装完全兼容。应用程序和数据库可以轻松地“直接迁移”而无需更改。                | 与 SQL Server 几乎 100% 兼容。可以使用 Azure 数据库迁移服务迁移大多数本地数据库，只需作最少的代码更改。 | 支持 SQL Server 的大多数核心数据库级功能。On-Premise 应用程序所依赖的某些功能可能不可用。                          |
| 架构              | SQL Server 实例安装在虚拟机中。每个实例可以支持多个数据库。                                             | 每个托管实例可以支持多个数据库。此外，实例池可用于在较小的实例之间有效地共享资源。                      | 您可以在专用的托管（逻辑）服务器中配置单个数据库；或者您可以使用弹性池在多个数据库之间共享资源并利用按需可扩展性。 |
| 可用性            | 99.99%                                                                                                  | 99.99%                                                                                                  | 99.995%                                                                                                            |
| 管理              | 您必须管理服务器的所有方面，包括操作系统和 SQL Server 更新、配置、备份和其他维护任务。                  | 全自动更新、备份和恢复。                                                                                | 全自动更新、备份和恢复。                                                                                           |
| 使用场景          | 当您需要迁移或扩展 On-Premise SQL Server 解决方案并保留对服务器和数据库配置的完全控制时，请使用此选项。 | 此选项适用于大多数云迁移方案，尤其是当您需要对现有应用程序进行最小更改时。                              | 此选项适用于新的云解决方案，或迁移具有最少实例级依赖性的应用程序。                                                 |

#### Azure 虚拟机上的 SQL Server

虚拟机上的 SQL Server 使您能够在云上使用完整版本的 SQL Server，而无需管理任何本地硬件。这是 IaaS 方法的一个示例。

在 Azure 虚拟机上运行的 SQL Server 有效地复制了在真实 On-Premise 硬件上运行的数据库。从 On-Premise 运行的系统迁移到 Azure 虚拟机与将数据库从一个 On-Premise 服务器移动到另一个服务器没有什么不同。

此方法适用于需要访问 PaaS 级别可能不支持的操作系统功能的迁移和应用程序。 SQL 虚拟机已为需要快速迁移到云且更改最少的现有应用程序做好了直接迁移的准备。您还可以使用 Azure VM 上的 SQL Server 将现有的本地应用程序扩展到混合部署中的云。

**_混合部署是一个系统，其中部分操作在本地运行，部分在云中运行。尽管数据库元素可能托管在云中，但您的数据库可能是在本地运行的更大系统的一部分。_**

您可以在虚拟机中使用 SQL Server 来开发和测试传统的 SQL Server 应用程序。使用虚拟机，您对 DBMS 和操作系统拥有完全的管理权限。当组织已经拥有可用于维护虚拟机的 IT 资源时，这是一个完美的选择。

这些功能使您能够：

- 当您不想购买 On-Premise 非生产 SQL Server 硬件时，创建快速开发和测试方案。
- 为需要快速迁移到云的现有应用程序做好直接迁移准备，只需极少更改或无需更改。
- 通过为虚拟机分配更多内存、CPU 算力和磁盘空间来扩展运行 SQL Server 的平台。您可以快速调整 Azure 虚拟机的规格，而无需重新安装在其上运行的软件。

**商业效益**

在虚拟机上运行 SQL Server 允许您通过结合 On-Premise 部署和云托管部署来满足独特且多样化的业务需求，同时在这些环境中使用相同的服务器产品、开发工具和专业知识集。

企业将其 DBMS 转换为完全托管的服务并不总是那么容易。为了迁移到需要对数据库和使用它的应用程序进行更改的托管服务，可能必须满足特定要求。出于这个原因，使用虚拟机可以提供解决方案，但使用它们并不能消除像在 On-Premise 一样仔细管理 DBMS 的需要。

#### Azure SQL 数据库托管实例

Azure SQL 托管实例有效地在云中运行完全可控的 SQL Server 实例。您可以在同一个实例上安装多个数据库。您可以完全控制此实例，就像控制 On-Premise 服务器一样。 SQL 托管实例可自动执行备份、软件修补、数据库监控和其它常规任务，但您可以完全控制数据库的安全性和资源分配。

托管实例依赖于其它 Azure 服务，例如用于备份的 Azure 存储、用于遥测的 Azure 事件中心、用于身份验证的 Azure Active Directory、用于透明数据加密 (TDE) 的 Azure Key Vault 以及一些提供安全性和可支持性功能的 Azure 平台服务。托管实例与这些服务建立连接。

所有通信都使用证书加密和签名。为了检查通信方的可信度，托管实例通过证书吊销列表不断验证这些证书。如果证书被吊销，托管实例将关闭连接以保护数据。

**使用场景**

如果您想将 On-Premise SQL Server 实例及其所有数据库直接迁移到云端，而不会产生在虚拟机上运行 SQL Server 的管理开销，请考虑使用 Azure SQL 托管实例。

Azure SQL 托管实例提供 Azure SQL Database 中不可用的功能。如果您的系统使用链接服务器、Service Broker（可用于跨服务器分发工作的消息处理系统）或数据库邮件（使您的数据库能够向用户发送电子邮件）等功能，那么您应该使用托管实例。要检查与现有本地系统的兼容性，您可以安装数据迁移助手 (Data Migration Assistant，DMA)。此工具分析您在 SQL Server 上的数据库并报告可能阻止迁移到托管实例的任何问题。

**商业效益**

Azure SQL 托管实例使系统管理员能够在管理任务上花费更少的时间，因为该服务可以为您执行这些任务或大大简化这些任务。自动化任务包括操作系统和数据库管理系统软件安装和修补、动态实例规格调整和配置、备份、数据库复制（包括系统数据库）、高可用性配置以及运行状况和性能监控数据流的配置。

Azure SQL 托管实例与本地运行的 SQL Server 企业版几乎 100% 兼容。

Azure SQL 托管实例支持 SQL Server 数据库引擎登录和与 Azure Active Directory (AD) 集成的登录。 SQL Server 数据库引擎登录包括用户名和密码。每次连接到服务器时都必须输入凭据。 Azure AD 登录使用与当前计算机登录关联的凭据，您无需在每次连接到服务器时都提供这些凭据。

#### Azure SQL Database

Azure SQL Database 是 Microsoft 的 PaaS 产品。您在云上创建托管数据库服务器，然后在此服务器上部署您的数据库。

**_SQL 数据库服务器是一个逻辑结构，它充当多个单一或共用数据库、登录、防火墙规则、审核规则、威胁检测策略和故障转移组的中央管理点。_**

Azure SQL Database 可用作单一数据库或弹性池。

**单一数据库**

此选项使您能够快速设置和运行单个 SQL Server 数据库。您在云上创建并运行数据库服务器，并通过该服务器访问您的数据库。 Microsoft 管理服务器，因此您所要做的就是配置数据库、创建表并用您的数据填充它们。如果您需要更多的存储空间、内存或处理能力，您可以扩展数据库。默认情况下，资源是预先分配的，您需要按小时为您请求的资源付费。您还可以指定无服务器配置。在此配置中，Microsoft 创建自己的服务器，该服务器可能由属于其他 Azure 订阅者的数据库共享。 Microsoft 确保您的数据库的隐私。您的数据库会自动扩展，并根据需要分配或取消分配资源。

**弹性池**

此选项与 Single Database 类似，不同之处在于默认情况下，多个数据库可以通过多租户共享相同的资源，例如内存、数据存储空间和处理能力。这些资源称为池。您创建池，并且只有您的数据库可以使用该池。如果您的数据库的资源需求随时间变化，此模型很有用，并且可以帮助您降低成本。例如，您的工资单数据库在每个月末处理工资单时可能需要大量的 CPU 算力，但在其它时候数据库可能会变得不那么活跃。您可能有另一个用于运行报告的数据库。随着管理报告的生成，该数据库可能会在月中的几天内处于活动状态，但在其他时间负载较轻。弹性池允许您使用池中可用的资源，然后在处理完成后释放资源。

**使用场景**

Azure SQL Database 为你提供了低成本和最少管理的最佳选择。它与 On-Premise SQL Server 安装不完全兼容。它通常用于新的云项目中，其中应用程序设计可以适应对应用程序的任何所需更改。

Azure SQL Database 通常用于：

- 需要使用最新稳定 SQL Server 功能的现代云应用程序。
- 需要高可用性的应用程序。
- 具有可变负载的系统需要数据库服务器快速向上和向下扩展。

**商业效益**

Azure SQL Database 会自动更新和修补 SQL Server 软件，以确保您始终运行最新且最安全的服务版本。

Azure SQL Database 的可扩展性功能确保您可以增加可用于存储和处理数据的资源，而无需执行成本高昂的手动升级。

该服务提供高可用性保证，以确保您的数据库至少有 99.995% 的时间可用。 Azure SQL Database 支持按时间点还原，使你能够将数据库恢复到过去任何时候的状态。数据库可以复制到不同的区域，以提供更多的弹性和灾难恢复。

高级威胁防护提供高级安全功能，例如漏洞评估，以帮助检测和修复数据库的潜在安全问题。威胁防护还检测异常活动，这些活动表明访问或利用您的数据库的异常和潜在有害尝试。它持续监控您的数据库中的可疑活动，并针对潜在漏洞、SQL 注入攻击和异常数据库访问模式提供即时安全警报。威胁检测警报提供可疑活动的详细信息，并就如何调查和缓解威胁提出建议。

审核跟踪数据库事件并将它们写入 Azure 存储帐户中的审核日志。审计可以帮助您保持合规性，了解数据库活动，并深入了解可能表明业务问题或可疑安全违规的差异和异常。

Azure SQL Database 通过提供加密保护存储在数据库中的数据（静态）和通过保护网络传输（动态）的数据来使您的数据更安全。

### 2.2 针对开源数据库的 Auzre 服务

除了 Azure SQL 服务之外，Azure 数据服务还可用于其它流行的关系型数据库系统，包括 MySQL、MariaDB 和 PostgreSQL。这些服务的主要原因是使在 On-Premise 应用程序中使用它们的组织能够快速迁移到 Azure，而无需对其应用程序进行重大更改。

#### MySQL、MariaDB 和 PostgreSQL 是什么？

MySQL、MariaDB 和 PostgreSQL 是为不同专业量身定制的关系型数据库管理系统。

MySQL 最初是一个简单易用的开源数据库管理系统。它是适用于 Linux、Apache、MySQL 和 PHP (LAMP) 技术栈应用程序的领先开源关系型数据库。它有多个版本；社区版、标准版和企业版。社区版是免费提供的，并且作为在 Linux 下运行的 Web 应用程序的数据库管理系统在历史上一直很受欢迎。也有 Windows 版本。标准版提供更高的性能，并使用不同的技术来存储数据。企业版提供了一套全面的工具和功能，包括增强的安全性、可用性和可扩展性。标准版和企业版是商业组织最常使用的版本，尽管这些版本的软件不是免费的。

MariaDB 是一种较新的数据库管理系统，由 MySQL 的原始开发人员创建。此后，数据库引擎已被重写和优化以提高性能。 MariaDB 提供与 Oracle 数据库的兼容性。 MariaDB 的一个显着特点是其对时态数据的内置支持。一个表可以包含多个版本的数据，使应用程序能够查询过去某个时间点出现的数据。

PostgreSQL 是一个混合关系对象数据库。您可以将数据存储在关系表中，但 PostgreSQL 数据库还使您能够存储自定义数据类型，以及它们自己的非关系属性。数据库管理系统可扩展；您可以将代码模块添加到数据库中，这些模块可以通过查询运行。另一个关键特性是能够存储和操作几何数据，例如线、圆和多边形。

PostgreSQL 有自己的查询语言，称为 pgsql。该语言是标准关系查询语言 SQL 的变体，具有使您能够编写在数据库中运行的存储过程的功能。

#### Azure Database for MySQL

Azure Database for MySQL 是 Azure 云上基于 MySQL 社区版的 PaaS 实现。

Azure Database for MySQL 服务包括无需额外费用的高可用性和所需的可伸缩性。您只需为使用的内容付费。提供自动备份和时间点恢复。

服务器提供连接安全性以强制执行防火墙规则，并且可以选择要求 SSL 连接。许多服务器参数使您能够配置服务器设置，例如锁定模式、最大连接数和超时。

Azure Database for MySQL 提供了一个全局数据库系统，可以扩展到大型数据库，而无需管理硬件、网络组件、虚拟服务器、软件补丁和其他底层组件。

Azure Database for MySQL 不支持某些操作。这些功能主要与安全和管理有关。 Azure 管理数据库服务器本身的这些方面。

**Azure Database for MySQL 的优势**

Azure Database for MySQL 具有以下功能：

- 内置高可用性功能。
- 可预测的性能。
- 轻松扩展，快速响应需求。
- 保护静态和动态数据。
- 过去 35 天的自动备份和时间点恢复。
- 企业级安全性和法规遵从性。

该系统使用即用即付定价，因此您只需为使用的产品付费。

Azure Database for MySQL 服务器提供监视功能以添加警报以及查看指标和日志。

#### Azure Database for MariaDB

Azure Database for MariaDB 是适用于在 Azure 中运行的基于 MariaDB 社区版的数据库管理系统的实现。

该数据库由 Azure 完全管理和控制。配置服务并传输数据后，系统几乎不需要额外的管理。

**Azure Database for MariaDB 的优势**

Azure Database for MariaDB 提供：

- 内置高可用性，无需额外费用。
- 可预测的性能，使用包容性的即用即付定价。
- 在几秒钟内根据需要进行伸缩。
- 对静态和动态敏感数据进行安全保护。
- 自动备份和时间点恢复长达 35 天。
- 企业级安全性和合规性。

#### Azure Database for PostgreSQL

如果你更喜欢 PostgreSQL，可以选择 Azure Database for PostgreSQL 在 Azure 云上运行 PostgreSQL 的 PaaS 实现。该服务提供与 MySQL 服务相同的可用性、性能、可扩展性、安全性和管理优势。

On-Premise PostgreSQL 数据库的某些功能在 Azure Database for PostgreSQL 中不可用。这些功能主要与用户可以添加到数据库以执行特定任务的扩展有关，例如用各种编程语言（除了可用的 pgsql）编写存储过程，以及直接与操作系统交互。支持一组最常用的扩展核心，并且可用扩展列表正在持续审查中。

**Azure Database for PostgreSQL Flexible Server**

PostgreSQL 的灵活服务器部署选项是完全托管的数据库服务。它提供高级别的控制和服务器配置定制，并提供成本优化控制。

**Azure Database for PostgreSQL 的优势**

Azure Database for PostgreSQL 是一个高可用性服务。它包含内置的故障检测和故障转移机制。

PostgreSQL 的用户熟悉 pgAdmin 工具，您可以使用它来管理和监控 PostgreSQL 数据库。你可以继续使用此工具连接到 Azure Database for PostgreSQL。但是，某些以服务器为中心的功能（例如执行服务器备份和还原）不可用，因为服务器由 Azure 管理和维护。

Azure Database for PostgreSQL 记录有关针对服务器上的数据库运行的查询的信息，并将它们保存在名为 azure_sys 的数据库中。您查询 query_store.qs_view 视图以查看此信息，并使用它来监控用户正在运行的查询。如果您需要微调应用程序执行的查询，这些信息将证明是非常宝贵的。

> 参考资料
>
> [1] [Exam DP-900: Microsoft Azure Data Fundamentals - learn.microsoft.com](https://learn.microsoft.com/en-us/certifications/exams/dp-900)