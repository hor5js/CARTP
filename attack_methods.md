# CARTP ラボマニュアル — 攻撃手法

出典: `Lab Manual.html`（Azure Cloud Attacks for Red and Blue Teams — CARTP）
作成日: 2026-10-07

本ドキュメントは、ラボマニュアルで解説されている攻撃手法を、前提となるトークン/MFA のセクションと各学習目標（Learning Objective）ごとに整理したものです。

> 注記: ツール名・製品名・攻撃手法の固有名詞（AADInternals、Mimikatz、Pass-the-PRT など）は、検索性を保つため英語表記のままにしています。

---

## サマリー表 — 学習目標 / 説明 / キルチェーン / ツール

| 学習目標 | 説明 | キルチェーン | ツール |
|----------|------|--------------|--------|
| LO1 | 未認証のテナント偵察 — Entra ID 利用の有無、テナント ID、メールアドレスの有効性確認 | 全KC · 偵察 | AADInternals |
| LO2 | ユーザー・グループ・デバイス・ディレクトリロール・アプリの認証済み列挙 | 全KC · 列挙 | Azure Portal |
| LO3 | Microsoft Graph 経由の認証済み列挙 | 全KC · 列挙 | Mg（MS Graph PowerShell）モジュール |
| LO4 | リソース・ロール・VM・アプリ・ストレージ・Key Vault の列挙 | 全KC · 列挙 | Az PowerShell モジュール |
| LO5 | VM・アプリ/関数アプリ・ストレージ・Key Vault の列挙 | 全KC · 列挙 | az CLI |
| LO6 | アプリロールと権限（同意グラフ）の列挙 | 全KC · 列挙 | ROADTools |
| LO7 | セキュリティ設定の確認（Security Defaults、ユーザー同意） | 全KC · 列挙 | Monkey365 |
| LO8 | 攻撃経路の分析（グローバル管理者ユーザー、Key Vault への到達経路） | 全KC · 列挙 | BloodHound CE、AzureHound |
| LO9 | 不正な同意付与（Illicit Consent Grant）フィッシング → アプリ管理者とそのワークステーションを侵害 | KC1 · 初期アクセス | 365-Stealer、MS Graph、xampp、FileFix |
| LO10 | 安全でないファイルアップロード → App Service を侵害 → マネージド ID の悪用 | KC1 · 初期アクセス | az CLI / Az PowerShell、curl（ファイルアップロード） |
| LO11 | サーバーサイドテンプレートインジェクション（SSTI）→ App Service を侵害 → マネージド ID の悪用 | KC2 · 初期アクセス | Web ブラウザ（SSTI）、az CLI |
| LO12 | 安全でないファイルアップロード + OS コマンドインジェクション → App Service を侵害 | KC3 · 初期アクセス | Web（ファイルアップロード／コマンドインジェクション）、az CLI |
| LO13 | 安全でないストレージ BLOB のデータマイニング → 2つ目のストレージアカウントへピボット | KC4 · 初期アクセス + データマイニング | Az PowerShell（Storage）、BLOB アクセス |
| LO14 | Automation アカウントの悪用 → クラウドからオンプレミスへのコマンド実行（ハイブリッドワーカー） | KC1 · 権限昇格 + 横展開（クラウド→オンプレミス） | Az／az CLI、MS Graph、Automation Runbook、xampp |
| LO15 | マネージド ID → Azure VM 上でのコマンド実行 + 資格情報の抽出 | KC1 · 権限昇格 | Az PowerShell、マネージド ID、Run Command |
| LO16 | Key Vault のシークレット抽出 + Copilot エージェントへのプロンプトインジェクション | KC2 · 初期アクセス（プロンプトインジェクション）+ 権限昇格 + データマイニング | Mg モジュール、Key Vault、IT Helpdesk Copilot エージェント |
| LO17 | Evilginx3 による AiTM フィッシング + 認証管理者（Authentication Administrator）の悪用 → コマンド実行 | KC2 · 初期アクセス + 権限昇格 | Evilginx3、Mg モジュール、365-Stealer |
| LO18 | Function App のマネージド ID → エンタープライズアプリ → Key Vault → デプロイ履歴から資格情報 | KC3 · 権限昇格 + データマイニング | Mg モジュール、AzureAD、Key Vault |
| LO19 | GitHub／CI-CD → Function App のマネージド ID → SSH 鍵 → 改変した関数のトリガー | KC4 · 権限昇格 + データマイニング | GitHub、az、SSH、Storage／BLOB |
| LO20 | PRT の抽出 → Pass-the-Certificate ／ カスタムスクリプト拡張 | KC2 · データマイニング + 横展開 | Mimikatz、roadtx、ROADTools、Custom Script Extension |
| LO21 | Pass-the-PRT + Intune デバイス管理構成ポリシーの悪用 | KC2 · データマイニング + 横展開（クラウド→オンプレミス） | Mimikatz、roadtx、Roadtoken、Intune、ADMX |
| LO22 | ゲスト招待による動的グループメンバーシップの悪用 | KC3 · 横展開（テナント間） | Mg モジュール、Chisel、FoxyProxy |
| LO23 | Application Proxy の悪用 → オンプレミスでの OS コマンド実行 + 資格情報の窃取 | KC4 · 横展開（テナント間） | Mg モジュール、Mimikatz、Application Proxy、Web（ファイルアップロード） |
| LO24 | ハイブリッド ID の悪用（MSOL_／ConnectSyncProvisioning_ アカウント） | KC1 · 権限昇格 + 横展開（オンプレミス→クラウド） | AADInternals、Mimikatz、CredSSP |
| LO25 | AD Connect へのバックドア設置 *（講師専用）* | KC2 · 横展開（オンプレミス↔クラウド） | AADInternals、Intune |
| LO26 | ADFS トークン署名証明書の窃取 ／ Golden SAML *（講師専用）* | KC4 · 横展開（オンプレミス→クラウド） | AADInternals（ADFS） |

> 凡例: KC = キルチェーン（攻撃経路 1〜4）／全KC = 全キルチェーン共通。フェーズ名: 偵察 (Recon)、列挙 (Enumeration)、初期アクセス (Initial Access)、権限昇格 (Privilege Escalation)、横展開 (Lateral Movement)、データマイニング (Data Mining)。

---

## 前提: トークン / MFA の手法

| # | 手法 | 説明 |
|---|------|------|
| P1 | 必須 MFA 強制の回避 | テナントで強制されている MFA 要件をバイパスする |
| P2 | ブラウザからの Graph API トークン抽出 | 認証済みブラウザセッションから Microsoft Graph のアクセストークンを取得する |
| P3 | ブラウザからの ARM トークン抽出 | 認証済みブラウザセッションから Azure Resource Manager のトークンを取得する |
| P4 | Device Code フローの悪用 | OAuth デバイスコード認証フローを通じてトークンをフィッシング／取得する |
| P5 | Authorization Code フローの悪用 | OAuth 認可コードフローの悪用によりトークンを取得する |

---

## 列挙（Enumeration）の手法

| # | 手法 | 使用ツール |
|---|------|------------|
| E1 | 未認証の組織偵察（テナント ID、Entra ID 利用の有無、メール有効性確認） | AADInternals |
| E2 | 認証済み列挙（ユーザー、グループ、デバイス、ロール、アプリ） | Azure Portal |
| E3 | Graph 経由の認証済み列挙 | Mg（Microsoft Graph）PowerShell モジュール |
| E4 | リソース／ロール／VM／アプリ／ストレージ／Key Vault の列挙 | Az PowerShell モジュール |
| E5 | リソースの列挙 | az CLI |
| E6 | アプリロールと権限の列挙 | ROADTools |
| E7 | セキュリティ設定の確認（Security Defaults、ユーザー同意設定） | Monkey365 |
| E8 | 条件付きアクセスポリシー（Conditional Access Policy）の列挙 | — |
| E9 | 攻撃経路の分析（グローバル管理者ユーザー、Key Vault への到達経路） | BloodHound Community Edition |

---

## 学習目標ごとの攻撃手法

### LO1 — 未認証のテナント偵察
AADInternals を用いて組織情報（Entra ID の利用有無、テナント ID、有効なメールアドレス）を収集する。*（トピック: 認証済み／未認証の列挙）*

### LO2〜LO8 — 認証済み列挙
Azure Portal、Mg モジュール、Az モジュール、az CLI、ROADTools、Monkey365、BloodHound を用いて、ユーザー／グループ／デバイス／ロール／アプリ／リソースを列挙する。*（トピック: 認証済み列挙）*

### LO9 — 不正な同意付与攻撃（フィッシング）
OAuth の不正な同意付与（Illicit Consent Grant）を利用して、アプリケーション管理者とそのワークステーションを侵害する（365-Stealer によるフィッシング、管理者同意の悪用、リバースシェル経由。コード実行の代替手段として FileFix）。*（キルチェーン 1 — 初期アクセス）*

### LO10 — 安全でないファイルアップロード → App Service 侵害
career アプリの安全でないファイルアップロード機能を悪用して App Service を侵害し、そのマネージド ID のサービスプリンシパルが持つ権限を悪用する。*（キルチェーン 1）*

### LO11 — サーバーサイドテンプレートインジェクション（SSTI）
SSTI の脆弱性を持つアプリケーションを特定・悪用して App Service を侵害し、そのマネージド ID を悪用する。*（キルチェーン 2）*

### LO12 — 安全でないファイルアップロード + OS コマンドインジェクション
安全でないファイルアップロードと OS コマンドインジェクションにより virusscanner の App Service を侵害し、そのマネージド ID を悪用する。*（キルチェーン 3）*

### LO13 — 安全でないストレージ BLOB のデータマイニング
安全でない／公開状態のストレージ BLOB を見つけてシークレットを収集し、2つ目のストレージアカウントへピボットしてさらにシークレットを抽出する。*（キルチェーン 4 — 初期アクセス & データマイニング）*

### LO14 — Automation アカウントの悪用 → クラウドからオンプレミスへの横展開
Azure Automation アカウントの権限を悪用して、クラウドからオンプレミスへの横展開を実行し、ハイブリッドワーカー上でコマンド実行を獲得する。*（キルチェーン 1 — 権限昇格 & 横展開）*

### LO15 — マネージド ID の悪用 → VM でのコマンド実行
`defcorphqcareer` App Service のマネージド ID を悪用して Azure VM 上でコマンドを実行し、資格情報を抽出する。*（キルチェーン 1 — 権限昇格）*

### LO16 — Key Vault シークレット抽出 + プロンプトインジェクション
`vaultfrontend` App Service のマネージド ID を悪用して Key Vault のシークレットを抽出し、IT HelpDesk Copilot エージェントにアクセスして、プロンプトインジェクション攻撃によりバックエンドのワークフロー／シークレットを漏えいさせる。*（キルチェーン 2 — プロンプトインジェクションによる初期アクセス & データマイニング）*

### LO17 — フィッシング + 認証管理者の悪用（Evilginx3）
認証管理者（Authentication Administrator）をフィッシングし（Evilginx3 による AiTM とセッションクッキーの窃取）、対象ユーザーのパスワードをリセットして、jumpvm 上でコマンド実行を獲得する。*（キルチェーン 2 — 初期アクセス & 権限昇格）*

### LO18 — Function App のマネージド ID → エンタープライズアプリ / Key Vault
`processfile` 関数アプリのマネージド ID を悪用してエンタープライズアプリケーションを侵害し、その権限を悪用して Key Vault のシークレットを抽出、さらにリソースグループのデプロイ履歴からユーザーの資格情報を抽出する。デバイスプラットフォームに基づく条件付きアクセスの回避を含む。*（キルチェーン 3）*

### LO19 — CI/CD / GitHub の悪用 → Function App
`defcorpcodebackup` ストレージ内のアプリバックアップから得たシークレットを使い、GitHub 経由で Function App に変更をプッシュし、そのマネージド ID を悪用して SSH 鍵を抽出、改変したコードを使う関数アプリをトリガーする。*（キルチェーン 4）*

### LO20 — PRT の抽出 & Pass-the-Certificate / カスタムスクリプト拡張
jumpvm からユーザーの PRT を抽出し、infradminsrv に対して Pass-the-Certificate 攻撃を実行する。代替として、ユーザーデータから平文の資格情報を抽出し、VM の Custom Script Extension を悪用してコード実行する。クライアントアプリに基づく条件付きアクセスの回避を含む。*（キルチェーン 2 — 横展開）*

### LO21 — Pass-the-PRT + Intune（デバイス管理）の悪用
infradminsrv から PRT を抽出し、Pass-the-PRT を実行する。Intune Administrator ロールがある場合、デバイス管理構成ポリシーを悪用して、ローカル管理者の追加、コマンド実行（ADMX テンプレートによる GPO RCE を含む）、PowerShell スクリプトの実行（GUI または `browserprtauth`/roadtx 経由）を行い、その後 PSRemote で資格情報をダンプする。PRT の抽出には Mimikatz／Roadtoken を使用。*（キルチェーン 2 — クラウドからオンプレミスへの横展開）*

### LO22 — 動的グループメンバーシップの悪用（テナント間）
動的グループを列挙し、ゲストユーザーを招待して、その属性を変更することで動的グループに自動参加させ、権限昇格を行う。ユーザー Thomas の MFA 回避、および Chisel を用いた位置情報ベースの条件付きアクセス回避（+ FoxyProxy）を含む。*（キルチェーン 3 — テナント間の横展開）*

### LO23 — Application Proxy の悪用 → オンプレミスでのコマンド実行
GitHub の `CreateUsers` リポジトリを悪用してユーザーを作成し、Application Proxy を使用するエンタープライズアプリを特定、ファイルアップロードの脆弱性を悪用してアプリをホストするオンプレミスサーバー上で OS コマンド実行を獲得し、資格情報を抽出する。*（キルチェーン 4 — テナント間の横展開）*

### LO24 — ハイブリッド ID の悪用（MSOL_ / ConnectSyncProvisioning_）
Entra Connect サーバーから `MSOL_*` および `ConnectSyncProvisioning_*` アカウントの資格情報を抽出し、`MSOL_*` でオンプレミスドメインを侵害、`ConnectSyncProvisioning_*` で同期されたクラウドユーザーを侵害する。ハイブリッド ID／PHS の悪用と CredSSP の有効化を扱う。*（キルチェーン 1 — オンプレミスからクラウドへの横展開）*

### LO25 — AD Connect のバックドア *（講師専用）*
defres.corp の AD Connect サーバーにバックドアを設置し、同期ユーザーとしてテナントにアクセスする。*（キルチェーン 2）*

### LO26 — ADFS トークン署名証明書の窃取（Golden SAML）*（講師専用）*
侵害済みのドメイン管理者資格情報を使って ADFS のトークン署名証明書を抽出し、別のユーザーとして deffin.com テナントにアクセスする。*（キルチェーン 4 — オンプレミスからクラウドへの横展開）*

---

## 条件付きアクセス / MFA 回避の手法（横断的）

- 必須 MFA 強制の回避
- デバイスプラットフォームに基づく条件付きアクセスポリシーの回避
- クライアントアプリに基づく条件付きアクセスポリシーの回避
- Chisel を用いた位置情報ベースの条件付きアクセスの回避（+ FoxyProxy）
- Evilginx3 による AiTM セッションクッキー窃取での MFA バイパス
- 条件付きアクセス要件を満たすためのデバイス登録／MFA 登録の悪用

---

## サイバーキルチェーンのマッピング（AttackingAzureAD.pdf より）

スライド資料では **Azure キルチェーン** を5つの連続するフェーズとして定義しており、さらに横断的なトピック（データマイニング、および3方向の横展開）があります。そのうえで、これらのフェーズを貫く4本の完全な攻撃経路 — **キルチェーン 1〜4** — を解説しています。

### Azure キルチェーンのフェーズ（PDF スライド 34〜273）

| フェーズ | PDF スライド | 対象範囲 |
|----------|--------------|----------|
| 1. 偵察 / ディスカバリ | 「Azure Kill Chain - Recon」 | 未認証のディスカバリ: テナント利用の有無、テナント ID／名、フェデレーション、ドメイン、サービス、メール推測 |
| 2. 初期アクセス | 「Azure Kill Chain - Initial Access」 | 最初の足がかりの獲得（資格情報の侵害、フィッシング／同意付与、アプリの脆弱性、プロンプトインジェクション） |
| 3. 列挙 | 「Azure Kill Chain - Enumeration」 | 既定のユーザー権限を用いた Entra ID／Azure リソースの認証済み列挙 |
| 4. 権限昇格 | 「Azure Kill Chain - Privilege Escalation」 | マネージド ID、ロール、Automation アカウント、Key Vault、Intune の悪用 |
| 5. 横展開 | 「Azure Kill Chain - Lateral Movement」 | クラウド→オンプレミス、オンプレミス→クラウド、テナント間の移動 |

> マニュアルのタグで使われている横断的トピック: **データマイニング**、および3方向の横展開（**クラウド→オンプレミス**、**オンプレミス→クラウド**、**テナント間**）。

### 4本の攻撃経路（キルチェーン 1〜4）

| 経路 | 経路に含まれる学習目標 | テーマ |
|------|------------------------|--------|
| **キルチェーン 1** | LO9、LO10、LO14、LO15、LO24 | 不正な同意付与 → App Service → Automation アカウント → VM → ハイブリッド ID（オンプレミス） |
| **キルチェーン 2** | LO11、LO16、LO17、LO20、LO21、LO25 | SSTI / Key Vault+Copilot → フィッシング → PRT / Pass-the-Cert → Intune → AD Connect |
| **キルチェーン 3** | LO12、LO18、LO22 | ファイルアップロード+コマンドインジェクション → Function App / エンタープライズアプリ → 動的グループ（テナント間） |
| **キルチェーン 4** | LO13、LO19、LO23、LO26 | 安全でない BLOB → GitHub/CI-CD → App Proxy → ADFS Golden SAML |

### 攻撃手法 → キルチェーンのフェーズ + 経路

| 攻撃手法（LO） | キルチェーンのフェーズ | 経路 |
|----------------|------------------------|------|
| P1〜P5 トークン/MFA 手法 | 初期アクセス | 全経路 |
| E1 未認証の組織偵察（AADInternals） | **偵察** | 全経路 |
| E2〜E9 / LO2〜LO8 列挙 | **列挙**（認証済み） | 全経路 |
| LO1 未認証のテナント偵察 | **偵察** | 全経路 |
| LO9 不正な同意付与フィッシング | 列挙 → **初期アクセス** | 1 |
| LO10 安全でないファイルアップロード → App Service | 列挙 → **初期アクセス** | 1 |
| LO11 SSTI → App Service | 列挙 → **初期アクセス** | 2 |
| LO12 ファイルアップロード + OS コマンドインジェクション | 列挙 → **初期アクセス** | 3 |
| LO13 安全でないストレージ BLOB のデータマイニング | **初期アクセス** + **データマイニング** | 4 |
| LO14 Automation アカウント → クラウドからオンプレミス | 列挙 → **権限昇格** → **横展開（クラウド→オンプレミス）** | 1 |
| LO15 マネージド ID → VM でのコマンド実行 | 列挙 → **権限昇格** | 1 |
| LO16 Key Vault + プロンプトインジェクション | 列挙 → **初期アクセス（プロンプトインジェクション）** → **権限昇格** → **データマイニング** | 2 |
| LO17 フィッシング + 認証管理者（Evilginx3） | 列挙 → **初期アクセス** → **権限昇格** → **データマイニング** | 2 |
| LO18 Function App MI → エンタープライズアプリ / KV | 列挙 → **権限昇格** → **データマイニング** | 3 |
| LO19 GitHub / CI-CD → Function App | 列挙 → **権限昇格** → **データマイニング** | 4 |
| LO20 PRT → Pass-the-Certificate / CSE | 列挙 → **データマイニング** → **横展開** | 2 |
| LO21 Pass-the-PRT + Intune 悪用 | 列挙 → **データマイニング** → **横展開（クラウド→オンプレミス）** | 2 |
| LO22 動的グループメンバーシップの悪用 | 列挙 → **横展開（テナント間）** | 3 |
| LO23 Application Proxy → オンプレミスでのコマンド実行 | 列挙 → **横展開（テナント間）** | 4 |
| LO24 ハイブリッド ID（MSOL_/ConnectSync） | 列挙 → **権限昇格** → **横展開（オンプレミス→クラウド）** | 1 |
| LO25 AD Connect バックドア *（講師専用）* | 列挙 → **権限昇格** → **横展開（オンプレミス↔クラウド）** | 2 |
| LO26 ADFS Golden SAML *（講師専用）* | 列挙 → **権限昇格** → **横展開（オンプレミス→クラウド）** | 4 |
| CA/MFA 回避の手法 | 横断的（初期アクセスと横展開を可能にする） | 2、3 |

> 注記: PDF 内のキルチェーン 1〜4 は **図（画像）** として提示されているため、上記の経路のグルーピングは、マニュアル内の各学習目標に付された「Part of - Kill Chain N」タグから再構成したものです。フェーズ名は PDF の「Azure Kill Chain - …」セクション見出しからそのまま採用しています。

---

## キルチェーン・フェーズ別テーブル — フェーズ / 説明 / ツール

Azure キルチェーンの各フェーズ（横断的トピックを含む）ごとに、内容と使用ツールをまとめたものです。

各フェーズの「攻撃手法」と「説明」は上から順に 1 対 1 で対応しています。

| キルチェーンのフェーズ | 攻撃手法 | 説明 | ツール |
|------------------------|----------|------|--------|
| 1. 偵察（Recon） | 1. 未認証テナント偵察<br>2. ユーザー名／メールアドレス列挙 | 1. テナントが Entra ID を利用しているか、テナント ID／名、フェデレーション構成、ドメイン、使用サービスを特定する<br>2. 有効なユーザー名／メールアドレスを推測・検証する | AADInternals |
| 2. 初期アクセス（Initial Access） | 1. 不正な同意付与（Illicit Consent Grant）フィッシング<br>2. 安全でないファイルアップロード<br>3. SSTI（サーバーサイドテンプレートインジェクション）<br>4. OS コマンドインジェクション<br>5. プロンプトインジェクション<br>6. AiTM フィッシング | 1. 悪意あるアプリへの OAuth 同意をユーザーに付与させ、アプリ管理者とそのワークステーションを侵害する<br>2. アプリのアップロード機能を悪用して Web シェル等を配置し App Service を侵害する<br>3. テンプレート処理の脆弱性を悪用してコード実行し App Service を侵害する<br>4. 入力検証の不備を悪用して基盤 OS 上で任意コマンドを実行する<br>5. Copilot エージェントのワークフローを操作し、バックエンドのシークレット／ワークフローを漏えいさせる<br>6. リバースプロキシでセッションクッキーを窃取し MFA をバイパスして足がかりを得る | 365-Stealer、Evilginx3、FileFix、xampp、Web ブラウザ、curl、az CLI、Get-AccessToken（Authorization Code フロー）、IT Helpdesk Copilot エージェント |
| 3. 列挙（Enumeration） | 1. 認証済み列挙<br>2. 攻撃経路分析<br>3. セキュリティ設定の確認 | 1. 既定のユーザー権限で Entra ID／Azure のユーザー・グループ・デバイス・ロール・アプリ・リソースを列挙する<br>2. ユーザー／グループ／リソース間の権限関係をグラフ化し、特権への到達経路を特定する<br>3. Security Defaults やユーザー同意設定など、悪用可能な構成を確認する | Azure Portal、Mg モジュール、Az PowerShell、az CLI、ROADTools、Monkey365、BloodHound CE、AzureHound |
| 4. 権限昇格（Privilege Escalation） | 1. マネージド ID の悪用<br>2. Automation アカウントの悪用<br>3. エンタープライズアプリ／サービスプリンシパル権限の悪用<br>4. 認証管理者の悪用（パスワードリセット） | 1. App Service／関数アプリのマネージド ID を悪用して他リソースへの権限を取得し、VM コマンド実行や Key Vault アクセスを行う<br>2. Automation アカウントの権限でランブックを実行し、ハイブリッドワーカー等でコード実行する<br>3. アプリ／サービスプリンシパルに付与された過大な権限を悪用し、シークレット抽出等を行う<br>4. Authentication Administrator 権限で対象ユーザーのパスワードをリセットしアカウントを乗っ取る | Az PowerShell、az CLI、Mg モジュール、AzureAD、Automation Runbook／Run Command、Key Vault、Intune |
| 5. 横展開（Lateral Movement） | 1. Pass-the-Certificate<br>2. Pass-the-PRT<br>3. Custom Script Extension の悪用<br>4. Intune デバイス管理構成ポリシーの悪用<br>5. 動的グループメンバーシップの悪用<br>6. Application Proxy の悪用<br>7. ハイブリッド ID の悪用（MSOL_／ConnectSyncProvisioning_）<br>8. ADFS Golden SAML | 1. 抽出した PRT から証明書を取得し、別 VM に対して認証・コマンド実行する<br>2. 窃取した PRT を悪用してユーザーになりすましトークンを取得する<br>3. VM の Custom Script Extension を使って任意コードを実行する<br>4. Intune 管理者権限でポリシーを配布し、登録端末にローカル管理者追加・コマンド実行・スクリプト実行を行う<br>5. ゲストユーザーの属性を操作して動的グループに参加させ、権限を獲得する<br>6. 公開アプリの脆弱性を悪用して、オンプレミスのホストサーバー上でコマンド実行する<br>7. Entra Connect の同期アカウント資格情報を抽出し、オンプレミスドメインや同期クラウドユーザーを侵害する<br>8. ADFS のトークン署名証明書を窃取し、任意ユーザーとしてクラウドにアクセスする | Mimikatz、roadtx、Roadtoken、ROADTools、Custom Script Extension、AADInternals、Application Proxy、Intune／ADMX、CredSSP |
| データマイニング（Data Mining）*（横断的）* | 1. ストレージ BLOB データマイニング<br>2. デプロイ履歴・バックアップ・ユーザーデータからのシークレット／資格情報抽出 | 1. 公開／安全でない BLOB を探索し、シークレットや別アカウントへの手がかりを収集する<br>2. リソースグループのデプロイ履歴、アプリバックアップ、VM のユーザーデータから資格情報を抽出する | Az PowerShell（Storage）、BLOB アクセス、GitHub、SSH |
| 条件付きアクセス / MFA 回避 *（横断的）* | 1. 必須 MFA 強制の回避<br>2. 条件付きアクセスの回避（デバイスプラットフォーム／クライアントアプリ／位置情報）<br>3. AiTM セッションクッキー窃取<br>4. トークン抽出（ブラウザ／Device Code／Authorization Code フロー） | 1. MFA が強制されないポータル経由でトークンを取得し MFA をバイパスする<br>2. デバイス種別・クライアントアプリ・送信元 IP 等の条件を偽装してポリシーを回避する<br>3. リバースプロキシでセッションクッキーを窃取し、認証済みセッションを再利用する<br>4. 各種 OAuth フローや認証済みブラウザからアクセストークンを取得する | ブラウザからのトークン抽出、Device Code フロー、Evilginx3、Chisel、FoxyProxy、roadtx |

---

## 参照ツール一覧

AADInternals · Az PowerShell / az CLI · Microsoft Graph（Mg）モジュール · ROADTools · roadtx ·
Monkey365 · BloodHound CE · 365-Stealer · Evilginx3 · Mimikatz · Roadtoken · Chisel · FoxyProxy · xampp
