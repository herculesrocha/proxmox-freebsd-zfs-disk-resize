# Runbook – Expansão de Disco FreeBSD com ZFS no Proxmox VE

## Objetivo
Expandir o disco de uma VM FreeBSD hospedada no Proxmox VE e permitir
que o pool ZFS utilize o novo espaço disponível, sem reinstalação
do sistema operacional.
OBS.: Essa atividade foi executada, desta forma, por não termos optado por acrescentar um novo HD.
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

⚠️ Este runbook **não abordará**:
- Criação, remoção ou redimensionamento de SWAP
- Swap em partição
- Swap em ZVOL
- Swap em arquivo

---

## Pré-requisitos

- VM dervá ser reiniciada
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
Optei por não utilizar print em produção
A imagem abaixo é meramente ilustrativa (gerada por IA)
<img src="https://github.com/herculesrocha/proxmox-freebsd-zfs-disk-resize/blob/main/Exemplo_Resize_Proxmox.png" alt="Um Exemplo Gerado por IA">
Size Increment (GiB): 100
```cpp

Resultado final:
```
Tamanho total do disco: 150 GB

## Etapa 2 – Inicialização e verificação do disco

Após o resize, inicialize a VM.

Em alguns cenários (No meu caso), o FreeBSD pode indicar inconsistência na GPT:
ada0 GPT (size mismatch) [CORRUPT]

## Etapa 3 – Recuperação da GPT

Execute o comando abaixo para ajustar a tabela GPT ao novo tamanho do disco:

```sh
gpart recover /dev/ada0
```
Esse comando é seguro e não altera dados existentes.

## Etapa 4 – Identificação das partiçõe
Liste o layout do disco:
```sh
gpart show
```
Ou especificamente:
```sh
gpart show -p ada0
```
Identifique:
A partição ZFS (ex.: ada0p3)
Espaço livre disponível no final do disco

## Etapa 5 – Redimensionamento da partição ZFS
Utilize gpart resize para expandir a partição ZFS.
Parâmetros

-i: índice da partição
-s: novo tamanho desejado
-a: alinhamento da partição

Exemplo:
```sh
gpart resize -i 3 -s 141G -a 4k ada0
```
Neste exemplo:
* A partição ada0p3 será expandida
* O tamanho final será 141 GB
* O alinhamento será ajustado para 4K

## Etapa 6 – Expansão do pool ZFS
Após redimensionar a partição, o pool ZFS ainda não utiliza automaticamente
o novo espaço.

Execute:
```sh
zpool online -e zroot ada0p3
```

## Etapa 7 – Validação

```sh
zpool status
zpool list
```
Confirme se o novo espaço está disponível no pool zroot.

🔁 Rollback (Resolvi Documentar como Procedimento de Estudo)

⚠️ Observação: rollback raramente é necessário neste procedimento,
mas esta seção foi criada para fins didáticos e estudo.

Cenários possíveis de rollback
* Erro no índice da partição
* Redimensionamento incorreto
* Pool não expandido conforme esperado

Estratégias de rollback \
1️⃣ Pool não expandiu

O redimensionamento pode ser reaplicado:
```sh
zpool online -e zroot ada0p3
```

2️⃣ Partição redimensionada incorretamente \
Se o ZFS ainda não foi expandido, é possível ajustar novamente:
```sh
gpart resize -i 3 -s <tamanho_anterior> ada0
```
⚠️ Após expansão do pool ZFS, não é possível reduzir o tamanho do pool.

3️⃣ Recuperação via snapshot / backup \
Caso o procedimento gere inconsistências: 
* Restaurar snapshot do Proxmox VE
* Restaurar backup completo da VM

Considerações finais
* Como já utilizar algumas vezes, posso dizer que o procedimento é seguro quando seguido corretamente
* Resolvi deixar do escopo a SWAP para reduzir riscos operacionais
* Ideal para ambientes virtualizados com crescimento gradual de storage


---

# 📋 TODOS OS COMANDOS JUNTOS (copiar e usar)

```sh
gpart recover /dev/ada0
```
```sh
gpart show
gpart show -p ada0
```
```sh
gpart resize -i 3 -s 141G -a 4k ada0
```
```sh
zpool online -e zroot ada0p3
```
```sh
zpool status
zpool list
```
