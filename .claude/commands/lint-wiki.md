---
description: 对 D:\note\wiki\ 跑一次 LLM Wiki 健康检查
argument-hint: (无参数)
---

## 检查清单(对应根 `CLAUDE.md` 的"维护"流程)

- [ ] **矛盾**:是否有页面结论冲突(优先用全文 grep 关键词)
- [ ] **过时**:近 3 个月未更新的概念/实体页是否还被引用
- [ ] **孤立**:没有入站链接的页面(`wiki/index.md` 之外)
- [ ] **缺失**:被多次提及但没有独立页的概念/实体
- [ ] **交叉引用**:broken `[[wikilink]]` 链接
- [ ] **数据空白**:能否用 web-search-prime 补强

## 输出

把发现写入 `D:\note\wiki\_lint-report.md`(若无该路径则创建),格式:

```
## [YYYY-MM-DD] lint 报告

### 🔴 必须修
- ...

### 🟡 建议修
- ...

### 🟢 已检查
- ...
```

不要自动改 wiki,只报告,等用户拍板。
