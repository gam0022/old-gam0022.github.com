---
layout: post
title: "Rsyncを使ってOctpressをVPS上で公開する"
date: 2012-07-23 10:02
comments: true
categories: 
- Octpress
- VPS
- Rsync
- ServersMan@VPS
---
今回もOctpressに関する記事です。

先日、Octpressを使ってGithubPages上にブログを運営しましたが、
気が変わってレンタルしているVPS上で運営したくなりました。

Octpressでは`rake deploy`を叩くだけでRsyncを使い自分のサーバ上にコンテンツを同期するようにできます。

今回はその方法を紹介したいと思います。

Rsyncを使って同期するためには次の2つの作業をする必要があります。

(もしかすると、Rakefileの設定だけでよいのかもしれませんが、未検証です。)

1. 公開鍵認証を使ってsshでログインできるようにする。
2. サーバ側の`~/.ssh/authorized_keys`にローカル側の公開鍵を登録する。

1.は最初から行なっている人が多いかもしれませんね。

ここでは、作成済みのuserというユーザからserverというドメインのサーバに公開鍵でログインするようにする手順を紹介します。

環境はServersMan@VPS(OS:Ubuntu, HDD:10GB,RAM:256MB)です。

## サーバ側の設定

まず、VPSに普通にパスワード認証でsshをして作業します。

### 公開鍵の生成

{% codeblock lang:bash %}
ssh root@server
login user
ssh-keygen -t rsa #公開鍵を生成 save the keyは空白でOK、パスフレーズは任意のものを指定する
cd ~/.ssh/
mv id_rsa.pub authorized_keys #公開鍵はauthorized_keysにリネーム
cat id_rsa #この内容は後でローカルに保存するので、テキストエディタなどに貼りつけておく
exit
{% endcodeblock %}

### sshの設定

vimなどで、サーバ側の`/etc/ssh/sshd_config`を編集します。
間違った設定をすると最悪sshでログインできなるので注意しましょうw

私の場合はこのような感じに**一部を書き換えました。**
この記事を書いている私ですが、サーバの設定は初心者なので、あまり信用しないようにしましょうw
あくまで参考程度で。

{% codeblock %}
AllowUsers user
RSAAuthentication yes 
PubkeyAuthentication yes 
AuthorizedKeysFile   %h/.ssh/authorized_keys
{% endcodeblock %}

VPS上の設定が終わったら、sshdを再起動します。

{% codeblock lang:bash %}
/etc/init.d/ssh restart
{% endcodeblock %}

## ローカル側の設定

前に`cat id_rsa`で表示した内容を、ローカルの`~/.ssh/foo.key`(fooは任意)で保存します。
sshの-iオプションで秘密鍵を指定してログインらしいです。じつはよくわかってませんw

{% codeblock lang:bash %}
cd ~/.ssh
vim foo.key #サーバのid_rsaをコピペして保存
chmod 600 foo.key #これしないと「WARNING: UNPROTECTED PRIVATE KEY FILE!」と出てログインできない
ssh -i foo.key user@server #これでログインできれば成功
{% endcodeblock %}

### サーバ側の~/.ssh/authorized_keysにローカル側の公開鍵を登録する。

前の手順で、サーバにログインできたと思うので、
ついでにサーバ側の`~/.ssh/authorized_keys`にローカルの公開鍵を追加します。

### Octpressの設定

ここまで来れば、あとはOctpressの設定だけです。`Rakefile`を編集します。

私の場合、GithubPages用にRakeFileを設定してしまったので、書き直します。

{% codeblock lang:ruby %}
## -- Rsync Deploy config -- ##
# Be sure your public key is listed in your server's ~/.ssh/authorized_keys file
ssh_user       = "user@server"
ssh_port       = "1234" #普通は22ですが、セキュリティ上の理由で(
document_root  = "/var/www/html/" #htmlの公開用のディレクトリを指定。ServersMan@VPSならこの設定でいいはず。
rsync_delete   = true
#deploy_default = "push"
deploy_default = "rsync" #rsyncで同期します。
{% endcodeblock %}

たぶんこれで設定は終わりです。

`rake deploy`でファイルが同期できたら成功です。

---

参考記事
:[Deploying With Rsync](http://octopress.org/docs/deploying/rsync/)
