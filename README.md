# yocto-ci-sbom

團隊共用的 Yocto build 設定 repo。「產品怎麼被 build 出來」的 single source of truth。

> 所有 Yocto 檔案（build 產物、sstate-cache、downloads）要放在 **Linux 檔案系統**（WSL2 `~/`），不要放在 `/mnt/c/...`。

---

## 目錄結構

```
repo/
├── kas/                  # kas 設定檔（base + 各 target）
├── docker/               # Build 環境容器定義（目前使用官方 kas 映像）
├── scripts/              # Glue scripts：SBOM 驗證、artifact 上傳、通知等
├── docs/                 # 使用與維護說明
└── .github/workflows/    # GitHub Actions CI pipeline
```

## Kas 設定架構：Base + Overlay

共用的東西集中在 `kas/base.yml`，每個 target 只寫「跟別人不同的部分」，透過 `header.includes` 疊加：

```
kas/
├── base.yml        ← 所有 target 共用（poky、DISTRO、共用 conf 變數）
├── qemu.yml        ← qemux86-64（本機/CI 練習，不需硬體）
└── <target>.yml    ← 之後各硬體 target 各一份
```

**不要複製 base.yml 的內容到各 target**，保持 DRY。平台差異用「不同的 .yml 檔」表達，不要用資料夾或 git 分支區分。

---

## Quick Start

```bash
# 建置 qemu target
kas build kas/qemu.yml

# 進入 devshell 除錯
kas shell kas/qemu.yml

# 用 Docker 直接跑（本機無 kas 時）
docker run --rm \
  --user $(id -u):$(id -g) \
  -v "$(pwd):/builder" \
  -v "$HOME/yocto-kas/sstate:/sstate-cache" \
  -v "$HOME/yocto-kas/downloads:/downloads" \
  ghcr.io/siemens/kas/kas \
  build /builder/kas/qemu.yml
```

---

## 分支策略

| 分支 | 用途 |
|------|------|
| `main` | 穩定、驗證過，組員可直接使用 |
| `dev` | 開發中、未驗證 |

**所有變更請走 PR review**，包含 `kas/*.yml`——改一行版本就可能影響整包 build。

---

## 新增 Build Target

見 [`docs/add-new-target.md`](docs/add-new-target.md)。

## CI Runner 需求

Workflow 使用 `runs-on: self-hosted`，runner 機器需要：
- Docker（執行 `ghcr.io/siemens/kas/kas` 容器）
- 掛載點：`$HOME/yocto-kas/sstate`、`$HOME/yocto-kas/downloads`（共約 50 GB+）
