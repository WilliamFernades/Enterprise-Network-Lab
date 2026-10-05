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

Três roteadores Cisco repr
