---
title: 驅動程式和工具相容性-適用於 MariaDB 的 Azure 資料庫
description: 本文說明與適用於 MariaDB 的 Azure 資料庫相容的適用于 mariadb 驅動程式和管理工具。
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.topic: conceptual
ms.date: 12/02/2019
ms.openlocfilehash: daec0aaf04cae26b6467cc4472305e75517cee5d
ms.sourcegitcommit: 6bb98654e97d213c549b23ebb161bda4468a1997
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/03/2019
ms.locfileid: "74772982"
---
# <a name="mariadb-drivers-and-management-tools-compatible-with-azure-database-for-mariadb"></a>與適用於 MariaDB 的 Azure 資料庫相容的適用于 mariadb 驅動程式與管理工具

本文說明與適用於 MariaDB 的 Azure 資料庫相容的驅動程式和管理工具。

## <a name="mariadb-drivers"></a>適用于 mariadb 驅動程式

適用於 MariaDB 的 Azure 資料庫使用適用于 mariadb 伺服器的「社區」版本。 因此，此版本與多種程式設計語言和驅動程式相容。 適用于 mariadb 的 API 和通訊協定與 MySQL 所使用的應用程式開發介面相容。 這表示使用 MySQL 的連接器也應該使用適用于 mariadb。

其目標是要支援適用于 mariadb 驅動程式的三個最新版本，並與開放原始碼社區的作者進行工作，以持續改善適用于 mariadb 驅動程式的功能和可用性。 下表提供已經過測試且發現與適用於 MariaDB 的 Azure 資料庫10.2 相容的驅動程式清單：

**驅動程式** | **連結** | **相容的版本** | **不相容的版本** | **注意事項**
---|---|---|---|---
PHP | https://secure.php.net/downloads.php | 5.5、5.6、7.x | 5.3 | 若要連接 PHP 7.0 與 SSL MySQLi，請在連接字串中加入 MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT。 <br> ```mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306, NULL, MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT);```<br> PDO 組：```PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT``` 選項為 false。
.NET | [ GitHub 上的 MySqlConnector](https://github.com/mysql-net/MySqlConnector) \(英文\) <br> [來自 Nuget 的安裝套件](https://www.nuget.org/packages/MySqlConnector/) \(英文\) | 0.27 及更新版本 | 0.26.5 及更舊版本 |
MySQL Connector/NET | [MySQL Connector/NET](https://github.com/mysql/mysql-connector-net) | 8.0、7.0、6.10 |  | 編碼錯誤 (bug) 可能會導致在某些非 UTF8 Windows 系統上的連線失敗。
Node.js |  [GitHub 上的 MySQLjs](https://github.com/mysqljs/mysql/) \(英文\) <br> 來自 NPM 的安裝套件：<br> 從 NPM 執行 `npm install mysql` | 2.15 | 2.14.1 及更舊版本
GO | https://github.com/go-sql-driver/mysql/releases | 1.3、1。4 | 1.2 及更舊版本 | 在1.3 版的連接字串中使用 `allowNativePasswords=true`。 1\.4 版包含修正程式，不再需要 `allowNativePasswords=true`。
Python | https://pypi.python.org/pypi/mysql-connector-python | 1.2.3、2.0、2.1、2.2 | 1.2.2 及更舊版本 |
Java | https://downloads.mariadb.org/connector-java/ | 2.1、2.0、1.6 | 1.5.5 及更舊版本 |

## <a name="management-tools"></a>管理工具

相容性的優點也會延伸到資料庫管理工具。 只要資料庫操作在使用者權限的範圍內運作，您現有的工具就應該繼續使用適用於 MariaDB 的 Azure 資料庫。 下表列出已經過測試且發現與適用於 MariaDB 的 Azure 資料庫10.2 相容的三種常見資料庫管理工具：

| | **MySQL Workbench 6.x 與更新版本** | **Navicat 12** | **PHPMyAdmin 4.x 與更新版本**
---|---|---|---
建立、更新、讀取、寫入、刪除 | X | X | X
SSL 連線 | X | X | X
SQL 查詢自動完成 | X | X |
匯入和匯出資料 | X | X | X
匯出成多種格式 | X | X | X
備份及還原 |  | X |
顯示伺服器參數 | X | X | X
顯示用戶端連線 | X | X | X

## <a name="next-steps"></a>後續步驟

- [針對適用於 MariaDB 的 Azure 資料庫的連線問題進行疑難排解](howto-troubleshoot-common-connection-issues.md)