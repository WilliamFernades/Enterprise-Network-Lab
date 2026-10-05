# Enterprise Network Lab

Laboratório de redes corporativas desenvolvido em **EVE-NG**, com o objetivo de simular uma infraestrutura empresarial composta por múltiplos provedores de Internet, roteamento BGP, firewalls, VLANs, switching, redundância, NAT e VPNs.

O projeto foi desenvolvido como um laboratório prático para demonstrar conhecimentos aplicados de **redes e infraestrutura**, abrangendo desde conectividade e roteamento até segmentação, segurança e comunicação entre sites.

> **Nota:** este projeto representa um ambiente de laboratório. A complexidade técnica apresentada não representa, por si só, experiência profissional em ambiente de produção.

---

# Objetivos

O laboratório tem como objetivos:

* Simular múltiplos provedores de Internet;
* Implementar comunicação entre diferentes Autonomous Systems;
* Utilizar **eBGP** entre provedores simulados;
* Implementar roteamento IP;
* Estruturar redes corporativas utilizando VLANs;
* Implementar switching com portas Access e Trunk;
* Implementar firewalls e políticas de controle de tráfego;
* Implementar NAT;
* Simular redundância e failover;
* Implementar VPN Site-to-Site;
* Implementar VPN de acesso remoto;
* Validar conectividade através de testes práticos;
* Documentar configurações, decisões técnicas, testes e troubleshooting.

---

# Arquitetura

O laboratório é dividido em duas camadas principais:

1. **Camada ISP**, responsável pela simulação da Internet e do roteamento entre Autonomous Systems;
2. **Camada Corporativa**, composta por Matriz e Filial, com firewalls, switching, VLANs e comunicação através de VPN.

```text
                         SIMULATED INTERNET
                                  |
              +-------------------+-------------------+
              |                   |                   |
           AS 65001            AS 65002            AS 65003
            CLARO               VIVO                  OI
           ISP-01              ISP-02               ISP-03
              \                   |                   /
               \                  |                  /
                +-----------------+-----------------+
                                  |
                         CORPORATE NETWORK
                                  |
                       +----------+----------+
                       |                     |
                    MATRIZ                  FILIAL
                   pfSense                 pfSense
                       |                     |
                   Core SW                Core SW
                       |                     |
                  Corporate             Corporate
                     VLANs                 VLANs
                       \                     /
                        \                   /
                         +---- IPsec VPN ---+
```

---

# Camada ISP

Três roteadores Cisco representam provedores de Internet independentes.

| Provedor | Equipamento |   ASN |
| -------- | ----------- | ----: |
| CLARO    | ISP-01      | 65001 |
| VIVO     | ISP-02      | 65002 |
| OI       | ISP-03      | 65003 |

Os provedores possuem conectividade entre si através de links de trânsito `/30` e estabelecem sessões **eBGP**.

## Links de trânsito

| Enlace       | Rede            | Lado A       | Lado B       |
| ------------ | --------------- | ------------ | ------------ |
| CLARO ↔ VIVO | `10.255.1.0/30` | `10.255.1.1` | `10.255.1.2` |
| CLARO ↔ OI   | `10.255.2.0/30` | `10.255.2.1` | `10.255.2.2` |
| VIVO ↔ OI    | `10.255.3.0/30` | `10.255.3.1` | `10.255.3.2` |

Cada ISP também anuncia um prefixo próprio através do BGP:

| ISP   |   ASN | Prefixo         |
| ----- | ----: | --------------- |
| CLARO | 65001 | `10.100.1.0/24` |
| VIVO  | 65002 | `10.100.2.0/24` |
| OI    | 65003 | `10.100.3.0/24` |

Os prefixos utilizados para representar as redes dos provedores são mantidos na tabela de roteamento através de interfaces `Null0`.

---

# WAN dos Sites

Os links WAN utilizados pelos firewalls são representados por prefixos `/30` anunciados pelos ISPs através do BGP.

### CLARO — AS 65001

```text
100.64.10.0/30
100.64.11.0/30
100.64.12.0/30
```

### VIVO — AS 65002

```text
100.64.20.0/30
100.64.21.0/30
100.64.22.0/30
```

### OI — AS 65003

```text
100.64.30.0/30
100.64.31.0/30
100.64.32.0/30
```

Esses prefixos representam, dentro do laboratório, as redes WAN utilizadas pelos equipamentos conectados aos provedores.

> Os endereços `100.64.0.0/10` são utilizados exclusivamente para representar a conectividade WAN dentro do ambiente de laboratório.

---

# Ambiente Corporativo

A infraestrutura corporativa é composta por dois sites:

* **Matriz**
* **Filial**

Cada site possui firewall, switching e redes segmentadas através de VLANs.

```text
                         SIMULATED ISP
                              |
                     +--------+--------+
                     |                 |
                  MATRIZ             FILIAL
                 pfSense             pfSense
                     |                 |
                 Core SW            Core SW
                     |                 |
          +----------+----------+     +----------+----------+
          |          |          |     |          |          |
         IT         RH       SERVERS IT     COMERCIAL   MARKETING
                               
                                      |
                                   SERVERS
```

---

# Matriz

A Matriz possui:

* Firewall pfSense;
* Switch Core;
* VLAN de TI;
* VLAN de RH;
* VLAN de servidores;
* Múltiplos links WAN;
* NAT;
* Políticas de firewall;
* VPN Site-to-Site;
* VPN de acesso remoto.

### Redes principais

| VLAN    | Rede             | Função     |
| ------- | ---------------- | ---------- |
| VLAN 20 | `172.16.20.0/28` | TI         |
| VLAN 30 | `10.10.30.0/30`  | Servidores |

---

# Filial

A Filial possui:

* Firewall pfSense;
* Switch Core;
* VLAN de TI;
* VLAN Comercial;
* VLAN Marketing;
* VLAN de servidores;
* Múltiplos links WAN;
* NAT;
* Políticas de firewall;
* VPN Site-to-Site;
* VPN de acesso remoto.

### Redes principais

| VLAN    | Rede             | Função     |
| ------- | ---------------- | ---------- |
| VLAN 21 | `172.16.21.0/29` | TI         |
| VLAN 31 | `172.16.31.0/30` | Servidores |

---

# Switching

Os switches Core são responsáveis pela conectividade da rede interna e pela segmentação lógica através de VLANs.

Foram implementados:

* VLANs;
* Portas Access;
* Portas Trunk;
* 802.1Q;
* Segmentação de redes;
* Gerenciamento através de VLAN dedicada;
* Transporte de múltiplas VLANs entre firewall e switch.

---

# Roteamento e BGP

A camada ISP utiliza três Autonomous Systems:

```text
             AS 65001
              CLARO
             /      \
          eBGP      eBGP
           /          \
     AS 65002 ------ AS 65003
       VIVO             OI
             eBGP
```

As sessões entre os provedores permitem o intercâmbio de prefixos e a existência de múltiplos caminhos para determinados destinos.

A validação do BGP foi realizada através de comandos como:

```text
show ip bgp summary
show ip bgp
show ip route bgp
```

Foram verificados:

* Estado das sessões BGP;
* Vizinhos estabelecidos;
* Prefixos recebidos;
* Prefixos anunciados;
* Instalação das rotas BGP na tabela de roteamento;
* Seleção de caminhos.

---

# Firewall e Segurança

Os firewalls pfSense são responsáveis pela aplicação das políticas de segurança e conectividade dos sites.

Foram implementados:

* Regras de firewall;
* NAT;
* Segmentação entre VLANs;
* Controle de tráfego entre redes;
* DHCP;
* DNS;
* Gerenciamento de interfaces;
* VPN Site-to-Site;
* VPN de acesso remoto;
* Políticas de acesso através da interface IPsec.

A existência de um túnel IPsec estabelecido não implica autorização automática do tráfego. As redes participantes da VPN possuem regras específicas de firewall para permitir a comunicação necessária.

---

# VPN Site-to-Site

Foi implementada uma VPN **IPsec Site-to-Site entre os firewalls pfSense da Matriz e da Filial**.

A comunicação entre os sites é segmentada através dos seletores definidos nas diferentes Phase 2.

### Redes participantes

| Matriz           | Filial           |
| ---------------- | ---------------- |
| `172.16.20.0/28` | `172.16.21.0/29` |
| `172.16.20.0/28` | `172.16.31.0/30` |
| `10.10.30.0/30`  | `172.16.21.0/29` |
| `10.10.30.0/30`  | `172.16.31.0/30` |

A implementação foi validada através de:

* Estado da Phase 1;
* Estado das Phase 2;
* Regras de firewall;
* Testes de conectividade entre redes;
* Análise dos logs dos firewalls.

A configuração detalhada e os testes estão documentados em:

```text
docs/vpn/site-to-site.md
```

---

# VPN de Acesso Remoto

O laboratório também possui uma implementação de VPN destinada ao acesso remoto à infraestrutura corporativa.

A documentação específica apresenta:

* Arquitetura;
* Configuração;
* Redes autorizadas;
* Políticas de acesso;
* Processo de autenticação;
* Testes de conectividade.

Documentação:

```text
docs/vpn/remote-access.md
```

---

# Redundância e Failover

A arquitetura possui múltiplos links WAN para os sites corporativos.

O objetivo é permitir a simulação de indisponibilidade de um caminho e observar o comportamento da infraestrutura diante da falha.

Foram realizados testes envolvendo:

* Disponibilidade dos gateways;
* Perda de conectividade de um link WAN;
* Seleção de caminho alternativo;
* Recuperação após retorno do link;
* Validação da conectividade após failover.

Documentação:

```text
docs/redundancy/failover.md
```

---

# Validação

O laboratório foi desenvolvido com foco em **implementação e validação prática**, e não apenas na configuração dos equipamentos.

Foram realizados testes envolvendo:

### ISP / BGP

* Comunicação entre os roteadores ISP;
* Estabelecimento das sessões eBGP;
* Recebimento de prefixos;
* Anúncio de prefixos;
* Instalação de rotas BGP;
* Seleção de caminhos.

### Switching

* Comunicação através de VLANs;
* Portas Access;
* Portas Trunk;
* Transporte de VLANs;
* Segmentação lógica.

### Firewall

* Regras de acesso;
* NAT;
* DHCP;
* DNS;
* Comunicação entre redes autorizadas;
* Bloqueio de tráfego conforme política.

### VPN

* Negociação IPsec;
* Phase 1;
* Phase 2;
* Comunicação entre redes da Matriz e Filial;
* Validação das regras de firewall;
* Testes de conectividade.

### Redundância

* Falha de links;
* Seleção de caminho alternativo;
* Recuperação do ambiente.

As evidências e procedimentos de teste serão documentados individualmente nos respectivos diretórios.

---

# Troubleshooting

O laboratório também é utilizado para reproduzir e solucionar problemas de conectividade e configuração.

Os incidentes serão documentados utilizando uma abordagem baseada em evidências:

```text
Problema
   ↓
Hipóteses
   ↓
Coleta de evidências
   ↓
Testes
   ↓
Identificação da causa raiz
   ↓
Correção
   ↓
Validação
```

Cada ocorrência de troubleshooting poderá apresentar:

1. Problema identificado;
2. Hipóteses levantadas;
3. Comandos utilizados;
4. Evidências coletadas;
5. Causa raiz;
6. Correção aplicada;
7. Teste final.

---

# Tecnologias

### Networking

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

### Security

* Firewall
* pfSense
* IPsec
* VPN Site-to-Site
* Remote Access VPN
* Network Segmentation

### Infrastructure

* DHCP
* DNS
* Failover
* Troubleshooting

### Virtualização / Laboratório

* EVE-NG
* Cisco IOS

---

# Estrutura da documentação

```text
enterprise-network-lab/
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
│       └── troubleshooting.md
│
└── diagrams/
    └── topology.png
```

---

# Status do Projeto

| Componente        | Status          |
| ----------------- | --------------- |
| Topologia ISP     | ✅ Concluído     |
| eBGP              | ✅ Concluído     |
| Roteamento        | ✅ Concluído     |
| VLANs             | ✅ Concluído     |
| Trunk             | ✅ Concluído     |
| Access Ports      | ✅ Concluído     |
| pfSense           | ✅ Concluído     |
| NAT               | ✅ Concluído     |
| Firewall          | ✅ Concluído     |
| DHCP              | ✅ Concluído     |
| DNS               | ✅ Concluído     |
| VPN Site-to-Site  | ✅ Concluído     |
| VPN Remote Access | ✅ Concluído     |
| Failover          | ✅ Concluído     |
| Testes            | ✅ Concluído     |
| Documentação      | 🚧 Em andamento |

---

# Observação sobre a simulação dos ISPs

Os três ISPs são representados por roteadores Cisco independentes dentro do ambiente EVE-NG.

A conectividade externa dos roteadores utiliza a mesma conexão física de Internet disponível no ambiente de laboratório. Portanto, a implementação representa uma **simulação lógica de múltiplos provedores**, e não três links físicos de operadoras distintas.

O objetivo dessa camada é demonstrar conceitos de:

* Autonomous Systems;
* eBGP;
* Anúncio de prefixos;
* Seleção de caminhos;
* Roteamento entre provedores;
* Redundância lógica;
* Troubleshooting de conectividade.

---

# Objetivo profissional

Este laboratório foi desenvolvido para consolidar conhecimentos práticos de **redes, infraestrutura e segurança**, utilizando uma arquitetura que permite estudar desde fundamentos de switching e roteamento até BGP, firewalls, VPN, redundância e troubleshooting.

A documentação busca registrar não apenas as configurações utilizadas, mas também as decisões técnicas, testes realizados, problemas encontrados e respectivas soluções.
