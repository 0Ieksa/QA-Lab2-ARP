# Команди для дослідження ARP

Середовище: Ubuntu WSL2. Нижче наведено порядок відтворення експерименту; результати залежать від актуальних адрес і конфігурації мережі.

## 1. Перевірка інтерфейсу, маршруту і таблиці сусідів

```bash
ip -br addr
ip -4 route
ip neigh show
ip neigh show dev eth0
```

## 2. Запис ARP/ICMP-трафіку

Термінал 1:

```bash
mkdir -p ~/lab2_arp
cd ~/lab2_arp
GW=$(ip -4 route show default | awk 'NR==1 {print $3}')
echo "$GW"
sudo tcpdump -i eth0 -nn -e -w arp_capture.pcap 'arp or icmp'
```

Поки працює захоплення, в іншому терміналі:

```bash
GW=$(ip -4 route show default | awk 'NR==1 {print $3}')
ping -c 4 "$GW"
ip neigh show "$GW" dev eth0
```

За потреби викликати нову ARP-резолюцію можна видаленням лише динамічного запису поточного шлюзу: `sudo ip neigh del "$GW" dev eth0`. Робити це лише у своєму тестовому середовищі: операція може короткочасно перервати зв'язок.

Завершити tcpdump через Ctrl+C у першому терміналі, далі:

```bash
ls -lh ~/lab2_arp/arp_capture.pcap
sudo tcpdump -nn -e -r ~/lab2_arp/arp_capture.pcap
```

## 3. Аналіз у Wireshark

Відкрити справжній `arp_capture.pcap` і застосувати фільтри:

- `arp` — увесь ARP-обмін;
- `arp.opcode == 1` — ARP Request;
- `arp.opcode == 2` — ARP Reply;
- `icmp` — ICMP-пакети;
- `arp || icmp` — обидва протоколи.

У захопленому експерименті виявлено ARP Request/Reply та чотири ICMP Echo Request. Echo Reply у відповідному дампі не спостерігались.

## 4. Перевірка кешу після експерименту

```bash
ip neigh show dev eth0
```

Вихідний та повторний знімки показали запис шлюзу у стані STALE, тому ці знімки окремо не доводять оновлення кешу.

> Команди створення статичного ARP-запису описані як тест-кейс [TC-ARP-03](TC-ARP-03.md), але фактичне виконання цього тесту тут не заявляється.
