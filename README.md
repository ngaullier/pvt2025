```
██████  ██    ██ ████████     ██████   ██████  ██████  ███████ 
██   ██ ██    ██    ██             ██ ██  ████      ██ ██      
██████  ██    ██    ██         █████  ██ ██ ██  █████  ███████ 
██       ██  ██     ██        ██      ████  ██ ██           ██ 
██        ████      ██        ███████  ██████  ███████ ███████ 
```

# Présentation

Ce projet montre comment réaliser un banc de test de codecs vidéo en utilisant uniquement des outils open-source. Tout est reproductible bout en bout.

C'est par essence un POC et le projet sera archivé à la fin de l'année.

Pré-requis: conçu pour Debian 13 (Trixie) / valable également sur WSL/Debian 13.

# Préparation

## Install

```bash
sudo apt install mpv nemo stow zenity
cd install
sudo ./setup install
```

## Sources (shorts)

4 clips 1080p25 pris sur pexels.com :

[Wolfgang Weiser - Photographie](https://www.pexels.com/fr-FR/@wolfgang-weiser-467045605/?filter=videos)

https://www.pexels.com/fr-fr/download/video/34100948/

https://www.pexels.com/fr-fr/download/video/32886287/

https://www.pexels.com/fr-fr/download/video/35007492/

https://www.pexels.com/fr-fr/download/video/32471238/

## Trim

Pour des programmes longs, on peut trimmer avec `mpv`:

[GitHub - aerobounce/trim.lua: Trim mode for mpv — Turn mpv into Lossless Audio / Video Editor](https://github.com/aerobounce/trim.lua)

💡 TIP: débrider le trim pour ne pas imposer un TCin sur keyframe.

## Encodage

### build ffmpeg incluant vmaf

[Releases · BtbN/FFmpeg-Builds · GitHub](https://github.com/BtbN/FFmpeg-Builds/releases)

Mettre ffmpeg dans le PATH, typiquement via /usr/local/bin.

### encodage hevc

```bash
ffmpeg -codecs|grep hevc
```

```bash
ffmpeg -h encoder=libx265
```

Helpers: [script unitaire](./encode) et [batch](./batch_process_all)

```bash
./encode media/train.mp4 hevc_vaapi
```

## Analyse syntaxique

### dump

```bash
ffmpeg -i media/train.mp4_libx265.mp4 -map v -c copy \
    -bsf trace_headers -t 5 -f null /dev/null
```

```bash
ffmpeg -i media/train.mp4_libx265.mp4 -map v -c copy \
    -time_base 1/25 -t 5 -f framecrc -
```

👍 Intégration à némo : `PVT analyzer`.
Comparaison des dumps avec `meld`.

### visualisation

`mpv` customisé : image par image et OSD avec TC et pict_type: `PVT player`.

👍 Intégration à némo : `PVT player`.

[input.conf](install/usr/local/etc/pvt_player/input.conf)

## Analyse VMAF

Boîte à outil [vmafstat](https://github.com/ngaullier/vmafstat).

```bash
git clone https://github.com/ngaullier/vmafstat
cd vmafstat
./install
```

ffsync

```bash
/srv/vmafstat/ffsync -V -r media/train.mp4 \
                        -i media/train.mp4_libx265.mp4
```

ffvmaf (incluant ffsync)

```bash
/srv/vmafstat/ffvmaf -V -r media/train.mp4 \
                        -i media/train.mp4_libx265.mp4 \
                        -o media/output.json
```

👍 Intégration à némo : `PVT vmaf-ize`.

## vivictpp

```bash
sudo snap install vivictpp
```

👍 Intégration à némo : `PVT vivictpp`.

## plotvmaf

Calcul statistique avec `R` et production d'un graphe dynamique avec `plotly`.

💡 TIP: Choix de la moyenne harmonique, cf [faq netflix](https://github.com/Netflix/vmaf/blob/6b75f37728b2eb70c11508ece93afaacc6572b45/resource/doc/faq.md#q-why-the-aggregate-vmaf-score-sometimes-may-bias-easy-content-too-much-issue-20).

👍 Intégration à némo : `PVT plotvmaf`.

## PVT player

💡 `mpv` customisé : mode multi-angles, côte à côte et "diff".

👍 Intégration à némo : `PVT player`.

[pvt_player](install/usr/local/bin/pvt_player)
[input_2.conf](install/usr/local/etc/pvt_player/input_2.conf)
[input_3.conf](install/usr/local/etc/pvt_player/input_3.conf)

## Intégration bureau gnome / GTK

Modélisation avec `glade`.
Wrapper GTK pour bash :  [GitHub - abecadel/gtkwrap: GTK gui in bash.](https://github.com/abecadel/gtkwrap)

```bash
wget \
'https://github.com/ngaullier/gtkwrap/releases/download/v1.0/gtk-wrap-1.0-0-g553a592-amd64.deb'
sudo apt install ./gtk-wrap-1.0-0-g553a592-amd64.deb
```

👍 Intégration à némo : `PVT player-ui`
