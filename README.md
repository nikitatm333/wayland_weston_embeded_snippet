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

Камера
/etc/init.d/S40camera start

