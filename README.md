# ESP32-C3 空气质量监测（ESPHome）

ESPHome 固件配置仓库，可用 **GitHub Actions 自动构建**，也可本地编译。

| 配置文件 | 说明 | 硬件 |
| --- | --- | --- |
| `air-monitor.yaml` | 固件：PM2.5/PM10/甲醛/温湿度 + OLED 三页轮播 + Web 服务器 | ESP32-C3 + PMS5003ST + SH1106 128x64 |

- ESPHome 版本：**2026.8.2**（CI 与本地保持一致）
- 平台：`esp32-c3-devkitm-1`，framework `arduino`（ESP-IDF 5.5.5 工具链）

## 引脚接线（air-monitor.yaml）

| 功能 | 引脚 |
| --- | --- |
| PMS5003ST SET/SLEEP 控制 | GPIO3 |
| PMS5003ST UART（ESP RX / TX） | GPIO4 / GPIO5 |
| OLED I2C SDA / SCL | GPIO6 / GPIO7 |

> 注意：ESP32-C3 的 GPIO2 / GPIO8 / GPIO9 是 strapping 引脚，尽量避免用作 I2C。

## GitHub Actions 自动构建

工作流：`.github/workflows/esphome.yml`（官方 action `esphome/build-action@v8.1.0`，Docker 镜像 `ghcr.io/esphome/esphome`）

- **push / PR / 手动触发**：编译 `air-monitor.yaml`，产物以 `<name>-<platform>` 命名的 artifact 提供（含 `.bin`、`manifest.json`、`.elf`）
- **打 tag 推送**（如 `git tag v1.0.0 && git push --tags`）：自动发布到 GitHub Release

触发方式：

```bash
git init -b main
git add .
git commit -m "ESPHome air monitor + CI"
git remote add origin https://github.com/<你的用户名>/esphome.git
git push -u origin main
```

### 仓库 Secrets（可选，建议配置）

`secrets.yaml` 已被 `.gitignore` 忽略，CI 会用下面 Secrets 重建；未配置时回退到默认值。

- `WIFI_SSID`
- `WIFI_PASSWORD`
- `AP_PASSWORD`

## 本地编译

```bash
# 需要 Python 3.11+
python3 -m venv .venv && source .venv/bin/activate
pip install esphome==2026.8.2

esphome compile air-monitor.yaml     # 产物: .esphome/build/air-monitor/build/*.bin
esphome run air-monitor.yaml         # 编译 + 烧录（USB）
esphome dashboard .                  # 网页版编辑器
```

首次编译会下载 ESP-IDF 工具链（约 1 GB，缓存在 `~/.esphome` / `~/.cache/esphome`）。

## 烧录方式

1. **USB**：`esphome run air-monitor.yaml`
2. **OTA**：设备联网后在 ESPHome Dashboard / Home Assistant 中一键更新
3. **网页刷机**：用 [esp-web-tools](https://esphome.github.io/esp-web-tools/) 加载 CI 产物里的 `manifest.json`

## 目录说明

```
air-monitor.yaml      固件配置
secrets.yaml          本地凭据（不入库）
fonts/DejaVuSans.ttf  OLED 字体（跨平台，勿用 C:/Windows/Fonts 绝对路径）
.esphome/             构建缓存（不入库）
```
