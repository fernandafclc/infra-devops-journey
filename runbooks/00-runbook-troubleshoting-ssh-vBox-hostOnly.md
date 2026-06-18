### Troubleshooting & Correção: SSH em VirtualBox Host-Only
## 1. Cenário/Situação Inicial
# Ambiente de laboratório usando VirtualBox com rede Host-Only (vboxnet0).
Infra:
- PC (host): Ubuntu 24.04
- VMs: server e client, ambos na rede 192.168.56.0/24.
Sintoma:
- Cliente SSH dentro de VM conecta normalmente ao server.
- Do PC (host) para server, recebe Connection refused, mesmo com IP e serviço SSH aparentemente corretos.

## 2. Sintoma e Hipótese Inicial
Sintoma

```bash
ssh -v USER@192.168.56.2
# Resultado:
# connect to address 192.168.56.2 port 22: Connection refused
```
> Ping funcionava, mas SSH não.
Hipóteses testadas
- Bloqueio de porta (firewall).
- SSH inativo na VM.
- Serviço SSH ouvindo apenas em localhost.
- Problema de rede VirtualBox.
- Conflito de IP/ARP/MAC.

## 3. Testes & Diagnóstico
# 3.1. Teste de conectividade e porta

```bash
ping 192.168.56.2
nc -vz 192.168.56.2 22
#nc retorna Connection refused, sugerindo resposta de algo (não timeout).
```

# 3.2. Verificação do serviço SSH
No server:

```bash
sudo systemctl status ssh
sudo netstat -tulnp | grep ':22'
#SSH está “LISTEN 0.0.0.0:22”.
```

#3.3. Checagem do host-only e IP
No PC:

```bash
ip -br addr show vboxnet0
# Saída:
# vboxnet0 ... UP ... inet 192.168.56.1/24
```

# 3.4. Diagnóstico de MAC/ARP
No PC:

```bash
arp -an | grep 192.168.56.2
```

No server:

```bash
cat /sys/class/net/enp0s8/address
#MACs não coincidem!
#Indica resposta de outro dispositivo ou estado “contaminado” da rede host-only.
```

## 4. Root Cause Analysis (RCA)
> Causa Encontrada
> Conflito de IP/ARP:
> A máquina que respondeu por 192.168.56.2 não era a VM esperada.
> Isso pode ocorrer por: conflito de IP, cache ARP antigo, VM não linkada corretamente ao vboxnet0, erro do VirtualBox.
Evidência
> tcpdump na VM não via pacotes chegando quando tentava SSH do notebook.
> ARP do notebook não batia com MAC da VM.

## 5. Correção Aplicada
# 5.1. Redefinir IPs das VMs
- Server: 192.168.56. ...
- Client: 192.168.56. ...
 
# 5.2. Limpar e reatribuir IP na interface host-only do PC

```bash
sudo ip addr flush dev vboxnet0
sudo ip addr add <IP_DO_PC>/24 dev vboxnet0

## 5.3. Validar ARP e conectividade

```bash
ip neigh show dev vboxnet0
ssh -vvv usuario@<IP_DO_SERVER>
# Agora, MAC do ARP do host bate com MAC real da VM.
# SSH funciona normalmente.
```
6. (Opcional) Fixar o IP do Host-Only pós-reboot
Netplan (Ubuntu 24.04)

```bash
sudo nano /etc/netplan/99-vboxnet0.yaml
```
Conteúdo:

```yaml
network:
  version: 2
  ethernets:
    vboxnet0:
      addresses: [<IP_HOST_ONLY>/24]
      dhcp4: false
```
Ativar:

```bash
sudo netplan apply
ip addr show vboxnet0
```

## 7. Evidências Finais

SSH OK:

```bash
ssh usuario@192.168.56.11
# Sucesso na conexão
```

ARP/Neighbor Table OK:

```bash
ip neigh show dev vboxnet0
# <IP_VM_SERVER> at <MAC da VM>
```
Ping OK:

```bash
ping -c 3 192.168.56.11
```

## 8. Lições e Valor para Infra/DevOps

- Identificar diferenciais entre timeout, refused e “IP errado” via ARP.
- Diagnóstico avançado com ip neigh, tcpdump, netstat, nmap.
- Uso prático de reset/flush de interface e ARP.
- Documentação de incidentes: passo‐a‐passo e evidências.
