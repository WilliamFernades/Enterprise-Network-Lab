# Enterprise Network Lab

Laboratório de infraestrutura e redes corporativas desenvolvido em **EVE-NG**, simulando uma arquitetura empresarial com múltiplos ISPs, eBGP, switching, VLANs, firewalls, NAT, VPN, redundância e troubleshooting.

O projeto foi construído com foco em **implementação prática, validação de conectividade, análise de falhas e documentação técnica**, buscando reproduzir cenários encontrados em ambientes corporativos.

> **Nota:** este projeto representa um ambiente de laboratório. A complexidade técnica apresentada não representa, por si só, experiência profissional em ambiente de produção.

---

## Objetivo

O objetivo deste laboratório é consolidar e demonstrar conhecimentos práticos em **Infraestrutura e Redes**, trabalhando desde a camada de switching e segmentação até roteamento, segurança, redundância e conectividade entre sites.

O projeto também foi estruturado para documentar não apenas configurações, mas o processo de:

```text
Arquitetura
    ↓
Implementação
    ↓
Validação
    ↓
Troubleshooting
    ↓
RCA
    ↓
Correção
    ↓
Validação final
```

---

# Arquitetura

![Topologia da rede](diagrams/topology.png)

A arquitetura é dividida em duas áreas principais:

### Camada ISP

Três roteadores Cisco são utilizados para representar Autonomous Systems independentes:

| ISP    |   ASN | Prefixo         |
| ------ | ----: | --------------- |
| ISP-01 | 65001 | `10.100.1.0/24` |
| ISP-02 | 65002 | `10.100.2.0/24` |
| ISP-03 | 65003 | `10.100.3.0/24` |

Os ISPs possuem conectividade entre si através de enlaces `/30` e utilizam **eBGP** para troca de rotas.

### Camada Corporativa

A infraestrutura corporativa é composta por:

* Matriz;
* Filial;
* Firewalls pfSense;
* Core ArubaOS-CX;
* VLANs;
* múltiplos links WAN;
* NAT;
* políticas de firewall;
* VPN Site-to-Site;
* VPN de acesso remoto;
* mecanismos de redundância.

---

# Camada ISP

## eBGP

Os três Autonomous Systems formam uma malha de conectividade:

```text
              ISP-01
             AS 65001
             /      \
          eBGP      eBGP
           /          \
      ISP-02 -------- ISP-03
      AS 65002       AS 65003
             eBGP
```

### Enlaces de trânsito

| Enlace          | Rede            |
| --------------- | --------------- |
| ISP-01 ↔ ISP-02 | `10.255.1.0/30` |
| ISP-01 ↔ ISP-03 | `10.255.2.0/30` |
| ISP-02 ↔ ISP-03 | `10.255.3.0/30` |

Cada ISP anuncia um prefixo próprio:

```text
ISP-01 → 10.100.1.0/24
ISP-02 → 10.100.2.0/24
ISP-03 → 10.100.3.0/24
```

Os prefixos são mantidos na tabela de roteamento através de interfaces `Null0`, permitindo sua utilização nas declarações `network` do BGP.

### Validação

Entre os comandos utilizados para análise:

```bash
show ip bgp summary
show ip bgp
show ip route bgp
```

Foram avaliados:

* estado das sessões;
* vizinhos BGP;
* prefixos recebidos;
* prefixos anunciados;
* tabela BGP;
* instalação das rotas;
* seleção de caminhos.

[Documentação de BGP](docs/isp/bgp.md)

---

# Conectividade WAN

Os enlaces WAN dos sites são representados por redes `/30` dentro do ambiente de laboratório.

## Matriz

| ISP    | Rede             | Gateway       | pfSense       |
| ------ | ---------------- | ------------- | ------------- |
| ISP-01 | `100.64.10.0/30` | `100.64.10.1` | `100.64.10.2` |
| ISP-02 | `100.64.20.0/30` | `100.64.20.1` | `100.64.20.2` |
| ISP-03 | `100.64.30.0/30` | `100.64.30.1` | `100.64.30.2` |

## Filial

| ISP    | Rede             | Gateway       | pfSense       |
| ------ | ---------------- | ------------- | ------------- |
| ISP-01 | `100.64.11.0/30` | `100.64.11.1` | `100.64.11.2` |
| ISP-02 | `100.64.21.0/30` | `100.64.21.1` | `100.64.21.2` |
| ISP-03 | `100.64.31.0/30` | `100.64.31.1` | `100.64.31.2` |

> As redes `100.64.0.0/10` são utilizadas exclusivamente para representar conectividade WAN dentro do laboratório.

[Plano de endereçamento](docs/topology/ip-addressing.md)

---

# Matriz

A Matriz utiliza pfSense como firewall de borda e possui três conexões WAN.

O ambiente interno é segmentado através de VLANs.

| VLAN | Nome     | Rede             | Finalidade           |
| ---: | -------- | ---------------- | -------------------- |
|   10 | USUARIOS | —                | Estações de trabalho |
|   20 | IT       | `172.16.20.0/28` | TI                   |
|   30 | SERVERS  | `10.10.30.0/30`  | Servidores           |
|   99 | MGMT     | `172.16.99.0/30` | Gerenciamento        |

A VLAN 99 possui:

```text
pf
```
