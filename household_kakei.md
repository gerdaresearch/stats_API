Python code  obtain official data via API from government stats sites 
日本　政府統計の総合窓口　e-stat から、APIで家計調査データにアクセス　～データ取得（JSON）～データフレーム～日付整形～csv出力
前もって、マイページ、APIキーの申請と取得が必要　　https://www.e-stat.go.jp/
以下のコードは、総務省「家計調査」　月次データの取得例
期間　1986年1月～2023年4月まで　
世帯区分  2人以上世帯、2人以上勤労者世帯
品目　消費支出、切花、園芸植物、園芸用品
Japan monthly household income and expenditure survey (Kakei chosa), Jan 1986 - April 2023, spending, cut flower and plants  

# 1. 統計表ID（事前取得）  ここでは家計調査　月次データ　（IDは事前に調べておく）　
 家計調査　2人以上月次　https://www.e-stat.go.jp/stat-search/database?page=1&statdisp_id=0003343671

# 2. APIキー（事前取得） 個人でe-statで申請・取得
api_key = "Your API key"

# 3. 下準備　パッケージ

# 4．統計に関するメタ情報取得　統計の内容確認
API仕様 確認　https://www.e-stat.go.jp/api/sites/default/files/uploads/2019/07/API-specVer3.0.pdf
APIリクエスト アプリケーションID＋パラメーター指定　
申請取得したAPIキー入力、パラメーターは "&パラメーター=対応コード," の形でつなげていく
バージョン 3.0 　xml,json, csv対応　それぞれurlが違うので、仕様書参照
url = f'http://api.e-stat.go.jp/rest/3.0/app/json/getMetaInfo?appId=<Your API key>&statsDataId=0003343671'

メタ情報リクエストURL  JSON
データを表示　統計の構造、必要な項目とコード、レベル等を確認・取得

# 5. 統計データ取得
表形式で出力

# 6. データ成型
日付データの変換 2000000101の形で出力されるので、上4桁（年）＋7～8桁（月）の形式（2000-01）に変換
列名の変更 コードを内容を表す列名に変える
セルの値（項目名）をテキスト表記に　元の列（コード）は保存、コピーして新しい列（テキスト表示）と併置

# 7.ファイルに書き出し
データフレームをCSVファイルとして保存 ヘッダー行書き込み
 

