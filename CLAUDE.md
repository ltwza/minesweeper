# 扫雷游戏 — AI 项目上下文

## 项目概述
纯前端扫雷游戏（HTML + CSS + JavaScript 单页应用），无后端依赖。部署在 GitHub Pages。

## 技术栈
- 纯原生 HTML / CSS / JavaScript（无框架）
- localStorage：设置持久化（ms_settings）、存档（ms_saves）
- IndexedDB：自定义音乐文件存储（ms_custom_track_db）
- Web Audio API：decodeAudioData 解码 + BufferSourceNode 循环播放
- CSS Grid：棋盘网格
- 响应式布局：动态计算 cell size

## 核心文件
- `index.html` — 唯一源码文件（HTML + CSS + JS 全部内联）
- `bg1.mp3` / `bg2.mp3` / `bg3.mp3` — 三首背景音乐
- `README.md` — 项目说明文档
- `CLAUDE.md` — AI 上下文（本文件）

## 版本号
- 文件头部注释：`v=1.0`
- 发布部署时通过 URL 参数 `?v=X.X` 控制缓存

## 数据结构

### localStorage: ms_settings
```json
{
  "sfxVolume": 0.7,
  "bgVolume": 0.5,
  "bgTrack": 0,
  "customTrackName": "xxx.mp3"
}
```
- `bgTrack`: 0=bg1, 1=bg2, 2=bg3, 3=自定义音轨

### localStorage: ms_saves
```json
[
  { "name": "自动存档", "auto": true, "status": "playing", "board": [...], "timer": 30, ... },
  { "name": "存档2", "difficulty": "easy", "difficultyLabel": "简单 9×9", ... }
]
```
- 数组格式，slot 0 始终为自动存档
- `status`: "playing" | "won" | "lost" | "cleared"（cleared = 自动存档被清空）
- 最多 10 个元素（slot 0 + 9 手动存档）

### localStorage: ms_records
```json
[
  { "difficulty": "easy", "difficultyLabel": "简单 9×9", "time": 42, "timeStr": "00:42", "date": 1700000000000, "dateStr": "2026/6/19", "rows": 9, "cols": 9, "mines": 10 }
]
```
- 扁平数组，按时间倒序，最多 10 条

### localStorage: ms_custom_config
```json
{ "rows": 15, "cols": 15, "mines": 50, "timeLimit": 0 }
```

### IndexedDB: ms_custom_track_db / tracks store
```json
{ "id": "custom_track", "name": "song.mp3", "data": ArrayBuffer }
```

## 存档系统规则（重要）
1. slot 0 始终固定，不会被删除或移除
2. 删除自动存档只清空内容（status='cleared', board=[]），保留 slot 位置显示"空"
3. 自动存档没有"覆盖"按钮
4. 自动存档名称不可编辑
5. 空的手动存档（slot > 0）不显示在弹窗中，仅保留最后一个空 slot 供"保存"
6. 存档弹窗中自定义难度显示雷数（如"自定义 50雷 (15×15)"）

## 排行榜
按难度拆分 5 个榜单：easy/medium/hard/hell/custom
- easy/medium/hard/hell：按用时从少到多排序
- custom：按通关时间倒序
- 每个榜单最多 10 条

## 音乐系统
- AudioManager 单例管理播放
- 3 个 MP3 通过 Audio 元素 + HTMLMediaElement 循环播放
- 自定义音轨通过 decodeAudioData 解码 + BufferSourceNode 循环播放
- 设置中可切换音轨、调节音效/背景音量
- 爆炸音效：sine 低频下扫 + 低通噪声

## 操作方式
- **电脑端**：左键揭格、右键插旗、双击 chord 揭周围
- **手机端**：点击揭格；**游戏开始后**长按弹出插旗菜单（可拖动选择"点开"/"插旗模式"）；游戏未开始时短按和长按都直接揭格开始游戏
- 插旗模式下点击格子自动插旗，屏幕顶部显示黄色指示条
- 切换难度自动退出插旗模式

## 本地测试（手机端）
### 开启 HTTP 服务器
```bash
cd d:\cocos\cocosProjects\trae_test
python -m http.server 8080
```

### 生成二维码（手机扫码访问）
```bash
# 在另一个终端执行：
$ip = (Get-NetIPAddress -AddressFamily IPv4 | Where-Object { $_.IPAddress -match '^192\.' }).IPAddress
start "https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=http://$ip`:8080"
```
> 手机和电脑需在**同一 WiFi** 下。

### 关闭服务器
```bash
# 查找并杀掉占用 8080 端口的进程
netstat -ano | findstr :8080
taskkill /PID <进程ID> /F
```

## 已知约定
- confirm 弹窗已替换为自定义 showConfirm Promise
- 所有 CSS 在 `<style>` 标签内，JS 在 `<script>` 标签内，均在 index.html
- 已添加缓存控制 meta 标签
