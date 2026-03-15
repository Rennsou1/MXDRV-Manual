# MXDRV 使用指南

MXDRV - 夏普 X68000 电脑音源驱动使用教程

---

## 目录

- [简介](#简介)
- [文件头](#文件头)
- [乐器制作与定义](#乐器制作与定义)
  - [FM 乐器定义](#fm-乐器定义)
  - [ADPCM 乐器](#adpcm-乐器)
    - [PDX 文件制作](#pdx-文件制作)
    - [PDX 文件使用](#pdx-文件使用)
- 编译器手册
  - [MXC 编译器](manual_MXC.md)
  - [NOTE 编译器](manual_NOTE.md)
  - [mml2pdx 编译器](Manual_mml2pdx.md)

---

## 简介

MXDRV 是夏普 X68000 电脑的音源驱动程序，支持 OPM（FM 合成）和 OKIM6258（ADPCM）音源芯片。本文档详细介绍了如何使用 MXC 和 NOTE 两种编译器编写 MML（Music Macro Language）并生成 MDX 音乐文件。

---

## 相关工具

- [mml2mdx](https://github.com/Rennsou1/mml2mdx) — MML → MDX 编译器
- [wav2pdx](https://github.com/Rennsou1/wav2pdx) — WAV → PDX 转换工具
- [mdx2wav](https://github.com/Rennsou1/mdx2wav) — MDX → WAV 渲染工具

---

## 文件头

### 设置曲名

```
#title "曲名"
```

### 指定 PCM 文件

```
#pcmfile "文件名.pdx"
```

设定一个 PDX 文件来播放 ADPCM。如需使用，请查看 [PDX 文件制作](#pdx-文件制作) 章节。

---

## 乐器制作与定义

### FM 乐器定义

#### 格式

```
@1~255={
AR, DR, SR, RR, SL, OL, KS, ML, DT1, DT2, AME,
AR, DR, SR, RR, SL, OL, KS, ML, DT1, DT2, AME,
AR, DR, SR, RR, SL, OL, KS, ML, DT1, DT2, AME,
AR, DR, SR, RR, SL, OL, KS, ML, DT1, DT2, AME,
CON, FL, OP
}
```

#### 参数说明

| 参数 | 名称 | 范围 | 说明 |
|------|------|------|------|
| AR | Attack Rate | 0~31 | 起音速率 |
| DR | Decay Rate | 0~31 | 衰减速率 |
| SR | Sustain Rate | 0~31 | 延音速率 |
| RR | Release Rate | 0~15 | 释音速率 |
| SL | Sustain Level | 0~15 | 延音电平 |
| OL | Operator Level | 0~127 | Operator 音量或调制等级 |
| KS | Key Scaling | 0~3 | ADSR 包络随音高缩放 |
| ML | Multiplier | 0~15 | Operator 音高倍数 |
| DT1 | Detune 1 | 0~7 | 弱失谐（0=不失谐，1~4=偏高，5~7=偏低）|
| DT2 | Detune 2 | 0~3 | 强失谐（数字越高越偏高音）|
| AME | Amplitude Modulation Enable | 0~1 | 调幅开关（0=关，1=开）|
| CON | Connection | 0~7 | FM 合成算法（Operator 连接方式）|
| FL | Feedback Level | 0~7 | 反馈等级（仅作用于 Operator 1）|
| OP | Operator Mask | - | 一般填 15（二进制 %1111，开启全部 4 个 Operator）|

**Operator Mask 对应关系：**
- 3 = 1 op
- 7 = 2 op
- 11 = 3 op
- 15 = 4 op

**参考资料：** [合成器基础：ADSR包络线 - 知乎](https://zhuanlan.zhihu.com/p/81771109)

---

### ADPCM 乐器

#### PDX 文件制作

PDX 文件是 MXDRV 中用于 PCM 通道的重要文件。以下是从零制作 PDX 文件的步骤：

##### 1. 转换 WAV 为 ADPCM

使用 WAV2ADP 将 `.wav` 文件转换为 raw ADPCM（`.pcm`）文件：

```bash
run68 wav2adp [filename.wav]
```

##### 2. 创建 PDL 文件

创建一个 `.txt` 文件并重命名为 `.pdl`，内容如下：

```
#ex-pdx 1

@0
1=文件名1.pcm
2=文件名2.pcm
3=文件名3.pcm
```

**说明：**
- `#ex-pdx` 启用扩展 PDX 格式，一个 bank 可载入约 90 个采样文件
- `@0` 定义 bank 编号
- 数字（1, 2, 3...）对应音符音高，映射到对应的 PCM 文件

##### 3. 生成 PDX 文件

在终端输入以下指令，或直接将 `.pdl` 文件拖入 MKPDX：

```bash
mkpdx filename.pdl
```

---

#### PDX 文件使用

PDX 文件只能在 PCM 通道（OKIM6258）播放。

##### 基础用法

使用音高音符指定采样：

```
o0 e4 e4 e4 e4
o0l4[e]4
```

##### 推荐用法

使用 `n#` 指令直接指定数值音高：

```
l8n1n2n3n2
n1,8n2,8n3,8n2,8
```

**示例：** 假设 1=kick.pcm，2=hat.pcm，3=clap.pcm

---

##### MXC 与 NOTE 编译器 PCM 通道对比

| 特性 | MXC 编译器 | NOTE 编译器 |
|------|-----------|-------------|
| PCM 通道数量 | 1 个 | 最多 8 个 |
| 音高调节 | 不支持 | 支持 |
| 音量调节 | 不支持 | 支持 |
| PCM 格式 | 仅 ADPCM | 支持 ADPCM 和 PCM |

**推荐：** 如对 PCM 通道有较高需求，建议使用 NOTE 编译器。

---

## 编译器手册

各编译器的详细 MML 指令请参阅对应手册：

- **[MXC 编译器手册](manual_MXC.md)** — MXC 编译教程与 MML 指令参考
- **[NOTE 编译器手册](manual_NOTE.md)** — NOTE 编译教程、命令行开关与 MML 指令参考
- **[mml2pdx 编译器手册](Manual_mml2pdx.md)** — mml2pdx 编译教程与扩展 MML 指令参考

---

## 其他

本文档基于 MXDRV 手册整理编写。

---

## 参考资料

- [合成器基础：ADSR包络线 - 知乎](https://zhuanlan.zhihu.com/p/81771109)
- MXDRV 文档
- X68000 技术参考手册

---

**文档版本：** 1.0  
**最后更新：** 2025/3/16
**编写者：** Ripo_2006
