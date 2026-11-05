# CLAUDE.md

## 模型分工

- 主会话用 Sonnet：统筹任务、写代码、改 bug。
- Opus 是顾问（advisor），只在这三个时候调用：
  1. 计划敲定之前
  2. 同一个错误第二次出现时
  3. 宣布完成之前，审核汇总后的改动
- 其他时候不调用顾问。

## 搜索和测试交给 Haiku 子代理

所有代码库搜索和测试都交给 Haiku 子代理，主会话不自己大范围读文件、grep 或跑检查，保持主上下文精简。

- `search`：梳理代码库结构，定位功能在哪些文件（只读）
- `grep`：查找函数、配置项、变量的调用和引用位置（只读）
- `audit`：在独立 worktree 里检查改动（不碰工作分支）

主会话只看子代理返回的结论，要改哪段再打开那一段。

`audit` 的 worktree 默认从 master 分出，看不到未提交的改动。先把改动提交到工作分支，再把分支名交给 `audit`。

## 项目速览

- `main.py`：主程序（抓取、关键词筛选、推送）
- `config/config.yaml`：主配置
- `config/frequency_words.txt`：关键词
- `.github/workflows/crawler.yml`：定时运行
- 没有单元测试。检查方式：解析 `main.py` 语法，并用 PyYAML 加载 `config/config.yaml`。
