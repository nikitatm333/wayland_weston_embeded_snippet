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
