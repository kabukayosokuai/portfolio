#RUN,COPYやADD等、レイヤーに差が生じやすくキャッシュ使えない為、ビルドに時間がかかる。だからそれらは後ろのほうに持ってく
# このままだとコンテナ内はrootユーザーとなり、ホストのファイルも変更できるようになる可能性がある
#しかし、一般ユーザーを作るとホストのUID差が生まれvolumeした時のファイル所有者の観点から修正できなくなってしまう
# →本番用と開発用にうまく分けて、開発用はrootコンテナでok。本番はユーザー指定できるように

# https://www.forcia.com/blog/002273.html
# https://www.slideshare.net/zembutsu/explaining-best-practices-for-writing-dockerfiles

#マルチステージビルドによって開発環境・本番環境で分ける。
# 分ける理由は開発環境ではvolumeによってホスト側を修正したらコンテナ側も修正できるようコンテナ

#ビルド用のコンテナ
FROM python:3.10-buster as builder

# コンテナ内に指定のフォルダを作成し、cdで入る。ここにrequirement.txtをコピーしてインストールさせる
WORKDIR /django

# requirements.txtをコンテナ内のdjangoフォルダにコピー
COPY requirements.txt .

# コピーしたrequirements.txtを元にpythonモジュールをインストール
RUN pip3 install -r requirements.txt



#本番用のコンテナ
FROM python:3.10-buster as production

#COPYは本来ローカルに対してだが、--from=builder(Asでステージに名前つけた)でビルドしたコンテナを参照できるようになる。
#ここで下記の通り必要最低限のものだけインストールすると容量が小さくて良くなる。uwsgiは違うフォルダにインストールされる為以下のようにしている
COPY --from=builder /usr/local/lib/python3.10/site-packages /usr/local/lib/python3.10/site-packages
COPY --from=builder /usr/local/bin/uwsgi /usr/local/bin/uwsgi

#apt-get clean では /var/cache/apt/archives にキャッシュされている全てのパッケージを削除
#rm -rf /var/lib/apt/lists/* では /var/cache/apt/list にキャッシュされている全てのパッケージリストを削除
#それぞれ削除することでイメージを軽くしている
RUN apt-get update \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

#セキュリティの観点からユーザーを作成する。
#この指定がなければコンテナ内はrootユーザーで動作する為、侵入された場合あらゆる権限で操作可能なうえコンテナ内のrootユーザとホストのrootユーザはuidが一緒のため、ホスト側もroot権限でアクセスが可能になります。
#コンテナ用にユーザーを作成し、ファイルは全てホスト側で編集しvolumeを介してコンテナに共有することでコンテナに侵入されても権限の観点から編集できない
# -s /bin/falseを指定することでログインもできないように設定。つまり本番用
RUN useradd -r -s /bin/false portfolio

#作成したユーザーに変更
USER portfolio




#開発用のコンテナ
FROM python:3.10-buster as local

# コンテナ内に指定のフォルダを作成し、cdで入る。ここにrequirement.txtをコピーしてインストールさせる
WORKDIR /django

# requirements.txtをコンテナ内のdjangoフォルダにコピー
COPY requirements.txt .

# コピーしたrequirements.txtを元にpythonモジュールをインストール
RUN pip3 install -r requirements.txt

#apt-get clean では /var/cache/apt/archives にキャッシュされている全てのパッケージを削除
#rm -rf /var/lib/apt/lists/* では /var/cache/apt/list にキャッシュされている全てのパッケージリストを削除
#それぞれ削除することでイメージを軽くしている
RUN apt-get update \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*