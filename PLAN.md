# macOS arm64 対応と GitHub Actions 自動リリース計画

## Summary
- 配布対象は **macOS arm64 のみ**にする。
- GitHub Release は **`v*` タグ push** で作成し、成果物は **未署名の zip** とする。
- 現在同梱されている `XADMaster.framework` と `UniversalDetector.framework` は x86_64 のみなので、ソースから arm64 用に再ビルドできる形へ移行する。
- 先例として、cooViewer の arm64 対応フォークでは依存 framework を submodule 化し、`xcodebuild -configuration Deployment -arch arm64` でビルドしている。

## Key Changes
- 依存 framework を再現可能にする。
  - `XADMaster` と `UniversalDetector` をソース依存として固定する。
  - 既存の x86_64-only framework バイナリは、arm64 ビルド済み framework に置き換えるか、CI 内で生成してからアプリに組み込む。
  - 実装時は submodule 方式を第一候補にし、依存元リポジトリと commit を固定する。

- Xcode project を arm64 リリースビルド向けに整理する。
  - `Deployment` configuration で `ARCHS=arm64` を明示する。
  - CI では `xcodebuild -project cooViewer.xcodeproj -scheme cooViewer -configuration Deployment -arch arm64 build` を基準にする。
  - `MACOSX_DEPLOYMENT_TARGET=10.8` は現代 Xcode で問題になる可能性が高いため、arm64 ビルドで必要なら最小限の引き上げを行う。Sequoia/Xcode 15+ で失敗する場合は `15.0` まで引き上げる判断を実装側で確認する。

- GitHub Actions workflow を追加する。
  - `.github/workflows/release.yml` を作成する。
  - `on.push.tags: ["v*"]` で起動する。
  - `macos-latest` で checkout、submodule 初期化、依存 framework ビルド、cooViewer arm64 ビルドを実行する。
  - `build/Deployment/cooViewer.app` を `ditto -c -k --sequesterRsrc --keepParent` で `cooViewer-${{ github.ref_name }}-macos-arm64.zip` に固める。
  - `softprops/action-gh-release` または `gh release create` で GitHub Release に zip を添付する。
  - 署名・公証は行わない。

## Test Plan
- ローカルまたは Actions 上で以下を確認する。
  - `xcodebuild ... -configuration Deployment -arch arm64 build` が成功する。
  - `lipo -info build/Deployment/cooViewer.app/Contents/MacOS/cooViewer` が `arm64` を返す。
  - 同梱 framework の `lipo -info` も `arm64` を返す。
  - `otool -L cooViewer.app/Contents/MacOS/cooViewer` で framework 参照が `@executable_path/../Frameworks/...` として解決される。
  - zip を展開して `.app` 構造と `Contents/Frameworks` が保持されている。
  - Apple Silicon Mac 上で起動し、画像ファイルと zip/cbz/rar 系アーカイブを最低1つずつ開ける。

## Assumptions
- リリースは `v1.2b26` のような Git tag 名を唯一の Release 名・zip 名のバージョンとして扱う。
- 配布物は arm64 のみで、x86_64/Universal は今回の対象外。
- 未署名配布のため、初回起動時の Gatekeeper 警告は許容する。
- 参照した先例: [P-Life の cooViewer Apple Silicon 対応記事](https://plife.techwp.net/2022/09/19/build-apple-silicon-ed-cooviewer-2/)
