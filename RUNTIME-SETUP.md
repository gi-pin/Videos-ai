# 影片執行環境（2026-10-22）

吉兒與騰益兩份 checkout 均已建立獨立 `.venv`（Python 3.11.15），安裝核心依賴、Piper TTS、MCP 測試 SDK 與 pytest；兩份 Python 套件版本完全一致，快照見 `requirements-workspace.lock.txt`。Remotion 依既有 package-lock.json 以 `npm ci` 安裝。

在這個儲存庫目錄執行，路徑有空格／括號時必須加引號：

```sh
source .venv/bin/activate
python -c 'from tools.tool_registry import registry; import json; registry.discover(); print(json.dumps(registry.provider_menu_summary(), indent=2))'
python render_demo.py --list
python render_demo.py code-to-screen
make hyperframes-doctor
```

最後一個 render_demo 指令會輸出完整原廠範例；本次驗收只渲染該範例的前 30 幀、半尺寸。正式影片製作仍須先讀 `AGENT_GUIDE.md` 並遵循所選 pipeline。

## 還原安裝

```sh
uv venv --python 3.11 .venv
uv pip install --python .venv/bin/python -r requirements-workspace.lock.txt
npm ci --prefix remotion-composer
npx --yes hyperframes@0.8.185 --version
```

Piper 英文測試模型 `en_US-lessac-medium` 已放在本機 `~/.local/share/piper/voices/`，兩份 checkout 根目錄的 `.onnx` 與 `.onnx.json` 為指向該模型的連結，已由既有 Git 忽略規則排除。重裝時可先啟用 venv，再執行：

```sh
python -m piper.download_voices --download-dir "$HOME/.local/share/piper/voices" en_US-lessac-medium
ln -s "$HOME/.local/share/piper/voices/en_US-lessac-medium.onnx" .
ln -s "$HOME/.local/share/piper/voices/en_US-lessac-medium.onnx.json" .
```

已有模型或連結時不用重建。此英文模型是環境測試用途，並未驗證中文配音品質。

## 實測結果與界線

- 兩份 checkout 均輸出 H.264 960×540 MP4（30 幀，容器約 1.045 秒）及 22,050 Hz PCM WAV。
- `tools.audio.piper_tts.PiperTTS.execute` 在兩份 checkout 均成功，包含預設模型名稱解析。
- FFmpeg、Remotion、HyperFrames 均被原廠 registry 判為可用；HyperFrames 0.8.185 doctor 的核心 runtime 檢查通過，但尚未實作 HyperFrames 成片測試。
- HyperFrames doctor 仍列出選配 whisper-cpp、Kokoro、MusicGen 與 Docker 未安裝；這些不是本次 Remotion／Piper 測試的依賴。
- registry 目前回報雲端／其他生圖 0/23、生影片 0/31、音樂生成 0/5 可用；沒有填入或呼叫付費 API。這是本機設定偵測，並非外部服務登入驗證。
- 原始測試媒體留在 `projects/runtime-smoke/renders/`（不進 Git）；沒有發佈影片、啟動正式製作或下載 GPU 模型。

原先 `WORKSPACE-INSTALL.md` 的「僅安裝技能、未測試執行環境」為初始安裝狀態；本文件更新以上已實測項目，其餘外部服務仍未驗證。
