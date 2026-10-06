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
pfSense → 172.16.99.1
Aruba   → 172.16.99.2
```

O Core ArubaOS-CX transporta as VLANs através de enlaces trunk 802.1Q.

[Topologia da rede](docs/topology/network-topology.md)

---

# Filial

A Filial possui um segundo pfSense e redes internas segmentadas.

| VLAN | Nome    | Rede             |
| ---: | ------- | ---------------- |
|   21 | IT      | `172.16.21.0/29` |
|   31 | SERVERS | `172.16.31.0/30` |

A comunicação entre Matriz e Filial é realizada através de uma VPN IPsec Site-to-Site.

---

# Switching

O Core da Matriz utiliza:

**ArubaOS-CX Virtual 10.14.1000**

Foram implementados:

* VLANs;
* SVI;
* Access Ports;
* Trunk 802.1Q;
* VLAN nativa;
* VLANs permitidas;
* segmentação lógica;
* rede de gerenciamento.

As VLANs utilizadas no Core são:

```text
VLAN 10 → USUARIOS
VLAN 20 → IT
VLAN 30 → SERVERS
VLAN 99 → MGMT
```

O enlace principal com o firewall utiliza trunk com:

```text
Native VLAN: 1
Allowed VLANs: 10,20,30,99
```

[Documentação de VLANs](docs/switching/vlans.md)

[Documentação de Trunking](docs/switching/trunking.md)

---

# Firewall

O pfSense é utilizado como camada de segurança e conectividade dos sites.

Foram implementados:

* regras de firewall;
* segmentação entre redes;
* controle de tráfego;
* NAT;
* Port Forward;
* VPN;
* failover WAN;
* acesso remoto;
* gerenciamento de interfaces.

A arquitetura foi estruturada de forma que a conectividade entre redes dependa de políticas explícitas de acesso.

[Documentação do pfSense](docs/firewall/pfsense.md)

[Regras de Firewall](docs/firewall/firewall-rules.md)

---

# NAT e publicação de serviços

Foi implementado um cenário de **Port Forward** para representar a publicação de um serviço interno através de uma interface WAN.

Fluxo:

```text
Cliente externo
      ↓
     WAN
      ↓
   pfSense
      ↓
Port Forward
      ↓
Servidor interno
```

A validação considerou:

* disponibilidade do serviço internamente;
* regra de NAT;
* regra de firewall;
* acesso através da WAN;
* teste final do serviço.

[Documentação de NAT](docs/firewall/nat.md)

---

# VPN Site-to-Site

Foi implementado um túnel **IPsec Site-to-Site** entre os pfSense da Matriz e da Filial.

Redes participantes:

| Matriz           | Filial           |
| ---------------- | ---------------- |
| `172.16.20.0/28` | `172.16.21.0/29` |
| `172.16.20.0/28` | `172.16.31.0/30` |
| `10.10.30.0/30`  | `172.16.21.0/29` |
| `10.10.30.0/30`  | `172.16.31.0/30` |

A validação considerou:

* Phase 1;
* Phase 2;
* seletores;
* regras de firewall;
* conectividade entre redes;
* análise dos logs.

[Documentação da VPN Site-to-Site](docs/vpn/site-to-site.md)

---

# VPN de Acesso Remoto

Também foi implementada uma VPN para acesso remoto à infraestrutura da Matriz.

Pool utilizado:

```text
10.0.8.0/28
```

A implementação foi validada considerando:

* autenticação;
* estabelecimento da conexão;
* endereçamento do cliente;
* políticas de acesso;
* conectividade com redes autorizadas.

[Documentação da VPN de acesso remoto](docs/vpn/remote-access.md)

---

# Redundância e Failover

A Matriz possui três caminhos WAN e utiliza um grupo de gateways denominado:

```text
GW_JANUS
```

O objetivo é simular a indisponibilidade de um caminho e validar a utilização de uma rota alternativa.

Foram realizados testes envolvendo:

* disponibilidade dos gateways;
* perda de conectividade;
* seleção de caminho alternativo;
* continuidade do tráfego;
* recuperação do link;
* retorno à condição normal.

[Documentação de Failover](docs/redundancy/failover.md)

---

# Troubleshooting e RCA

O laboratório também foi utilizado para reproduzir falhas e aplicar uma metodologia de análise baseada em evidências.

A abordagem utilizada é:

```text
Problema
   ↓
5 Porquês
   ↓
Hipóteses
   ↓
Coleta de evidências
   ↓
Testes
   ↓
Causa raiz
   ↓
Correção
   ↓
Validação final
```

Os casos documentados incluem:

### VLAN 99 sem conectividade

Análise de conectividade entre ArubaOS-CX e pfSense, envolvendo VLAN, trunk, tagging 802.1Q e SVI.

[Ver RCA](docs/troubleshooting/vlan-99-sem-conectividade.md)

### BGP Neighbor Down

Análise de uma sessão eBGP que não estabelecia corretamente, incluindo validação de conectividade e configuração dos neighbors.

[Ver RCA](docs/troubleshooting/bgp-neighbor-down.md)

### Falha no Failover WAN

Análise do comportamento do grupo `GW_JANUS` durante uma indisponibilidade de WAN.

[Ver RCA](docs/troubleshooting/failover-wan.md)

### VPN UP sem tráfego

Análise de um cenário em que o túnel IPsec estava estabelecido, mas o tráfego entre redes permanecia bloqueado.

[Ver RCA](docs/troubleshooting/vpn-tunel-up-sem-trafego.md)

### NAT Port Forward sem acesso externo

Análise de uma falha na publicação de um serviço interno através da WAN.

[Ver RCA](docs/troubleshooting/nat-port-forward.md)

---

# Tecnologias

## Networking

* IPv4
* Subnetting
* Routing
* eBGP
* Autonomous Systems
* VLAN
* 802.1Q
* Access Port
* Trunk
* WAN
* NAT
* Failover

## Security

* pfSense
* Firewall
* Network Segmentation
* IPsec
* Site-to-Site VPN
* Remote Access VPN
* Port Forward

## Switching

* ArubaOS-CX
* VLAN
* SVI
* Trunking
* Access Ports
* 802.1Q

## Virtualização

* EVE-NG
* Cisco IOS
* ArubaOS-CX Virtual
* pfSense

---

# Estrutura do projeto

```text
Enterprise-Network-Lab/
│
├── README.md
│
├── docs/
│   │
│   ├── topology/
│   │   ├── network-topology.md
│   │   └── ip-addressing.md
│   │
│   ├── isp/
│   │   ├── isp-topology.md
│   │   ├── bgp.md
│   │   └── routing.md
│   │
│   ├── switching/
│   │   ├── vlans.md
│   │   └── trunking.md
│   │
│   ├── firewall/
│   │   ├── pfsense.md
│   │   ├── nat.md
│   │   └── firewall-rules.md
│   │
│   ├── vpn/
│   │   ├── site-to-site.md
│   │   └── remote-access.md
│   │
│   ├── redundancy/
│   │   └── failover.md
│   │
│   └── troubleshooting/
│       ├── vlan-99-sem-conectividade.md
│       ├── bgp-neighbor-down.md
│       ├── failover-wan.md
│       ├── vpn-tunel-up-sem-trafego.md
│       └── nat-port-forward.md
│
└── diagrams/
    └── topology.png
```

---

# Validação

O projeto foi validado através de testes práticos em diferentes componentes da infraestrutura.

### Routing / BGP

* conectividade entre ISPs;
* estabelecimento de sessões eBGP;
* recebimento de prefixos;
* anúncio de prefixos;
* instalação de rotas;
* seleção de caminhos.

### Switching

* criação de VLANs;
* comunicação através das VLANs;
* SVI;
* Access Ports;
* Trunks;
* transporte de VLANs;
* conectividade da VLAN de gerenciamento.

### Firewall / NAT

* regras de acesso;
* segmentação;
* NAT;
* Port Forward;
* publicação de serviços;
* bloqueio de tráfego não autorizado.

### VPN

* estabelecimento do IPsec;
* Phase 1;
* Phase 2;
* comunicação entre Matriz e Filial;
* acesso remoto;
* validação das políticas de firewall.

### Redundância

* indisponibilidade de links;
* alteração do caminho;
* continuidade da conectividade;
* recuperação após retorno do link.

---

# Simulação dos ISPs

Os três ISPs são representados por roteadores Cisco independentes dentro do EVE-NG.

A conectividade externa utilizada pelo ambiente de laboratório é compartilhada pela infraestrutura física onde o EVE-NG está hospedado.

Portanto, o projeto **não representa três links físicos de operadoras diferentes**.

A camada ISP foi criada para estudar e demonstrar:

* Autonomous Systems;
* eBGP;
* anúncio de prefixos;
* seleção de caminhos;
* roteamento entre diferentes AS;
* redundância lógica;
* troubleshooting de conectividade.

---

# Documentação

A documentação foi organizada por domínio técnico para facilitar a consulta:

* [Topologia](docs/topology/network-topology.md)
* [Endereçamento IP](docs/topology/ip-addressing.md)
* [Topologia dos ISPs](docs/isp/isp-topology.md)
* [BGP](docs/isp/bgp.md)
* [Routing](docs/isp/routing.md)
* [VLANs](docs/switching/vlans.md)
* [Trunking](docs/switching/trunking.md)
* [pfSense](docs/firewall/pfsense.md)
* [Firewall](docs/firewall/firewall-rules.md)
* [NAT](docs/firewall/nat.md)
* [VPN Site-to-Site](docs/vpn/site-to-site.md)
* [VPN Remote Access](docs/vpn/remote-access.md)
* [Failover](docs/redundancy/failover.md)
* [Troubleshooting](docs/troubleshooting/)

---

# Status

| Componente            | Status      |
| --------------------- | ----------- |
| Topologia             | ✅ Concluído |
| Endereçamento         | ✅ Concluído |
| ISP-01/02/03          | ✅ Concluído |
| eBGP                  | ✅ Concluído |
| Routing               | ✅ Concluído |
| VLANs                 | ✅ Concluído |
| Trunking              | ✅ Concluído |
| Switching             | ✅ Concluído |
| pfSense               | ✅ Concluído |
| Firewall              | ✅ Concluído |
| NAT                   | ✅ Concluído |
| VPN Site-to-Site      | ✅ Concluído |
| VPN Remote Access     | ✅ Concluído |
| Failover              | ✅ Concluído |
| Troubleshooting / RCA | ✅ Concluído |
| Testes                | ✅ Concluído |
| Documentação          | ✅ Concluído |

---

# Considerações

Este laboratório foi desenvolvido como um ambiente de estudo e portfólio para consolidar conhecimentos de **Infraestrutura e Redes**.

O foco do projeto não está apenas na configuração dos equipamentos, mas na capacidade de compreender uma arquitetura, implementar seus componentes, validar o funcionamento, investigar falhas e documentar a causa raiz.

A complexidade apresentada deve ser interpretada dentro do contexto de um **laboratório técnico**, não como substituição de experiência profissional em ambientes de produção.
