# 开发久胜 Code 插件

[分发测试插件](distribution-check)演示一个只提供 Skill 的插件。复制示例目录，用自己的 ID、名称、发布者和语义版本替换清单中的值，再编辑 `skills/distribution-check/SKILL.md`。该文件的 YAML 头部声明技能名称和用途，正文写给 Agent。

## 清单与宿主接口

宿主从包根目录读取 `.jiusheng-plugin/plugin.json`。版本 2 的清单填写 `schema_version: "plugin-v2"`。包内路径使用相对路径。插件 ID 跨版本保持一致，每次分发新内容递增版本。用编辑器将清单关联到 [JSON Schema](schema/plugin-v2.json)。宿主也会检查字段之间的约束以及入口文件是否存在。

| 能力 | 清单字段 | 入口约定 |
| --- | --- | --- |
| 技能 | `skills` | 列出随包提供的 `SKILL.md` 路径 |
| 工具 | `mcp` | 声明协议版本、`stdio`、包内程序和工具名。可用 `startup_policy: "on_demand"` 按需启动 |
| 工作区界面 | `ui` | 入口为自包含 ES Module，默认导出 HTMLElement 子类，由宿主注册。声明 `slot: "workspace"` 及所属上下文 |
| 设置页 | `ui`、`configuration` | 设置页使用 `slot: "configuration"` 和 `context: "global"`。设备操作在清单中声明，由宿主传输到对应执行设备 |

## 打包与上架

在改好的插件目录执行 `python3 -m zipfile -c ../my-plugin.jsext .jiusheng-plugin skills`。该命令适用于本示例，增加工具或界面时将其入口目录也加入包。用 `python3 -m zipfile -l ../my-plugin.jsext` 检查清单和每个入口位于正确路径。可在此仓库提交资源 PR，附插件包、版本、目标平台、原作者与许可。

官方目录分两层。第一层是[目录](schema/catalog-v1.json)，按插件、版本及目标平台定位发布清单。第二层是[发布清单](schema/plugin-release-v1.json)，包含插件声明、最低宿主版本、平台、包下载地址、字节数和 SHA-256。两层分别使用 [Ed25519 签名信封](schema/signed-envelope-v1.json)。签名覆盖 Base64 解码后的原始载荷字节。宿主使用内置可信公钥验证两层签名，并核对包摘要后安装。开发者只提交自己的包与清单，目录签名由发行维护者完成。
