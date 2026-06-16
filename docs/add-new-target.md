# 如何新增一個 Build Target

## 前提

- 你已知道新 target 的 `MACHINE` 名稱（通常在 BSP layer 的文件裡）
- 你已知道要加哪個 BSP layer（Git URL + branch）

---

## 步驟

### 1. 建立 `kas/<target-name>.yml`

只寫「跟 base 不同的部分」，其餘由 `includes: base.yml` 繼承。

**最小範例（公開 layer，無額外 BSP）：**

```yaml
header:
  version: 14
  includes:
    - base.yml

machine: <MACHINE_NAME>
```

**有額外 BSP layer 時（加在 repos 區段）：**

```yaml
header:
  version: 14
  includes:
    - base.yml

machine: <MACHINE_NAME>

repos:
  meta-<bsp>:
    url: https://github.com/<org>/meta-<bsp>
    branch: scarthgap
    layers:
      meta-<bsp>:
```

> `repos` 區段是疊加的，不是覆蓋——base.yml 裡的 poky 仍然有效。

### 2. 本機驗證

在 WSL2 Linux 環境（非 `/mnt/c/...`）執行：

```bash
kas build kas/<target-name>.yml
```

或用 Docker：

```bash
docker run --rm \
  --user $(id -u):$(id -g) \
  -v "$(pwd):/builder" \
  -v "$HOME/yocto-kas/sstate:/sstate-cache" \
  -v "$HOME/yocto-kas/downloads:/downloads" \
  ghcr.io/siemens/kas/kas \
  build /builder/kas/<target-name>.yml
```

**驗收**：build 完成、`build/tmp/deploy/images/<MACHINE_NAME>/` 目錄有產物。

### 3. 更新 CI（如需 CI 支援）

在 `.github/workflows/build.yml` 新增一個 job，參考現有的 `qemu` job，修改：
- `kas/qemu.yml` → `kas/<target-name>.yml`
- artifact 名稱與路徑中的 `qemux86-64` → `<MACHINE_NAME>`

### 4. 開 PR

- 分支從 `dev` 開，命名建議：`add-target-<target-name>`
- PR description 說明：machine 名稱、對應的硬體/平台、用到的 BSP layer 來源

---

## 不要做的事

| 錯誤做法 | 正確做法 |
|---------|---------|
| 把 base.yml 整個複製一份再改 | 只寫 `includes: base.yml` + 差異 |
| 為這個 target 開一個新資料夾 | 一個 target = 一個 `.yml` 檔在 `kas/` |
| 為這個 target 開一條新分支維護 | 所有 target 在同一條分支並列維護 |

---

## 完成後的檔案清單

```
kas/
├── base.yml              ← 沒有動過
├── qemu.yml              ← 沒有動過
└── <target-name>.yml     ← 你新增的
```
