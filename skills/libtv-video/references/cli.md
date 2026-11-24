# LibTV CLI 执行与检查

2026-11-24 本机 CLI 1.0.2 实测通过：浏览器授权、个人/团队画布查询、模型 schema 查询、创建并连接参考图节点、修改及读回参数、提交一次生成、查询完成状态、下载及本地检查。单镜头测试使用 Seedance 2.5、图片参考、1080P、9:16、5秒、1条、开启声音，实际输出1080×1920、5.056秒、24fps、HEVC视频/AAC双声道音频。此记录是已验证路径；当前用户指定的默认模型、清晰度、画幅和声音设置见 SKILL.md，时长和数量按任务确定。批量生成、失败退款、4K实际输出仍未验证。版本升级后重新查看命令帮助。

## 安装与身份

官方安装脚本：https://liblibai-web-static.liblib.cloud/cli/latest/install-libtv-cli.ps1
版本清单：https://liblibai-web-static.liblib.cloud/cli/latest/manifest.json

Windows 常规安装位置 `%USERPROFILE%\.libtv\libtv.exe`。若新进程未刷新 PATH，使用该文件的完整路径；不要将 PATH 不可用误报为 CLI 未安装。已有可用 CLI 不重复覆盖升级。

```powershell
& "$env:USERPROFILE\.libtv\libtv.exe" --version
& "$env:USERPROFILE\.libtv\libtv.exe" account info
& "$env:USERPROFILE\.libtv\libtv.exe" login web --open
```

仅在未登录或凭据失效时启动登录。`login web` 启动本机回调并打印 LibTV 官方授权链接；等待成功回调，再运行 `account info` 验证。凭据由 CLI 保存到用户目录，不读取或输出 `credentials.json`，不复制到项目或技能中。

账号可能默认进入团队空间。先看 `account info` 中的 activeAccount/teamId；`account list` 可查看可用空间。不要为连接测试擅自切换账号、退出登录或创建凭据。

## 只读连接验证

```text
libtv project list --page-size 50
libtv project list --team-id 0 --page-size 50
libtv node list --project <目标画布UUID>
libtv model search --type video Seedance
libtv model <modelKey或完整modelName>
```

`project list` 默认查询当前账号空间，`--team-id 0` 显式查个人。列表可能分页；检查目标是否在当前页，不从第一页没有结果推断画布不存在。同名画布可能有多张，按用户提供的链接中 `projectId`/UUID 和空间核对，不按名称任意选择写入对象。

连接验证以读取已有画布节点成功为准，不需要建测试节点或提交生成。结果记录画布链接、节点数量和实际验证范围；只读成功不代表付费生成已验证。

画布链接格式：`https://www.liblib.tv/canvas?projectId=<UUID>`。

CLI 的 project 指画布；不要把它和网页中可能包含多张画布的文件夹/项目容器混为一谈。

## 制作时的绑定、写入与运行

- 默认用命令的 `--project <UUID>` 显式限定画布；先查相关子命令 `--help`，确认是否支持。需要目录绑定时在具体视频项目目录中执行 `project use <UUID>`，不能在通用技能目录或公共工作区根目录随意绑定。
- `project use` 会写当前目录 `.libtv/project.json` 并清除默认分组绑定；先检查已有绑定，不覆盖其他项目上下文。
- `node list --project <UUID>` 和 `node <ID> --project <UUID>` 不附加写入选项时为查询。节点名称可能重复，写入优先用唯一 ID。
- 模型查询返回的 `modelKey` 可用于拉 schema；创建/修改节点的 `--set model=...` 使用模型完整 `modelName`。其他参数和素材限制从实时 schema 读取，不照抄别的模型。
- `--prompt`、`--set` 修改生成参数；`--name` 修改展示名；`--update` 写节点顶层属性。不要将 `--update params=...` 当成修改生成参数。
- `node create` 仅新建，必须有 `-t video` 等类型；先创建不运行，读回确认节点、素材、参数及连线。创建结果不确定时先查询再重试。
- `--left` 为确保入边，可能调整已有关系；参考素材顺序和槽位用模型 schema 及读回结果核对。更新旧节点前检查其现有连线，不能盲目重写。
- `--run`/`-r` 会触发生成，每次都要在审核通过、报出该次实际费用并获得用户明确确认后才使用。查询状态不带它；不能用总预算、历史授权或整组操作绕过逐次费用确认。
- `upload --help`、`download --help` 查询当前参数；上传通常会创建资源节点，不属于只读验证。下载指定节点且使用新文件名，保存实际输出位置。
- 节点状态字段以实际返回为准；不要发明统一状态路径。对提交超时、排队、失败分别核对任务，避免重复扣点。

## 已跑通的生成、查询与下载流程

以下是 PowerShell 模板，变量由当前用户任务和实时 schema 填写，不固定绑定这次 KONO 测试的画布或节点。命令执行后检查 `$LASTEXITCODE`，失败时先查远端实际状态。

### 1. 准备草稿，不运行

```powershell
$libtvExe = Join-Path $env:USERPROFILE '.libtv\libtv.exe'
# $canvasUuid、$referenceId、$shotName 取自当前任务；$promptPath 指向已审核的 UTF-8 文件。
$shotPrompt = Get-Content -LiteralPath $promptPath -Raw -Encoding utf8
& $libtvExe node create $shotName --project $canvasUuid --type video `
  --prompt $shotPrompt --set 'model=Seedance 2.5' --set 'modeType=image2video' `
  --set 'ratio=9:16' --set 'resolution=1080p' --set 'duration=5' `
  --set 'count=1' --set 'enableSound=on' --left $referenceId
```

从返回的 `newNodeKey` 保存 `$shotNodeId`。以上参数只是本次成功示例；实际按用户选择和模型 schema 配置。图片参考 `image2video` 可以将产品三视图用于外观约束；提示词明确三视图是同一件产品，画面只出现一件，不把三视图布局作为首帧。只支持首帧的模式不能这样假设。

### 2. 费用与提交前核对

CLI 1.0.2 本次未返回该任务的实际报价或扣点。需要报价时，在网页选中**同一个节点**，核对生成面板上的模型、清晰度、时长、比例、数量、声音及参考输入后，读取生成按钮点数，不为报价点击生成。

已见报价仅作故障说明：Mini/720P/静音的80积分、2.5/4K/静音的900积分，都不能套用到2.5/1080P/带声音任务。参数变化后旧报价失效。团队总余额会因其他任务变化，不能仅用前后余额差推断本任务扣点。报价和实际扣点分列记录，未知就留空。

用户已明确要求：**每一次生成的费用，报价确认以后再生成。** 无法读取该次实际报价就说明具体阻碍并等待，不以“即可生成”“按保存参数生成”等一般指令跳过费用确认。参数或数量变化影响价格时重新报价确认；每次重新生成、重拍、付费重试也重新确认。该次费用已明确确认且参数和报价未变时，只提交一次，不重复询问同一次确认；查询/等待/下载同一任务不算新生成。

```powershell
& $libtvExe node $shotNodeId --project $canvasUuid
# 仅在读回值与当前授权不符时恢复已授权参数，例如：
& $libtvExe node $shotNodeId --project $canvasUuid `
  --set 'resolution=1080p' --set 'enableSound=on'
& $libtvExe node $shotNodeId --project $canvasUuid
```

读回 JSON 核对 `data.params`：`model`、`modeType`、`count`、`prompt`、`imageList`/参考顺序，以及 `settings` 的 `ratio`、`resolution`、`duration`、`enableSound`。提交前保存参数快照。检查是否已经有本次提交的任务或结果；同一镜头新版本用新节点/新名称，保留旧结果。若只是续接进行中的任务，直接查询，不再提交。

**已遇问题：** CLI 保存1080P/开启声音后，后续读回又出现4K/关闭声音；网页也曾仍显示旧参数。并发网页写入或同步机制的确切原因未确认。以提交前远端读回为检查依据；避免同时在网页编辑同一节点。网页仍显示旧值时先刷新、重新选中节点核对，不将旧网页的值覆盖回 CLI。发现持续回退或正在被他人编辑时，说明冲突并先协调，不能在竞争写入状态下提交。

### 3. 提交一次并跟进完成

```powershell
& $libtvExe node $shotNodeId --project $canvasUuid --run
# 后续状态查询不带 --run、--set、--left 或其他写入参数：
& $libtvExe node $shotNodeId --project $canvasUuid
```

`--run` 本次等待约数分钟才输出完成结果，中途无输出不表示提交失败。进程仍在运行时保留会话，按约20–30秒间隔查询同一节点，并定期简短更新用户。需要检查进程输出时轮询原执行会话，不重跑提交命令。

本次实际字段：`data.taskInfo.taskId`、`loading`、`status`、`progressPercent`；生成中见 `status=1/loading=true`，完成见 `status=2/loading=false/progressPercent=100` 且 `data.url` 有视频链接。`--run` 完成返回也包含顶层 `taskId/status`。其他状态码未实测，不凭整数猜测退款或可安全重试。任务失败/长时间卡住时保存原始错误与任务ID；先判断状态，每次付费重试都重新报价并取得该次费用确认。

### 4. 下载、验证、交付

```powershell
& $libtvExe download --node $shotNodeId --project $canvasUuid `
  --out $deliveryDir --without-ai-watermark
ffprobe -v error -show_entries 'format=duration,size:stream=index,codec_name,codec_type,width,height,r_frame_rate,sample_rate,channels' -of json $videoPath
ffmpeg -hide_banner -loglevel error -i $videoPath `
  -vf 'fps=2,scale=216:384,tile=5x2' -frames:v 1 $contactSheetPath
ffmpeg -hide_banner -i $videoPath -vn -af volumedetect -f null NUL
ffmpeg -hide_banner -loglevel error -i $videoPath `
  -c:v libx264 -crf 18 -preset fast -pix_fmt yuv420p `
  -c:a aac -b:a 160k -movflags +faststart $previewPath
```

`--without-ai-watermark` 是 CLI 支持的下载偏好，是否可取原片取决于真实账号权限，不通过 `--vip` 伪造会员资格。先用 `download --help` 核实当前支持。CLI 按节点名称命名文件，实际输出路径从返回和目录核对；目录/文件用新版本命名，不覆盖以前成片。

**已遇问题：** 下载图片或视频曾先打印 `canva.xingliu.art` 的水印附加端点404，随后仍输出文件路径且退出码为0，下载文件可以正常探测和解码。因此不能只看到 `API Request Error` 就判定下载失败：检查退出码、文件存在且非空、ffprobe及抽帧解码结果。若文件无效再重试下载或用返回的官方媒体链接下载，不重新生成视频。不要根据该警告声称输出一定有/没有水印。

本例5秒竖屏抽10格，示例拼图适用于该时长；其他时长/画幅调整缩放和布局，保证检查覆盖完整视频。音量检测只证明数值和信号存在，不证明实际听感；标签小字、实际运动流畅度和未试听的声音应如实标注检查边界。H.264预览版用于播放器兼容，保留HEVC原片供下载。

交付包含可播放的本地预览、原片链接、画布链接、实际文件参数及检查范围。在 `tasks.csv` 记录任务ID、最终参数、状态、输出路径及实际已知费用；本次生成成功不等于未来任务免审核、免预算确认或允许自动重拍。
