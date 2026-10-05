# VPN de Acesso Remoto

## Objetivo

Este documento descreve a implementação de uma VPN de acesso remoto no pfSense da Matriz.

Diferentemente da VPN Site-to-Site, utilizada para conectar redes inteiras entre Matriz e Filial, a VPN de acesso remoto permite que usuários individuais estabeleçam uma conexão segura com a infraestrutura corporativa a partir de redes externas.

```text id="xq7m2p"
                  INTERNET
                     |
                     |
              +------+------+
              |   pfSense   |
              |    MATRIZ   |
              +------+------+
                     |
              VPN Acesso Remoto
                     |
              10.0.8.0/28
                     |
              +------+------+
              |    Usuário  |
              |    remoto   |
              +-------------+
```

---

## 1. Cenário

A VPN foi configurada no pfSense da Matriz para permitir conexões remotas de usuários autorizados.

Ao estabelecer a conexão, o cliente VPN recebe um endereço pertencente ao pool:

```text
10.0.8.0/28
```

Essa rede é dedicada aos clientes da VPN de acesso remoto.

A separação do pool permite identificar e controlar o tráfego originado por usuários conectados remotamente.

---

## 2. Pool de Endereços

A faixa utilizada para os clientes VPN é:

| Parâmetro             | Valor             |
| --------------------- | ----------------- |
| Rede                  | `10.0.8.0/28`     |
| Máscara               | `255.255.255.240` |
| Endereços totais      | 16                |
| Endereços utilizáveis | 14                |
| Finalidade            | Clientes VPN      |

A faixa foi reservada exclusivamente para os clientes do acesso remoto.

```text
10.0.8.0/28

10.0.8.0   → endereço da rede
10.0.8.1   → utilizável
10.0.8.2   → utilizável
...
10.0.8.14  → utilizável
10.0.8.15  → broadcast
```

---

## 3. Fluxo de Conexão

O fluxo esperado é:

```text
Usuário remoto
      |
      | Internet
      v
   pfSense
      |
      | Autenticação
      v
VPN estabelecida
      |
      | IP 10.0.8.x
      v
Rede corporativa
```

Após a autenticação e estabelecimento do túnel, o cliente passa a possuir conectividade através da infraestrutura do pfSense, de acordo com as políticas de firewall configuradas.

---

## 4. Segmentação

Os clientes da VPN não são tratados como se estivessem fisicamente conectados a uma VLAN interna.

O tráfego originado da faixa:

```text
10.0.8.0/28
```

é controlado pelo firewall antes de alcançar os recursos corporativos.

Exemplo:

```text
VPN Client
10.0.8.x
    |
    v
pfSense
    |
    +----> Recursos autorizados
    |
    +----> Redes internas
    |
    +----> Internet
```

O acesso efetivo depende das regras de firewall aplicadas ao ambiente.

---

## 5. Controle de Acesso

A VPN de acesso remoto foi integrada às políticas de segurança do pfSense.

A existência de uma conexão VPN estabelecida não implica acesso irrestrito à infraestrutura.

As políticas podem controlar:

* Redes internas acessíveis;
* Servidores acessíveis;
* Portas e protocolos;
* Acesso administrativo;
* Comunicação com outras redes;
* Acesso à Internet através da VPN.

O princípio utilizado é permitir somente os recursos necessários ao usuário remoto.

---

## 6. Integração com a Rede Corporativa

A VPN permite que usuários externos alcancem recursos internos autorizados sem a necessidade de exposição direta desses serviços na Internet.

Exemplo:

```text
                 INTERNET
                    |
                    v
             VPN REMOTE ACCESS
                    |
               10.0.8.0/28
                    |
                 pfSense
                    |
          +---------+---------+
          |                   |
       VLAN 20             VLAN 30
         IT                SERVERS
```

O pfSense permanece como ponto central de controle do tráfego.

---

## 7. Validação

A implementação foi validada através do estabelecimento de uma conexão VPN a partir de um cliente remoto.

A validação considerou:

1. Estabelecimento da conexão VPN;
2. Atribuição de endereço pertencente à faixa `10.0.8.0/28`;
3. Comunicação com os recursos autorizados;
4. Aplicação das regras de firewall;
5. Teste de conectividade após o estabelecimento do túnel.

Fluxo validado:

```text
Cliente remoto
      |
      v
VPN estabelecida
      |
      v
IP 10.0.8.x
      |
      v
pfSense Matriz
      |
      v
Recurso corporativo
```

---

## 8. Troubleshooting

Em caso de falha na conexão ou acesso aos recursos internos, a análise pode seguir as seguintes camadas:

```text
1. Conectividade com a Internet
            ↓
2. Serviço VPN
            ↓
3. Autenticação
            ↓
4. Estabelecimento do túnel
            ↓
5. Atribuição do IP
            ↓
6. Rotas
            ↓
7. Firewall
            ↓
8. Serviço de destino
```

A primeira distinção importante é determinar se o problema está no **estabelecimento da VPN** ou no **acesso após a VPN estar conectada**.

Por exemplo:

```text
VPN não conecta
        ↓
investigar autenticação/túnel

VPN conecta, mas não acessa servidor
        ↓
investigar IP, rota e firewall
```

---

## 9. Resultado

A VPN de acesso remoto foi implementada no pfSense da Matriz e validada.

Os clientes remotos recebem endereços da faixa:

```text
10.0.8.0/28
```

A solução permite acesso remoto controlado à infraestrutura corporativa sem necessidade de expor diretamente os recursos internos à Internet.

A implementação complementa a VPN Site-to-Site existente:

```text
                 VPNs
                  |
        +---------+---------+
        |                   |
   Site-to-Site        Remote Access
        |                   |
   Matriz ↔ Filial      Usuário ↔ Matriz
```
