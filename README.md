NTEmacs64 Ver 26.3 (日本語パッチ版)
=========

Windows 版 Emacs (通称 NTEmacs) の 64bit 版 version 26.3


バイナリ説明
------------
26.2から公式ビルドに日本語パッチが正式に導入されたようなので、基本的に26.3では日本語パッチは必要ないと思っていたのですが・・・
公式ビルド(26.3)を使ってみて

* IME自体は動作しているが文節区切り、変換し直しなどうまくいかない
* そもそもW32パッチを当ててない場合のIMEの設定方法が判らないし、適当な資料も見つからない
* 日本語パッチを当てたバイナリが見つからない

というわけで、自分で環境を構築してバイナリを作成してみることにしました。
パッチを当ててバイナリを作成するにあたって、[chuntaro氏のNTEmacs64 Ver25.2](https://github.com/chuntaro/NTEmacs64/blob/master/README.md)のページを参考にさせていただきました。

このドキュメントも基本的に上記ファイルに準じた形としています。

公式ビルドを使用するには以下から取得出来ます。  
<http://ftpmirror.gnu.org/emacs/windows/>

ファイル | 説明
------------- | -------------
[**emacs-26.3-x64-IME-patched.zip**](https://github.com/Akari-Kaminoto/NTEmacs64/raw/master/emacs-26.3-x64-IME-patched.zip) | IMEパッチ適用版

### 起動方法
**emacs-26.3-IME-patched.zip** を展開すると **emacs-26.3/** フォルダが出来るので **emacs-26.3/bin/runemacs.exe** を実行します。

### 特徴
* MSYS2 (MSYS の改良版) を使用してビルドしているので Cygwin に依存していません
  * Emacs 上でのパスの扱いなどが自然になります
  * 公式ビルドでも MSYS2 を使用しています (ビルド手順も普通にビルドするだけなので、ほぼ一緒と思われます)
  * MSYS2 でビルドはしていますが、実際に使用されているコンパイラツールチェインは MinGW-w64(mingw64) になります
* gcc に **-Ofast -march=x86-64 -mtune=corei7** を付けて最適化ビルドされています
* バージョン 25.2 から追加された目玉機能のダイナミックモジュールを有効にしてビルドされています
  * 公式ビルドはダイナミックモジュールを有効にしてビルドされていません
  * ダイナミックモジュールは DLL に実装された機能を Emacs から直接使用する為の機能ですが、load-path 上に DLL を配置して、なおかつ DLL に専用のエントリーポイントが必要なので任意の DLL がロード可能になるわけではありません
  * [Introduction to Emacs modules](http://diobla.info/blog-archive/modules-tut.html) ← このサイトを参考にモジュール(DLL)を作成して動作確認済みです
* 画像対応させる為の最低限の DLL を同梱しています (**GIF, PNG, JPEG, TIFF, XPM**)
  * 本来は SVG の表示にも対応可能ですが、依存 DLL(主に GTK+ 関連) が多すぎるので含めていません
* **libxml2, GnuTLS** の DLL も同梱しています
  * elisp で実装されたテキストブラウザ M-x eww も動作確認済みです
  * 追加した DLL は全て emacs-26.3/bin/ 以下にあります (bin/*.dll 以外追加したファイルはありません)
* IMEパッチを適用してビルドされています
  * <https://gist.github.com/rzl24ozi> で最小構成に整理されたIMEパッチがアップされたので、それを適用しました  
パッチを整理していただきありがとうございます！  
IMEを有効にするには以下の設定が必要です  
  ```emacs-lisp
    ;; (set-language-environment "UTF-8") ;; UTF-8 でも問題ないので適宜コメントアウトしてください
    (setq default-input-method "W32-IME")
    (setq-default w32-ime-mode-line-state-indicator "[--]")
    (setq w32-ime-mode-line-state-indicator-list '("[--]" "[あ]" "[--]"))
    (w32-ime-initialize)
    ;; 日本語入力時にカーソルの色を変える設定 (色は適宜変えてください)
    (add-hook 'w32-ime-on-hook '(lambda () (set-cursor-color "coral4")))
    (add-hook 'w32-ime-off-hook '(lambda () (set-cursor-color "black")))

    ;; 以下はお好みで設定してください
    ;; 全てバッファ内で日本語入力中に特定のコマンドを実行した際の日本語入力無効化処理です
  
    ;; ミニバッファに移動した際は最初に日本語入力が無効な状態にする
    (add-hook 'minibuffer-setup-hook 'deactivate-input-method)
  
    ;; isearch に移行した際に日本語入力を無効にする
    (add-hook 'isearch-mode-hook '(lambda ()
                                    (deactivate-input-method)
                                    (setq w32-ime-composition-window (minibuffer-window))))
    (add-hook 'isearch-mode-end-hook '(lambda () (setq w32-ime-composition-window nil)))
  
    ;; helm 使用中に日本語入力を無効にする
    (advice-add 'helm :around '(lambda (orig-fun &rest args)
                                 (let ((select-window-functions nil)
                                       (w32-ime-composition-window (minibuffer-window)))
                                   (deactivate-input-method)
                                   (apply orig-fun args))))
  ```

### 注意事項
* 言語の詳細設定の「アプリウィンドウごとに異なる入力方式を設定する」にチェックを入れていると日本語入力に切り替わらない問題があるようです。これは26.3でも変わっていません。  
詳細は以下の chuntaro氏のissue をご参照ください。  
https://github.com/chuntaro/NTEmacs64/issues/3

ビルド方法
----------

### ビルド環境
基本的にchuntaro氏のNTEmacs 25.2の場合とほぼ同様です。

|エディション|バージョン|OS ビルド|
|---|---|---|
|Windows 10|1709|16299.1087|

### MSYS2 のインストール
<http://msys2.github.io/>
から **msys2-x86_64-20190524.exe** (2019/10/23 時点の最新) を取得し、インストールします。

### 64ビット環境用のシェルの起動

スタートメニューの **MSYS2 MinGW 64-bit** で MSYS2 のシェルを起動後、上記サイトに記載されているように MSYS2 を最新にしておきます。

    $ pacman -Syu

    MSYS2 のシェルを再起動します
    上記コマンドを初めて実行すると、一旦終了してもう一度実行するように促されるので、
    右上の×をクリックしてシェルを終了後、もう一度 $ pacman -Syu を実行します

    $ pacman -Su

### ビルド関連パッケージのインストール
    http://git.savannah.gnu.org/cgit/emacs.git/tree/nt/INSTALL.W64
    に書いてあるように以下のコマンドでビルド関連のパッケージをインストールします

    $ pacman -S base-devel \
      mingw-w64-x86_64-toolchain \
      mingw-w64-x86_64-xpm-nox \
      mingw-w64-x86_64-libtiff \
      mingw-w64-x86_64-giflib \
      mingw-w64-x86_64-libpng \
      mingw-w64-x86_64-libjpeg-turbo \
      mingw-w64-x86_64-librsvg \
      mingw-w64-x86_64-libxml2 \
      mingw-w64-x86_64-gnutls \
      mingw-w64-x86_64-zlib

### ソースの取得と検証
    $ wget http://ftpmirror.gnu.org/emacs/emacs-26.3.tar.gz
    $ wget http://ftpmirror.gnu.org/emacs/emacs-26.3.tar.gz.sig
    $ wget http://ftp.gnu.org/gnu/gnu-keyring.gpg

    Emacs のソースと GNU 関連の公開鍵を取得し、ソースを検証します


    $ gpg --verify --keyring ./gnu-keyring.gpg emacs-26.3.tar.gz.sig
    gpg: keybox'/c/home/ando//.gnupg/pubring.kbx'が作成されました
    gpg: 署名されたデータが'emacs-26.3.tar.gz'にあると想定します
    gpg: 2019年08月29日 05時58分51秒 JSTに施された署名
    gpg:                RSA鍵D405AA2C862C54F17EEE6BE0E8BCD7866AFCF978を使用
    gpg: /c/home/***//.gnupg/trustdb.gpg: 信用データベースができました
    gpg: "Nicolas Petton <nicolas@petton.fr>"からの正しい署名 [不明の]
    gpg:                 別名"Nicolas Petton <petton.nicolas@gmail.com>" [不明の]
    gpg:                 別名"Nicolas Petton <nicolas@foretagsplatsen.se>" [不明の]
    主鍵フィンガープリント: 28D3 BED8 51FD F3AB 57FE  F93C 2335 87A4 7C20 7910

    https://www.gnu.org/software/emacs/download.html
    上記サイトの中ほどにフィンガー・プリントが記載されているので確認します

    https://gist.github.com/rzl24ozi
    上記サイトよりIMEパッチ emacs-26.1-rc1-w32-ime.diff を取得します

### ビルドとインストール
    これも基本的にchuntaro氏のNTEmacs 25.2の場合とほぼ同様です。

    $ zip -dc emacs-26.3.tar.gz | tar xvpf -
    $ cd emacs-26.3/

    ソースを展開してディレクトリに移動します。

    Emacs はビルドしたディレクトリのフルパスを emacs.exe に含めて記憶し
    *Help* から C のソースへ飛ぶ場合にそのパスを使います。(もちろん init.el で変更可能)
    なので、ソースは c:\ の直下に展開した方がいいです。(個人的なディレクトリ名を含めない為でもあります)

    私はc:/emacs26.3/emacs-26.3にソースを展開し、c:/emacs-26.3にバイナリを作成しました。

    $ patch.exe -p0 < ../emacs-26.1-rc1-w32-ime.diff
    $ autoconf

    IMEパッチを適用後 configure スクリプトを更新します。

    $ CFLAGS='-Ofast -march=x86-64 -mtune=corei7 -static' ./configure --prefix=c:/emacs-25.2 \
      --without-dbus --without-compress-install --with-modules

    configure を実行します。
    その際に CFLAGS で最適化オプションを指定します。
    -static は公式ビルドで指定されていたので同じように指定します。

    configure のオプションは --with-modules 以外は公式ビルドで指定されていたので同じように指定します。
    --with-modules でダイナミックモジュールが有効になります。

    25.2 から --prefix を付けないと c:/msys64/mingw64/ 以下にインストールされてしまうようなので --prefix を付けます
    インストール先は適宜変更してください。

    $ make bootstrap && make install-strip

    終了後 --prefix で指定したディレクトリにインストールされているはずです。

## 謝辞
私は、X68000時代にmicroEmacsに出会ってから、Nemacsやmuleを経て、現在もEmacsを使い続けています。
といっても、なかなか仕事マシンでEmacsを使う機会に恵まれなかったため、muleあたりからいきなり
Emacs25.2ぐらいまで飛んでいるのですが。

LinuxとWindowsで同時に使っている時期もありましたが、現在はWindows環境のみなので、余計必要性に駆られているというのもありました。


最後に、このような素晴らしいソフトウェアを生み出し改良し続けている関係者の方々に最大限の感謝をこめて。  





ビルド関連追記
--------------

### emacs-26.3/bin/*.dll の依存関係など
* 以下の DLL 以外追加したファイルはありません
* DLL は全て MSYS2(mingw64/bin) からコピーしたものです
* 依存関係は Windows に標準インストールされているものは含めていません
```
 XPM
 libXpm-noX4.dll
 
 JPEG
 libjpeg-8.dll
 
 PNG
 libpng16-16.dll
 └ zlib1.dll
 
 GIF
 libgif-7.dll
 
 TIFF
 libtiff-5.dll
 ├ libjpeg-8.dll
 ├ liblzma-5.dll
 └ zlib1.dll
 
 LIBXML2
 libxml2-2.dll
 ├ libiconv-2.dll
 ├ liblzma-5.dll
 └ zlib1.dll
 
 GnuTLS
 libgnutls-30.dll
 ├ libwinpthread-1.dll
 ├ libgmp-10.dll
 ├ libhogweed-4.dll
 │ ├ libgmp-10.dll
 │ └ libnettle-7.dll
 ├ libidn-11.dll
 │ ├ libiconv-2.dll
 │ └ libintl-8.dll
 ├ libintl-8.dll
 │ └ libiconv-2.dll
 ├ libnettle-6.dll
 ├ libp11-kit-0.dll
 │ ├ libffi-6.dll
 │ └ libintl-8.dll
 ├ libtasn1-6.dll
 ├ libunistring-2.dll
 │ └ libiconv-2.dll
 └ zlib1.dll
```


------------
Akari Kaminoto / akari@bear.club.ne.jp 