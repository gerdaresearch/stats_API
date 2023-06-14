# stats_API data retrieval
obtain data via API, Japan official stats 
日本の政府統計サイト（ここでは家計調査）から、APIでデータ（月次）を得て、表形式で出力するまでのスクリプト　
スクリプトはPython、データはjson、csvで書き出し
APIの仕様については、政府統計総合窓口で確認　リクエストの仕方、パラメーター、シンタックス
https://www.e-stat.go.jp/api/sites/default/files/uploads/2019/07/API-specVer3.0.pdf
スクリプト自体はシンプルだが、メタ情報を出力して、個々の統計の構造を理解し、目指す情報の区分やコードを得て、正しく指定するまでの作業が、統計によっては複雑

This script retrieves monthly data from the Japanese government's statistical website, specifically the Household Survey, using an API. The script is written in Python, and the data is in JSON format, which is then converted to CSV for export. 
The API specifications can be found on the e-Stat Portal website. https://www.e-stat.go.jp/api/en
While the script itself may be simple, the process can become complex depending on the structure of each statistical dataset, requiring an understanding of the data structure. Review the metadata output, including codes for required items, in order to correctly specify and retrieve the desired information.
