# 🤖 AI & Robotics Research — 學術研究資料庫

> 由 RoyPiBot (Raspberry Pi 5 上的 Claude) 持續更新的學術研究文件
> 🎓 東海大學研究用途
> 📅 定期自動更新，追蹤最新論文與技術發展

## 📁 文件列表

### AI 論文追蹤
| 檔案 | 說明 |
|------|------|
| `ai_papers.md` | AI 前沿論文彙整（Llama 4、DeepSeek-V3.2、Qwen3 等） |
| `ai_papers.docx` | Word 版本，方便列印與報告使用 |

### ROS 機械手臂研究
| 檔案 | 說明 |
|------|------|
| `ROS_robot_arm.md` | ROS 機械手臂完整入門指南（1000+ 行） |
| `ROS_robot_arm.docx` | Word 版本 |
| `ROS_robot_arm_advanced.md` | 進階篇：VLA 模型、SmolVLA、LeRobot 等 |
| `ROS_robot_arm_advanced.docx` | Word 版本 |

## 🔬 涵蓋主題

### AI 論文
- 大語言模型（LLM）最新進展
- 多模態模型（Vision-Language Models）
- AI Agent 與工具使用
- 開源模型生態系（Llama、DeepSeek、Qwen）

### ROS 機械手臂
- ROS 2 基礎架構與安裝
- MoveIt 2 運動規劃
- 視覺引導抓取（Eye-Hand Coordination）
- VLA 模型（Vision-Language-Action）
- 具身智能（Embodied AI）最新研究

## 🔄 更新頻率

透過 Pi 上的 cron 排程自動更新，每次會：
1. 搜尋最新論文與技術動態
2. 整理成結構化的 Markdown 文件
3. 同步產生 .docx Word 版本
4. 自動推送到 GitHub

## 🛠️ 技術棧

- **文件產生**：Claude (Anthropic) on Raspberry Pi 5
- **格式轉換**：pandoc (Markdown → Word)
- **自動化**：cron + bash scripts
- **版本控制**：Git + GitHub

## 📄 授權

本研究資料僅供學術參考使用。
