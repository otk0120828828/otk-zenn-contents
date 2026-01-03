---
title: "Ubuntu + Lutris における日本版『黒い砂漠』環境構築ログ"
author: "otk"
date: 2026-01-02
emoji: 🎮
type: idea
topics: ["ubuntu", "lutris"]
published: true
---

# Ubuntu + Lutris における日本版『黒い砂漠』環境構築ログ
※AIの要約を (備忘録的に) そのまま記事にしました。

## 1. 概要
* **目的**: Ubuntu (Linux) 上の Lutris を使用して、Steam版ではない「日本公式版（Pearl Abyss版）」の黒い砂漠を動作させる。
* **環境**:
    * OS: Ubuntu (24.04系)
    * GPU: Hybrid構成 (AMD Radeon 680M + NVIDIA RTX 3060 Laptop)
    * Runner: Lutris-GE-Proton / Wine-GE
* **結果**: インストール完了およびランチャー起動、ゲームプレイ可能状態へ到達。

---

## 2. 実施した主な作業プロセス

### フェーズ1：Lutrisの基本セットアップ
1.  **Lutrisへのゲーム登録**:
    * 自動スクリプトは使用せず、「Add locally installed game」から手動で登録。
    * Runnerには「Wine」を選択。
2.  **Runner (Wine) の導入**:
    * Steam用の `GE-Proton` と Lutris用の `lutris-ge-proton` の違いを理解し、Lutrisに適したバージョンを導入。
3.  **グラフィック設定**:
    * ハイブリッドGPU環境のため、NVIDIA RTX 3060 を優先使用するように設定 (`Enable NVIDIA Prime Render Offload: ON`).

### フェーズ2：依存ライブラリの解決
1.  **32bitアーキテクチャの追加**:
    * `dpkg --add-architecture i386` を実行。
2.  **必須パッケージのインストール**:
    * `wine32`, `libnvidia-gl:i386`, `libvulkan1:i386` などの導入。

### フェーズ3：インストールとパス修正
1.  **インストーラーの実行**:
    * ターミナル経由で環境変数を指定し、強制的にインストーラーを起動。
2.  **実行ファイルの切り替え**:
    * インストール完了後、Lutrisの参照先をインストーラー (`.exe`) からゲームランチャー (`BlackDesertLauncher.exe`) に変更。

---

## 3. 発生した問題とアプローチ

### トラブル①：GameModeのエラーで即時終了する
* **現象**: ログに `ERROR: ld.so: object 'libgamemodeauto.so.0'... wrong ELF class` が表示され起動しない。
* **原因**: 32bitインストーラーに対し、64bit版のGameModeライブラリがロードされようとして競合していた。
* **アプローチ**:
    * LutrisのSystem optionsで **「Enable Feral GameMode」をOFF** に設定。

### トラブル②：`Invalid Wine prefix path` エラーのループ
* **現象**: 設定保存時や起動時に「Prefixのパスが無効である」という `OSError` が発生。
* **原因**:
    1.  **設定ミス**: Lutrisの `Wine version` 指定で、フォルダではなく内部のバイナリファイル (`/bin/wine`) を直接指定していた。
    2.  **構造不一致**: 手動配置したProtonのフォルダ構造が、Lutrisの期待する構造（直下に `bin/` がある状態）と異なっていた。
* **アプローチ**:
    * `Wine version` の指定を「ファイル選択」から**「ドロップダウンリストからの選択」**（またはフォルダ指定）に変更。
    * 手動ダウンロードしたProtonの中身を整理し、正しいディレクトリ構造に配置し直した。

### トラブル③：インストーラーが起動しない (`Return code: 256`)
* **現象**: エラーログすら出ずにプロセスが即終了する、または `kernel32.dll not found (status c0000135)` が出る。
* **原因**:
    * インストーラー自体は32bitアプリケーションだが、Ubuntuシステム側に **32bit版のWine実行環境 (`wine32`) が欠落** していた。
* **アプローチ**:
    * `sudo apt install wine32:i386` を実行し、OSレベルでの32bitサポートを完備させた。

### トラブル④：パッケージリポジトリのエラー (`questing Release`)
* **現象**: `apt install` 実行時にリポジトリエラーが出てインストールできない。
* **原因**: 存在しないUbuntuバージョン名（コードネーム `questing`）が誤ってリポジトリリストに登録されていた。
* **アプローチ**:
    * `add-apt-repository --remove` で壊れたPPAを削除。
    * `universe` リポジトリを正しく有効化 (`add-apt-repository universe`)。

---

## 4. 最終的な成功構成（Lutris設定）

| 設定項目 | 値 / 状態 |
| :--- | :--- |
| **Runner** | Wine (lutris-ge-proton8-26-x86_64) |
| **Executable** | `.../drive_c/Program Files (x86)/BlackDesert/BlackDesertLauncher.exe` |
| **DXVK / VKD3D** | ON / ON |
| **NVIDIA Prime** | ON (RTX 3060使用) |
| **Feral GameMode**| OFF |
