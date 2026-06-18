### Lab 01 - Troubleshooting
## Problema de conexão do PC para a VM

## (1) Sintoma inicial: SSH recusado

```bash
# O -vvv mostra o motivo da falha. Apareceu Connection refused, indicando que o host respondeu, mas a porta/serviço não aceitou (ou não estava chegando na máquina esperada).
ssh -vvv usuario@192.168.56.2
```
## (2) Teste de porta (para separar rede vs porta/serviço)
No PC:

```bash
# Se fosse problema de rota/caminho, o mais comum seria timeout. Connection refused sugere “algo respondeu naquele IP/porta recusando”
nc -vz 192.168.56.2 22
```

## (3) Verificação de interface host-only no notebook
No PC:

```bash
# Confirma que o vboxnet0 existe e está na mesma sub-rede (192.168.56.1/24)
ip a | grep -E '192\.168\.56\.|vboxnet|VirtualBox' -n
```

## (4) Hipótese-chave do caso: MAC não coincidente (ARP/IP conflitante)

```bash
# Não estava falando com a VM correta (conflito de IP/ARP), ou a VM não estava realmente conectada à host-only esperada.
# Ferramentas usadas para confirmar:
# Ver vizinhança/ARP no host:
ip neigh show dev vboxnet0
arp -an | grep 192.168.56
# MAC real na VM (server), interface host-only:
cat /sys/class/net/enp0s8/address
# ou
ip -br link show enp0s8
```

## (5) Flush da tabela ARP (limpar cache)
No PC:

```bash
#Remover entradas ARP/neighbor para forçar uma nova descoberta do MAC correto.
ip neigh flush all
# ou específico:
ip neigh flush dev vboxnet0
```

## (6) Ação corretiva final que resolveu
# Reendereçamento VMs para .11 (server) e .12 (cliente)
# Resetar vboxnet0 no PC:

```bash
#Por que funcionou:
#Evitou o IP problemático (possível conflito/estado anterior).
#Forçou a reconstrução de vizinhança (ARP) e do estado do vboxnet0.
sudo ip addr flush dev vboxnet0
sudo ip addr add 192.168.56.1/24 dev vboxnet0
```

## Evidências
# Evidência do erro (SSH)
Trecho relevante:

```bash
# Conexão recusada na tentaiva de conexão ssh entre PC e VM
connect to address 192.168.56.2 port 22: Connection refused
# MAC address na arp do PC, divergente do host (server) configurado com o IP 192.168.56.2
? (192.168.56.2) em 08:00:27:31:9a:93 [ether] em vboxnet0 # saída arp -n comandado no PC
# MAC address do host (server) configurado com o IP 192.168.56.2
link/ether 08:00:27:49:d5:c6 brd ff:ff:ff:ff:ff:ff
inet 192.168.56.2/24 brd 192.168.56.255 scope global enp0s8 # ip a comandado na VM server
```
> MAC do ARP no PC atrelado a um IP deveria ser igual ao MAC real da interface da VM nesse IP.
> Quando não coincide → forte indício de IP em conflito ou mapeamento/host-only errado.
> Evidência final
> Após a reconfiguração funciona a conexão via ssh

