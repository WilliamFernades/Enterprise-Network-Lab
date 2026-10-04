# Enterprise Network Lab

Laboratório de redes corporativas desenvolvido em **EVE-NG**, com o objetivo de simular uma infraestrutura empresarial composta por **múltiplos provedores de Internet, roteamento BGP, firewalls, VLANs, switching, redundância e VPNs**.

O projeto foi desenvolvido como laboratório prático para demonstrar conhecimentos aplicados de redes e infraestrutura, desde a camada de conectividade e roteamento até a comunicação segura entre sites.

> **Nota:** este projeto representa um ambiente de laboratório. A complexidade técnica apresentada não representa, por si só, experiência profissional em produção.

---

## Objetivos

O laboratório tem como principais objetivos:

* Simular múltiplos provedores de Internet;
* Implementar roteamento entre diferentes Autonomous Systems;
* Utilizar **eBGP** para comunicação entre os provedores simulados;
* Estruturar uma rede corporativa segmentada por VLANs;
* Implementar firewall e políticas de segurança;
* Simular redundância e failover;
* Implementar NAT;
* Implementar VPN Site-to-Site;
* Implementar acesso remoto via VPN;
* Trabalhar com trunking e portas de acesso;
* Validar conectividade e funcionamento através de testes práticos;
* Documentar configurações, testes e resultados.

---

# Arquitetura

O laboratório é dividido em duas partes principais:

### Camada ISP

Três roteadores Cisco representam três provedores de Internet:

| Provedor | Equipamento |   ASN |
| -------- | ----------- | ----: |
| CLARO    | ISP-01      | 65001 |
| VIVO     | ISP-02      | 65002 |
| OI       | ISP-03      | 65003 |

Os três roteadores possuem conectividade entre si através de links de trânsito /30 e estabelecem sessões **eBGP**.

### Topologia ISP

```text
                         INTERNET
                            |
              +-------------+-------------+
              |             |             |
           AS 65001      AS 65002      AS 65003
            CLARO          VIVO            OI
           ISP-01        ISP-02         ISP-03
              |\             /|
              | \           / |
              |  \         /  |
              |   \       /   |
              +----+-----+----+
```

### Links de trânsito

| Enlace       | Rede            | Lado A       | Lado B       |
| ------------ | --------------- | ------------ | ------------ |
| CLARO ↔ VIVO | `10.255.1.0/30` | `10.255.1.1` | `10.255.1.2` |
| CLARO ↔ OI   | `10.255.2.0/30` | `10.255.2.1` | `10.255.2.2` |
| VIVO ↔ OI    | `10.255.3.0/30` | `10.255.3.1` | `10.255.3.2` |

Cada ISP anuncia uma rede própria através do BGP:

| ISP   | Prefixo anunciado |
| ----- | ----------------- |
| CLARO | `10.100.1.0/24`   |
| VIVO  | `10.100.2.0/24`   |
| OI    | `10.100.3.0/24`   |

Os prefixos utilizados para representar as redes dos provedores são mantidos através de interfaces `Null0`.

---

# Ambiente Corporativo

A segunda parte do laboratório representa uma empresa com dois sites:

```text
                         INTERNET
                             |
                    +--------+--------+
                    |  MULTI-WAN ISP |
                    +--------+--------+
                             |
                       +-----+-----+
                       |   MATRIZ  |
                       |  pfSense  |
                       +-----+-----+
                             |
                       Core Switch
                             |
              +--------------+--------------+
              |              |              |
             TI              RH           SERVERS


                             |
                       VPN SITE-TO-SITE
                             |
                       +-----+-----+
                       |  FILIAL   |
                       |  pfSense  |
                       +-----+-----+
                             |
                       Core Switch
                             |
          +------------------+------------------+
          |                  |                  |
         TI             COMERCIAL           MARKETING
                                             
                             |
                          SERVERS
```

## Matriz

A matriz possui:

* Firewall pfSense;
* Switch Core;
* VLAN de TI;
* VLAN de RH;
* VLAN de servidores.

## Filial

A filial possui:

* Firewall pfSense;
* Switch Core;
* VLAN de TI;
* VLAN Comercial;
* VLAN Marketing;
* VLAN de servidores.

---

# Firewalls

Os firewalls pfSense foram utilizados nos dois sites.

Foram implementados e validados:

* Failover;
* DHCP;
* DNS;
* NAT;
* Regras de firewall;
* VLANs;
* Port Trunk;
* VPN Site-to-Site;
* VPN de acesso remoto.

---

# Switching

Os switches utilizados como camada de Core possuem configuração de:

* VLANs;
* Portas Access;
* Portas Trunk;
* Segmentação lógica da rede;
* Conectividade entre as diferentes redes VLAN.

---

# Roteamento e BGP

A camada de provedores utiliza três Autonomous Systems:

```text
CLARO
AS 65001
   |
   | eBGP
   |
VIVO
AS 65002
   |
   | eBGP
   |
OI
AS 65003
```

Além das sessões diretas entre os provedores, a topologia possui caminhos alternativos entre os AS, permitindo observar o comportamento do BGP diante de diferentes caminhos disponíveis.

As sessões BGP foram validadas através do `show ip bgp summary` e apresentam os vizinhos estabelecidos e prefixos recebidos.

---

# Endereçamento

### Redes de trânsito

```text
10.255.1.0/30    CLARO ↔ VIVO
10.255.2.0/30    CLARO ↔ OI
10.255.3.0/30    VIVO ↔ OI
```

### Redes simuladas dos ISPs

```text
CLARO
100.64.10.0/30
100.64.11.0/30
100.64.12.0/30

VIVO
100.64.20.0/30
100.64.21.0/30
100.64.22.0/30

OI
100.64.30.0/30
100.64.31.0/30
100.64.32.0/30
```

---

# Validação

O laboratório não foi desenvolvido apenas como configuração teórica.

Os componentes foram configurados e submetidos a testes de conectividade e funcionamento.

Entre as validações realizadas estão:

* Comunicação entre os roteadores ISP;
* Estabelecimento das sessões eBGP;
* Recebimento de prefixos via BGP;
* Instalação de rotas BGP na tabela de roteamento;
* Comunicação entre VLANs conforme as políticas definidas;
* Funcionamento de NAT;
* Funcionamento de DHCP e DNS;
* Funcionamento das regras de firewall;
* Funcionamento da VPN Site-to-Site;
* Funcionamento da VPN de acesso remoto;
* Funcionamento de trunk e portas access;
* Funcionamento do failover.

As evidências dos testes serão documentadas individualmente nas próximas etapas do projeto.

---

# Tecnologias

* EVE-NG
* Cisco IOS
* pfSense
* VLAN
* 802.1Q / Trunk
* Access Port
* IPv4
* Subnetting
* Routing
* eBGP
* Autonomous System
* NAT
* Firewall
* DHCP
* DNS
* VPN Site-to-Site
* Remote Access VPN
* Failover

---

# Estrutura da documentação

A documentação será organizada por componente para facilitar a consulta:

```text
enterprise-network-lab/
│
├── README.md
│
├── docs/
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

## Observação sobre a simulação dos ISPs

Os três ISPs são representados por roteadores Cisco independentes dentro do ambiente EVE-NG.

A conectividade externa dos roteadores utiliza a mesma conexão física de Internet do ambiente de laboratório. Portanto, a implementação representa uma **simulação lógica de múltiplos provedores**, e não três links físicos de operadoras distintas.

O objetivo dessa camada é demonstrar conceitos de:

* Autonomous Systems;
* eBGP;
* anúncios de prefixos;
* seleção de caminhos;
* roteamento entre provedores;
* redundância lógica;
* troubleshooting de conectividade.

---
