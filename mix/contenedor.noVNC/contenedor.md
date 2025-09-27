
# Contenedor con Chromium + VNC + noVNC en Fedora (puerto 8080)

Este contenedor permite ejecutar Chromium en un entorno sin GUI (Fedora minimal), renderizado con Xvfb y accesible vía navegador web usando noVNC en el puerto 8080.

Como ejemplo, usamos una lista de reproduccion de videos.

---

## ✅ Características

- Corre Chromium con salida gráfica en Xvfb
- VNC servido por x11vnc
- Accesible desde navegador web vía noVNC (HTML5 + WebSockets)
- Todo dentro de un contenedor Podman compatible

---

## 🛠️ Estructura del proyecto

```
.
├── Dockerfile
├── start.sh
├── watch_once.sh
├── x11vnc.sh
└── noVNC/          <-- Clonado desde GitHub
```

---

## Preparar el ambiente

```
sudo dnf install -y podman
sudo dnf install -y podman git wget tar unzip
sudo dnf install -y python3-pip


sudo rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9
curl -Iv https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9
sudo dnf install -y   https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf makecache

sudo mkdir -p /run/user/$(id -u)
sudo chown "$USER":"$USER" /run/user/$(id -u)
ls -ld /run/user/$(id -u)


```


## 📄 Dockerfile

```
FROM fedora:40

RUN dnf install -y \
    chromium \
    xorg-x11-server-Xvfb \
    x11vnc \
    git \
    python3 \
    python3-pip \
    procps \
    mesa-dri-drivers \
    mesa-libGL \
    libXScrnSaver \
    libXcursor \
    libXrandr \
    alsa-lib \
    wget \
    tar \
    xdpyinfo \
    fluxbox \
    xdotool \
    && dnf clean all

# Clonar noVNC y websockify
RUN git clone https://github.com/novnc/noVNC.git /opt/noVNC && \
    git clone https://github.com/novnc/websockify /opt/noVNC/utils/websockify

RUN useradd -ms /bin/bash usuario

COPY start.sh watch_once.sh x11vnc.sh /home/usuario/
RUN chown usuario:usuario /home/usuario/*.sh && chmod +x /home/usuario/*.sh

USER usuario
WORKDIR /home/usuario

ENV DISPLAY=:0

CMD ["./start.sh"]

```

---

## ▶️ start.sh

```bash
#!/bin/bash

LOG_DIR="/tmp/kiosk-logs"
mkdir -p "$LOG_DIR"
echo "📁 Logs guardados en $LOG_DIR"

echo "🚀 Lanzando Xvfb en DISPLAY=:0"
Xvfb :0 -screen 0 1920x1080x24 > "$LOG_DIR/xvfb.log" 2>&1 &
XVFB_PID=$!
echo "🔧 Xvfb PID: $XVFB_PID"

export DISPLAY=:0

# Esperar hasta 10 segundos a que el DISPLAY esté disponible
for i in {1..10}; do
  if xdpyinfo -display $DISPLAY > "$LOG_DIR/xdpyinfo.log" 2>&1; then
    echo "✅ Xvfb activo en DISPLAY=$DISPLAY"
    break
  else
    echo "⏳ Esperando a que el display $DISPLAY esté disponible... ($i)"
    sleep 1
  fi
done

if ! xdpyinfo -display $DISPLAY > /dev/null 2>&1; then
  echo "❌ ERROR: El display $DISPLAY no está disponible después de 10 segundos."
  echo "📝 Contenido de $LOG_DIR/xvfb.log:"
  cat "$LOG_DIR/xvfb.log"
  exit 1
fi

echo "📋 xdpyinfo:"
cat "$LOG_DIR/xdpyinfo.log"

echo "🚀 Iniciando x11vnc"
./x11vnc.sh > "$LOG_DIR/x11vnc.log" 2>&1 &
X11VNC_PID=$!
echo "🔧 x11vnc PID: $X11VNC_PID"

echo "🌐 Lanzando noVNC en puerto 8080"
nohup /opt/noVNC/utils/novnc_proxy --vnc localhost:5900 --listen 8080 > "$LOG_DIR/novnc.log" 2>&1 &
NOVNC_PID=$!
echo "🔧 noVNC PID: $NOVNC_PID"

sleep 3

echo "🎥 Lanzando Chromium vía watch_once.sh"
./watch_once.sh > "$LOG_DIR/chromium.log" 2>&1 &
CHROME_PID=$!
echo "🔧 Chromium PID: $CHROME_PID"

echo "🛠️ Iniciando simulador de subtítulos (tecla 'c' cada 15 min)"
(
  while true; do
    sleep 900  # 15 minutos
    echo "$(date '+%Y-%m-%d %H:%M:%S') ⏱️ Simulando tecla 'c'" >> "$LOG_DIR/subtitles_simulator.log"
    xdotool key c >> "$LOG_DIR/subtitles_simulator.log" 2>&1
  done
) &
SIMULATOR_PID=$!
echo "🔧 Simulador de subtítulos PID: $SIMULATOR_PID"

echo ""
echo "📊 Procesos activos:"
ps -ef | grep -E "Xvfb|x11vnc|chromium|novnc|xdotool" | grep -v grep

echo ""
echo "📡 Kiosk corriendo. Accedé desde tu navegador a:"
echo "   👉 http://<IP-de-la-máquina>:8080/vnc.html"
echo ""
echo "📦 Contenedor se mantendrá activo. Logs vivos en $LOG_DIR"

# Mantener contenedor activo
tail -f /dev/null


```

## ▶️ start.sh modificado con otras opciones

```
#!/bin/bash

LOG_DIR="/tmp/kiosk-logs"
mkdir -p "$LOG_DIR"
echo "📁 Logs guardados en $LOG_DIR"

echo "🚀 Lanzando Xvfb en DISPLAY=:0"
Xvfb :0 -screen 0 1920x1080x24 > "$LOG_DIR/xvfb.log" 2>&1 &
XVFB_PID=$!
echo "🔧 Xvfb PID: $XVFB_PID"

export DISPLAY=:0

# Esperar hasta 10 segundos a que el DISPLAY esté disponible
for i in {1..10}; do
  if xdpyinfo -display $DISPLAY > "$LOG_DIR/xdpyinfo.log" 2>&1; then
    echo "✅ Xvfb activo en DISPLAY=$DISPLAY"
    break
  else
    echo "⏳ Esperando a que el display $DISPLAY esté disponible... ($i)"
    sleep 1
  fi
done

if ! xdpyinfo -display $DISPLAY > /dev/null 2>&1; then
  echo "❌ ERROR: El display $DISPLAY no está disponible después de 10 segundos."
  echo "📝 Contenido de $LOG_DIR/xvfb.log:"
  cat "$LOG_DIR/xvfb.log"
  exit 1
fi

echo "📋 xdpyinfo:"
cat "$LOG_DIR/xdpyinfo.log"

echo "🚀 Iniciando x11vnc"
./x11vnc.sh > "$LOG_DIR/x11vnc.log" 2>&1 &
X11VNC_PID=$!
echo "🔧 x11vnc PID: $X11VNC_PID"

echo "🌐 Lanzando noVNC en puerto 8080"
nohup /opt/noVNC/utils/novnc_proxy --vnc localhost:5900 --listen 8080 > "$LOG_DIR/novnc.log" 2>&1 &
NOVNC_PID=$!
echo "🔧 noVNC PID: $NOVNC_PID"

sleep 3

echo "🎥 Lanzando Chromium vía watch_once.sh"
./watch_once.sh > "$LOG_DIR/chromium.log" 2>&1 &
CHROME_PID=$!
echo "🔧 Chromium PID: $CHROME_PID"

echo "🛠️ Iniciando simulador de subtítulos (tecla 'c' cada 15 min)"
(
  while true; do
    sleep 900  # 15 minutos
    echo "$(date '+%Y-%m-%d %H:%M:%S') ⏱️ Simulando tecla 'c'" >> "$LOG_DIR/subtitles_simulator.log"
    xdotool key c >> "$LOG_DIR/subtitles_simulator.log" 2>&1
  done
) &
SIMULATOR_C_PID=$!
echo "🔧 Simulador de subtítulos PID: $SIMULATOR_C_PID"

echo "🛠️ Iniciando simulador de aleatorio + siguiente (cada 10–25 min)"
(
  while true; do
    RANDOM_MIN=$(shuf -i 10-25 -n 1)
    echo "$(date '+%Y-%m-%d %H:%M:%S') ⏳ Esperando $RANDOM_MIN minutos antes de 'aleatorio + siguiente'" >> "$LOG_DIR/random_skip_simulator.log"
    sleep "$((RANDOM_MIN * 60))"

    CHROME_WIN=$(xdotool search --name "YouTube" | head -n 1)
    if [[ -n "$CHROME_WIN" ]]; then
      xdotool windowactivate "$CHROME_WIN"
      sleep 1
    fi

    echo "$(date '+%Y-%m-%d %H:%M:%S') 🎲 Simulando 'Shift+R' (aleatorio)" >> "$LOG_DIR/random_skip_simulator.log"
    xdotool key shift+r >> "$LOG_DIR/random_skip_simulator.log" 2>&1

    sleep 1

    echo "$(date '+%Y-%m-%d %H:%M:%S') ⏭️ Simulando 'Shift+N' (siguiente)" >> "$LOG_DIR/random_skip_simulator.log"
    xdotool key shift+n >> "$LOG_DIR/random_skip_simulator.log" 2>&1


    echo "$(date '+%Y-%m-%d %H:%M:%S') 🔁 Click en botón de loop" >> "$LOG_DIR/random_skip_simulator.log"
    xdotool mousemove 1830 950 click 1 >> "$LOG_DIR/random_skip_simulator.log" 2>&1

  done
) &
SKIP_SIMULATOR_PID=$!
echo "🔧 Simulador aleatorio/siguiente PID: $SKIP_SIMULATOR_PID"

echo ""
echo "📊 Procesos activos:"
ps -ef | grep -E "Xvfb|x11vnc|chromium|novnc|xdotool" | grep -v grep

echo ""
echo "📡 Kiosk corriendo. Accedé desde tu navegador a:"
echo "   👉 http://<IP-de-la-máquina>:8080/vnc.html"
echo ""
echo "📦 Contenedor se mantendrá activo. Logs vivos en $LOG_DIR"

# Mantener contenedor activo
tail -f /dev/null

```


---

## ▶️ watch_once.sh

```bash
#!/bin/bash

export DISPLAY=:0

URL="https://www.youtube.com/playlist?list=PLBrdJqPHEZjuuzjTs7pZ_mnDagZHgqTfG"

while true; do
  echo "Iniciando Chromium en modo kiosko..."
  chromium-browser --no-sandbox --disable-gpu --disable-software-rasterizer \
    --autoplay-policy=no-user-gesture-required \
    --disable-features=MediaSessionService \
    --window-size=1024,768 --start-fullscreen --kiosk "$URL"
  echo "Chromium terminó. Reiniciando en 5 segundos..."
  sleep 5
done


```

## ▶️ Adaptando Watch_once.sh

1) Poniendo una playlist directamente "https://www.youtube.com/playlist?list=PLBrdJqPHEZjuuzjTs7pZ_mnDagZHgqTfG"

2) Reduciendo la pantalla a 1024,768 para que sea mas visible

3) Hay que entrar una vez, seleccionar que inicie el playlist y luego validar loop / shuffle, playback speed y close captions

4) Revisar que durante el dia ... no cambie nada... 

---

## ▶️ x11vnc.sh (opcional)

```bash
#!/bin/bash

export DISPLAY=:0

# Esperar a que el display esté disponible antes de lanzar x11vnc
while ! xdpyinfo -display $DISPLAY &>/dev/null; do
  echo "Esperando a que el display $DISPLAY esté disponible..."
  sleep 1
done

echo "Display $DISPLAY disponible. Iniciando x11vnc..."
x11vnc -display $DISPLAY -nopw -forever -shared -rfbport 5900

```

---

## 🧪 Cómo construir y ejecutar

```bash
podman build -t youtube-kiosk .

podman run -d --rm --privileged --user 0 -p 8080:8080 youtube-kiosk

```

---

## 🌐 Acceder desde el navegador

Conectate desde tu navegador a:

```
http://IP_DEL_SERVIDOR:8080/vnc.html
```

---

## 🧹 Detener y eliminar contenedores

```bash
podman stop youtube_kiosk
podman rm youtube_kiosk
```

---


## :bomb: Detener y elimnar todo 

```
podman ps -q | xargs -r podman stop
podman rm -a -f
podman container prune
podman rmi --all --force
```

