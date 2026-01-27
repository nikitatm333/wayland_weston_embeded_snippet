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

RK3568 SoC с ISP2 (rkisp версии v02.03.00).
