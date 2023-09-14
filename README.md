Задание к лабораторной работе
Вариант №1
1.	информацию о продуктах из США или Канады стоимостью более 30;
2.	Информацию о продуктах, чьё количество в заказе более 100;
3.	Информацию о заказах, имеющие от 1 до 3 позиций;
4.	Первые 10 самых ранних заказов от клиентов из Франции;
5.	Количество заказов по каждому клиенту, отсортировать по фамилии и имени клиента;
6.	Для каждого клиента определить общую стоимость по каждому продукту. Вывести фамилию, имя клиента, название продукта и его стоимость;
7.	Поставщиков, которые поставляют только один товар. Вывести имя компании, город, страну и наименование товаров;
8.	Количество каждого продукта в заказах и его общую стоимость. Вывести информацию о продукте и его количество, и общую стоимость в заказах
Создать следующие представления:
1.	Количество поставляемых поставщиками различных товаров. Вывести имя компании, город, страну и наименование товаров, и их количество;
2.	Сведения о заказах по каждому продукту. Вывести информацию о продукте, его количество, общую стоимость в заказах и сведения о поставщике.

1)
SELECT * 
FROM dbo.Product prod
INNER JOIN dbo.[Supplier] sup ON sup.Id = prod.SupplierId
WHERE UnitPrice > 30 AND sup.Country IN ('USA', 'Canada');


2)
SELECT * 
FROM dbo.Product prod
INNER JOIN dbo.[OrderItem] OI ON OI.ProductId = prod.Id
WHERE Quantity > 100;

3)
SELECT *
FROM dbo.[Order] ord
INNER JOIN dbo.[OrderItem] OI ON OI.OrderId = ord.Id
WHERE Quantity BETWEEN 1 AND 3;

4)
SELECT TOP(10) *
FROM dbo.[Order] ord
INNER JOIN dbo.[Customer] cust ON cust.Id = ord.CustomerId
WHERE Country = N'France'
ORDER BY OrderDate ASC;

5)
SELECT DISTINCT COUNT(ord.CustomerId) AS 'Количество заказов', cust.FirstName AS 'Имя', cust.LastName AS 'Фамилия'
FROM dbo.[Order] ord
INNER JOIN dbo.[Customer] cust ON cust.Id = ord.CustomerId
GROUP BY ord.CustomerId, cust.LastName, cust.FirstName
ORDER BY cust.LastName, cust.FirstName;

6)
SELECT DISTINCT cust.FirstName AS 'Имя', cust.LastName AS 'Фамилия', prod.ProductName AS 'Название продукта', SUM(prod.UnitPrice) AS 'Общая стоимсоть'
FROM dbo.[Customer] cust
LEFT JOIN dbo.[Order] ord ON cust.Id = ord.CustomerId
LEFT JOIN dbo.[OrderItem] ordItem ON ord.Id = ordItem.OrderId
LEFT JOIN dbo.[Product] prod ON prod.Id = ordItem.ProductId
GROUP BY cust.LastName, cust.FirstName, prod.ProductName
ORDER BY cust.LastName, cust.FirstName, SUM(prod.UnitPrice) DESC;

7)
SELECT DISTINCT sup.CompanyName, sup.City, sup.Country, productSupplier.ProductName
FROM dbo.[Supplier] sup
INNER JOIN(
SELECT DISTINCT ProductName, SupplierId
FROM dbo.[Product]
GROUP BY SupplierId, ProductName
HAVING COUNT(SupplierId) = 1
) AS productSupplier ON sup.Id = productSupplier.SupplierId
ORDER BY productSupplier.ProductName ASC;

8)
USE SalesDB;

SELECT DISTINCT prod.ProductName, prod.SupplierId, prod.UnitPrice, prod.Package, SUM(oi.Quantity) AS 'Количество', SUM(prod.UnitPrice * oi.Quantity) AS 'Общая стоимость'
FROM dbo.Product prod
   INNER JOIN dbo.[OrderItem] oi ON prod.Id = oi.ProductId
GROUP BY prod.ProductName, prod.SupplierId, prod.UnitPrice, prod.Package, oi.ProductId
ORDER BY 'Общая стоимость';

Представления:
1)	
CREATE VIEW [dbo].[ViewSupplierInfo]
AS
SELECT sup.CompanyName, sup.ContactName, sup.Country, sup.City,
       OI.ProductId, OI.UnitPrice, OI.Quantity,
       P.SupplierId, P.ProductName
FROM dbo.Supplier sup
    INNER JOIN Product P ON sup.Id = P.SupplierId
    INNER JOIN OrderItem OI ON P.Id = OI.ProductId;

select CompanyName, City, Country, ProductName, Quantity
from dbo.ViewSupplierInfo

2)	
CREATE VIEW [dbo].[ViewOrderInfo]
AS
SELECT o.OrderDate, o.OrderNumber, o.TotalAmount,
	 p.ProductName, p.Package, p.UnitPrice,
	 oi.Quantity,
	  sup.CompanyName, sup.ContactName, sup.City, sup.Country,sup.Phone
from dbo.Supplier sup
inner join dbo.Product p on sup.Id = p.SupplierId
inner join dbo.OrderItem oi on oi.ProductId = p.Id
inner join dbo.[Order] o on o.Id = oi.OrderId;

select ProductName, sum(Quantity), sum(Quantity * UnitPrice), CompanyName, ContactName, City, Country, Phone
from dbo.ViewOrderInfo
group by ProductName, CompanyName, ContactName, City, Country, Phone;
