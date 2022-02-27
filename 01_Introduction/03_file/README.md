# fileコマンド

## fileコマンドの実行結果
```console
$ file /usr/bin/file
/usr/bin/file: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=2b26928f841d92afa31613c2c916a3abc96bbed8, stripped
```

## -iを付けた場合の結果
```console
$ file -i /usr/bin/file
/usr/bin/file: application/x-sharedlib; charset=binary
```

## fileコマンドの内容
以下の順序に基づいてファイルの種別を判断
- デバイス, ディレクトリ, シンボリックリンクなどのスペシャルファイルチェック  
\[デバイス\]
```console
$ file /dev/tty0
/dev/tty0: character special (4/0)
```
\[ディレクトリ\]
```console
$ file ../03_file/
../03_file/: directory
```
\[シンボリックリンク\]
```console
$ file ./back
./back: symbolic link to ../
```
- 圧縮ファイルのチェック
```console
$ file test.txt.gz 
test.txt.gz: gzip compressed data, was "test.txt", last modified: Sun Feb 27 09:44:06 2022, from Unix
```
- tarファイルのチェック
```console
$ file test.tar
test.tar: POSIX tar archive (GNU)
```
- magicデータベースに基づくチェック。/usr/share/file/magic.mgcと照合して,シグネチャから種別を特定する。
- ASCII, Unicodeなどのテキストファイル種別チェック
