# Windows 桌面脚本开发 — VBScript + Inno Setup 踩坑笔记

> 本文件是 2026-06-16 EasyTier 启动脚本增强任务沉淀的 Windows 桌面脚本开发陷阱清单。
> 适用场景: VBScript 启动器 / Windows 服务脚本 / Inno Setup 安装包。
> 关键命令: cscript.exe, wscript.exe, schtasks, netsh advfirewall, Inno Setup Compiler。

---

## 1. VBScript 编码: UTF-8 是死路,必须 UTF-16 LE + BOM

**问题**: VBScript 引擎(wscript/cscript)**不直接支持 UTF-8**。中文 MsgBox/InputBox 字面量在 UTF-8 文件里会触发"未结束的字符串常量"(列号偏移到中文字符第 43 字节附近)。

**验证**:
```cmd
cscript script.vbs
REM (111, 9) Microsoft VBScript 编译错误: 缺少语句
```

**解法**: 文件必须存为 **UTF-16 LE + BOM**。
```python
# Python 转换
with open(path, "rb") as f:
    data = f.read()
text = data.decode("utf-8")  # 假设源是 UTF-8
out = b"\xff\xfe" + text.encode("utf-16-le")  # 加 BOM
with open(path, "wb") as f:
    f.write(out)
```

**禁区**:
- ❌ 记事本"另存为 UTF-8" → 触发上述错误
- ❌ VS Code 默认 UTF-8 → 同上
- ✅ VS Code 右下角编码 → "UTF-16 LE BOM" → 保存
- ✅ Python 转换后再也不要用编辑器打开覆盖

---

## 2. UTF-16 文件反复 patch 时 BOM 累积

**问题**: 反复用 Python `read → patch → write` 同一个 UTF-16 文件,每次 `text.encode("utf-16-le")` 后再 `b"\xff\xfe" + ...` 会**叠加多个 BOM**(`ff fe ff fe ff fe`),导致 cscript 报 `(1, 1) 无效字符`。

**症状**: `cscript` 退出码 1,无任何输出(被 //B 吞掉)。
```cmd
python -c "
with open('D:/.../Start_EasyTier.vbs','rb') as f:
    data = f.read()
print('first 20 bytes hex:', data[:20].hex())
"
# fffefffefffe27002000...  ← 三个 BOM!
```

**解法**: 每次 patch 前先 strip 所有 BOM,再 encode 时只加一个:
```python
data = f.read()
while data.startswith(b"\xff\xfe"):
    data = data[2:]  # 剥光 BOM
text = data.decode("utf-16-le")
out = b"\xff\xfe" + text.encode("utf-16-le")  # 只一个
```

---

## 3. .replace() 跨平台换行符不匹配

**问题**: 在 MSYS / Python heredoc 里写的字符串含 `\n`,但 VBS 源文件实际是 `\r\n`(尤其经过 UTF-16 转换后),`.replace("... \n ...", "...")` 找不到匹配,patch 静默失败。

**症状**: `data.replace(..., 1)` 返回 0(实际是返回 -1 但很多人没检查返回值),函数看起来没动文件但其实什么都没改。

**解法**: 用 `re.sub` 加 `\r?\n` 容错:
```python
import re
text = re.sub(
    r"(Sub Foo\(\)\r?\n    )(.*?)(\nEnd Sub)",
    r"\1new body\3",
    text,
    flags=re.DOTALL,
)
```

---

## 4. cscript 输出捕获坑

**问题**: `cscript //B //NoLogo` 在某些环境下吞错误输出。语法错误只体现在 exit=1,看不到任何 stderr。

**调试姿势**:
```cmd
cscript script.vbs > out.log 2>&1
# exit=1, out.log 为空 → 看不出哪里错
```
**正解**: 去掉 `//B`(去掉 batch mode),重定向到文件:
```cmd
cscript script.vbs > out.log 2>&1
# 现在能看到完整错误:
# D:\...\script.vbs(111, 9) Microsoft VBScript 编译错误: 缺少语句
```

---

## 5. Option Explicit + 重复 Dim

**问题**: `Option Explicit` 强制所有变量显式声明。VBScript **不允许同一作用域重复 Dim**(不是覆盖,而是报"重复定义")。

**症状**: 函数顶部 `Dim x, y`,函数体中又有 `Dim x : x = ...` → `(N, N) 重复定义`。
**解法**:
- 函数顶部 `Dim x, y, z` 列全部变量
- 函数体内**不再 Dim**,直接 `x = ...`
- 如果用 `read_file` 看到的内容跟实际文件不一致,先 cat / od 验证编码

---

## 6. UAC 自提权 + cscript 无 GUI session

**问题**: VBS 用 `Shell.ShellExecute "wscript.exe", ..., "runas", 1` 触发 UAC 重启自己。在非交互终端里跑 cscript,UAC 弹窗没人点,cscript 永久挂起。

**场景**: 自动化测试 / 远程 SSH 跑 cscript → 看似死锁。
**处理**: 知道这是预期行为,**不要用 cscript 验证需要管理员权限的完整流程**。改用静态语法分析(读源码看 Dim 数量、Sub/Function 配对)或用 `wscript` 在真实桌面 session 跑。

---

## 7. schtasks 创建开机自启模板

```vbs
cmd = "schtasks /Create /F /TN ""EasyTierAutoStart"" " & _
      "/TR ""wscript.exe """" & scriptPath & """" elevate""" & _
      " /SC ONLOGON /RL HIGHEST /RU SYSTEM /DELAY 0001:00"
WshShell.Run cmd, 0, True
```

要点:
- `/RL HIGHEST + /RU SYSTEM` → 最高权限运行
- `/DELAY 0001:00` → 延迟 1 分钟(避开登录后各种 race)
- `wscript.exe` 包启动脚本 + `elevate` 参数(自己脚本要识别 elevate 跳过再提权)
- 删除: `schtasks /Delete /F /TN "..."`

查询存在: 用 `WshShell.Exec` + `StdOut.ReadAll` + `InStr(..., TASK_NAME)`。

---

## 8. netsh advfirewall 添加程序例外

```cmd
netsh advfirewall firewall delete rule name="MyRule" >nul 2>&1
netsh advfirewall firewall add rule name="MyRule" dir=in action=allow program="C:\path\to\app.exe" enable=yes profile=any
```

- 必须 `dir=in`(入站);不加默认双向但有些版本行为不一致
- `program=` 必须是绝对路径,不能用通配符
- `profile=any` 让规则在所有网络配置生效(家用/公网/域)
- 失败不阻塞主流程,用 `Err.Number` 检测

查询: `netsh advfirewall firewall show rule name="MyRule"`

---

## 9. Inno Setup 编译依赖

**问题**: 任务交付 Inno Setup 安装包时,如果用户机器没装 Inno Setup 编译器,只能停在 `.iss` 源码阶段。

**打包步骤(交付物)**:
```
installer-package/
├── install.iss         ← Inno Setup 脚本
├── files/              ← SourceDir,放所有要打包的二进制
│   ├── easytier-core.exe
│   ├── easytier-web-embed.exe
│   └── ... dll/sys
└── Output/             ← 编译后输出 setup.exe
```

**编译命令**:
- GUI: 打开 Inno Setup Compiler → File → Open → `install.iss` → Build → Compile
- CLI: `"C:\Program Files (x86)\Inno Setup 6\ISCC.exe" install.iss`

**交付话术模板**(告知用户下一步):
> 安装包脚本已就绪,需要 Inno Setup 6+ 编译。本机未检测到 ISCC.exe。步骤:
> 1. 下载 https://jrsoftware.org/isdl.php
> 2. 把 outputs\install.iss + files\ 子目录放同一目录
> 3. 打开 install.iss → Build → Compile → 输出 Output\EasyTier-Setup-X.X.exe

---

## 10. Inno Setup Pascal Script 关键模式

**检测 D: 盘 → 用 D: 路径,否则 C:**:
```pascal
function GetInstallPath(Param: string): string;
begin
  if DirExists('D:\') then
    Result := 'D:\Family_Ai\Software\app-name'
  else
    Result := 'C:\Family_Ai\Software\app-name';
end;
```

**安装后自动运行启动器**:
```ini
[Run]
Filename: "{app}\{#MyAppLauncherName}"; Description: "{cm:LaunchProgram,{#MyAppName}}"; Flags: nowait postinstall skipifsilent runascurrentuser
```

**PrivilegesRequired**: 默认 admin,程序需要写 ProgramFiles 或注册表时务必保留。

---

## 11. 启动器 / 安装器任务的标准交付物清单

无论 VBScript / PowerShell / Inno Setup / NSIS,产物至少包含:
- 主启动脚本(`Start_X.vbs` / `setup.exe` / `install.ps1`)
- 配置文件(`.toml` / `.json` / `.ini` 模板)
- README 或 INSTALL.md: 说明编码要求、依赖、验证命令
- 如果有安装包:`files/` 目录的 source 列表

**禁止**: 把 .vbs 用 UTF-8 交付(死路)、把 .iss 直接打包成 .exe 但没编译、忘了给 firewall / schtasks / taskkill 错误做日志。

---

## 12. cscript / wscript 选型

| 场景 | 用什么 |
|------|--------|
| 调试语法、看错误信息 | `cscript script.vbs`(不带 //B) |
| 静默批处理 / 重定向 stdout | `cscript //B //NoLogo script.vbs` |
| 真实用户双击运行 | `wscript script.vbs`(GUI 模式,无控制台窗口) |
| UAC 自提权后运行 | `wscript` + `elevate` 参数 |

---

## 13. 调试用临时 patch 的禁忌

如果用 Python 给 VBS 文件打临时 patch 加 `WScript.Quit 0` 跳过 UAC 验证语法,**千万**:
- ✅ 用 `re.sub` + `\r?\n` 容错
- ✅ 提前 strip BOM
- ✅ 验证后再还原(还原前 sha 校验)
- ❌ 不要用 `sed`(MSYS 对 CRLF 处理不可控)
- ❌ 不要用 heredoc + `>`(Windows 路径 + BOM 会乱)
- ❌ 不要 patch 函数体内部的局部 Dim(容易跟函数顶部 Dim 重复,触发新错误)
