---
title: 使用具體化視圖進行效能微調
description: 當您使用具體化視圖來改善查詢效能時，應該知道的建議和考慮。
services: sql-data-warehouse
author: XiaoyuMSFT
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: development
ms.date: 09/05/2019
ms.author: xiaoyul
ms.reviewer: nibruno; jrasnick
ms.custom: seo-lt-2019
ms.openlocfilehash: c1cfd3b4c365a04c3d4704f37e4ed4177fa74619
ms.sourcegitcommit: 609d4bdb0467fd0af40e14a86eb40b9d03669ea1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/06/2019
ms.locfileid: "73692978"
---
# <a name="performance-tuning-with-materialized-views"></a>使用具體化視圖進行效能微調 
Azure SQL 資料倉儲中的具體化視圖可針對複雜的分析查詢提供較低的維護方法，以在沒有任何查詢變更的情況下取得快速效能。 本文討論使用具體化視圖的一般指引。


## <a name="materialized-views-vs-standard-views"></a>具體化視圖與標準視圖的比較
Azure SQL 資料倉儲支援標準和具體化的視圖。  兩者都是使用 SELECT 運算式建立的虛擬資料表，並以邏輯資料表形式呈現給查詢。  Views 會封裝一般資料計算的複雜性，並將抽象層新增至計算變更，因此不需要重寫查詢。  

每次使用 view 時，標準視圖都會計算其資料。  磁片上不會儲存任何資料。 人們通常會使用標準視圖作為工具，以協助組織資料庫中的邏輯物件和查詢。  若要使用標準視圖，查詢必須直接參考它。 

具體化視圖會預先計算、儲存和維護其在 Azure SQL 資料倉儲中的資料，就像資料表一樣。  每次使用具體化視圖時，都不需要重新計算。  這就是為什麼在具體化視圖中使用全部或子集資料的查詢，可能會得到更快的效能。  更棒的是，查詢可以使用具體化視圖，而不需要直接參考它，因此不需要變更應用程式程式碼。  

標準視圖的大部分需求仍然適用于具體化視圖。 如需具體化視圖語法和其他需求的詳細資訊，請參閱[CREATE 具體化 view AS SELECT](https://docs.microsoft.com/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?view=azure-sqldw-latest)。



| 比較                     | 檢視                                         | 具體化檢視模式             
|:-------------------------------|:---------------------------------------------|:--------------------------------------------------------------| 
|檢視定義                 | 儲存在 Azure 資料倉儲中。              | 儲存在 Azure 資料倉儲中。    
|視圖內容                    | 每次使用 view 時就會產生。   | 在建立視圖期間，預先處理並儲存在 Azure 資料倉儲中。 已更新，因為資料已加入基礎資料表中。                                             
|資料重新整理                    | 一律更新                               | 一律更新                          
|從複雜查詢取得視圖資料的速度     | 一些                                         | 快速  
|額外儲存空間                   | 否                                           | 是                             
|語法                          | 建立視圖                                  | 建立具體化視圖做為選取           
     
## <a name="benefits-of-using-materialized-views"></a>使用具體化 views 的優點

設計正確的具體化視圖可以提供下列優點：

- 使用聯結和彙總函式，縮短複雜查詢的執行時間。 查詢越複雜，執行時間儲存的可能性就越高。 當查詢的計算成本很高，而產生的資料集很小時，就會獲得最大效益。  

- Azure SQL 資料倉儲中的優化工具可以自動使用已部署的具體化視圖來改善查詢執行計畫。  這個程式對於提供較快速查詢效能的使用者而言是透明的，不需要查詢就能直接參考具體化的視圖。 

- 需要在 views 上進行低維護。  基表的所有累加資料變更都會以同步方式自動新增至具體化 views。  這種設計可讓查詢具體化視圖傳回與直接查詢基表相同的資料。 
- 具體化視圖中的資料可以用不同于基表的方式散發。  
- 具體化視圖中的資料會與一般資料表中的資料取得相同的高可用性和復原優勢。  
 
相較于其他資料倉儲提供者，在 Azure SQL 資料倉儲中執行的具體化視圖也提供下列額外的優點： 

- 資料表中的資料變更，自動和同步資料重新整理。 因此，使用者不需要採取任何動作。 
- 廣泛的彙總函式支援。 請參閱[CREATE 具體化 VIEW AS SELECT （transact-sql）](https://docs.microsoft.com/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?view=azure-sqldw-latest)。
- 支援查詢特定的具體化視圖建議。  請參閱[說明（transact-sql）](https://docs.microsoft.com/sql/t-sql/queries/explain-transact-sql?view=azure-sqldw-latest)。

## <a name="common-scenarios"></a>常見案例  

在下列案例中，通常會使用具體化視圖： 

**需要針對大小龐大的資料，改善複雜分析查詢的效能**

複雜的分析查詢通常會使用更多的彙總函式和資料表聯結，導致在查詢執行中進行更大量的計算作業，例如洗牌和聯結。  這就是為什麼這些查詢需要較長的時間才能完成，特別是在大型資料表上。  使用者可以針對從一般查詢計算傳回的資料建立具體化的視圖，因此當查詢需要此資料時，不需要進行任何重新計算，這可讓計算成本較低，而且查詢回應更快。 

**無需或最少查詢變更，需要更快的效能**

資料倉儲中的架構和查詢變更通常會保持最低，以支援一般的 ETL 作業和報告。  如果視圖所產生的成本可能會因查詢效能的提升而位移，人員可以使用具體化視圖進行查詢效能微調。 相較于其他調整選項（例如縮放和統計資料管理），建立和維護具體化視圖的影響力生產變更比較少，而且其潛在效能提升也會更高。

- 建立或維護具體化視圖並不會影響對基表執行的查詢。
- 查詢最佳化工具可以自動使用已部署的具體化視圖，而不需要查詢中的直接 view 參考。 這項功能可減少對效能調整進行查詢變更的需求。 

**需要不同的資料散發策略，以提供更快的查詢效能**

Azure 資料倉儲是分散式大量平行處理（MPP）系統。   資料倉儲資料表中的資料會使用三種[散發策略](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute)（雜湊、round_robin 或複寫）中的其中一種散發到60節點。  資料散發是在資料表建立時指定的，而且會維持不變，直到卸載資料表為止。 具體化 view 是磁片上的虛擬資料表支援雜湊和 round_robin 資料散發。  使用者可以選擇與基表不同的資料散發，但最佳的是使用最多視圖之查詢的效能。  

## <a name="design-guidance"></a>設計指引 

以下是使用具體化視圖來改善查詢效能的一般指引：

**針對您的工作負載設計**

在您開始建立具體化視圖之前，請務必先深入瞭解查詢模式、重要性、頻率和產生的資料大小等方面的工作負載。  

使用者可以針對查詢最佳化工具建議的具體化視圖，執行說明 WITH_RECOMMENDATIONS < SQL_statement >。  由於這些建議是查詢專屬的，因此，針對相同工作負載中的其他查詢，受益于單一查詢的具體化視圖可能不是最理想的。  請考慮您的工作負載需求來評估這些建議。  理想的具體化視圖是可受益于工作負載效能的瀏覽器。  

**請留意速度較快的查詢與成本之間的取捨** 

針對每個具體化視圖，會有資料儲存成本，以及維護此視圖的成本。  當基表中的資料變更時，具體化視圖的大小會增加，而且其實體結構也會變更。  為了避免查詢效能降低，每個具體化的視圖都會由資料倉儲引擎分別維護。  當具體化視圖和基表變更的數目增加時，維護工作負載會變高。   使用者應該檢查所有具體化視圖所產生的成本是否可以由查詢效能提升來位移。  

您可以針對資料庫中的具體化視圖清單執行此查詢： 

```sql
SELECT V.name as materialized_view, V.object_id 
FROM sys.views V 
JOIN sys.indexes I ON V.object_id= I.object_id AND I.index_id < 2;
``` 

減少具體化視圖數的選項： 

- 識別工作負載中複雜查詢經常使用的一般資料集。  建立具體化視圖以儲存這些資料集，讓優化工具可以在建立執行計畫時，將它們當做建立區塊使用。  

- 捨棄具有低使用量或不再需要的具體化視圖。  已停用的具體化視圖不會維護，但仍會產生儲存體成本。  

- 結合在相同或相似基表上建立的具體化 views，即使其資料不重迭也一樣。  合併具體化視圖的大小可能會比個別視圖的總和大得多，不過，視圖維護成本應該會減少。  例如：

```sql

-- Query 1 would benefit from having a materialized view created with this SELECT statement

SELECT A, SUM(B)
FROM T
GROUP BY A

-- Query 2 would benefit from having a materialized view created with this SELECT statement

SELECT C, SUM(D)
FROM T
GROUP BY C

-- You could create a single mateiralized view of this form

SELECT A, C, SUM(B), SUM(D)
FROM T
GROUP BY A, C

```

**並非所有效能微調都需要查詢變更**

資料倉儲優化工具可以自動使用已部署的具體化視圖來改善查詢效能。  這項支援是以透明的方式套用至查詢，而不會參考在建立具體化視圖時不支援的匯總視圖和查詢。  不需要變更任何查詢。 您可以檢查查詢的估計執行計畫，以確認是否使用具體化視圖。  

**監視具體化視圖** 

具體化視圖會儲存在資料倉儲中，就像是具有叢集資料行存放區索引（CCI）的資料表一樣。  從具體化視圖讀取資料包括掃描 CCI 索引區段，以及套用基表的任何累加式變更。 當累加變更的數目過高時，從具體化視圖解析查詢可能需要比直接查詢基表更長的時間。  為了避免查詢效能降低，最好是執行[DBCC PDW_SHOWMATERIALIZEDVIEWOVERHEAD](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-pdw-showmaterializedviewoverhead-transact-sql?view=azure-sqldw-latest)來監視視圖的 overhead_ratio （total_rows/max （1，base_view_row））。  如果使用者的 overhead_ratio 太高，則應該重建具體化視圖。 

**具體化視圖和結果集快取**

這兩項功能會在 Azure SQL 資料倉儲中引進，以進行查詢效能微調。  結果集快取是用來針對靜態資料，從重複的查詢取得高並行性和快速回應。  若要使用快取的結果，要求查詢的快取形式必須符合產生快取的查詢。  此外，快取的結果必須套用至整個查詢。  具體化 views 允許基表中的資料變更。  具體化視圖中的資料可以套用至查詢的某個片段。  這項支援可讓共用一些計算的不同查詢使用相同的具體化視圖，以提高效能。

## <a name="example"></a>範例

這個範例會使用類似 TPCDS 的查詢，尋找在目錄中花費更多成本的客戶，而不是在商店中，識別慣用的客戶及其來源國家/地區。   查詢牽涉到從包含 SUM （）和 GROUP BY 的三個子 SELECT 語句的聯集內選取前100筆記錄。 

```sql
WITH year_total AS (
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
       ,'s' sale_type
FROM customer
     ,store_sales 
     ,date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
       ,'c' sale_type
FROM customer
     ,catalog_sales 
     ,date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
       ,'w' sale_type
FROM customer
     ,web_sales 
     ,date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
         )
  SELECT TOP 100 
                  t_s_secyear.customer_id
                 ,t_s_secyear.customer_first_name
                 ,t_s_secyear.customer_last_name
                 ,t_s_secyear.customer_birth_country
FROM year_total t_s_firstyear
     ,year_total t_s_secyear
     ,year_total t_c_firstyear
     ,year_total t_c_secyear
     ,year_total t_w_firstyear
     ,year_total t_w_secyear
WHERE t_s_secyear.customer_id = t_s_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_c_secyear.customer_id
   AND t_s_firstyear.customer_id = t_c_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_secyear.customer_id
   AND t_s_firstyear.sale_type = 's'
   AND t_c_firstyear.sale_type = 'c'
   AND t_w_firstyear.sale_type = 'w'
   AND t_s_secyear.sale_type = 's'
   AND t_c_secyear.sale_type = 'c'
   AND t_w_secyear.sale_type = 'w'
   AND t_s_firstyear.dyear+0 =  1999
   AND t_s_secyear.dyear+0 = 1999+1
   AND t_c_firstyear.dyear+0 =  1999
   AND t_c_secyear.dyear+0 =  1999+1
   AND t_w_firstyear.dyear+0 = 1999
   AND t_w_secyear.dyear+0 = 1999+1
   AND t_s_firstyear.year_total > 0
   AND t_c_firstyear.year_total > 0
   AND t_w_firstyear.year_total > 0
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_s_firstyear.year_total > 0 THEN t_s_secyear.year_total / t_s_firstyear.year_total ELSE NULL END
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_w_firstyear.year_total > 0 THEN t_w_secyear.year_total / t_w_firstyear.year_total ELSE NULL END
ORDER BY t_s_secyear.customer_id
         ,t_s_secyear.customer_first_name
         ,t_s_secyear.customer_last_name
         ,t_s_secyear.customer_birth_country
OPTION ( LABEL = 'Query04-af359846-253-3');
```

檢查查詢的估計執行計畫。  有18個洗牌和17個聯結作業，這需要更多時間來執行。 現在，讓我們為三個子 SELECT 語句的每個建立一個具體化視圖。   

```sql
CREATE materialized view nbViewSS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.store_sales
     ,dbo.date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
GO
CREATE materialized view nbViewCS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
          , count_big(*) as cb
FROM dbo.customer
     ,dbo.catalog_sales
     ,dbo.date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

GO
CREATE materialized view nbViewWS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.web_sales
     ,dbo.date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

```
再次檢查原始查詢的執行計畫。  現在，聯結的數目會從17變更為5，且不會再播放。  按一下計畫中的篩選作業圖示，其 [輸出] 清單會顯示從具體化視圖中讀取的資料，而不是基表。  

 ![Plan_Output_List_with_Materialized_Views](media/performance-tuning-materialized-views/output-list.png)

使用具體化視圖，相同的查詢執行速度會更快，而且不會變更任何程式碼。  

## <a name="next-steps"></a>後續步驟
如需更多開發秘訣，請參閱 [SQL 資料倉儲開發概觀](sql-data-warehouse-overview-develop.md)。
