# cooViewer Appleシリコン対応版
これは、[cooViewer](https://github.com/coo-ona/cooViewer)をForkしAppleシリコン用にビルドするリポジトリです。
`v*` 形式のタグを GitHub に push すると、GitHub Actions が未署名の macOS arm64 zip を Release に添付します。
Release zip にはアプリ本体と第三者ライセンス文書を同梱します。

## ビルド
依存ライブラリは submodule として管理しています。

```
git submodule update --init --recursive
xcodebuild -project cooViewer.xcodeproj -scheme cooViewer -configuration Deployment -arch arm64 CODE_SIGNING_ALLOWED=NO MACOSX_DEPLOYMENT_TARGET=11.0 build
```

## 著作権、免責等
cooViewerはMITライセンスです。
ライセンスについては添付のLicence.txtを参照してください。

このソフトウェアはXAD library system ( http://sourceforge.net/projects/libxad/ ) を使用しています。<br>
ライセンスについては添付のLicence_xad.txtを参照してください。

このソフトウェアはRemote Control Wrapper ( http://www.martinkahr.com/source-code/ ) を使用しています。<br>
ライセンスについては添付のLicence_RemoteControlWrapper.txtを参照してください。
