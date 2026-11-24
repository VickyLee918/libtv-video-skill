# LibTV Video Skill

把 `lenmu-video` 的导演工作方法适配到 LibTV，让支持 Agent Skills 的 AI 助手完成从故事、分镜到视频生成、下载和本地后期的工作。

本项目基于 [chesterkuo/lenmu-ai](https://github.com/chesterkuo/lenmu-ai) 改编，使用 LibTV 的账号及积分，不需要 Lenmu 积分，也不会自动调用即梦。它是社区适配项目，不是 LibTV 官方产品。

## 能做什么

- 故事开发、开头设计、分镜、景别与运镜、配色草图。
- 角色和产品一致性、首尾帧检查、图片及视频提示词、生成前审核。
- 优先通过 LibTV CLI 查询画布和模型、创建节点、提交生成、查询状态及下载结果。
- 抽帧检查、音轨检查、剪辑修复、字幕及本地后期的操作指导。
- 短剧与互动故事的分支、变量、跨集连续性和素材规划。

默认视频参数为 **Seedance 2.5、1080P、9:16、开启声音**。时长、数量和参考素材按任务确定。**每次生成必须先报出实际积分，收到用户明确确认后才提交**；重新生成和付费重试也适用。

## 安装与使用

使用支持多种 Agent 的 skills 安装器：

```bash
npx skills add VickyLee918/libtv-video-skill --skill libtv-video
```

也可以使用 Codex 的 skill-installer 安装：

```text
$skill-installer https://github.com/VickyLee918/libtv-video-skill/tree/main/skills/libtv-video
```

手动安装：

将仓库中的 `skills/libtv-video` 文件夹复制到个人 Codex skills 目录：

```text
Windows: %USERPROFILE%\.codex\skills\libtv-video
macOS / Linux: ~/.codex/skills/libtv-video
```

如果已存在同名 skill，先保存旧版。安装后在新一轮对话调用 `$libtv-video`。

还需要从 LibTV 官方渠道安装 CLI，并使用自己的账号完成授权。Windows 的官方安装脚本：

[LibTV CLI Windows installer](https://liblibai-web-static.liblib.cloud/cli/latest/install-libtv-cli.ps1)

```text
libtv --version
libtv login web --open
libtv account info
```

使用本地后期前需要 Python 3、ffmpeg/ffprobe；配色与帧检查脚本需要 `numpy` 和 `Pillow`。缺少其他可选依赖时，助手应按当前任务检查，而非一次安装所有工具。

示例请求：

> 用 $libtv-video，使用我指定的 LibTV 画布里的产品素材做一个 5 秒竖屏测试，先给我确认创意、参数和积分费用。

## 验证范围

已在 Windows、LibTV CLI 1.0.2 上跑通单镜头的参考图连接、Seedance 2.5 视频生成、状态查询、下载和文件检查。测试输出为1080×1920、约5秒，包含音轨，并另存 H.264 预览版。

模型、价格与 CLI 版本可能变化，每次以实时能力和报价为准。角色库、局部重拍、平台内剪辑、互动播放器、托管发布等功能没有全部完成端到端验证。故事方法已迁移不代表对应平台接口已接通，详见 [能力迁移表](skills/libtv-video/references/platform-map.md)。

仓库仅包含 skill 文档和辅助脚本，不包含账号凭据、私人画布链接、产品素材或测试成片。

## 文件结构

```text
skills/libtv-video/
  SKILL.md
  references/
    cli.md
    directing.md
    platform-map.md
    postproduction.md
    series.md
  scripts/
    colorboard.py
    framecheck.py
```

## 来源与许可

导演工作方法及配套的配色/帧检查脚本改编或保留自 [chesterkuo/lenmu-ai](https://github.com/chesterkuo/lenmu-ai)，原项目采用 MIT License，版权声明保留在 [LICENSE](LICENSE) 中。

本适配版增加 LibTV CLI 工作流、默认参数、逐次积分确认及能力边界说明。原版的 Lenmu 专用接口和价格不作为 LibTV 的能力或价格保证。
