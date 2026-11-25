# Команди для лабораторної №2

## Перевірка мережі

```bash
ip -br addr
ip -4 route
ip neigh show
ip neigh show dev eth0
```

## Захоплення ARP та ICMP

У першому терміналі:

```bash
mkdir -p ~/lab2_arp
cd ~/lab2_arp
GW=$(ip -4 route show default | awk 'NR==1 {print $3}')
echo "$GW"
sudo tcpdump -i eth0 -nn -e -w arp_capture.pcap 'arp or icmp'
```

У другому терміналі:

```bash
GW=$(ip -4 route show default | awk 'NR==1 {print $3}')
ping -c 4 "$GW"
ip neigh show "$GW" dev eth0
```

Після завершення захоплення (Ctrl+C) перевірив файл:

```bash
ls -lh ~/lab2_arp/arp_capture.pcap
sudo tcpdump -nn -e -r ~/lab2_arp/arp_capture.pcap
```

## Фільтри у Wireshark

```text
arp
arp.opcode == 1
arp.opcode == 2
icmp
arp || icmp
```

У файлі були ARP Request/Reply та чотири ICMP Echo Request без Echo Reply.

На завершення повторно переглянув кеш:

```bash
ip neigh show dev eth0
```
