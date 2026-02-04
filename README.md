# wayland_weston_embeded_snippet

# Создаем XDG_RUNTIME_DIR
mkdir -p /run/user/0
chmod 700 /run/user/0
export XDG_RUNTIME_DIR=/run/user/0

# Запускаем Weston через openvt
openvt -s -- weston 

## Проверка сокета  
ls /run/user/0

# Запускаем видеопоток с камеры
export WAYLAND_DISPLAY=wayland-1
gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert ! waylandsink

--------------

ip link set end1 up
ip addr add 10.0.0.1/24 dev end1

//////////////

sudo ip link set enx28ee52015936 up
sudo ip addr add 10.0.0.2/24 dev enx28ee52015936


ip addr flush dev end0  # Очистить старые адреса, если были

# Камера
/etc/init.d/S40camera start
# Экспозиция
v4l2-ctl -d /dev/video0 --set-ctrl=exposure=200



cmake ..     -DCMAKE_SYSROOT=$SYSROOT     -DARCH=aarch64     -DRKAIQ_TARGET_SOC="rk356x"     -DBUILROOT_BUILD_PROJECT=ON     -DCMAKE_C_FLAGS="-DISP_HW_VERSION=21 -DISP_HW_V21"     -DCMAKE_CXX_FLAGS="-DISP_HW_VERSION=21 -DISP_HW_V21"

RK3568 SoC с ISP2 (rkisp версии v02.03.00)


# Сборка camera_engine_rkaiq

# Репозиторий
git clone https://gitlab.com/rk3588_linux/linux/external/camera_engine_rkaiq.git
cd camera_engine_rkaiq

# Переключаемся на ветку для RK356x
git checkout rk356x

# Настройка переменных окружения
export SDK_PATH=/home/tnv/SOVA2.0/SDK/aarch64-buildroot-linux-gnu_sdk-buildroot
export CC=$SDK_PATH/bin/aarch64-buildroot-linux-gnu-gcc
export CXX=$SDK_PATH/bin/aarch64-buildroot-linux-gnu-g++
export SYSROOT=$SDK_PATH/aarch64-buildroot-linux-gnu/sysroot

#  КРИТИЧЕСКОЕ ИСПРАВЛЕНИЕ iq_parser_v2/CMakeLists.txt:
sed -i 's/\${ISP_HW_VERSION}/-DISP_HW_VERSION=20 -DISP_HW_V20/g' iq_parser_v2/CMakeLists.txt

# Проверка замены
grep "ISP_HW" iq_parser_v2/CMakeLists.txt | head -5

# Отключение -Werror в основном CMakeLists.txt
sed -i 's/-Werror/-Wno-error/g' CMakeLists.txt

# Сборка 
mkdir build && cd build

cmake .. \
    -DCMAKE_SYSROOT=$SYSROOT \
    -DARCH=aarch64 \
    -DRKAIQ_TARGET_SOC="rk356x" \
    -DBUILROOT_BUILD_PROJECT=ON \
    -DCMAKE_C_FLAGS="-DISP_HW_VERSION=20 -DISP_HW_V20" \
    -DCMAKE_CXX_FLAGS="-DISP_HW_VERSION=20 -DISP_HW_V20"

make -j$(nproc)

# rkaiq_3A_server не соберется! Соберем основную библиотеку librkaiq.so
# Проблема в rkaiq_3A_server/CMakeLists.txt - там жестко прописан 32-битный ARM компилятор (arm-linux-gnueabihf), а у нас ARM64 (aarch64). Вот как это исправить:

# Перейдем в директорию проекта
cd ~/SOVA2.0/modules/camera_engine_rkaiq
# Отредактируем файл
nano rkaiq_3A_server/CMakeLists.txt

# либо комментируем:
# SET(CMAKE_C_COMPILER "/home/camera/camera/rv1109_sdk/prebuilts/gcc/linux-x86/arm/gcc-arm-8.3-2019.03-x86_64-arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc")
# SET(CMAKE_CXX_COMPILER "/home/camera/camera/rv1109_sdk/prebuilts/gcc/linux-x86/arm/gcc-arm-8.3-2019.03-x86_64-arm-linux-gnueabihf/bin/arm-linux-gnueabihf-g++")

# Либо ставим наши кросс-компиляторы:
SET(CMAKE_C_COMPILER "/home/tnv/SOVA2.0/SDK/aarch64-buildroot-linux-gnu_sdk-buildroot/bin/aarch64-buildroot-linux-gnu-gcc")
SET(CMAKE_CXX_COMPILER "/home/tnv/SOVA2.0/SDK/aarch64-buildroot-linux-gnu_sdk-buildroot/bin/aarch64-buildroot-linux-gnu-g++")

# Проблема rkaiq_3A_server не может найти статические библиотеки алгоритмов (librkaiq_ae.a, librkaiq_awb.a и т.д.):
# Заменяем эту строку SET(RK_AIQ_LIB_DIR ${RK_AIQ_SOURCE_DIR}/all_lib/${CMAKE_BUILD_TYPE}) на эту:
SET(RK_AIQ_LIB_DIR ${CMAKE_BINARY_DIR}/all_lib/${CMAKE_BUILD_TYPE})




# Рецепт сборки camera_engine_rkaiq для rk3568 от MacroGroup 
# Структура 
```text
.
├── patch/                       # патчи для camera_engine_rkaiq
└── camera_engine_rkaiq/         # штатный репо https://gitlab.com/rk3588_linux/linux/external/camera_engine_rkaiq
```
Для полного понимания происходящего можно обратиться сюда https://github.com/MacroGroup/buildroot/tree/macro/package/diasom/rockchip/camera-engine-rkaiq

```
git clone https://gitlab.com/rk3588_linux/linux/external/camera_engine_rkaiq
```
```
cd ~/camera_engine_rkaiq
```
```
git checkout 77f10089
```

# Все патчи из папки ../patch/
```
for patch in ../patch/*.patch; do
    echo "Применяем $(basename "$patch")"
    patch -p1 < "$patch"
done
```

#  Сборка
```
export SDK_PATH=/home/tnv/SOVA2.0/SDK/aarch64-buildroot-linux-gnu_sdk-buildroot
export CC=$SDK_PATH/bin/aarch64-buildroot-linux-gnu-gcc
export CXX=$SDK_PATH/bin/aarch64-buildroot-linux-gnu-g++
export SYSROOT=$SDK_PATH/aarch64-buildroot-linux-gnu/sysroot
```
```
mkdir build && cd build
```
```
cmake .. \
    -DCMAKE_BUILD_TYPE=MinSizeRel \
    -DISP_HW_VERSION=-DISP_HW_V21 \
    -DRKAIQ_TARGET_SOC=rk356x \
    -DBUILDROOT_BUILD_PROJECT=TRUE \
    -DARCH=aarch64 \
    -DCMAKE_C_COMPILER=$SDK_PATH/bin/aarch64-buildroot-linux-gnu-gcc \
    -DCMAKE_CXX_COMPILER=$SDK_PATH/bin/aarch64-buildroot-linux-gnu-g++ \
    -DCMAKE_SYSROOT=$SDK_PATH/aarch64-buildroot-linux-gnu/sysroot \
    -DCMAKE_INSTALL_PREFIX=./install  
```
```
make -j$(nproc)
```



void *engine_thread(void *arg)
{
    int ret = 0;
    int isp_fd;
    unsigned int stream_event = -1;
    struct rkaiq_media_info *media_info;

    media_info = (struct rkaiq_media_info *) arg;

    isp_fd = open(media_info->vd_params_path, O_RDWR);
    if (isp_fd < 0) {
        ERR("open %s failed %s\n", media_info->vd_params_path, strerror(errno));
        return NULL;
    }

    /* NOTE: раньше init_engine вызывался здесь. Теперь вызываем его
       внутри цикла, чтобы при каждом старте стрима можно было прочитать
       актуальный V4L2 input и подставить нужный sensor */
    subscrible_stream_event(media_info, isp_fd, true);

    for (;;) {
        /* 1) Попробуем прочитать текущий input на mainpath (e.g. /dev/video0) */
        int main_fd = open(media_info->mainpath, O_RDONLY);
        if (main_fd >= 0) {
            int input = 0;
            if (xioctl(main_fd, VIDIOC_G_INPUT, &input) == 0) {
                DBG("%s: current V4L2 input: %d\n", media_info->mainpath, input);

              
                if (input == 0) {
                    strncpy(media_info->sensor_entity_name, "imx462 10-001a", sizeof(media_info->sensor_entity_name)-1);
                } else if (input == 1) {
                    strncpy(media_info->sensor_entity_name, "imx462 11-001a", sizeof(media_info->sensor_entity_name)-1);
                } else {
                    DBG("%s: unknown input %d, using default sensor_entity_name '%s'\n",
                        media_info->mainpath, input, media_info->sensor_entity_name);
                }
                media_info->sensor_entity_name[sizeof(media_info->sensor_entity_name)-1] = '\0';
            } else {
                DBG("VIDIOC_G_INPUT on %s failed: %s\n", media_info->mainpath, strerror(errno));
            }
            close(main_fd);
        } else {
            DBG("open %s failed: %s\n", media_info->mainpath, strerror(errno));
        }

        init_engine(media_info);
        subscrible_stream_event(media_info, isp_fd, true);
        start_engine(media_info);

        DBG("%s: wait stream start event...\n", media_info->mdev_path);
        wait_stream_event(isp_fd, CIFISP_V4L2_EVENT_STREAM_START, -1);
        DBG("%s: wait stream start event success ...\n", media_info->mdev_path);

        DBG("%s: wait stream stop event...\n", media_info->mdev_path);
        wait_stream_event(isp_fd, CIFISP_V4L2_EVENT_STREAM_STOP, -1);
        DBG("%s: wait stream stop event success ...\n", media_info->mdev_path);

        stop_engine(media_info);
        deinit_engine(media_info);
    }

    subscrible_stream_event(media_info, isp_fd, false);
    close(isp_fd);

    return NULL;
}
