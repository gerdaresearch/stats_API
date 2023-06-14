# Python code  obtain official data via API from government stats sites 
# 日本　政府統計の総合窓口　e-stat から、APIで家計調査データにアクセス　～データ取得（JSON）～データフレーム～日付整形～csv出力
# 前もって、マイページ、APIキーの申請と取得が必要　　https://www.e-stat.go.jp/
# 以下のコードは、総務省「家計調査」　月次データの取得例
# 期間　1986年1月～2023年4月まで　
# 世帯区分  2人以上世帯、2人以上勤労者世帯
# 品目　消費支出、切花、園芸植物、園芸用品
# Japan monthly household income and expenditure survey (Kakei chosa), Jan 1986 - April 2023, spending, cut flower and plants  


# 1. 統計表ID（事前取得）  ここでは家計調査　月次データ　（IDは事前に調べておく）　
# 家計調査　2人以上月次　https://www.e-stat.go.jp/stat-search/database?page=1&statdisp_id=0003343671

# 2. APIキー（事前取得） 個人でe-statで申請・取得
api_key = "Your API key"

# 3. 下準備　パッケージ
import pandas as pd
import numpy as np
from datetime import datetime

import requests
import json
import csv


# 4．統計に関するメタ情報取得　統計の内容確認
# API仕様 確認　https://www.e-stat.go.jp/api/sites/default/files/uploads/2019/07/API-specVer3.0.pdf
# APIリクエスト アプリケーションID＋パラメーター指定　
# 申請取得したAPIキー入力、パラメーターは "&パラメーター=対応コード," の形でつなげていく
# バージョン 3.0 　xml,json, csv対応　それぞれurlが違うので、仕様書参照
url = f'http://api.e-stat.go.jp/rest/3.0/app/json/getMetaInfo?appId=<Your API key>&statsDataId=0003343671'

# メタ情報リクエストURL  JSON
res = requests.get(url) 

# データを表示　統計の構造、必要な項目とコード、レベル等を確認・取得
res.json()


# 5. 統計データ取得
appId_estat= <Your_API_Key>
url = "https://api.e-stat.go.jp/rest/3.0/app/json/getStatsData?"
params = {
    "appId": api_key,
    "statsDataId": "0003343671",
    "cdArea" : "00000",
    "cdTimeFrom" : "1986000202",
    "cdTimeTo" : "2023000404",
    "cdCat01": "001100000,090203060,090204050,090204060",
}

res = requests.get(url, params)
out = res.json()
out

# 出力内容確認
list(out['GET_STATS_DATA'].keys())
out['GET_STATS_DATA']['STATISTICAL_DATA'][ 'DATA_INF']['VALUE']

# 表形式で出力
df = pd.DataFrame(out['GET_STATS_DATA']['STATISTICAL_DATA'][ 'DATA_INF']['VALUE'])
df


# 6. データ成型
# 6.1 日付データの変換
# 2000000101の形で出力されるので、上4桁（年）＋7～8桁（月）の形式（2000-01）に変換
# @time列を新しい列に変換して追加
df['yyyymm'] = df['@time'].apply(lambda x: f"{x[:4]}-{x[6:8]}")

# 日付データとして保存
df['yyyymm'] = pd.to_datetime(df['yyyymm'])


# 6.2 列名の変更 コードを内容を表す列名に変える
# マッピングを定義
column_mapping = {
    '@cat01': '品目分類',
    '@cat02': '世帯区分',
    '@area': '地域区分',
    '@time': '日付',
    '@unit': '単位',
    '$': '円'
}

# 列名の変更 inplace引数で置換
df.rename(columns=column_mapping, inplace=True)

# 6.3 セルの値（項目名）をテキスト表記に　元の列（コード）は保存、コピーして新しい列（テキスト表示）と併置
df['品目'] = df['品目分類'].copy()

# 世帯区分の列をコピーして新しい列を作成
df['世帯'] = df['世帯区分'].copy()

# 地域の列をコピーして新しい列を作成
df['地域'] = df['地域区分'].copy()

# 品目分類の値を置換（各セル）
df.loc[df['品目'] == '001100000', '品目'] = '消費支出'
df.loc[df['品目'] == '090203060', '品目'] = '切り花'
df.loc[df['品目'] == '090204050', '品目'] = '園芸用植物'
df.loc[df['品目'] == '090204060', '品目'] = '園芸用品(2014年までは園芸用植物を含む)'

# 世帯区分の値を置換（各セル）
df.loc[df['世帯'] == '03', '世帯'] = '2人以上'
df.loc[df['世帯'] == '04', '世帯'] = '2人以上勤労'
df.loc[df['世帯'] == '01', '世帯'] = '2人以上農家除'
df.loc[df['世帯'] == '02', '世帯'] = '2人以上農家除勤労'

# 世帯区分の値を置換（各セル）
df.loc[df['地域'] == '00000', '地域'] = '全国'

# 列の順序を定義　変換前後の列が隣接し、照合できるようにする
new_columns = ['@tab', '品目分類', '品目', '世帯区分', '世帯', '地域区分', '地域', '日付', 'yyyymm', '単位', '円']

# 列の順序を入れ替え
df = df.reindex(columns=new_columns)

# 7.ファイルに書き出し
# データフレームをCSVファイルとして保存 ヘッダー行書き込み
# 文字フレーム　cp932(shift-jis系)を指定 （UTF-8では文字化け）  
df.to_csv('kakei_out.csv', index=False, encoding='cp932')
print("データをCSVファイルに保存しました。")   

