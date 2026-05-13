# Janus Gateway - Debian 13 Trixie Build

## Contexte

Le paquet `janus` a ete retire des depots officiels Debian 13 (Trixie).
Ce projet fournit un workflow de cross-compilation et un mecanisme d'installation
automatique pour les architectures ARM ou janus n'est plus disponible via apt.

## Architecture

```
install.sh
  └── ensure_deps()
        └── scripts/ensure_janus.sh
              ├── janus deja installe ?  → skip
              ├── apt-cache show janus ? → apt install (Bookworm)
              └── sinon → download .deb depuis GitHub Releases (Trixie+)

Runtime :
  moonraker_obico/janus_config_builder.py
    ├── find_system_janus_paths()  → dpkg -L janus (systeme / .deb)
    └── find_precompiled_dir()     → binaires embarques (RPi, MKS)
```

## Workflow : build-janus-deb.yml

### Declenchement

Manuel via `workflow_dispatch` :

```bash
gh workflow run build-janus-deb.yml \
  --repo Yumi-Lab/moonraker-yumi-lab \
  -f janus_version=v1.2.4
```

### Etapes du build

1. **QEMU user-static** : emulation armhf sur runner x86_64
2. **Container** : `debian:trixie` (platform `linux/arm/v7`)
3. **Build** : autotools (`autogen.sh` → `configure` → `make`)
4. **Package** : `dpkg-deb` avec detection auto des dependances (`dpkg-shlibdeps`)
5. **Release** : upload du `.deb` en GitHub Release

### Options de build janus-gateway

```
--disable-docs
--disable-data-channels
--disable-rabbitmq
--disable-mqtt
--disable-nanomsg
--disable-unix-sockets
--enable-websockets        ← requis pour moonraker-obico
--enable-turn-rest-api     ← requis pour TURN/obico
```

### Dependances de build

```
build-essential pkg-config git ca-certificates
automake autoconf libtool gengetopt
libmicrohttpd-dev libjansson-dev libssl-dev
libsrtp2-dev libglib2.0-dev libnice-dev
libwebsockets-dev libconfig-dev libcurl4-openssl-dev
libopus-dev libogg-dev
```

### Sortie

- **Artifact** : `janus-gateway-armhf-trixie`
- **Release** : `janus-v<version>-trixie-armhf`
- **Asset** : `janus-gateway_<version>_armhf.deb`

## ensure_janus.sh

Script d'installation automatique appele par `install.sh` :

| Situation | Action |
|-----------|--------|
| `janus` deja dans PATH | Skip |
| `apt-cache show janus` OK (Bookworm) | `apt install janus` |
| Pas dans apt (Trixie+) | Query API GitHub → download `.deb` → `dpkg -i` + `apt install -f` |

Le script est non-bloquant : si l'installation echoue, un warning est affiche
mais l'installation de moonraker-obico continue.

## Integration avec moonraker-obico

### Detection runtime (janus_config_builder.py)

```python
# Priorite 1 : janus systeme (installe via apt ou .deb)
(janus_bin_path, system_janus_lib_path) = find_system_janus_paths()
# → utilise dpkg -L janus pour trouver le binaire et les libs

# Priorite 2 : binaires precompiles embarques
# → precomplied/<board>.debian.<ver>.<bits>-bit/
```

### Compatibilite board

| Board | board_id | Precompiled | Systeme janus |
|-------|----------|-------------|---------------|
| Raspberry Pi | `rpi` | Oui (Debian 11, 12) | Fallback |
| MKS | `mks` | Oui (Debian 10) | Fallback |
| SmartPi ONE (AllWinner H3) | `NA` | Non | .deb Trixie |
| Autre | `NA` | Non | apt ou .deb |

## Ajout d'une nouvelle architecture

Pour ajouter arm64 Trixie par exemple :

1. Modifier le workflow pour ajouter un job `build-arm64` avec `--platform linux/arm64`
2. Changer `Architecture: armhf` → `Architecture: arm64` dans le control file
3. `ensure_janus.sh` detecte automatiquement l'architecture via `dpkg --print-architecture`

## Versions testees

| Janus | Debian | Arch | Statut |
|-------|--------|------|--------|
| v1.2.4 | Trixie (13) | armhf | OK - build ~15min |
