# Foundations of Computer Vision Chapters 1–3 EPUB 製作說明

這個 EPUB 收錄 *Foundations of Computer Vision* 的 Chapter 1–3，使用 Quarto book project 建置。

## 輸出檔案

- EPUB：`Foundations-of-Computer-Vision-—-Chapters-1–3.epub`
- 建置後大小：41,619,621 bytes（約 39.7 MiB）
- SHA-256：`71c014ce219bdf942005b36734522077291223fabcb36fce33002363a0354d1c`

## 教材來源

原始教材為：

- 官方網站：<https://visionbook.mit.edu/>
- 官方 source repository：<https://github.com/Foundations-of-Computer-Vision/visionbook>
- 作者：Antonio Torralba、Phillip Isola、William T. Freeman
- 出版社：MIT Press

本 EPUB 使用以下 Quarto 來源檔：

- `taxonomy.qmd`：Chapter 1, *The Challenge of Vision*
- `simplesystem.qmd`：Chapter 2, *A Simple Vision System*
- `visionscience.qmd`：Chapter 3, *Looking at Images*
- `index.qmd`：電子書首頁
- `figures/`：Chapter 1–3 所使用的圖片

教材網站標示內容採用 **CC BY-NC-ND** 授權。使用、分享或重新發佈前，應自行確認最新授權條款，並保留原作者與來源資訊。本次建置沒有改寫 Chapter 1–3 的教材正文。

## Quarto 設定

`_quarto.yml` 的主要設定如下：

```yaml
project:
  type: book
  output-dir: _gaze

book:
  title: "Foundations of Computer Vision — Chapters 1–3"
  author:
    - Antonio Torralba
    - Phillip Isola
    - William T. Freeman
  chapters:
    - index.qmd
    - taxonomy.qmd
    - simplesystem.qmd
    - visionscience.qmd

format:
  epub:
    toc: true
    syntax-highlighting: none
```

## 建置環境

- Mac architecture：Intel `x86_64`
- macOS：13.7.8 Ventura
- 成功建置的 Quarto：1.6.43
- Quarto 1.6.43 內建 Dart Sass：1.70.0
- Pandoc：3.4.0（由 Quarto 1.6.43 內建）

系統當時安裝的 Quarto 1.11.1 不能用於這台 Mac，因為它所附的 x86_64 Dart Sass runtime 要求最低 macOS 14.0。直接執行 Sass 時會出現：

```text
VM initialization failed: Current Mac OS X version 13.0 is lower than minimum supported version 14.0
```

這不是 arm64/x86_64 選錯的問題。Quarto 確實選到 x86_64 binary，但該 binary 的 `LC_BUILD_VERSION` 標示 `minos 14.0`。

為了不修改 `/Applications` 中的系統 Quarto，本次改用官方 Quarto 1.6.43 macOS tarball。該版 Dart binary 的最低 macOS 版本為 10.14，可在 macOS 13.7.8 上正常執行。

## 下載與驗證 Quarto 1.6.43

官方 release：<https://github.com/quarto-dev/quarto-cli/releases/tag/v1.6.43>

```bash
mkdir -p /private/tmp/visionbook-quarto-1.6.43

curl -fL --retry 3 \
  -o /private/tmp/visionbook-quarto-1.6.43/quarto.tar.gz \
  https://github.com/quarto-dev/quarto-cli/releases/download/v1.6.43/quarto-1.6.43-macos.tar.gz

curl -fL --retry 3 \
  -o /private/tmp/visionbook-quarto-1.6.43/checksums.txt \
  https://github.com/quarto-dev/quarto-cli/releases/download/v1.6.43/quarto-1.6.43-checksums.txt

shasum -a 256 /private/tmp/visionbook-quarto-1.6.43/quarto.tar.gz
grep 'macos.tar.gz' /private/tmp/visionbook-quarto-1.6.43/checksums.txt
```

當時下載檔案與官方 checksum 皆為：

```text
1ead22fde52301cdc07d3c294a6003a05b36f1c570805cfefbff0e8dc4beb2cd
```

解壓並檢查：

```bash
tar -xzf /private/tmp/visionbook-quarto-1.6.43/quarto.tar.gz \
  -C /private/tmp/visionbook-quarto-1.6.43

/private/tmp/visionbook-quarto-1.6.43/bin/quarto --version
/private/tmp/visionbook-quarto-1.6.43/bin/quarto check
```

## 補齊缺少的圖片

第一次 render 顯示本地缺少：

```text
figures/visionscience/shadow.png
```

`visionscience.qmd` 中原本已有這個引用，而官方 repository 也有對應檔案，因此直接從官方 source repository 補下載，沒有修改正文：

```bash
curl -fL --retry 3 \
  -o figures/visionscience/shadow.png \
  https://raw.githubusercontent.com/Foundations-of-Computer-Vision/visionbook/main/figures/visionscience/shadow.png

file figures/visionscience/shadow.png
```

圖片格式檢查結果為 1024 × 667 RGB PNG。補齊後，專案內 Chapter 1–3 圖片總數為 70。

## 建置 EPUB

在專案根目錄執行：

```bash
cd ~/Downloads/visionbook-gaze
/private/tmp/visionbook-quarto-1.6.43/bin/quarto render
```

輸出位置：

```text
_gaze/Foundations-of-Computer-Vision-—-Chapters-1–3.epub
```

## EPUB 基本完整性檢查

```bash
epub='_gaze/Foundations-of-Computer-Vision-—-Chapters-1–3.epub'

ls -lh "$epub"
file "$epub"
shasum -a 256 "$epub"
unzip -t "$epub"
unzip -Z1 "$epub"
```

圖片封裝數量：

```bash
unzip -Z1 "$epub" \
  | grep -Ei '\.(png|jpe?g|gif|svg)$' \
  | wc -l
```

驗證結果：

- `file` 識別為 EPUB document。
- `unzip -t` 報告所有壓縮內容皆正常。
- EPUB 第一個 ZIP member 是未壓縮的 `mimetype`。
- 包含首頁以及 Chapter 1–3。
- EPUB 內封裝 70 張圖片。
- 本機沒有安裝 `epubcheck`，因此做的是 ZIP/manifest/章節/圖片層級的基本 integrity check，不是完整 EPUBCheck conformance test。

## 已知警告

### 尚未收錄的章節參照

Build 會警告無法解析：

```text
@sec-imaging
@sec-image_derivatives
```

這兩個參照指向 Chapter 1–3 以外的後續章節。由於本 EPUB 只收錄前三章，這是預期的非致命警告。

### 舊式 TeX `\bf` 語法

原教材部分公式使用 `\bf`。Pandoc 嘗試轉成 MathML 時會警告 `unexpected control sequence \bf`，並改以 TeX 形式保留。為了避免任意修改教材正文，本次沒有改寫這些公式。

## 重要注意事項

1. 在 macOS 13 上不要直接用本機當時的 Quarto 1.11.1 build；它的 Dart Sass 要求 macOS 14。
2. 不要為了消除 cross-reference 警告而刪除或改寫原教材段落。
3. 圖片路徑與檔名大小寫必須與 QMD 引用一致，因為 GitHub/Linux 環境會區分大小寫。
4. 大型 EPUB 使用 Git push 時，如 HTTP/2 出現 `RPC failed; HTTP 400`，可針對這次 push 改用 HTTP/1.1：

   ```bash
   git -c http.version=HTTP/1.1 \
       -c http.postBuffer=104857600 \
       push origin main
   ```

5. GitHub 一般 Git repository 的單檔限制為 100 MiB。這個 EPUB 約 39.7 MiB，不需 Git LFS；如後續收錄更多章節導致檔案超過限制，應改用 Git LFS 或 GitHub Releases。
6. 公開 repo 前應再次確認 CC BY-NC-ND 授權與教材圖片的使用條件。目前 GitHub repository 設為 private。

## 重新建置的最短流程

在 Quarto 1.6.43 已解壓、圖片也已齊全的情況下：

```bash
cd ~/Downloads/visionbook-gaze

/private/tmp/visionbook-quarto-1.6.43/bin/quarto check
/private/tmp/visionbook-quarto-1.6.43/bin/quarto render

unzip -t '_gaze/Foundations-of-Computer-Vision-—-Chapters-1–3.epub'
shasum -a 256 '_gaze/Foundations-of-Computer-Vision-—-Chapters-1–3.epub'
```

`/private/tmp` 可能被 macOS 清理。如 Quarto 1.6.43 不存在，需先重新下載、核對 checksum 並解壓，或將相容版 Quarto 安裝到持久的 user-local 目錄。
