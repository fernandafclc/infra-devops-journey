# Lab 00 — Setup do Ambiente (VirtualBox)

## Objetivo
Criar ambiente de laboratório para executar tarefas e oportunidade de melhorias de infraestrutura e contrução de consciência DevOps.
- Criar a VM `vm-srv01` (Ubuntu Server 24.04 LTS).
- Criar a VM `vm-cli01` (Ubuntu Server 24.04 LTS).

## Especificações da VM-SRV01 
- Nome: vm-srv01
- SO: Ubuntu Server 24.04 LTS
- vCPU: 2
- RAM: 3 GB
- Disco: 30 GB (VDI, dinâmico)

## Rede
- Adaptador 1: NAT
- Adaptador 2: Host-Only (vboxnet0)
  - Subnet 192.168.56.0/24

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
# Atribuir senha para o root
sudo passwd root
# Atualização do catálogo de pacotes do sistema
sudo apt update
# Instalação dos pacotes de sistema mais atuais
sudo apt -y full-upgrade
# Limpeza pós atualização
sudo apt -y autoremove
# Reinicialização completa do sistema
sudo reboot
```
Snapshot vm-srv01 criado: Snapshot(clean)_v00
Data: 2026-06-16 22:37

## Especificações da VM-CLI01
- Clone da vm-srv01 (clean)
- Nome: vm-cli01
- SO: Ubuntu Server 24.04 LTS
- vCPU: 1
- RAM: 2 GB
- Disco: 30 GB (VDI, dinâmico)

>
- Ganha tempo (sem reinstalar).
- Mantém as duas VMs com a mesma “base limpa” (ótimo pra troubleshooting e documentação).
- Evita desvio de foco: este laboratório tem objetico de treinar rede/SSH/infra, não ficar repetindo instalação.

## Adequação para cliente
Configurações VirtualBox
- Marcação “reinitialize MAC address / gerar novos MACs” para evitar conflito de hardware.
- Adequação de recursos de CPu e RAM.
- Full clone para evitar dependências de disco.
- Rede 
   - NAT(DHCP)
   - Host-Only (IP fixo)
	- Subnet 192.168.56.0/24
 
Comandos executados

```bash
# Renomear o nome do host para o nome correto (cliente)
sudo hostnamectl set-hostname vm-cli01
# Verificar se as configurações do host estão corretas
sudo nano /etc/hosts 
# Gerar novo machine-ID para evitar conflitos (por ser clone de outra VM)
sudo rm -f /etc/machine-id
sudo systemd-machine-id-setup
# Reinicialização completa do sistema
sudo reboot

```

Snapshot vm-cli01 criado: Snapshot(clean)
Data: 2026-06-17 00:45

## Configuração de netplan

Comandos executados:

```bash
# Listar configurações iniciais
ls -l /etc/netplan
# Não existia um arquivo netplan válido, foi necessário criar um novo com as configurações requeridas
sudo nano /etc/netplan/90-lab.yaml
```

# Conteúdo do arquivo 90-lab.yaml (cliente e servidor)
```yaml
network:
  version: 2
  ethernets:
    enp0s3:      # NAT (internet)
      dhcp4: true
    enp0s8:      # Host-Only (lab interno)
      dhcp4: false
      addresses:
        - 192.168.56.2/24   # (Ajuste: .3 no cliente)
```

```bash
# Teste e aplicação da segurança
sudo netplan try
# Aplicação definitiva
sudo netplan apply
```

## Testes de conectividade de rede

```bash
# Na vm-cli01
# Verificação do status das interfaces de rede
ip -br a
# Ping cliente -> servidor
ping -c 4 <IP_HOSTONLY_DO_SERVER>
# Conexão via ssh
ssh <usuario>@<IP_HOSTONLY_DO_SERVER>

# Na vm-srv01
# Ping servidor -> cliente
ping -c 4 <IP_HOSTONLY_DO_CLIENTE>
# Conexão via ssh
ssh <usuario>@<IP_HOSTONLY_DO_CLIENTE>
```

