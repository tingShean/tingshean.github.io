# Google Summit 11/07

## 開場
- GSuite 介紹 & 應用
- Auto ML 介紹 & 應用
- 合作企業

## Accelerate application development 加速應用程式開發
### 打造無伺服器(Serverless)應用
- 起源
	- 希望各位把server丟掉
- 使用之前
	- 開發flow的複雜度過高
- 優點
	- 只要付費使用的部份
	- 人力、維護成本
	- 人力可以移往開發
- Compute continuum (<-Highly customizable, Highly managed->)
	- Compute Engine
	- Kubernetes Engine
	- App Engine (--Serverless--)
	- Cloud Functions
	- Firebase
- 情境 (Compute)
	- Data Processing
	- AI (ML)
	- Storage (GS)
	- Messaging (PubSub)
	- Database (Cloud SQL)
	- Data warehourse (Big Query)
- GCP serverless platform
	- Cloud Firestore
	- BigQuery
	- Cloud Dataflow(ETL)
	- Cloud Storage
	- Cloud Machine Learning
	- Cloud PubSub
	- App Engine (Serverless compute)
	- Cloud Functions (Serverless compute)
- Serverless: Build highly scalable applications
	- Minimal config deployments (gcloud app deploy)
	- Auto-scaling to support peak traffic spikes
	- Pay only while your codd runs
- Serverless 支援
	- Python 2.7, 3.7
	- PHP 5.7, 7.2
	- NodeJs
- Functions: Connect & extend your services
	- Simple
	- Auto-scaling to support any traffic workload
	- Run code in response to events
	- Pay only while your code runs
- Functions 支援
	- NodeJs 8.11 async/await
	- Python 3.7 Flask
- New
	- VPC & VPN
	- Security controls (GCP IAM)
	- Cloud SQL Direct Connect (直連)
	- Environment Variables (DevOps 環境區分)
- GCP serverless platform (最大化產值)
	- Comprehensive Serverless Ecosystem
	- Hight Developer productivity
	- Pay only for what you consume
- Demo: Auto-scaling with zero toil
	- https://hammer-strike-172817.appspot.com/advanced.html
- Demo: real-time sentiment analysis
	- https://demo.firebase.cool/

### iKala 遇見AI趨勢，掌握產業智慧化新機會
- 簡介
- Premier Patrner
	- Data Analytics
- 預測
- Game
	- 深藍
	- Alpha Go
	- GTA V
	- SC II
	- 毀滅戰士
- Robotics
	- Kiva 2014 收購
	- 無人卡車
	- 2018 無人車駕照 (不需要駕駛員)
	- 2018 Parkour (跑酷機器人)
- KOL 網紅搜尋
- Shoplus
	- 下一個AI要解決的是客服
- Google Duplex (機器人打電話)
	- pixel 可用
- Google Contact Center AI, 2018
- Legal
	- 辦識合約 (AI 94% vs 人類84%)
- Technology
- ImageNet Challenge
	- 2017 下降低於5%
	- Deep Angle (把不要的東西從照片上挖除)
- BigGan, Deep Mine
	- 自動合成照片
- Cloud + Edge
	- Edge IoT Core
- 神經網絡
	- CapsNet (new)
	- CNN
- OpenAI
	- ICM
	- RDN
- Machine Meta-Learning (讓機器自己修改自己)
- Opportunit open AI
	- Marketing and sales 1.4~2.6 trillion
- 文化的不同造就不同的AI
	
### 並非所有資料庫都一樣：如何選擇適用的資料庫
- 以資料大小區分使用

### 5分鐘內將雲端搬遷至Google Cloud - 透過Velostrata協助，快速將工作負載量遷移
- velostrata
- cloudend

### Kubernetes與Istio：妥善管理基硬架構的有效方法
- container 好處
	- Immutable infrastructure
	- Faster deployments
	- Reusability
	- Versioning
	- Isolation
	- Portability
	- Introspection
	- Ease of sharing
- Other problems solved by K8s
	- Utilization
	- Scaling
	- Availability & failover
	- Rolling upgrades
	- Vendor lock-in
- Support for Regional Persistent Disks
	- Regional Control panel (node 可跨區)
- Integration with CDN
- Multi-cluster ingress
	- 不同的Region可用同一個LB
- Stackdrvier K8s Monitoring
- GKE Serverless Add-on
	- K8s with serverless
- GKE on-platform
	- 可在私人的server上使用GKE管理他們想要管理的server
- Sam wants to test his features with a small audience
	- New microservices
	- Feature iterations
- Istio
	- 開放式框架
	- 管理服務 (不一定是微服務)
	- 所有交通管理
- Istio-enabling a service
	- 在image都包上一個proxy
	- 全部的交通都透過proxy
	- 可設定container交通比重
	- 可對進來的交通正規化後決定去向
	- 可設定黑白名單
	- 通訊加密 (citadel)
- Istio 還可以做到什麼事
	-	Traffic control
	-	observability
	-	security
	-	fault-injection
	-	hybrid cloud
-	https://cloud.google.com/containers/
-	https://cloud.google.com/kubernetes/
-	https://cloud.google.com/istio/
### 雲端安全：如何確保使用者及其資料安全
- Why cloud for security (三個維度)
	- Platform Security (Auto-update single platform)
	- Application (Security secure from every path)
	- AI First (Enhance security by AI)
- Purpose-built Hardware Infrastructure (全都自己來)
	- Purpose-built chips
	- Purpose-built servers
	- Purpose-built storage
	- Purpose-built network
	- Purpose-built data centers
- Here is Titan
	- Secure Boot with a trusted platform Module
		> Host power up
		> PCH reads bootloader code
		> CPU executes bootloader, loads OS
		> TPM verifies boot files
	- Secure Boot with Titan
		> Host powers up
		> Titan boots, holds host CPU in reset
		> Titan verifies boot firmware flash
		> Titan allows host to boot
- Boot disk
	- Show images with shielded VM features
		> open security list
- Encryption by Default (不讓任何server有完整的資料，而且每個都加密)
	- All connections to Google Cloud require TLS
	- Data is chunked and each chunk is encrypted with its own data encryption key
	- Data encryption keys(DEKs) are wrapped using a key encryption key(KEK)
	- Encrypted chunks and wrapped encryption keys are distributed across Google's storage infrastructure.
- Find vulnerabilities that impact everyone (兩大漏洞晶片)
	- MELTDOWN
	- SPECTRE
- Update at scale, no disruptions
	- build
	- release
	- operate, monitor
- Live Migration
	- Compute Engine live migrates your running instances to another host in the same zone rather than requiring your VMs to be rebooted.
		> 事實上不需要擔心GCP更新server的相關操作
- Audit logs
	- Admin Activity Log
	- Data Access Log
		> 不管是讀取寫入都會放在stackdriver裡
- Export log for further analysis/notification
	- Cloud Pub/Sub
 	- BigQuery
	- Cloud Storage
- Network Security
	- Cloud Armor
		- DDos
		- IP/Location Whitelist & Blacklist
		- XSS/SQLi
		- Partner ecosystem
		- IAP
	- VPC Service Control
		- Perimeter security
	- VPC Firewall
- Cloud Armor Security Policy (GCP Network Security)
- XSS and SQLi defense
- Custom Rules
- AI for DDos Identification: Cloud Armor
- DLP
- Safely unlock more of the cloud
	- DLP API
	- Analytics, ML, NLP
	- App development
	- sharing
- Third-party audits and certifications