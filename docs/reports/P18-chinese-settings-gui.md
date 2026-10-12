# P18 中文設定頁與正體中文介面

**日期：** 2026-10-12
**結果：** ✅ PASS（使用者在啟動器與遊戲中測試：「目前測試很順暢」）
**引擎：** `ultima8-zh-tw-dev` @ `e1220ab65f`、`0da37d5d54`

## 目標

使用者在 U7（Exult）專案做過遊戲內的「Chinese Setup」選單。U8 討論後採方案 A：在 ScummVM 的遊戲設定加一個中文設定頁，並把 ScummVM 介面改為正體中文。U7 的「英文強制套用中文體」與「UI 介面等比縮放」不做（使用者決定）。

## 中文設定頁

`engines/ultima/ultima8/misc/options_widget.{h,cpp}`：`UltimaMetaEngine::buildEngineOptionsWidget` 對 `ultima8` 回傳 `Ultima8::OptionsWidget`（其他 Ultima 遊戲沿用原本的勾選框）。

- 上方：原本的遊戲選項勾選框（與 `GUI::ExtraGuiOptionsWidget` 相同的讀寫方式）。
- 下方各區塊：對話文字（高解析 / 點陣）、台詞框、書本與捲軸、墓碑與牌匾、操作；最後是「恢復預設」。
- 字型：下拉選單，列出 extrapath（含命令列 `--extrapath`，在暫時設定區）、遊戲資料夾、macOS bundle（`__OSX_BUNDLE__`）中的 `.ttf` / `.otf`。
- 數值：滑桿，0.1 單位的設定（高解析的字距等）用 10 倍刻度。
- 書本與捲軸、墓碑與牌匾各寫兩個設定（`font_cjk_book_*` + `font_cjk_scroll_*` 等）。
- 寫入規則：只有改過的值才寫；等於繼承值（`[scummvm]` 或預設）時從 `[ultima8]` 移除，避免把預設值寫死在 ini。
- 即時生效：引擎在設定視窗關閉後呼叫 `applyGameSettings()` → `setupFontOverrides()`，本來就會重新建立中文字型，不需另外處理。

## 正體中文介面

- ScummVM 官方 `po/zh_Hant.po` 是空的（3,257 條全未翻）。`tools/gui/make_zh_hant.py` 由 `zh_Hans.po`（2,903 條）以 OpenCC `s2twp` 轉為台灣用語，再修正常見詞（全域性→全域、好的→確定、鍵盤佈局→按鍵對應、聲音字型→SoundFont 等），並加入 U8 遊戲選項與設定頁的 68 條。
- `gui/themes/translations.dat` 以全部 po 重建（WSL 編譯 `devtools/create_translations`），Windows 的 `scummvm.exe` 會內嵌它；macOS bundle 也會複製它。介面字型（思源黑體繁中）原本就在內嵌的 `fonts-cjk.dat` 裡。
- 套件預設介面語言：`package/scummvm.ini` 的 `gui_language=zh_Hant`。macOS 版沒有套件設定檔，安裝說明告知如何切換。
- 玩家文件的按鈕名稱改為中文介面上的字（附英文）。

## 驗證

- MSVC 與 WSL g++ 都能編譯（g++ 無警告）。
- 執行檔內含新的譯文字串；遊戲啟動正常。
- 使用者測試：啟動器、遊戲設定、遊戲中「4.選項」皆正常。

## 已知限制

- 355 條連簡體中文都沒有翻譯的介面字串維持英文。
- 介面翻譯是自動轉換加局部修正，用詞若有不順，修改 `make_zh_hant.py` 的 FIXES / EXACT 後重新產生。
