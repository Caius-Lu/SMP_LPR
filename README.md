# SMP_LPR

基于树莓派的车牌识别智能限高杆（2018 年作品，现仅维护文档，不再开发）。

系统分两部分：辨识部分以树莓派 3B+ 为核心处理器，激光对准模块判断车辆是否超高，超高则用摄像头抓拍，Python + OpenCV 做高斯去噪、灰度化、边缘检测、二值化、闭操作、字符分割、SVM 字符识别；识别结果经串口按自定义协议帧发给警示部分。警示部分以 STC12C5A32S2 为核心，点阵屏闪烁显示车牌，警示超高司机。

## 入口文件（先看这里）

| 文件 | 作用 | 怎么跑 |
| --- | --- | --- |
| `surface.py` | 桌面演示界面（tkinter） | `python surface.py` |
| `send.py` | 树莓派端主程序 | 在树莓派上 `python send.py` |
| `predict.py` | 识别核心 `CardPredictor` | 被 `surface.py` / `send.py` 引用；直接跑则对 `test/66.jpg` 做一次识别演示 |
| `config.js` | 两组识别调参 | 识别参数文件，不直接运行 |

桌面界面点“来自图片”选图，或点“来自摄像头”实时识别（约每 2 秒识别一次），显示车牌字符、车牌位置图和颜色（绿牌 / 黄牌 / 蓝牌）。

树莓派端流程：GPIO 4 检测到超高车辆 → picamera 拍照存为 `img_bgr.jpg` → 识别 → 串口按 `#车牌%` 格式发送（`/dev/ttyAMA0`，9600 波特）。

## 依赖

- 桌面：Python 3、opencv-python、numpy、Pillow
- 树莓派：另需 RPi.GPIO、picamera、pyserial
- 模型文件 `svm.dat`、`svmchinese.dat` 已在仓库内，启动时由 `CardPredictor.train_svm()` 加载

## 新环境注意

代码写于 Python 3.5 时代。`surface.py` 用到的 `Image.ANTIALIAS` 在 Pillow 10 起已移除，新环境需改为 `Image.LANCZOS`。

## 文件说明

- `surface.py` / `predict.py` / `send.py`：见上表
- `config.js`：两组识别参数（开运算、形态学核尺寸、行列数上限）
- `svm.dat` / `svmchinese.dat`：SVM 模型数据（数字字母 / 中文省份）
- `aa.jpg`–`gg.jpg`：示例图片

## 安全说明

2026-10-30 清理：删除了不被任何文件引用的 `MySQLCon.py`、`TheQt.py`、`UART.py`。其中 `MySQLCon.py` 是测试片段，且写明了远程 MySQL 的明文口令。该口令自 2018 年起公开可见，**请务必更换该数据库口令**；仅删除文件不能清除 git 历史中的记录，如需彻底清除需重写历史并强制推送，改动较大，另行确认后再做。
