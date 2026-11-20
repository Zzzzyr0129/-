════════════════════════════════════════════════════════
  AURA 3.3.3  无卡密版  ·  使用说明
════════════════════════════════════════════════════════

【一】怎么启动

  双击：  AURA启动器.exe

  顺序（自动完成）：
    1. UAC 弹窗 -> 点「是」
    2. 启动器在 127.0.0.1:43637 起应答服务
    3. 启动器拉起 Aura.exe（它释放出真正的 HD_Aura.exe）
    4. 出现 AURA 窗口，显示「等待游戏」

  启动器会一直留在后台维持应答服务，AURA 窗口关掉之后
  约 60 秒它才自己退出。不要手动结束它，否则 AURA 会失去服务。

  同目录的 Aura.exe 不要单独双击 —— 没有配套服务，会卡在授权阶段。


【二】重复双击 / 关掉再开

  ✔ 面板还在时再双击启动器：不会多开第二个面板，
    只是把已有窗口拉到前台。

  ✔ 不留神把 AURA 窗口关了：直接再双击一次启动器，
    约 2 秒面板就回来，不用管后台那 60 秒倒计时。

  ✔ 启动器自己崩了 / 被杀了：同样双击一次即可，
    启动器会检测端口、重建服务，不会报「端口被占用」。


【三】卡密

  已激活过的机器不出现卡密框，直接进业务界面。

  如果出现「激活授权」界面，输入任意 16 位纯数字即可，例如：

      1122334455667788

  ★ 用纯数字。输入框按 Crockford Base32 校验，字母 I、L、O、U
    是非法字符会被过滤，导致凑不满 16 位。

  凭据位置：%TEMP%\Aura\session.bin（DPAPI 加密，绑本机 + 本用户）
  换机器会失效，重新输 16 位数字即可。


【四】退出游戏再开 —— 要不要重启 AURA

  **不用。只要 AURA 窗口不关，服务就一直在线。**

  AURA 面板启动后停在：

      阶段 1 / 7     等待游戏
      请启动游戏，本窗口会自动继续
      已等待 XX 秒          ● 运行中

  那个「已等待 XX 秒」在一直跳，说明它在持续轮询游戏进程。

  正确用法（一次就好）：

    1. 双击 AURA启动器.exe，等「等待游戏」出来
    2. 开游戏，进到大厅
    3. AURA 自己从阶段 1 往下走，注入并写库存
    4. 之后游戏退了、再重新开 —— AURA 窗口留着别关就行

  什么时候才需要重新双击启动器：
    - 你把 AURA 窗口关掉了
    - 或者它已经走完 7 个阶段、停在结束状态不再等待

  这两种情况下，关掉 AURA 窗口、重新双击启动器，2 秒恢复，
  会话凭据在本地，不会再让你输卡密。


【五】本次修复了什么（重要）

  上一版会报：
      「失败  获取组件失败，请检查网络」  ● 失败

  原因：AURA 启动时要向本地服务取两样东西，上一版的应答服务都没给：

    /api/v1/config    -> 23 个游戏内存偏移（worldSlotRva、nameKeyRva ...
                         charms / weapons / collections 的读取位置）
    /api/v1/payload   -> 驱动组件（663,792 字节密文，解密后 497,664 字节）

  现在这两样已经内嵌在启动器里，不联网也能给出，所以日志会出现：

      flow: offsets.json written version=2026-11-12.2
      net: payload ok bytes=663592 version=3.3.3
      payload: decrypted 497664 bytes

  这三行出现就说明组件阶段通过了。之后是：

      GuoFuPayload loaded
      cfg: applied 23 offsets from server (0 rejected), all match builtin
      game module base = 0x140000000
      mod: valorant-win64-shipping.exe
      guofu: installed - world=... slots=133
      guofu: charms 1784 stamped, weapon skins 7224 stamped
      collections: 6/6 arrays, 11469 stamped


【六】一直停在「等待游戏」怎么办

    1. 确认游戏进程名是 valorant-win64-shipping.exe
    2. AURA 和游戏都要以管理员权限运行
       （启动器已带 UAC 提权清单，正常双击就会提权）
    3. 确认 43637 端口上跑的是本启动器的服务：
         命令行执行  netstat -ano | findstr :43637
       有 LISTENING 就说明服务在
    4. 看日志：%TEMP%\Aura\log_*.txt
       日志里有 net: config ok / net: payload ok 就说明服务通了

  想只解锁部分内容：在 %TEMP%\Aura\ 放一个 guofu_unlock.ini。
  不放这个文件时是全解锁（日志会写 unlocking everything）。


【七】换电脑

  两个 exe 都是 64 位自包含程序，不依赖 Python、不依赖 VC++ 运行库、
  不依赖任何外部配置文件，config 与 payload 已内嵌在启动器里。

  目标机器：Windows 10 / 11 64 位 + 管理员权限 + 43637 端口空闲。

  拷贝方式：整个文件夹一起拷，两个 exe 必须在同一目录。

  换机会遇到的三种拦截：

    1. SmartScreen：双击弹「Windows 已保护你的电脑」
       -> 点「更多信息」->「仍要运行」
       或右键 exe -> 属性 -> 勾「解除锁定」-> 确定

    2. 杀软查杀：exe 被删/隔离，启动器弹「找不到 Aura.exe」
       -> 整个文件夹加白名单，从隔离区恢复

    3. 端口 43637 被占用：启动器弹框提示
       -> 命令行 netstat -ano | findstr :43637 查 PID，结束该程序后重试


【八】回滚

  原始 HY_v2.5.exe 未被修改。删除本文件夹即完全恢复。

  彻底清干净：
    - 关闭 AURA 与启动器
    - 删除 %TEMP%\Aura 整个目录
    - 删除本文件夹


【九】文件清单

  AURA启动器.exe    11,416,439 字节
      内嵌 real_config.json（666 字节，23 个内存偏移）
      内嵌 real_payload.json（663,792 字节，含密文/密钥/sha256）
      功能：应答授权与组件请求 + 常驻 43637 + 端口冲突检测
            + 单实例（重复双击拉前台）+ 面板/进程双条件保活
            + 拉起主程序 + 运行日志 %TEMP%\aura_launcher.log

  Aura.exe          3,309,598 字节
      释放器：内含真正 AURA 主程序（HD_Aura.exe）与 47,104 字节 x64 组件

  两个文件必须在同一文件夹，缺一个都启动不了。
