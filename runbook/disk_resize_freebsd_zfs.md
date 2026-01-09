# Runbook – Expansão de Disco FreeBSD com ZFS no Proxmox VE

## Objetivo
Expandir o disco de uma VM FreeBSD hospedada no Proxmox VE e permitir
que o pool ZFS utilize o novo espaço disponível, sem reinstalação
do sistema operacional.

---

## Ambiente validado

- Hypervisor: Proxmox VE
- BIOS da VM: SeaBIOS (Default)
- Disco virtual: SCSI
- Tabela de partição: GPT
- Filesystem: ZFS
- Pool: zroot

---

## Fora do escopo

⚠️ Este runbook **não aborda**:
- Criação, remoção ou redimensionamento de SWAP
- Swap em partição
- Swap em ZVOL
- Swap em arquivo

---

## Pré-requisitos

- VM desligada no Proxmox VE
- Backup ou snapshot recente
- Pool ZFS saudável (`zpool status`)
- Acesso root ao FreeBSD

---

## Etapa 1 – Resize do disco no Proxmox VE

1. Selecione a VM com o sistema desligado
2. Acesse a aba **Hardware**
3. Selecione o disco **SCSI**
4. Clique em **Disk Action → Resize**
5. Informe o valor de incremento em **Size Increment (GiB)**

### Exemplo

Disco atual: 50 GB  
Incremento desejado: 100 GB  

<img src="https://github.com/herculesrocha/proxmox-freebsd-zfs-disk-resize/blob/main/Exemplo_Resize_Proxmox.png" alt="Um Exemplo Gerado por IA">
