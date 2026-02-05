# Claude Code 配置仓库

## 克隆后初始化

```bash
git submodule update --init --recursive

cd .upstream/anthropics/skills
git sparse-checkout set skills/doc-coauthoring skills/docx skills/pdf skills/pptx skills/xlsx skills/web-artifacts-builder
cd ../../..
```

## 更新上游 skills

```bash
git submodule update --remote .upstream/anthropics/skills
```
