# Redroid GMS Dockerfile

这个项目提供了一个 `Dockerfile.a14`，用于在构建过程中自动下载并集成 GApps (Google Mobile Services) 到 Redroid Android 14 镜像中。

## 前提条件

你需要一个有效的 MindTheGapps (或其他 GApps 包) 的**直链下载地址**。
由于 GApps 的下载链接经常变动，你需要自己在构建时提供。

## 构建方法

`Dockerfile.a14` 中已经内置了默认的 GApps 下载链接（20250203 版本）。你可以直接构建：

```bash
docker build -f Dockerfile.a14 -t redroid-gms:14.0.0-latest .
```

如果你想使用其他版本的 GApps，可以通过 `--build-arg` 覆盖：

```bash
# 示例：使用其他版本的下载链接
export GAPPS_LINK="https://github.com/MindTheGapps/14.0.0-arm64/releases/download/..."

docker build \
  --build-arg GAPPS_URL="$GAPPS_LINK" \
  -t redroid-gms:14.0.0-latest \
  .
```

## 运行容器

构建完成后，你可以像运行普通 Redroid 镜像一样运行它：

```bash
docker run -itd \
    --privileged \
    --name redroid14 \
    -v ~/data14:/data \
    -p 5555:5555 \
    redroid-gms:14.0.0-latest \
    androidboot.redroid_width=720 \
    androidboot.redroid_height=1280 \
    androidboot.redroid_gpu_mode=host
```

## 说明

- **多阶段构建**: Dockerfile 使用了多阶段构建。第一阶段使用 `alpine` 镜像下载并解压 GApps，第二阶段将文件复制到 `redroid` 镜像中。这样可以避免在最终镜像中安装 `wget` 或 `unzip` 等不必要的工具，保持镜像体积小巧。
- **权限**: 构建过程中会自动执行 `chown -R 0:0` 来修复 `/system` 等目录的权限，确保 GApps 能正常加载。
