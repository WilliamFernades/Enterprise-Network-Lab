# Topologia da Rede

## Visão geral

Este laboratório representa uma arquitetura de rede corporativa com redundância de conectividade, roteamento dinâmico, segmentação de redes, firewall, VPN e infraestrutura distribuída entre Matriz e Filial.

O ambiente foi construído no EVE-NG utilizando equipamentos virtuais para reproduzir cenários encontrados em ambientes corporativos.

> **Nota:** este projeto representa um ambiente de laboratório. A complexidade técnica apresentada não representa, por si só, experiência profissional em produção.

---

## Arquitetura

![Topologia da rede](../../diagrams/topology.png)

A arquitetura está organizada em diferentes camadas:

```text
                         ISP-01
                       /        \
                      /          \
                 ISP-02 -------- ISP-03
                    |              |
                    |              |
              ┌─────┴─────┐  ┌────┴─────┐
              │  pfSense  │  │  pfSense │
              │  MATRIZ   │  │  FILIAL  │
              └─────┬─────┘  └────┬─────┘
                    │              │
                 Aruba          Switching
                  Core
                    │
              ┌─────┼─────┐
              │     │     │
             VLAN   VLAN  VLAN
              10     20    30
           Usuários   IT  Servers
```

---

## Camada de conectividade

O ambiente possui três roteadores simulando provedores independentes:

| Equipamento |   ASN | Prefixo       |
| ----------- | ----: | ------------- |
| ISP-01      | 65001 | 10.100.1.0/24 |
| ISP-02      | 65002 | 10.100.2.0/24 |
| ISP-03      | 65003 | 10.100.3.0/24 |

Os três equipamentos formam uma malha eBGP, permitindo a troca de rotas entre os sistemas autônomos.

O objetivo é reproduzir um cenário onde a organização possui múltiplos caminhos de conectividade.

---

## Matriz

A Matriz utiliza um pfSense com três conexões WAN:

* ISP-01
* ISP-02
* ISP-03

O firewall é responsável por:

* conectividade WAN;
* NAT;
* regras de firewall;
* failover;
* VPN;
* segmentação;
* publicação de serviços.

O gerenciamento e a distribuição das redes internas são realizados através do ambiente de switching.

---

## Segmentação da Matriz

O Core ArubaOS-CX utiliza VLANs para separar os diferentes segmentos:

| VLAN | Nome     | Função               |
| ---: | -------- | -------------------- |
|   10 | USUARIOS | Estações de trabalho |
|   20 | IT       | Administração/TI     |
|   30 | SERVERS  | Servidores           |
|   99 | MGMT     | Gerenciamento        |

A separação reduz o domínio de broadcast e permite aplicar políticas de segurança diferentes entre os segmentos.

---

## Filial

A Filial possui um segundo pfSense conectado aos três ISPs simulados.

As redes internas da Filial são segmentadas em:

| VLAN | Nome    | Rede           |
| ---: | ------- | -------------- |
|   21 | IT      | 172.16.21.0/29 |
|   31 | SERVERS | 172.16.31.0/30 |

A comunicação entre Matriz e Filial é realizada através de um túnel IPsec site-to-site.

---

## VPN Site-to-Site

O túnel IPsec conecta os ambientes internos da Matriz e da Filial.

São utilizados diferentes seletores de Phase 2 para permitir comunicação entre as redes necessárias.

```text
MATRIZ                         FILIAL

172.16.20.0/28  ────────────  172.16.21.0/29
10.10.30.0/30   ────────────  172.16.31.0/30
```

Além da VPN site-to-site, o pfSense da Matriz possui uma VPN de acesso remoto para clientes externos.

Pool utilizado:

```text
10.0.8.0/28
```

---

## Redundância

A Matriz utiliza três conexões WAN e um grupo de gateways denominado:

```text
GW_JANUS
```

O objetivo é manter a conectividade mesmo diante da indisponibilidade de um dos caminhos WAN.

O failover é baseado no monitoramento dos gateways e na alteração do caminho utilizado pelo firewall.

---

## Switching e Trunking

O Core utiliza ArubaOS-CX Virtual.

O enlace entre o Core e o pfSense utiliza trunk 802.1Q.

VLANs transportadas:

```text
10
20
30
99
```

A VLAN 1 é utilizada como VLAN nativa.

A VLAN 99 é utilizada para gerenciamento.

---

## Serviços de segurança e publicação

O pfSense também representa funções de segurança de borda, incluindo:

* filtragem de tráfego;
* NAT;
* Port Forward;
* publicação de serviços;
* controle entre segmentos;
* VPN;
* redundância WAN.

Um cenário de Port Forward foi implementado para representar a publicação de um serviço interno para acesso externo.

---

## Fluxo lógico

O fluxo geral da arquitetura pode ser representado como:

```text
                  INTERNET SIMULADA
                         │
              ┌──────────┼──────────┐
              │          │          │
           ISP-01     ISP-02     ISP-03
              │          │          │
              └──────────┼──────────┘
                         │
                    pfSense WAN
                         │
                  Firewall / NAT
                         │
                    Core Aruba
                         │
        ┌────────────────┼────────────────┐
        │                │                │
     VLAN 10          VLAN 20          VLAN 30
    Usuários             IT             Servers
                         │
                      VLAN 99
                        MGMT
                         
                         │
                      IPsec
                         │
                     Filial
```

---

## Objetivos técnicos demonstrados

A topologia foi construída para demonstrar conhecimentos em:

* IPv4;
* subnetting;
* VLANs;
* trunking 802.1Q;
* switching;
* roteamento;
* eBGP;
* múltiplos ISPs;
* NAT;
* firewall;
* failover;
* IPsec;
* VPN de acesso remoto;
* troubleshooting;
* análise de conectividade.

---

## Resultado

A arquitetura implementada permite reproduzir diferentes cenários de infraestrutura de rede corporativa dentro de um único laboratório.

A combinação de múltiplos ISPs, eBGP, firewall, segmentação, redundância e VPN permite testar tanto a operação normal quanto cenários de falha e recuperação.

O projeto foi estruturado de forma modular para facilitar a análise individual de cada componente e também a avaliação do funcionamento da infraestrutura como um todo.
