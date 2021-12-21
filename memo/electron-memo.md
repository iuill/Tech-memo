# Electron メモ書き

## 参考

最新版で学ぶElectron入門 ウェブ技術でPCアプリを開発しよう  
https://ics.media/entry/7298/

## バージョン

```console
> node --version
> v16.13.1
```

```console
> npm --version
> 8.1.2
```

## インストール

```console
> npm init -y
> npm install --save electron
```

```console
> npm view electron version
> 16.0.5
```

## サンプルコード

略

## アプリ実行

プロジェクトのフォルダで下記実行

```console
> npx electron ./src
```

## パッケージング(exe生成)

パッケージ名を myApp、ソースフォルダを src として。

```console
> npm install --save electron-packager
> npx electron-packager src myApp --platform=win32 --arch=x64 --overwrite
```

