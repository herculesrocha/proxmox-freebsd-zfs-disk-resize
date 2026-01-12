# Proxmox VE – Disk Resize em VM FreeBSD com ZFS

Este repositório documenta o procedimento técnico para expansão de disco
em máquinas virtuais FreeBSD utilizando ZFS, hospedadas no Proxmox VE.

O foco é a ampliação segura do armazenamento, preservando a integridade
da tabela GPT e permitindo que o pool ZFS utilize o novo espaço disponível.

Esse atividade foi executada por não termos optado por acrescentar um novo HD.

---

## Escopo do documento

✅ Proxmox VE  
✅ FreeBSD  
✅ ZFS (zroot)  
✅ Disco virtual SCSI  
✅ BIOS Legacy (SeaBIOS)  

❌ Gerenciamento ou recriação de SWAP (fora do escopo)

> **Observação:** este procedimento considera ambientes onde a swap
é gerenciada separadamente ou não é redimensionada junto ao disco.

---

## Visão geral do processo

1. Expandir o disco virtual no Proxmox VE
2. Corrigir possíveis inconsistências da GPT
3. Redimensionar a partição ZFS
4. Expandir o pool ZFS
5. Validar o novo espaço disponível

📘 O procedimento completo está documentado no runbook:

➡️ [`runbook/disk_resize_freebsd_zfs.md`](/runbook/disk_resize_freebsd_zfs.md)

---

## ⚠️ Avisos importantes

- Realize backup antes de qualquer operação em disco
- Execute todos os comandos como `root`
- Confirme o identificador correto do disco (ex.: `ada0`)
