### Lottie 预览
https://lottiefiles.com/preview
代理添加规则： DOMAIN-SUFFIX,lottiefiles.com,自主选择

### Fluter Rive
```bash
官方不推荐: https://rive.app/docs/editor/mcp/integration (实验性功能)

1. 安装 Rive Early Access 桌面版并保持它运行。
2. codex mcp add rive --url http://localhost:9791/sse
   or
   claude mcp add --transport sse rive http://localhost:9791/sse

官方推荐: https://rive.app/docs/editor/ai-agent/ai-agent   
    $17/mo

RiveMCP: https://rivemcp.stunning.gg
    $10/mo

Rive 中文文档:
https://www.rive.org.cn/docs/guide/introduction
```

### App Rive Skill

- 新增 `skills/app-rive/SKILL.md`
- 用于：
  - 判断 Pencil 设计稿中的图片组件是否适合转为 Rive
  - 给 Rive 新手输出一步一步的 Rive Editor 操作指导
  - 约束 `.riv` 资源在 Flutter App 中的接入边界
