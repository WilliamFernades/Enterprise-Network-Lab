# Plano de Endereçamento IP

## Objetivo

Este documento consolida o plano de endereçamento utilizado no laboratório, permitindo identificar os segmentos de rede, interfaces WAN, enlaces entre roteadores e redes internas.

---

## ISPs

| Equipamento |   ASN | Prefixo       |
| ----------- | ----: | ------------- |
| ISP-01      | 65001 | 10.100.1.0/24 |
| ISP-02      | 65002 | 10.100.2.0/24 |
| ISP-03      | 65003 | 10.100.3.0/24 |

---

## Enlaces eBGP

| Enlace          | Rede          |
| --------------- | ------------- |
| ISP-01 ↔ ISP-02 | 10.255.1.0/30 |
| ISP-01 ↔ ISP-03 | 10.255.2.0/30 |
| ISP-02 ↔ ISP-03 | 10.255.3.0/30 |

---

## WAN — Matriz

| ISP    | Rede           | pfSense     |
| ------ | -------------- | ----------- |
| ISP-01 | 100.64.10.0/30 | 100.64.10.2 |
| ISP-02 | 100.64.20.0/30 | 100.64.20.2 |
| ISP-03 | 100.64.30.0/30 | 100.64.30.2 |

Gateways:

```text
ISP-01 → 100.64.10.1
ISP-02 → 100.64.20.1
ISP-03 → 100.64.30.1
```

---

## WAN — Filial

| ISP    | Rede           | pfSense     |
| ------ | -------------- | ----------- |
| ISP-01 | 100.64.11.0/30 | 100.64.11.2 |
| ISP-02 | 100.64.21.0/30 | 100.64.21.2 |
| ISP-03 | 100.64.31.0/30 | 100.64.31.2 |

Gateways:

```text
ISP-01 → 100.64.11.1
ISP-02 → 100.64.21.1
ISP-03 → 100.64.31.1
```

---

## Matriz — Redes internas

| VLAN | Nome     | Rede                   |
| ---: | -------- | ---------------------- |
|   10 | USUARIOS | conforme implementação |
|   20 | IT       | 172.16.20.0/28         |
|   30 | SERVERS  | 10.10.30.0/30          |
|   99 | MGMT     | 172.16.99.0/30         |

VLAN 99:

```text
pfSense → 172.16.99.1
Aruba   → 172.16.99.2
```

---

## Filial — Redes internas

| VLAN | Nome    | Rede           |
| ---: | ------- | -------------- |
|   21 | IT      | 172.16.21.0/29 |
|   31 | SERVERS | 172.16.31.0/30 |

---

## VPN de acesso remoto

Pool destinado aos clientes da VPN:

```text
10.0.8.0/28
```

---

## Observação sobre os endereços WAN

As redes `100.64.0.0/10` utilizadas neste projeto representam endereçamento de laboratório para simular os enlaces WAN.

O ambiente não representa três conexões físicas independentes de Internet.

Os três ISPs são roteadores virtuais executados dentro do EVE-NG e a conectividade externa utilizada pelo laboratório é compartilhada pelo ambiente físico onde o EVE-NG está hospedado.

---

## Resumo

O plano de endereçamento foi estruturado para separar:

* enlaces entre ISPs;
* redes WAN;
* redes de usuários;
* redes de TI;
* servidores;
* gerenciamento;
* VPN;
* comunicação entre Matriz e Filial.

A segmentação facilita a aplicação de políticas de roteamento, firewall, NAT, VPN e troubleshooting.
