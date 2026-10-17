V23 双机器人辅助穿衣：本轮完整代码与迭代记录备份

保存范围
这是本机 V23_Dual_Robot_Dressing 的私密代码备份，不是新的训练成果。
当前源码版本：v23_22_far_field_roll。保留当前 participant、57个测试文件、所有顶层启动器与分析器、验证摘要、19次实验的不可变 participant_snapshot，以及一次 MediaPipe 人体姿态候选实验的代码、模型、输入图和来源信息。
当前 participant 与第19个实验的 participant_snapshot 一致。v22 冻结权重没有重新训练；本轮主要改动感知、双机器人协调、抓衣搬运、稳定性判断和有界转腕控制。
已观察到双抓和持衣搬运等局部进展；尚未完成袖口穿入或完整穿衣。CPU测试不等于物理仿真成功。最后一次实验由用户中止，没有完整官方评分。
本备份创建于独立目录，没有修改原 V22 项目或原 V23 实验目录。

两个下载文件
1. V23_all_iterations.bundle：完整 Git 仓库备份。含本轮当前代码、19份历史代码快照和保留的结果摘要；Git自动去重重复文件。需要完整恢复时使用它。
2. V23_current_source.zip：便于直接查看的当前源码、权重、测试和说明。保留最佳已验证双抓搬运GUI所依赖的那一次历史快照，以及姿态探针的原始输入图；不包含其余18份历史快照。完整迭代恢复请使用 bundle。

恢复完整代码
在已安装Git的终端中执行：
  git clone V23_all_iterations.bundle V23_Dual_Robot_Dressing
  cd V23_Dual_Robot_Dressing
  git submodule update --init --recursive
官方仿真依赖固定为 https://github.com/rirolab/PhyRC_2027_Challenge.git
提交：b974ac0121cb0be4d9dc867a9b1ca093688e4d3e。
Git bundle包含本仓库对象，不包含官方子模块仓库对象；子模块初始化需要网络和上游访问权限。
ZIP没有Git元数据。如只使用ZIP，需另行将官方仓库克隆到 official/ 并检出上述提交。

代码入口
participant/policy.py：官方runner适配器和各实验变体选择。
participant/cooperative_policy.py 及同目录模块：当前双机器人感知、导航、抓取、同步搬运和接近策略。
participant/frozen_v22/：保持哈希不变的v22父策略和 weights/best.pt。
participant/development_runner.py：使用官方环境/评分器的开发实验runner。
run_experiment.py：检查环境与权重、保存本次代码快照、启动独立Docker实验。
launch_progress_gui.py / show_dual_progress.py：基于已保存历史版本的GUI进展演示。
inspect_results.py / inspect_sleeve_observation.py / inspect_equilibrium_wrist.py：离线分析器。
tests/：CPU回归测试；在具备numpy、opencv、torch等依赖的环境中从项目根目录运行：
  python -m unittest discover -s tests
pose_probe/：一次独立MediaPipe Lite CPU实验，包含模型与许可证资料。
tools/：本轮临时状态查看及定点停止脚本原件；stop_v12_after_roll.py绑定旧实验绝对路径，不应在新实验中直接复用。

运行环境与复现限制
这份代码备份不是解压即运行的仿真安装包。未打包Isaac/Docker镜像、NVIDIA驱动、准备后的runtime资产、cache、X11认证文件、虚拟环境、原始RGB-D数据或全部逐步测量轨迹。
环境使用 NVIDIA Isaac Sim 6.0.1，Docker镜像标签 phyrc-2027:isaac-6.0.1；原测试镜像ID及已准备资产哈希记录在 validation/official_provenance.json。
原 run_experiment.py 会严格核验官方Git提交/洁净状态、准备后的资产和Docker镜像ID。重新构建的镜像即便使用同一Dockerfile也可能有不同ID；不要把修改检查后的新环境称为原环境完全相同的复现。
官方仓库自身提供 run.sh build / run.sh prepare 和 scripts/prepare_assets.py。官方prepare默认生成 official/.runtime，而V23启动器读取项目根目录 runtime/。本轮没有找到额外独立保存的V23 runtime准备脚本；恢复时需在独立准备目录完成官方流程并明确配置/复制到V23所需布局，再逐项核验来源清单。
本轮历史 launch.json 中的命令和若干工具含原电脑绝对路径；它们作为实验记录保留。GUI启动器还复用历史启动参数，因此换目录/换机器需创建明确记录的新启动配置并重新核验挂载路径。未验证跨机器运行。
launch_progress_gui.py依赖 results/evaluate_20261017T084012_614537Z/launch.json 及其完整 participant_snapshot，这些均保留。生成GUI认证cookie的代码可读，但实际cookie、cache/gui.xauth、.Xauthority和登录凭据均未打包。
完整历史代码已保留，但仅保留结果摘要、最终公开诊断JSON和一张姿态探针输入图；未保留全部progress.jsonl、samples.jsonl、measurements.json和相机图片。因此可检查原报告及重跑代码，不能只用本备份重新计算全部原实验统计或独立复核每个图像判断。

姿态候选实验
使用Python3.12独立虚拟环境安装 pose_probe/requirements.lock.txt；不要向Isaac环境直接安装这组依赖。
  python pose_probe/probe_pose.py --model pose_probe/pose_landmarker_lite.task --sheet results/evaluate_20261017T064335_904702Z/diagnostics/episode_001/public_rgb_step_00642.png --output pose_probe/reproduced
模型SHA256：59929e1d1ee95287735ddd833b19cf4ac46d29bc7afddbbf6753c459690d574a。
该模型仅提出2D肩肘腕候选；预设六关节观察门槛未通过，不证明3D人体轴、袖口或穿袖能力。

来源与校验
EXPORT_MANIFEST.json列出本次导出文件及其SHA256；未给整个组合仓库创造新的统一许可证。
官方许可证/声明在 third_party/official_reference/，姿态模型来源/许可证在 pose_probe/。
各历史 results/evaluate_*/source_sha256.json 保留原文件身份；对应 participant_snapshot 中所有清单项完整保留。
包外 SHA256SUMS.txt 核验 bundle、ZIP和附带清单。本地打包程序不执行上传；远端上传状态另行核验。
