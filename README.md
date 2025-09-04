PS1 Worker 自动更新工作流
本仓库包含一个 GitHub Actions 工作流（Obfuscate.yml），用于从官方 BPB-Worker-Panel 仓库获取、构建并部署最新版本的 BPB Worker 到 Cloudflare Worker。
概述
该工作流执行以下任务：

检查最新发布版本：通过 GitHub API 查询 BPB-Worker-Panel 的最新非预发布、非草稿版本。
比较版本：将本地版本（存储在 version.txt 中）与最新发布标签比较，判断是否需要更新。
拉取并构建：如果需要更新，则克隆官方仓库的最新标签，安装依赖，构建项目生成 worker.js。
自定义混淆：对 worker.js 进行自定义 JavaScript 混淆，生成独特的 _worker.js，以降低 Cloudflare 因代码相似性触发 1101 错误的风险。
提交并部署：将生成的 _worker.js 和更新后的 version.txt 提交到仓库，触发 Cloudflare Worker 的自动部署。

为什么需要自定义混淆？
官方 BPB Worker 构建包含易于识别的模式（例如 // Build 和 // @ts-nocheck 注释，以及类似 t,e,n,r,c,i,s,a,o,l 的可预测变量名）。这些模式在广泛使用时可能触发 Cloudflare 的风险检测，导致 1101 错误。为缓解此问题，工作流：

删除生成的 worker.js 中的前两行注释。
使用 javascript-obfuscator 以随机种子和随机字符串数组编码（RC4）等转换重新混淆代码，确保输出的 _worker.js 与官方构建不同。

这种方法最大限度地降低了 Cloudflare 因代码相似性而标记部署 Worker 的可能性。
工作流详情
工作流（Obfuscate.yml）运行触发条件：

推送到 main 分支时。
每天凌晨 1 点（UTC+8）通过定时任务（0 17 * * *）。
手动触发（workflow dispatch）。

步骤：

初始化仓库：检出 ps1 仓库。
获取本地版本：读取 version.txt 中的当前版本（如果存在）。
获取最新发布信息：通过 GitHub API 查询 BPB-Worker-Panel 的最新标签。
检查是否需要更新：比较本地版本与最新标签，决定是否需要更新。
构建和混淆：
克隆 BPB-Worker-Panel 仓库的最新标签。
安装 Node.js（22.x 版本）和依赖项。
运行 npm run build 生成 worker.js。
删除 worker.js 的前两行注释。
使用 javascript-obfuscator 以随机种子和自定义设置重新混淆 worker.js。
将结果输出为仓库根目录下的 _worker.js。


提交更改：将 _worker.js 和更新后的 version.txt 提交，提交信息记录同步的版本。
部署：提交的 _worker.js 触发 Cloudflare Worker 的自动部署。

关键文件：

_worker.js：部署到 Cloudflare 的混淆 Worker 脚本。
version.txt：记录当前同步的 BPB-Worker-Panel 版本。
.github/workflows/Obfuscate.yml：自动化工作流。

设置说明

克隆本仓库：git clone <你的-ps1-仓库-URL>


配置 Cloudflare Workers：
设置 Cloudflare Worker，使其从本仓库的 main 分支自动部署更改。
确保 Worker 配置以 _worker.js 作为入口点。


GitHub Actions：
工作流会在定时任务、推送或手动触发时自动运行。
无需额外的密钥或配置，使用默认的 GITHUB_TOKEN。


验证部署：
检查 GitHub Actions 日志，确认构建和提交成功。
监控 Cloudflare Worker，确认部署成功且无 1101 错误。



注意事项

自定义混淆使用随机种子和特定设置（例如 RC4 编码、字符串数组转换）以区分官方 BPB 构建。
如果 Cloudflare 1101 错误仍然出现，可考虑调整工作流中的混淆设置（例如修改 string-array-threshold 或启用其他转换）。
确保 Node.js 22.x 与你的环境兼容，如工作流中指定。
