# Mousecore —_controle e teste do Attack Shark X1_

Aplicativo livre para **configurar e testar o mouse Attack Shark X1 PAW3395 Pro** em Linux, com uma interface HTML puro que também funciona em Windows, macOS, Android e navegador (Chrome/Chromium com WebHID).

A interface foi criada com auxílio do OpenCode a partir do Windows 11 Pro, onde conexões, leituras e gravações foram validadas no mouse. Os instaladores Linux foram gerados em seguida com Electron.

---

## ✨ Recursos

- **Visão geral** — status da conexão, bateria, DPI ativo e atalhos para as demais abas.
- **Iluminação** — cores/efeitos do mouse e pré-visualização do dock (RGB).
- **DPI & polling** — 6 estágios de DPI (50 a 26000) e taxas de 125/250/500/1000 Hz.
- **Teste do mouse** — verificação visual dos botões esquerdo, meio, direito, voltar, avançar e DPI.
- **Conectividade** — USB com fio (`1d57:2111`), dongle 2.4 GHz (`1d57:fa60`, somente leitura) e Bluetooth (WebBluetooth).
- **Presets** — salvar, aplicar, excluir, exportar e importar configurações em JSON.
- **Firmware** — verificação de atualização e validação de arquivo (gravação mantida bloqueada até o protocolo de bootloader ser validado).
- **Modo demo** — explorar a interface sem o mouse conectado.
- Escala da interface ajustável; a janela é fixa (1200×900) e redimensionada pelos controles de escala.

## 📦 Instalação no Linux

### Forma rápida (sem root)

```bash
cd "/caminho/para/ATTACK SHARK X1"

./instalar.sh            # versão atual (Electron 45, x64)
./instalar-legacy.sh     # versão legada (Electron 22, glibc antiga, x64)
./desinstalar.sh         # remove apenas o app, o atalho e os ícones
```

Os scripts copiam o AppImage para `~/.local/bin`, criam o atalho **Mousecore** no menu e instalam o ícone em `~/.local/share/icons`. Nenhum outro aplicativo (Pianocore, Wpcore, ZPL_PRINT, Attack Shark X82 etc.) é alterado.

> No CachyOS/Arch Linux use **AppImage, `.sh` ou Flatpak**. Os pacotes `.deb`, `.rpm` e `.apk` são para Debian/Ubuntu, Fedora/RHEL e Alpine, respectivamente; `pacman` não instala esses formatos.

### Permissão HID (necessária no Linux)

Sem a regra udev, o app exibe `Falha ao abrir HID: Failed to open the device.` libere os dispositivos com:

```bash
./instalar-udev.sh        # pede sudo; cria /etc/udev/rules.d/99-mousecore.rules
./desinstalar-udev.sh     # remove apenas essa regra
```

A regra libera somente `1d57:2111` (mouse/cabo) e `1d57:fa60` (dongle 2.4 GHz). Depois de instalá-la: feche e reabra o app, desconecte e reconecte o USB; em último caso, faça logout e login.

### Instalar diretamente pelo pacote em `dist/`

| Formato | Comando | Observação |
|---|---|---|
| AppImage | `chmod +x dist/Mousecore-1.0.0.AppImage && ./dist/Mousecore-1.0.0.AppImage` | qualquer distro, sem root |
| AppImage ARM64 | `./dist/Mousecore-1.0.0-arm64.AppImage` | em máquina ARM |
| AppImage legado | `./dist/Mousecore-1.0.0-legacy-x86_64.AppImage` | glibc antiga |
| Instalador `.sh` | `bash dist/Mousecore-1.0.0.sh` | instala em `$HOME/.local/bin`; remover com `--remove` |
| `.deb` | `sudo apt install ./dist/Mousecore-1.0.0.deb` | Debian/Ubuntu/Mint/Pop!_OS |
| `.rpm` | `sudo dnf install ./dist/Mousecore-1.0.0.rpm` | Fedora/RHEL/openSUSE; remover com `sudo rpm -e mousecore` |
| `.apk` | `apk add --allow-untrusted dist/Mousecore-1.0.0-x64.apk` | Alpine (depende de `gcompat` e `fuse`) |
| Flatpak | `flatpak install --bundle dist/Mousecore-1.0.0.flatpak && flatpak run com.solmus.Mousecore` | remover com `flatpak uninstall com.solmus.Mousecore` |

O pacote `.deb`/`.rpm` instala os arquivos em `/usr/share/mousecore` e cria o atalho `/usr/bin/mousecore`.

## 🛠️ Compilar a partir do código

Requisitos: **Node.js com npm**, Python 3 e (apenas para o Flatpak) `flatpak-builder`.

```bash
cd electron
npm install
npm run build:all
cd ..
```

Os artefatos são gerados em `dist-electron/` e reunidos em `dist/`:

- `Mousecore-1.0.0.AppImage` / `-arm64.AppImage` / `-legacy-x86_64.AppImage`
- `Mousecore-1.0.0.deb` / `-arm64.deb` / `-legacy-amd64.deb`
- `Mousecore-1.0.0.rpm` / `-arm64.rpm` / `-legacy-x64.rpm`
- `Mousecore-1.0.0.sh` / `-arm64.sh` / `-legacy-x64.sh`
- `Mousecore-1.0.0-x64.apk` e `Mousecore-1.0.0.flatpak`

Comandos individuais (dentro de `electron/`):

```bash
npm start                       # rodar direto do projeto, sem instalar
npm run dist                    # moderno x64 (AppImage + deb)
npm run dist:legacy             # legado x64 (AppImage + deb)
npx electron-builder --config builder.modern-arm64.json
python3 build-rpm.py --unpacked ../dist-electron/modern-x64/linux-unpacked \
  --output ../dist-electron/Mousecore-1.0.0.rpm --arch x64
bash build-sh.sh
npm run dist:apk                # pacote Alpine
npm run flatpak                 # bundle Flatpak
bash collect.sh                 # copia os artefatos para dist/
```

**Compatibilidade de build:** o `fpm` portátil do electron-builder precisa de `libcrypt.so.1` com `GLIBC_2.2.5`; o arquivo está em `electron/.compat/` e `build-all.sh` já exporta `LD_LIBRARY_PATH`. Se executar o electron-builder/fpm sozinho, exporte:

```bash
export LD_LIBRARY_PATH="$PWD/.compat${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

O Flatpak usa `org.freedesktop.Platform//26.08` e `org.freedesktop.Sdk//26.08`; se `flatpak-builder` não estiver no PATH:

```bash
PATH="/caminho/para/flatpak-builder:$PATH" npm run build:all
```

O `.rpm` é montado em Python (lead + cabeçalhos + payload cpio, com SHA1/SHA256), sem exigir `rpmbuild`. O `.apk` é formato v2 com datahash e checksums SHA1 por arquivo (validado com apk-tools 3.0.9). Não há mais build ia32; o legado é somente x64.

## 📁 Estrutura do repositório

| Caminho | Conteúdo |
|---|---|
| `index.html` | Interface HTML/CSS/JS do app (todas as abas e o protocolo HID) |
| `electron/` | Aplicação Electron, configs de build, scripts para deb/rpm/apk/sh/Flatpak e `node_modules` |
| `electron/app/` | Código empacotado: interface + `main.js` (janela e permissões) + `preload.js` |
| `dist/` | Pacotes prontos para distribuição |
| `dist-electron/` | Saída bruta do electron-builder |
| `botom/` | SVGs usados no teste dos botões (`Botom_0.svg` = solto; `Botom_1..6.svg` = botões 0 a 5) |
| `Vetores/` | Arte vetorial do projeto |
| `X1UserManual.pdf` | Manual do usuário do mouse |
| `Attack Shark Software/` | Software/driver original do fabricante para Windows (DLLs e licenças próprias) |
| `INSTALACAO.txt` | Orientações completas de instalação em português |
| `instalar*.sh` / `desinstalar*.sh` | Instaladores do AppImage e da regra udev |

`icon.png` é usado na interface; `icon1.png` é o ícone do ambiente desktop (janela, `.desktop`, AppImage, deb, rpm, apk e Flatpak).

## 🔌 Detalhes técnicos e limitações

- Protocolo X-series: feature reports `0x04` (DPI/RGB), `0x05` (iluminação), `0x06` (polling), `0xA0` (leitura) e mensagens de bateria `0x40/0x41`. Baseado em documentação pública de drivers open source da comunidade.
- **O dongle 2.4 GHz é somente leitura**: não gravamos DPI/RGB/polling nele. O handshake de leitura `0xA0` é o mesmo do Windows e não desfaz o pareamento. Para gravar, use o cabo USB (`1d57:2111`).
- O botão de gravação de firmware permanece desabilitado até o protocolo de bootloader ser validado — nunca gravar firmware "no escuro".
- No Wayland, se aparecer aviso de Vulkan (`--ozone-platform=wayland` is not compatible with Vulkan), execute com `ELECTRON_DISABLE_GPU=1`, por exemplo `ELECTRON_DISABLE_GPU=1 ./dist/Mousecore-1.0.0.AppImage`.

## ❓ Solução de problemas

- **Botões não aparecem na aba Teste** → instale a regra udev antes de abrir o app e reconecte o USB.
- **`WebHID não disponível`** → use o app Electron ou Chrome/Chromium em HTTPS/localhost.
- **`Failed to open the device`** → verifique a regra udev (`/etc/udev/rules.d/99-mousecore.rules`) e faça logout/login.
- **Arch/CachyOS** → prefira AppImage, `.sh` ou Flatpak; `pacman` não instala `.deb`/`.rpm`/`.apk`.

## 🤝 Contribuindo e reportando problemas

Abra uma issue em [github.com/solmus](https://github.com/solmus) com:

- descrição do comportamento, sistema operacional e navegador;
- IDs USB do dispositivo (`1d57:2111` ou `1d57:fa60`);
- bytes dos feature reports `0x04`, `0x05` e `0x06`, quando relevantes.

Nunca cole dados sensíveis. Como alternativa, use a aba **Reportar** dentro do próprio app.

> **Publicando no GitHub:** os binários em `dist/` passam de 100 MB (limite de arquivo por repositório) e `electron/node_modules/` tem centenas de MB. O `.gitignore` deste repositório já mantém `dist/`, `dist-electron/`, `electron/node_modules/` e os diretórios de build fora do commit. Publique os pacotes em **Releases**.

## 📄 Licença

MIT — texto completo no arquivo [LICENSE](LICENSE) (o mesmo declarado nos metadados do projeto). O diretório `Attack Shark Software/` contém componentes do fabricante com licenças próprias (`directui license.txt` e `duilib license.txt`).

---

Feito por **solmus** · [github.com/solmus](https://github.com/solmus) · versão 1.0.0
