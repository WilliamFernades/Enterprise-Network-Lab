# pfSense — Firewalls Corporativos

## Objetivo

Os firewalls pfSense são utilizados como principal camada de controle e segurança das redes corporativas da **Matriz** e da **Filial**.

Cada site possui um firewall responsável por conectar as redes internas à infraestrutura WAN, aplicar políticas de segurança e fornecer serviços de rede.

---

# Arquitetura

```text
                         SIMULATED ISP
                              |
                +-------------+-------------+
                |                           |
          pfSense MATRIZ               pfSense FILIAL
                |                           |
           Core Switch                 Core Switch
                |                           |
        Corporate VLANs             Corporate VLANs
```

Os firewalls também estabelecem uma conexão **IPsec Site-to-Site** para permitir comunicação controlada entre as redes internas dos dois sites.

---

# Matriz

O pfSense da Matriz atua como gateway das redes corporativas do site.

### Principais funções

* Conectividade WAN;
* Multi-WAN;
* Roteamento;
* NAT;
* Firewall;
* DHCP;
* DNS;
* VLANs;
* VPN IPsec;
* Controle de tráfego entre redes;
* Gerenciamento da infraestrutura.

### Interfaces WAN

| Provedor | Interface | Endereço         |
| -------- | --------- | ---------------- |
| CLARO    | WAN       | `100.64.10.2/30` |
| VIVO     | WAN       | `100.64.20.2/30` |
| OI       | WAN       | `100.64.30.2/30` |

### Redes internas

| VLAN    | Rede             | Função        |
| ------- | ---------------- | ------------- |
| VLAN 20 | `172.16.20.0/28` | TI            |
| VLAN 30 | `10.10.30.0/30`  | Servidores    |
| VLAN 10 | `172.16.10.0/28` | Usuarios      |
| VLAN 99 | `172.16.99.0/30` | Gerenciamento |

---

# Filial

O pfSense da Filial possui função equivalente, atendendo as redes corporativas do segundo site.

### Principais funções

* Conectividade WAN;
* Multi-WAN;
* Roteamento;
* NAT;
* Firewall;
* DHCP;
* DNS;
* VLANs;
* VPN IPsec;
* Controle de tráfego entre redes;
* Gerenciamento da infraestrutura.

### Interfaces WAN

| Provedor | Interface | Endereço         |
| -------- | --------- | ---------------- |
| CLARO    | WAN       | `100.64.11.2/30` |
| VIVO     | WAN       | `100.64.21.2/30` |
| OI       | WAN       | `100.64.31.2/30` |

### Redes internas

| VLAN    | Rede             | Função     |
| ------- | ---------------- | ---------- |
| VLAN 21 | `172.16.21.0/29` | TI         |
| VLAN 11 | `172.16.11.0/28` | Comercial  |
| VLAN 12 | `172.16.12.0/28` | Marketing  |
| VLAN 31 | `172.16.31.0/30` | Servidores |

---

# Multi-WAN

Cada firewall possui três conexões WAN através dos provedores simulados.

```text
                  +----------------+
                  | ISP SIMULATION |
                  +-------+--------+
                          |
              +-----------+-----------+
              |           |           |
            CLARO       VIVO          OI
              |           |           |
              +-----------+-----------+
                          |
                       pfSense
```

A utilização de múltiplos links permite simular cenários de redundância e indisponibilidade de um provedor.

Os testes de failover são documentados separadamente em:

```text
docs/redundancy/failover.md
```

---

# VLANs

As redes internas são segmentadas através de VLANs.

A segmentação permite separar diferentes grupos e funções da infraestrutura, reduzindo o domínio de broadcast e permitindo aplicação de políticas específicas de firewall.

Exemplo da Matriz:

```text
VLAN 20
172.16.20.0/28
      |
      +---- TI

VLAN 30
10.10.30.0/30
      |
      +---- SERVERS
```

O mesmo princípio é aplicado à Filial.

---

# Firewall

O pfSense atua como ponto de controle entre:

* Redes internas;
* WAN;
* VLANs;
* VPN;
* Serviços publicados.

As regras são aplicadas de acordo com a origem, destino, protocolo e serviço necessários.

O princípio utilizado no laboratório é permitir somente o tráfego necessário para cada comunicação.

A documentação detalhada das políticas está disponível em:

```text
docs/firewall/firewall-rules.md
```

---

# NAT

O NAT é utilizado para permitir que redes internas tenham acesso através dos links WAN.

O comportamento de NAT é documentado separadamente em:

```text
docs/firewall/nat.md
```

---

# DHCP e DNS

Os firewalls também fornecem serviços de infraestrutura de rede para as redes internas.

### DHCP

Responsável pela distribuição de parâmetros como:

* Endereço IP;
* Máscara;
* Gateway;
* Servidores DNS.

### DNS

Utilizado para resolução de nomes dentro do ambiente de laboratório e como parte da infraestrutura de conectividade das redes internas.

---

# VPN IPsec

Os dois firewalls pfSense estabelecem uma VPN Site-to-Site utilizando IPsec.

```text
MATRIZ                         FILIAL

pfSense
100.64.10.2
    |
    |       IPsec
    +=======================+
                            |
                       100.64.11.2
                         pfSense
```

A comunicação através da VPN é controlada pelos seletores de tráfego da Phase 2 e pelas regras de firewall.

A implementação detalhada está documentada em:

```text
docs/vpn/site-to-site.md
```

---

# Gerenciamento

O gerenciamento dos equipamentos é realizado através de uma rede dedicada de gerenciamento quando aplicável.

O objetivo da separação é evitar que o acesso administrativo aos equipamentos dependa das mesmas redes utilizadas pelos usuários finais.

---

# Validação

A implementação dos firewalls foi validada através de testes de:

* Conectividade WAN;
* Conectividade entre VLANs;
* NAT;
* DHCP;
* DNS;
* Regras de firewall;
* Comunicação através da VPN;
* Failover entre links WAN;
* Recuperação após retorno do link.

Os resultados individuais são documentados nos respectivos componentes do projeto.

---

# Resultado

Os firewalls pfSense funcionam como camada central de **roteamento, segurança, conectividade WAN e interligação dos sites**.

A arquitetura permite simular cenários comuns em ambientes corporativos, incluindo:

* Múltiplos provedores;
* Segmentação por VLAN;
* Controle de acesso;
* NAT;
* VPN Site-to-Site;
* Redundância;
* Troubleshooting de conectividade.

O objetivo do componente não é apenas fornecer conectividade, mas demonstrar a utilização do firewall como ponto central de **controle e segmentação da infraestrutura de rede**.
