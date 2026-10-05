# Firewall Rules

## Objetivo

Este documento descreve as políticas de firewall implementadas no ambiente corporativo do laboratório, utilizando o pfSense como gateway de segurança entre as redes internas, Internet, VLANs e túneis VPN.

As regras foram estruturadas com foco em:

* Segmentação entre redes;
* Princípio do menor privilégio;
* Controle de acesso entre VLANs;
* Proteção da rede de gerenciamento;
* Controle de tráfego entre Matriz e Filial;
* Permissão explícita para serviços necessários;
* Bloqueio de tráfego não autorizado.

> **Nota:** este projeto representa um ambiente de laboratório. A complexidade técnica apresentada não representa, por si só, experiência profissional em produção.

---

## 1. Princípio de Segurança

A política adotada considera que o tráfego entre diferentes segmentos deve ser explicitamente autorizado quando necessário.

A arquitetura utiliza o pfSense como ponto de controle entre:

```text
                    INTERNET
                       |
                +------+------+
                |   pfSense   |
                +------+------+
                       |
        +--------------+--------------+
        |              |              |
     VLAN 20        VLAN 30        VLAN 99
       IT           SERVERS           MGMT
        |
        +---- VPN IPsec ---- Filial
```

A lógica geral é:

```text
Permitir somente o necessário
            +
Bloquear o restante
            =
Menor superfície de exposição
```

---

# 2. Segmentação das Redes

## Matriz

| VLAN    | Rede             | Função        |
| ------- | ---------------- | ------------- |
| VLAN 20 | `172.16.20.0/28` | TI            |
| VLAN 10 | `172.16.10.0/28` | RH            |
| VLAN 30 | `10.10.30.0/30`  | Servidores    |
| VLAN 99 | `172.16.99.0/30` | Gerenciamento |

## Filial

| VLAN    | Rede             | Função     |
| ------- | ---------------- | ---------- |
| VLAN 21 | `172.16.21.0/29` | TI         |
| VLAN 11 | `172.16.11.0/28` | Comercial  |
| VLAN 12 | `172.16.12.0/28` | Marketing  |
| VLAN 31 | `172.16.31.0/30` | Servidores |

---

# 3. Matriz de Acesso

A matriz abaixo representa a política lógica utilizada para determinar quais segmentos podem se comunicar.

| Origem          | Destino         | Política | Objetivo                                |
| --------------- | --------------- | -------- | --------------------------------------- |
| VLAN 20 IT      | Internet        | ALLOW    | Acesso externo                          |
| VLAN 10 IT      | Internet        | ALLOW    | Acesso externo                          |
| VLAN 20 IT      | VLAN 30 Servers | ALLOW    | Acesso aos serviços corporativos        |
| VLAN 20 IT      | VLAN 99 MGMT    | RESTRICT | Administração controlada                |
| VLAN 30 Servers | Internet        | RESTRICT | Permitir somente serviços necessários   |
| VLAN 30 Servers | VLAN 20 IT      | RESTRICT | Evitar acesso iniciado pelos servidores |
| VLAN 99 MGMT    | Infraestrutura  | ALLOW    | Administração                           |
| Internet        | Redes internas  | DENY     | Bloqueio de acesso não solicitado       |
| Matriz          | Filial          | ALLOW    | Comunicação via IPsec                   |
| Filial          | Matriz          | ALLOW    | Comunicação via IPsec                   |

A política efetiva pode variar de acordo com o serviço utilizado e com as regras específicas configuradas no pfSense.

---

# 4. Regras da Matriz

## VLAN 20 — IT

A VLAN de TI possui acesso aos serviços necessários para administração da infraestrutura e aos recursos externos.

Exemplos de tráfego autorizado:

* DNS;
* HTTP/HTTPS;
* Serviços corporativos;
* Servidores internos;
* Comunicação com a Filial através do túnel IPsec.

Exemplo lógico:

```text
VLAN 20
   |
   +----> Internet       ALLOW
   |
   +----> VLAN 30        ALLOW
   |
   +----> VPN IPsec      ALLOW
   |
   +----> VLAN 99        RESTRICT
```

---

## VLAN 30 — Servers

A rede de servidores possui uma política mais restritiva.

Os servidores não devem possuir acesso irrestrito às demais redes apenas por estarem atrás do firewall.

O acesso deve ser liberado conforme a necessidade do serviço.

Exemplo:

```text
VLAN 30
   |
   +----> Serviços necessários     ALLOW
   |
   +----> Internet                 RESTRICT
   |
   +----> VLAN 20                  RESTRICT
   |
   +----> VLAN 99                  DENY/RESTRICT
```

---

## VLAN 99 — Management

A VLAN 99 é destinada à administração dos equipamentos de infraestrutura.

O acesso administrativo deve ser restrito a origens autorizadas.

Serviços considerados para administração:

* HTTPS;
* SSH;
* ICMP para diagnóstico;
* Outros serviços de gerenciamento necessários.

Exemplo:

```text
VLAN 20 / Administração autorizada
              |
              v
         VLAN 99 MGMT
              |
       +------+------+
       |             |
    pfSense       Switch
```

A VLAN de gerenciamento não deve ser utilizada como rede comum de usuários.

---

# 5. Regras de Acesso à Internet

O tráfego iniciado pelas redes internas pode ser encaminhado para a Internet conforme as políticas estabelecidas no firewall.

Fluxo:

```text
Cliente
   |
VLAN
   |
pfSense
   |
Firewall Rule
   |
NAT
   |
Multi-WAN
   |
ISP
   |
Internet
```

O NAT é responsável pela tradução dos endereços privados utilizados internamente para os endereços utilizados nas interfaces WAN.

As regras de firewall determinam **quem pode iniciar o tráfego**, enquanto o NAT trata da tradução dos endereços.

---

# 6. Regras da VPN IPsec

O estabelecimento do túnel IPsec não significa automaticamente que qualquer tráfego entre as redes será permitido.

Por isso, foram utilizadas regras específicas na interface IPsec.

Redes envolvidas:

```text
MATRIZ
172.16.20.0/28
10.10.30.0/30
        |
        | IPsec
        |
FILIAL
172.16.21.0/29
172.16.31.0/30
```

As regras permitem somente os segmentos necessários para a comunicação entre Matriz e Filial.

Exemplo:

```text
Matriz VLAN 20
      |
      | ALLOW
      v
    IPsec
      |
      v
Filial VLAN 21
```

O mesmo princípio é aplicado às redes de servidores.

As regras e os seletores de Phase 2 devem ser coerentes entre si. Um túnel pode estar estabelecido enquanto o tráfego continua bloqueado pelo firewall.

---

# 7. Acesso Administrativo

O acesso administrativo aos equipamentos de infraestrutura deve ocorrer a partir de uma rede autorizada.

Exemplo:

```text
Origem autorizada
       |
       +---- HTTPS ----> pfSense
       |
       +---- SSH ------> Switch
       |
       +---- ICMP -----> Infraestrutura
```

O acesso administrativo proveniente diretamente da Internet não faz parte da política padrão do laboratório.

---

# 8. Política de Negação

A política de segurança segue o princípio:

```text
ALLOW explícito
      +
DENY implícito
```

Tráfego que não corresponde a uma regra permitida não deve ser considerado automaticamente autorizado.

Isso permite reduzir a superfície de ataque e evita que novos serviços sejam expostos apenas porque foram adicionados à rede.

---

# 9. Validação

As regras foram validadas utilizando testes de conectividade e análise dos estados do firewall.

Testes realizados:

### Comunicação permitida

```text
VLAN 20 -> Internet
VLAN 20 -> VLAN 30
Matriz -> Filial
Filial -> Matriz
Rede autorizada -> Management
```

### Comunicação bloqueada/restrita

```text
Internet -> VLAN 20
Internet -> VLAN 30
Internet -> VLAN 99
Redes não autorizadas -> Management
```

A validação deve considerar não apenas o resultado do `ping`, mas também o serviço utilizado.

Por exemplo:

```text
ICMP
TCP/443
TCP/22
DNS/53
```

Um teste ICMP bem-sucedido não significa necessariamente que determinado serviço TCP esteja autorizado.

---

# 10. Troubleshooting

Quando uma comunicação esperada não funciona, a investigação segue uma abordagem por camadas:

```text
1. Interface
      ↓
2. VLAN
      ↓
3. Gateway
      ↓
4. Roteamento
      ↓
5. Firewall Rule
      ↓
6. NAT
      ↓
7. Serviço/Porta
      ↓
8. Retorno do tráfego
```

No pfSense, devem ser analisados principalmente:

* Status das interfaces;
* Gateway utilizado;
* Regras aplicadas à interface correta;
* Logs do firewall;
* Estados das conexões;
* Regras IPsec;
* NAT;
* Rotas;
* Portas e protocolos utilizados.

Exemplo de raciocínio:

```text
Ping não funciona
       |
       v
Interface UP?
       |
      SIM
       |
       v
Gateway alcançável?
       |
      SIM
       |
       v
Existe rota?
       |
      SIM
       |
       v
Firewall permite?
       |
      NÃO
       |
       v
Corrigir regra
       |
       v
Executar novo teste
```

---

# 11. Resultado

A implementação das regras de firewall permitiu aplicar segmentação lógica entre os diferentes ambientes da infraestrutura.

O firewall passou a atuar não apenas como gateway de Internet, mas como ponto central de controle entre:

* Redes internas;
* Servidores;
* Gerenciamento;
* Internet;
* Matriz;
* Filial;
* VPN IPsec.

A abordagem adotada prioriza acesso explícito, segmentação e menor privilégio, mantendo o ambiente preparado para futuras políticas de segurança e expansão da infraestrutura.
