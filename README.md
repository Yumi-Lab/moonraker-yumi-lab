# Moonraker-Yumi-Lab (Legacy — SmartPad V1)

> **⚠️ Ce repo est en mode maintenance uniquement.**
> Pour les nouvelles installations (YumiOS V2 / SmartPi ONE), utilisez :
> **[Yumi-Lab/moonraker-app-yumi-lab](https://github.com/Yumi-Lab/moonraker-app-yumi-lab)**

---

Plugin Moonraker pour connecter les imprimantes 3D Klipper au serveur Yumi Lab.
Fournit la détection IA de défauts, le streaming WebRTC via Janus, et le monitoring à distance.

## Plateformes supportées

| Plateforme | Repo | Service |
|------------|------|---------|
| SmartPad V1 (arm64, Bookworm) | **ce repo** | `moonraker-obico` |
| SmartPi ONE V2 (armhf, Trixie) | [moonraker-app-yumi-lab](https://github.com/Yumi-Lab/moonraker-app-yumi-lab) | `moonraker-yumi` |

## Installation (SmartPad V1 uniquement)

```bash
cd ~
git clone https://github.com/Yumi-Lab/moonraker-yumi-lab.git
cd moonraker-yumi-lab
chmod +x install.sh
./install.sh
```

## Désinstallation

```bash
sudo systemctl stop moonraker-obico.service
sudo systemctl disable moonraker-obico.service
sudo rm /etc/systemd/system/moonraker-obico.service
sudo systemctl daemon-reload
rm -rf ~/moonraker-yumi-lab
rm -rf ~/moonraker-obico-env
```

## Documentation

- [Wiki Yumi Lab](https://wiki.yumi-lab.com)
- [Discord Yumi Lab](https://discord.yumi-lab.com)
