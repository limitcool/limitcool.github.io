---
title: "如何使用FFmpeg处理音视频文件"
description: 本文提供了FFmpeg处理音视频文件的完整指南，包括将单张图片转换为视频、拼接多个视频、设置转场特效等多种操作。
date: 2022-07-25T14:05:04+08:00
draft: true
slug: ffmpeg
image:
categories: ffmpeg
tags: ffmpeg
---

# `ffmpeg`图片转视频

使用单张图片生成5秒视频

```bash
# -loop 1 指定开启单帧图片loop
# -t 5 指定loop时长为5秒
# -i input 指定输入图片文件路径 示例:pic.jpg
# -pix_fmt 指定编码格式为yuv420p
# -y 若输出文件已存在,则强制进行覆盖。
# ffmpeg会根据输出文件后缀,自动选择编码格式。
# 也可以使用 -f 指定输出格式
ffmpeg -loop 1 -t 5 -i <filename>.jpg -pix_fmt yuv420p -y output.ts
```

# `ffmpeg`拼接视频

```bash
# windows
# -i input 指定需要合并的文件,使用concat进行合并.示例:"concat:0.ts|1.ts|2.ts"
# -vcodec 指定视频编码器的参数为copy
# -acodec 指定音频编码器的参数为copy
# -y 若输出文件已存在,则强制进行覆盖。
ffmpeg -i "concat:0.ts|1.ts" -vcodec copy -acodec copy -y output.ts
```

# `ffmpeg`设置转场特效

```bash
# Linux
ffmpeg -i v0.mp4 -i v1.mp4 -i v2.mp4 -i v3.mp4 -i v4.mp4 -filter_complex \
"[0][1:v]xfade=transition=fade:duration=1:offset=3[vfade1]; \
 [vfade1][2:v]xfade=transition=fade:duration=1:offset=10[vfade2]; \
 [vfade2][3:v]xfade=transition=fade:duration=1:offset=21[vfade3]; \
 [vfade3][4:v]xfade=transition=fade:duration=1:offset=25,format=yuv420p; \
 [0:a][1:a]acrossfade=d=1[afade1]; \
 [afade1][2:a]acrossfade=d=1[afade2]; \
 [afade2][3:a]acrossfade=d=1[afade3]; \
 [afade3][4:a]acrossfade=d=1" \
-movflags +faststart out.mp4
```

| 输入文件 | 输入文件的视频总长 |  +   | previous xfade `offset` |  -   | xfade `duration` | `offset` = |
| :------- | :----------------- | :--: | :---------------------- | :--: | :--------------- | :--------- |
| `v0.mp4` | 4                  |  +   | 0                       |  -   | 1                | 3          |
| `v1.mp4` | 8                  |  +   | 3                       |  -   | 1                | 10         |
| `v2.mp4` | 12                 |  +   | 10                      |  -   | 1                | 21         |
| `v3.mp4` | 5                  |  +   | 21                      |  -   | 1                | 25         |

// 将音频转为单声道

```
ffmpeg  -i .\1.mp3 -ac 1  -ar 44100 -ab 16k -vol 50 -f  1s.mp3 
ffmpeg -i one.ts -i 1s.mp3 -map 0:v -map 1:a -c:v copy -shortest -af apad -y one1.ts
```

