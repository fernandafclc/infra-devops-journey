# Lab 00 — Setup do Ambiente (VirtualBox)

## Objetivo
Criar a VM `vm-srv01` (Ubuntu Server 24.04 LTS) para os labs de Infra/DevOps/SRE.

## Especificações da VM
- Nome: vm-srv01
- SO: Ubuntu Server 24.04 LTS
- vCPU: 2
- RAM: 3 GB
- Disco: 30 GB (VDI, dinâmico)

## Rede
- Adaptador 1: NAT
- Adaptador 2: Host-Only (vboxnet0)

## Instalação
- ISO: ubuntu-24.04-live-server-amd64.iso
- OpenSSH Server: instalado (sim)

## Seleção de Snaps (instalador)
- Decisão: não instalar snaps opcionais para manter VM “clean”
- Motivo: reduzir consumo de recursos, facilitar troubleshooting e snapshots
- Planejado para depois:
  - aws-cli (Nível 4 – Cloud)
  - Prometheus (Nível 6 – Observabilidade)
  - Opcional/avançado: microk8s, keepalived, etcd

## Pós-instalação: atualização do sistema
Comandos executados:

```bash
sudo passwd root
sudo apt update
sudo apt -y full-upgrade
sudo apt -y autoremove
sudo reboot
```
Snapshot criado: Snapshot(clean)_v00
Data: 2026-06-16 22:37

