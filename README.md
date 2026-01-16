# wayland_weston_embeded_snippet

## Создаем XDG_RUNTIME_DIR
mkdir -p /run/user/0
chmod 700 /run/user/0
export XDG_RUNTIME_DIR=/run/user/0

## Запускаем Weston через openvt
openvt -s -- weston

## Проверка сокета
ls /run/user/0

## Запускаем видеопоток с камеры
export WAYLAND_DISPLAY=wayland-НОМЕР_СОКЕТА
gst-launch-1.0 v4l2src device=/dev/video_НОМЕР_КАМЕРЫ ! videoconvert ! waylandsink

