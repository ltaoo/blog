---
title: python 按顺序合并 ts 文件
categories: Python
tags:
- python
- ts video
date: 2018/10/22
---

cctalk 缓存视频后，实际缓存的是视频切片并加密的 ts 文件，将视频片段解密后，需要解决的问题就是如何合并视频片段。

<!--more-->

每个 ts 文件都会包含时间信息，通过`ffmpeg`可以查看：

```bash
ffmpeg -i 0a7599cc995bdc2b4d0ff70b443f76f2.ts
```

对应的输出：

```bash
ffmpeg version 4.0.2 Copyright (c) 2000-2018 the FFmpeg developers
  built with Apple LLVM version 9.1.0 (clang-902.0.39.2)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/4.0.2 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-gpl --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma
  libavutil      56. 14.100 / 56. 14.100
  libavcodec     58. 18.100 / 58. 18.100
  libavformat    58. 12.100 / 58. 12.100
  libavdevice    58.  3.100 / 58.  3.100
  libavfilter     7. 16.100 /  7. 16.100
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  1.100 /  5.  1.100
  libswresample   3.  1.100 /  3.  1.100
  libpostproc    55.  1.100 / 55.  1.100
Input #0, mpegts, from '0a7599cc995bdc2b4d0ff70b443f76f2.ts':
  Duration: 00:00:09.67, start: 7583.773333, bitrate: 562 kb/s
  Program 1
    Stream #0:0[0x100]: Video: h264 (High) ([27][0][0][0] / 0x001B), yuv420p(progressive), 1920x1080 [SAR 1:1 DAR 16:9], 25 fps, 25 tbr, 90k tbn, 50 tbc
    Stream #0:1[0x101](eng): Audio: aac (LC) ([15][0][0][0] / 0x000F), 48000 Hz, stereo, fltp, 145 kb/s
```

内容非常多，我们只需要关心倒数第四行，`Duration: 00:00:09.67, start: 7583.773333, bitrate: 562 kb/s`，其中的`start`就是该视频的开始时间了。

> 只有视频能正确输出这个时间的才能排序。

所以我们的问题就变成了从终端获取信息，并且给视频设置有顺序的文件名。

## 读取终端输出

```python
import subprocess
import re

def extract_timestamp(video_path):
    command = 'ffmpeg -i ' + video_path
    result = subprocess.Popen(command, shell=True, stdin=subprocess.PIPE, stderr=subprocess.STDOUT, stdout=subprocess.PIPE, close_fds=True)
    # out 就是终端的输出
    out, err = result.communicate()

    re_result = re.search('start: ([0-9]+)', out)
    print('regexp result is ', re_result)
    timestamp = re_result.group(1)

    return timestamp
```

## 获取正确顺序的文件名

我们最终是使用`cat *.ts > index.ts`来完成合并的，该命令会单纯按照文件的排序来合并，所以需要有正确的顺序。

这里的做法是根据视频的开始时间，直接将时间作为文件名。如有一个视频片段开始时间是 58:00，那么这个文件文件名就应该是 005800。从这里也能看出视频长度最好不超过 60 小时，因为超过 60 小时后文件名长度不一样，排序就会出现问题。

```python
import datetime
def seconds_to_str(seconds):
    time = str(datetime.timedelta(seconds=seconds))
    res = []
    for i in time.split(':'):
        res.append(prefix(i, num = 2))
    return ''.join(res)

def prefix(value, num = 6):
  index_list = list(str(value))
  if num > len(index_list):
    res = (num - len(index_list)) * '0' + str(value)
    return res
  return value
```

## 更新文件名并合并

```python
import os
import sys
import shutil

def sort_videos(path):
    # 原始视频文件夹
    videos_path = 'output'
    # 排序后视频文件夹
    output_path = 'sorted'
    for file in os.listdir(os.path.join(path, videos_path)):
        if file == '.DS_Store':
            continue
        # output dir
        sorted_dir = os.path.join(path, output_path)
        # 确保文件夹必然存在
        prepare_dir([sorted_dir])
        file_path = os.path.join(path, videos_path, file)
        print('prepare process file is ' + file_path)
        timestamp = extract_timestamp(file_path)
        file_name = seconds_to_str(int(timestamp)) + '.ts'
        new_file_path = os.path.join(sorted_dir, file_name)
        print('new file is ' + new_file_path)
        # 从视频原始文件夹「拷贝并重命名」到排序后视频文件夹中
        shutil.copyfile(file_path, new_file_path)

    print('sort videos finish')
    concat_command = 'cat ' + os.path.join(sorted_dir, '*.ts') + ' > ' + os.path.join(path, 'index.ts')
    print(concat_command)
    os.popen(concat_command)
    # 删除排序后视频文件夹
    shutil.rmtree(sorted_dir)

def prepare_dir(dirs):
    for dir in dirs:
        if not os.path.exists(dir):
            os.mkdir(dir)
```


