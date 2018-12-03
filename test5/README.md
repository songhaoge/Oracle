# Oracle实验五：PL/SQL编程
                   
## 实验前提
假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。  

本实验以实验四为基础。  

## 一、第1步：创建一个包(Package)，包名是MyPack，并定义相关包头，实验SQL语句及结果如下：  
```
create or replace PACKAGE MyPack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
  FUNCTION Put_SaleAmount(V_EMPLOYEE_ID NUMBER) RETURN NUMBER;
  PROCEDURE Get_Employees(V_EMPLOYEE_ID NUMBER);
END MyPack;

Package MYPACK 已编译  
```
### 二、第2步：在MyPack中创建一个函数SaleAmount ，查询部门表，统计每个部门的销售总金额，每个部门的销售额是由该部门的员工(ORDERS.EMPLOYEE_ID)完成的销额之和。函数SaleAmount要求输入的参数是部门号，输出部门的销售金额。  

### 创建函数语句  
```
 FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER
AS
 N NUMBER(20,2);
 BEGIN
  SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O,EMPLOYEES E
  WHERE O.EMPLOYEE_ID=E.EMPLOYEE_ID AND E.DEPARTMENT_ID =V_DEPARTMENT_ID;
  RETURN N;
END;  
```
### 查询语句  
```
select count(*) from orders;
select MyPack.Get_SaleAmount(11) AS 销售部1应收金额,MyPack.Get_SaleAmount(12) AS 销售部2应收金额 from dual;  
```
查询结果如下图


## 三、第3步：在MyPack中创建一个过程，在过程中使用游标，递归查询某个员工及其所有下属，子下属员工。过程的输入参数是员工号，输出员工的ID,姓名，销售总金额。信息用dbms_output包中的put或者put_line函数。输出的员工信息用左添加空格的多少表示员工的层次（LEVEL）。比如下面显示5个员工的信息：  
***
### 首先创建一个函数Put_SaleAmount,要求得出每个员工的销售总金额，创建函数语句如下：  
```
FUNCTION Put_SaleAmount(V_EMPLOYEE_ID NUMBER) RETURN NUMBER
AS
N NUMBER(20,2);
BEGIN
  SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O
  WHERE O.EMPLOYEE_ID=V_EMPLOYEE_ID;
  RETURN N;
END;    
```
### 然后创建过程GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)，创建过程语句如下：  
```
PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)
AS
LEFTSPACE VARCHAR(2000);
begin
  --通过LEVEL判断递归的级别
  LEFTSPACE:=' ';
  --使用游标
  for v in
  (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID,Put_SaleAmount(EMPLOYEE_ID) as a FROM employees
  START WITH employees.EMPLOYEE_ID = V_EMPLOYEE_ID
  CONNECT BY PRIOR employees.EMPLOYEE_ID = MANAGER_ID
  )
  LOOP
   DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')||
                         V.EMPLOYEE_ID||' '||v.NAME||' '||v.a);
  END LOOP;
END;   
```
### 查询语句：  
```
set serveroutput on
DECLARE
V_EMPLOYEE_ID NUMBER;    
BEGIN
V_EMPLOYEE_ID := 1;
MYPACK.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;  
V_EMPLOYEE_ID := 11;
MYPACK.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;    
END;  
```
### 查询结果如下图：
image

## 四、思考：由于订单只是按日期分区的，上述统计是全表搜索，因此统计速度会比较慢，如何提高统计的速度呢？  
***
可以按日期进行分区查询统计。  

可以加上索引。  

可以进行分页查询。  

## 五、总结分析  

这一次实验了解PL/SQL语言结构和PL/SQL变量和常量的声明和使用方法。  

学会了条件语句的使用方法、分支语句的使用方法以及循环语句的使用方法  

熟悉了常用的PL/SQL函数以及包，过程，函数的用法。
