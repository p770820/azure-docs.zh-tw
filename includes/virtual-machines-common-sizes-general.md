---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 08/08/2019
ms.author: azcspmt;jonbeck;cynthn;joelpell
ms.custom: include file
ms.openlocfilehash: 3fc20288f4ec80c85bd0109799d5ed45b504d359
ms.sourcegitcommit: b8f2fee3b93436c44f021dff7abe28921da72a6d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77430043"
---
一般用途的虛擬機器大小可讓 CPU 與記憶體比例達到平均。 適用於測試和開發、小型至中型資料庫，以及低至中流量 Web 伺服器。 本文提供 vCPU 數量、資料磁碟和 NIC 的相關資訊，以及此群組中各種大小之儲存體輸送量的相關資訊。

- [DC 系列](#dc-series)是 Azure 中的一系列虛擬機器，可在公用雲端中進行處理時，協助保護您的資料和程式碼的機密性和完整性。 這些機器由最新一代的 3.7GHz Intel XEON E-2176G 處理器 (具 SGX 技術) 支援。 在 Intel Turbo Boost 技術加持下，這些機器最高可達到 4.7GHz 的時脈。 DC 系列執行個體可讓客戶建置安全的飛地型應用程式，以在其程式碼與資料使用期間保護其安全。

- Av2 系列 VM 可以部署在各種不同的硬體類型和處理器上。 A 系列 VM 的 CPU 效能及記憶體設定最適合初階的工作負載，例如開發及測試， 根據硬體節流大小，為執行中的執行個體提供一致的處理器效能，不論硬體部署的位置。 若要判斷此大小部署所在的實體硬體，請從虛擬機器內查詢虛擬硬體。

  使用案例範例包括開發與測試伺服器、低流量網頁伺服器、小型至中型資料庫、概念證明和程式碼存放庫。

- Dv2 系列是原始 D 系列的升級版，搭載更強大的 CPU 及最佳的 CPU 記憶體設定，更適合大多數生產工作負載。 Dv2 系列的速度比 D 系列快35%。 Dv2 系列執行于 Intel®的® 8171M 2.1 GHz （Skylake）、Intel®的® E5-2673 v4 2.3 GHz （Broadwell），或 Intel®的® E5-2673 v3 2.4 g h z （Haswell）處理器與 Intel Turbo 加速技術2.0。 Dv2 系列的記憶體和磁碟組態和 D 系列一樣。

- Dv3 系列執行于 Intel®的® 8171M 2.1 GHz （Skylake）、Intel®的® E5-2673 v4 2.3 g h z （Broadwell），或超執行緒設定中的 Intel®（r）® E5-2673 v3 2.4 g h z （Haswell）處理器，為最常見的提供更好的價值主張目的工作負載。  除了記憶體已擴充 (從 ~3.5 GiB/vCPU 到 4 GiB/vCPU)，磁碟和網路限制也已就個別核心進行調整，以符合移轉至超執行緒的需求。  Dv3 系列不再具有 D/Dv2 系列的高記憶體 VM 大小，而是已移至適用于[Windows](https://docs.microsoft.com/azure/virtual-machines/windows/sizes-memory#ev3-series)和[Linux](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory#ev3-series)的記憶體優化 Ev3 系列。

  D 系列使用案例的範例包括企業級應用程式、關係資料庫、記憶體內部快取及分析。

- Dav4 系列和 Dasv4 系列是利用 AMD 的2.35 版 Ghz EPYC<sup>TM</sup> 7452 處理器的新大小，在多執行緒設定中，最多256可將 8 Mb 的 l3 快取專門提供給每8個核心，增加客戶選擇來執行其一般用途的工作負載。 Dav4 系列和 Dasv4 系列具有與 D & Dsv3 系列相同的記憶體和磁片設定。
  
## <a name="b-series"></a>B 系列

進階儲存體：支援

進階儲存體快取：不支援

B 系列高載的 VM 非常適合不需要持續性完整 CPU 效能的工作負載，例如 Web 伺服器、小型資料庫和開發與測試環境。 這些工作負載通常具有高載的效能需求。 B 系列讓這些客戶所購買到的 VM 大小具有價格公道的基準效能，可在 VM 利用低於其基底效能時，讓 VM 執行個體累積點數。 當 VM 累積點數時，VM 可以在您的應用程式需要較高的 CPU 效能時，使用最多 100% 的 CPU 來高載高於 VM 基準。

範例使用案例包括開發與測試伺服器、低流量網頁伺服器、小型資料庫、微服務、用於概念證明的伺服器、組建伺服器。


| 大小             | vCPU  | 記憶體：GiB | 暫存儲存體 (SSD) GiB | VM 的基礎 CPU 效能 | VM 的最大 CPU 效能 | 初始信用額度 | 點數累積 / 小時 | 最大累積點數 | 最大資料磁碟 | 最大快取和暫存儲存體輸送量： IOPS/MBps | 最大取消快取的磁碟輸送量︰IOPS / MBps | 最大 NIC |          
|---------------|-------------|----------------|----------------------------|-----------------------|--------------------|--------------------|--------------------|----------------|----------------------------------------|-------------------------------------------|-------------------------------------------|----------|
| Standard_B1ls<sup>1</sup>  | 1           | 0.5              | 4                          | 5%                   | 100%                   | 30                   | 3                  | 72            | 2                                      | 200/10                                  | 160/10                                  | 2  |
| Standard_B1s  | 1           | 1              | 4                          | 10%                   | 100%                   | 30                   | 6                  | 144            | 2                        | 400 / 10                                  | 320 / 10                                  | 2  |
| Standard_B1ms | 1           | 2              | 4                          | 20%                   | 100%                   | 30                   | 12                 | 288           | 2                         | 800 / 10                                  | 640 / 10                                  | 2  |
| Standard_B2s  | 2           | 4              | 8                          | 40%                   | 200%                   | 60                   | 24                 | 576            | 4                                      | 1600 / 15                                 | 1280 / 15                                 | 3  |
| Standard_B2ms | 2           | 8              | 16                         | 60%                   | 200%                   | 60                   | 36                 | 864            | 4                                      | 2400 / 22.5                               | 1920 / 22.5                               | 3  |
| Standard_B4ms | 4           | 16             | 32                         | 90%                   | 400%                   | 120                   | 54                 | 1296           | 8                                      | 3600 / 35                                 | 2880 / 35                                 | 4  |
| Standard_B8ms | 8           | 32             | 64                         | 135%                  | 800%                   | 240                   | 81                 | 1944           | 16                                     | 4320 / 50                                 | 4320 / 50                                 | 4  |
| Standard_B12ms | 12           | 48             | 96                         | 202%                  | 1200%                   | 360                   | 121                 | 2909           | 16                                     | 6480/75                                 | 4320 / 50                                 | 6  |
| Standard_B16ms | 16           | 64             | 128                         | 270%                  | 1600%                   | 480                   | 162                 | 3888           | 32                                     | 8640/100                                 | 4320 / 50                                 | 8  |
| Standard_B20ms | 20           | 80             | 160                         | 337%                  | 2000%                   | 600                   | 203                 | 4860           | 32                                     | 10800/125                                 | 4320 / 50                                 | 8  |

<sup>1</sup>只有在 Linux 上才支援 B1ls

## <a name="dsv3-series-sup1sup"></a>Dsv3 系列 <sup>1</sup>

ACU：160-190

進階儲存體：支援

進階儲存體快取：支援

Dsv3 系列大小執行于 Intel®的® 8171M 2.1 GHz （Skylake）、Intel®的® E5-2673 v4 2.3 GHz （Broadwell），或 Intel®的® E5-2673 v3 2.4 g h z （Haswell）處理器，搭配 Intel Turbo 加速技術2.0 並使用 premium storage。 Dsv3 系列大小提供 CPU、記憶體與暫存儲存憶體組合，適用於大多數生產環境工作負載。


| 大小             | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大快取和暫存儲存體輸送量︰IOPS / MBps (以 GiB 為單位的快取大小) | 最大取消快取的磁碟輸送量︰IOPS / MBps | 最大 NIC/預期的網路頻寬 (Mbps) |
|------------------|--------|-------------|----------------|----------------|-----------------------------------------------------------------------|-------------------------------------------|------------------------------------------------|
| Standard_D2s_v3  | 2      | 8           | 16             | 4              | 4000/32 （50）                                                       | 3200/48                                | 2 / 1000                                   |
| Standard_D4s_v3  | 4      | 16          | 32             | 8              | 8000/64 （100）                                                      | 6400/96                                | 2 / 2000                                   |
| Standard_D8s_v3  | 8      | 32          | 64             | 16             | 16000/128 （200）                                                    | 12800/192                              | 4 / 4000                                      |
| Standard_D16s_v3 | 16     | 64          | 128            | 32             | 32000/256 （400）                                                    | 25600/384                              | 8 / 8000                                      |
| Standard_D32s_v3 | 32     | 128          | 256            | 32             | 64000/512 （800）                                                    | 51200/768                              | 8 / 16000                                               |
| Standard_D48s_v3 | 48     | 192          | 384            | 32             | 96000/768 （1200）                                                    | 76800/1152                               | 8 / 24000                                               |
| Standard_D64s_v3 | 64     | 256          | 512            | 32             | 128000/1024 （1600）                                                    | 80000/1200                              | 8 / 30000                                               |

<sup>1</sup> Dsv3 系列 VM 的功能 Intel® 超執行緒技術

## <a name="dasv4-series"></a>Dasv4 系列

ACU：230-260

進階儲存體：支援

進階儲存體快取：支援

Dasv4 系列大小以2.35 版 Ghz AMD EPYC<sup>TM</sup> 7452 處理器為基礎，可達成 3.35 ghz 的提升頻率，並使用 premium SSD。 Dasv4 系列大小提供 vCPU、記憶體和暫存儲存體的組合，可用於大部分的生產環境工作負載。

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大快取和暫存儲存體輸送量︰IOPS / MBps (以 GiB 為單位的快取大小) | 最大取消快取的磁碟輸送量︰IOPS / MBps | 最大 Nic/預期的網路頻寬（MBps） |
|-----|-----|-----|-----|-----|-----|-----|-----|
| Standard_D2as_v4|2|8|16|4|4000/32 （50）|3200/48|2 / 1000 |
| Standard_D4as_v4|4|16|32|8|8000/64 （100）|6400/96|2 / 2000 |
| Standard_D8as_v4|8|32|64|16|16000/128 （200）|12800/192|4 / 4000 |
| Standard_D16as_v4|16|64|128|32|32000/255 （400）|25600/384|8 / 8000 |
| Standard_D32as_v4|32|128|256|32|64000/510 （800）|51200/768|8 / 16000 |
| Standard_D48as_v4 <sup>**</sup>|48|192|384|32| | | 
| Standard_D64as_v4 <sup>**</sup>|64|256|512|32| | | 
| Standard_D96as_v4 <sup>**</sup>|96|384|768|32| | | 

<sup>**</sup>這些大小目前為預覽狀態。  如果您想要試用這些較大的大小，請在[https://aka.ms/AzureAMDLargeVMPreview](https://aka.ms/AzureAMDLargeVMPreview)註冊。

## <a name="dv3-series-sup1sup"></a>Dv3 系列 <sup>1</sup>

ACU：160-190

進階儲存體：不支援

進階儲存體快取：不支援

Dv3 系列大小執行于 Intel®的® 8171M 2.1 GHz （Skylake）、Intel®的® E5-2673 v4 2.3 GHz （Broadwell），或 Intel®的® E5-2673 v3 2.4 g h z （Haswell）處理器與 Intel Turbo 加速技術2.0。 Dv3 系列大小提供 CPU、記憶體與暫存儲存體組合，適用於大多數生產環境工作負載。

資料磁碟儲存體與虛擬機器分開計費。 若要使用進階儲存體磁碟，請使用 Dsv3 大小。 Dsv3 大小的定價和計費方式與 Dv3 系列相同。 


| 大小            | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大暫存儲存體輸送量：IOPS / 讀取 MBps / 寫入 MBps | 最大 Nic/網路頻寬（Mbps） |
|-----------------|-----------|-------------|----------------|----------------|----------------------------------------------------------|------------------------------|
| Standard_D2_v3  | 2         | 8           | 50             | 4              | 3000/46/23                                               | 2 / 1000                    |
| Standard_D4_v3  | 4         | 16          | 100            | 8              | 6000/93/46                                               | 2 / 2000                    |
| Standard_D8_v3  | 8         | 32          | 200            | 16             | 12000/187/93                                             | 4 / 4000                    |
| Standard_D16_v3 | 16        | 64          | 400            | 32             | 24000/375/187                                            | 8 / 8000                    |
| Standard_D32_v3 | 32        | 128         | 800            | 32             | 48000/750/375                                            | 8 / 16000                   |
| Standard_D48_v3 | 48        | 192          | 1200            | 32             | 96000/1000/500                                            | 8 / 24000                             |
| Standard_D64_v3 | 64        | 256         | 1600           | 32             | 96000/1000/500                                           | 8 / 30000                   |

<sup>1</sup> Dv3 系列 VM 的功能 Intel® 超執行緒技術

## <a name="dav4-series"></a>Dav4 系列

ACU：230-260

進階儲存體：不支援

進階儲存體快取：不支援

Dav4 系列大小以2.35 版 Ghz AMD EPYC<sup>TM</sup> 7452 處理器為基礎，可達到最高的 3.35 ghz 的提升頻率。 Dav4 系列大小提供 vCPU、記憶體和暫存儲存體的組合，可用於大部分的生產環境工作負載。 資料磁碟儲存體與虛擬機器分開計費。 若要使用 premium SSD，請使用 Dasv4 大小。 Dasv4 大小的定價和計費方式與 Dav4 系列相同。

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大暫存儲存體輸送量：IOPS / 讀取 MBps / 寫入 MBps | 最大 Nic/預期的網路頻寬（MBps） |
|-----|-----|-----|-----|-----|-----|-----|
| Standard_D2a_v4 |  2  | 8  | 50  | 4  | 3000 / 46 / 23   | 2 / 1000 |
| Standard_D4a_v4 |  4  | 16 | 100 | 8  | 6000 / 93 / 46   | 2 / 2000 |
| Standard_D8a_v4 |  8  | 32 | 200 | 16 | 12000 / 187 / 93 | 4 / 4000 |
| Standard_D16a_v4|  16 | 64 | 400 |32  | 24000 / 375 / 187 |8 / 8000 |
| Standard_D32a_v4|  32 | 128| 800 | 32 | 48000 / 750 / 375 |8 / 16000 |
| Standard_D48a_v4 <sup>**</sup> | 48 | 192| 1200 | 32 | | |
| Standard_D64a_v4 <sup>**</sup> | 64 | 256 | 1600 | 32 | | |
| Standard_D96a_v4 <sup>**</sup> | 96 | 384 | 2400 | 32 | | |

<sup>**</sup>這些大小目前為預覽狀態。  如果您想要試用這些較大的大小，請在[https://aka.ms/AzureAMDLargeVMPreview](https://aka.ms/AzureAMDLargeVMPreview)註冊。

## <a name="dsv2-series"></a>DSv2 系列

ACU：210 - 250

進階儲存體：支援

進階儲存體快取：支援

DSv2 系列大小執行于 Intel®的® 8171M 2.1 GHz （Skylake）或 Intel®的® E5-2673 v4 2.3 g h z （Broadwell）或 Intel®的® E5-2673 v3 2.4 g h z （Haswell）處理器，搭配 Intel Turbo 加速技術2.0 並使用 premium 儲存體。

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大快取和暫存儲存體輸送量︰IOPS / MBps (以 GiB 為單位的快取大小) | 最大取消快取的磁碟輸送量︰IOPS / MBps | 最大 NIC/預期的網路頻寬 (Mbps) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_DS1_v2 |1 |3.5 |7 |4 |4000 / 32 (43) |3200/48 |2 / 750 |
| Standard_DS2_v2 |2 |7 |14 |8 |8000 / 64 (86) |6400/96 |2 / 1500 |
| Standard_DS3_v2 |4 |14 |28 |16 |16000/128 （172） |12800/192 |4 / 3000 |
| Standard_DS4_v2 |8 |28 |56 |32 |32000/256 （344） |25600/384 |8 / 6000 |
| Standard_DS5_v2 |16 |56 |112 |64 |64000/512 （688） |51200/768 |8 / 12000 |

## <a name="dv2-series"></a>Dv2 系列

ACU：210 - 250

進階儲存體：不支援

進階儲存體快取：不支援

DSv2 系列大小執行于 Intel®的® 8171M 2.1 GHz （Skylake）或 Intel®的® E5-2673 v4 2.3 GHz （Broadwell）或 Intel®的® E5-2673 v3 2.4 g h z （Haswell）處理器，搭配 Intel Turbo 加速技術2.0。

| 大小           | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大暫存儲存體輸送量：IOPS / 讀取 MBps / 寫入 MBps | 最大資料磁碟 | 輸送量：IOPS | 最大 NIC/預期的網路頻寬 (Mbps) |
|----------------|------|-------------|------------------------|------------------------------------------------------------|----------------|------------------|----------------------------------------------|
| Standard_D1_v2 | 1    | 3.5         | 50                     | 3000 / 46 / 23                                             | 4              | 4x500            | 2 / 750                                      |
| Standard_D2_v2 | 2    | 7           | 100                    | 6000 / 93 / 46                                             | 8              | 8x500            | 2 / 1500                                     |
| Standard_D3_v2 | 4    | 14          | 200                    | 12000 / 187 / 93                                           | 16             | 16x500           | 4 / 3000                                       |
| Standard_D4_v2 | 8    | 28          | 400                    | 24000 / 375 / 187                                          | 32             | 32x500           | 8 / 6000                                       |
| Standard_D5_v2 | 16   | 56          | 800                    | 48000 / 750 / 375                                          | 64             | 64x500           | 8 / 12000                                    |

## <a name="av2-series"></a>Av2 系列

ACU：100

進階儲存體：不支援

進階儲存體快取：不支援

| 大小            | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大暫存儲存體輸送量：IOPS / 讀取 MBps / 寫入 MBps | 最大資料磁碟 / 輸送量︰IOPS | 最大 NIC/預期的網路頻寬 (Mbps) | 
|-----------------|-----------|-------------|----------------|----------------------------------------------------------|-----------------------------------|------------------------------|
| Standard_A1_v2  | 1         | 2           | 10             | 1000 / 20 / 10                                           | 2 / 2x500               | 2 / 250                 |
| Standard_A2_v2  | 2         | 4           | 20             | 2000 / 40 / 20                                           | 4 / 4x500               | 2 / 500                 |
| Standard_A4_v2  | 4         | 8           | 40             | 4000 / 80 / 40                                           | 8 / 8x500               | 4 / 1000                     |
| Standard_A8_v2  | 8         | 16          | 80             | 8000 / 160 / 80                                          | 16 / 16x500             | 8 / 2000                     |
| Standard_A2m_v2 | 2         | 16          | 20             | 2000 / 40 / 20                                           | 4 / 4x500               | 2 / 500                 |
| Standard_A4m_v2 | 4         | 32          | 40             | 4000 / 80 / 40                                           | 8 / 8x500               | 4 / 1000                     |
| Standard_A8m_v2 | 8         | 64          | 80             | 8000 / 160 / 80                                          | 16 / 16x500             | 8 / 2000                     |

## <a name="dc-series"></a>DC 系列

進階儲存體：支援

進階儲存體快取：支援



| 大小          | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大快取和暫存儲存體輸送量︰IOPS / MBps (以 GiB 為單位的快取大小) | 最大取消快取的磁碟輸送量︰IOPS / MBps | 最大 NIC/預期的網路頻寬 (Mbps) |
|---------------|------|-------------|------------------------|----------------|-------------------------------------------------------------------------|-------------------------------------------|----------------------------------------------|
| Standard_DC2s | 2    | 8           | 100                    | 2              | 4000 / 32 (43)                                                          | 3200 /48                                  | 2 / 1500                                     |
| Standard_DC4s | 4    | 16          | 200                    | 4              | 8000 / 64 (86)                                                          | 6400 /96                                  | 2 / 3000                                     |
