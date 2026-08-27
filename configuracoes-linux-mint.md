# Configurações do Sistema — Linux Mint 22.3 (Zena)

> Documento de referência pessoal. Última atualização: 27/08/2026.

---

## Índice

- [Especificações do Sistema](#especificações-do-sistema)
- [1. Hibernar](#1-hibernar)
  - [Como funciona](#como-funciona)
  - [Swapfile](#swapfile)
  - [Parâmetros do GRUB](#parâmetros-do-grub)
  - [Initramfs](#initramfs)
  - [Polkit](#polkit-permissão-para-hibernar-sem-sudo)
  - [Atualizar tudo após mudanças](#atualizar-tudo-após-mudanças)
  - [Hibernar pelo terminal](#hibernar-pelo-terminal)
  - [Atalho no desktop](#atalho-no-desktop-botão-direito)
- [2. Bateria — Controle de Carga](#2-bateria--controle-de-carga-lenovo-ideapad)
- [3. Dual Boot](#3-dual-boot)
- [4. NVM](#4-nvm-node-version-manager)
- [5. Nemo — Ações de Contexto](#5-nemo--ações-de-contexto-botão-direito)
- [Referência Rápida](#referência-rápida)

---

## Especificações do Sistema

- **OS:** Linux Mint 22.3 (Zena) — base Ubuntu Noble
- **Kernel atual:** 7.0.0-30-generic
- **RAM:** 16 GB
- **Disco:** NVMe (`/dev/nvme1n1`)
- **Setup:** Dual boot com Windows 11

---

## 1. Hibernar

O Linux Mint não habilita hibernação por padrão. Toda a configuração abaixo foi feita manualmente.

### Como funciona

A hibernação salva o conteúdo da RAM no swapfile em disco, desliga o PC e restaura a sessão no próximo boot — igual ao Windows.

### Swapfile

O swapfile precisa ter tamanho igual ou maior que a RAM (16 GB):

```
/swapfile — 16 GB — montado em /etc/fstab
```

Para recriar do zero (se necessário):

```bash
sudo swapoff /swapfile
sudo rm /swapfile
sudo dd if=/dev/zero of=/swapfile bs=1M count=16384 status=progress
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Parâmetros do GRUB

Arquivo: `/etc/default/grub`

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=ea780bf5-b25f-49b6-bf22-79a9b4ff26a2 resume_offset=4069376"
```

- `resume=UUID=...` — UUID da partição raiz (`/`) que contém o swapfile
- `resume_offset=4069376` — posição do swapfile no disco (obtido com `sudo filefrag -v /swapfile`)

> Se recriar o swapfile, o `resume_offset` muda. Rode `sudo filefrag -v /swapfile | awk 'NR==4{print $4}' | tr -d '.'` e atualize o GRUB.

### Initramfs

Arquivo: `/etc/initramfs-tools/conf.d/resume`

```
RESUME=UUID=ea780bf5-b25f-49b6-bf22-79a9b4ff26a2
```

### Polkit (permissão para hibernar sem sudo)

Arquivo: `/etc/polkit-1/rules.d/10-hibernate.rules`

```javascript
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.login1.hibernate" ||
        action.id == "org.freedesktop.login1.hibernate-multiple-sessions") {
        if (subject.isInGroup("sudo")) {
            return polkit.Result.YES;
        }
    }
});
```

### Atualizar tudo após mudanças

```bash
sudo update-grub
sudo update-initramfs -u
```

### Hibernar pelo terminal

```bash
sudo systemctl hibernate
```

### Atalho no desktop (botão direito)

Arquivo: `~/.local/share/nemo/actions/hibernate.nemo_action`

Aparece como opção "Hibernar" no menu de contexto do desktop.

---

## 2. Bateria — Controle de Carga (Lenovo IdeaPad)

Aliases definidos em `~/.bashrc` para controlar o limite de carga da bateria via `conservation_mode`.

### Travar em 60% (recomendado para uso plugado)

```bash
bat60
```

O que faz: `echo 1 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode`

### Liberar até 100%

```bash
bat100
```

O que faz: `echo 0 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode`

### Carregamento Rápido (Rapid Charge)

```bash
rapido-on    # ativa carregamento rápido
rapido-off   # desativa carregamento rápido
```

> O carregamento rápido e o `bat60` são mutuamente exclusivos — não ativar os dois ao mesmo tempo.

---

## 3. Dual Boot

- Windows 11 detectado automaticamente pelo GRUB via `os-prober`
- O nome "Ubuntu" que aparece no menu do GRUB para o Mint é **intencional** — o arquivo `/etc/default/grub.d/50_linuxmint.cfg` força `GRUB_DISTRIBUTOR="Ubuntu"` por compatibilidade com EFI. Não alterar.

---

## 4. NVM (Node Version Manager)

Instalado e configurado no `~/.bashrc`. Para usar:

```bash
nvm list           # versões instaladas
nvm use <versão>   # trocar versão do Node
nvm install <versão>
```

---

## 5. Nemo — Ações de Contexto (Botão Direito)

### Abrir pasta com VS Code

Arquivo: `~/.local/share/nemo/actions/open-in-vscode.nemo_action`

```ini
[Nemo Action]
Name=Abrir com VS Code
Comment=Abrir pasta no Visual Studio Code
Exec=code %F
Icon-Name=visual-studio-code
Selection=any
Extensions=dir;
```

Clique com botão direito em qualquer pasta no Nemo e a opção "Abrir com VS Code" aparece. Nemo detecta automaticamente, sem reiniciar.

---

## Referência Rápida

| O que fazer | Comando |
|---|---|
| Hibernar | `sudo systemctl hibernate` |
| Travar bateria em 60% | `bat60` |
| Liberar bateria até 100% | `bat100` |
| Carregamento rápido ON | `rapido-on` |
| Carregamento rápido OFF | `rapido-off` |
| Atualizar GRUB | `sudo update-grub` |
| Atualizar initramfs | `sudo update-initramfs -u` |
| Abrir pasta com VS Code (botão direito) | Nemo action em `~/.local/share/nemo/actions/` |
