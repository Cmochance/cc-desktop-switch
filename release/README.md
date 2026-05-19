# release/ —— 测试 fixture(非 cc-desktop-switch 自己的发布物)

本目录里所有文件都是从姊妹仓库 `Cmochance/codex-app-transfer` 拉来的**测试固件**,
**不**是 cc-desktop-switch 的官方 release 资产:

| 文件 | 用途 | 来源 |
|---|---|---|
| `Codex-App-Transfer-release-public.pem` | `src-tauri/src/admin/signature.rs` `include_str!` 引用,验签客户端嵌入公钥 | codex-app-transfer v2.1.6 release |
| `latest.json` + `latest.json.sig` | `signature.rs` / `update.rs` 几个验签测试的真签名 fixture(`real_release_latest_json_signature_verifies` / `tampered_data_rejected` / `trims_whitespace_around_signature` / `fetch_latest_json_verifies_real_signature_end_to_end`) | codex-app-transfer v1.0.3 真实发布的签名 manifest |

## 已知问题(留 Phase 2 解决)

- **`latest.json` 里的 asset URL 是 relative**(例如 `"url": "Codex-App-Transfer-v1.0.3-Linux-x86_64"`):chatgpt-codex-connector 在 PR #2 review thread 指出,如果真把这份 manifest 当成 update endpoint 内容发给客户端,`download_asset_impl` 里的 `validate_update_url` 会拒(只接受 http / https),导致更新检查通过但下载失败。
  - **当前不影响 prod**:此文件**只在 cargo test 走 mock server 验签时被读**;客户端默认 update URL 指向 GitHub release 的 latest.json,不是这份 fixture。
  - **不能修这一份**:重新写 URL 会让 RSA 签名失效,而我们没有 codex-app-transfer 私钥。
  - **Phase 2 fix**:`docs/refactor/codex-app-transfer-id-cleanup.md` 里规划的 release pipeline 重生成时,用 cc-desktop-switch 自己的私钥重新签 `latest.json`,届时统一改用绝对 URL(`https://github.com/Cmochance/cc-desktop-switch/releases/download/.../...`)。

## 真 release 怎么走

正式发布通过 GitHub Actions `release.yml`(`.github/workflows/release.yml`)出包 + 签名 + 上传到 GitHub Releases。end-user 客户端的 update 检查从 GitHub release 的 `latest.json` 拉,**不**读本仓的 `release/latest.json`。
