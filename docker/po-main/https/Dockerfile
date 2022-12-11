#マルチステージビルドによって開発環境・本番環境で分ける。
# 分ける理由は開発環境ではvolumeによってホスト側を修正したらコンテナ側も修正できるようコンテナ

#本番用のコンテナ
FROM steveltn/https-portal:1 as production

#セキュリティの観点からユーザーを作成する。
#この指定がなければコンテナ内はrootユーザーで動作する為、侵入された場合あらゆる権限で操作可能なうえコンテナ内のrootユーザとホストのrootユーザはuidが一緒のため、ホスト側もroot権限でアクセスが可能になります。
#コンテナ用にユーザーを作成し、ファイルは全てホスト側で編集しvolumeを介してコンテナに共有することでコンテナに侵入されても権限の観点から編集できない
# -s /bin/falseを指定することでログインもできないように設定。つまり本番用
# RUN useradd -r -s /bin/false portfolio

# #作成したユーザーに変更
# USER portfolio




#開発用のコンテナ
FROM steveltn/https-portal:1 as local