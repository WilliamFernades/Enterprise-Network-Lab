# RCA — BGP Neighbor Down

> **Tipo:** Incidente reproduzido em laboratório
> **Área:** Routing / BGP
> **Equipamentos:** ISP-01 / ISP-02

## Problema identificado

A sessão eBGP entre o **ISP-01 (AS 65001)** e o **ISP-02 (AS 65002)** não estava estabelecida.

### 5 Porquês

**1. Por que a sessão BGP estava Down?**

Porque os roteadores não conseguiam estabelecer corretamente a sessão TCP utilizada pelo BGP.

**2. Por que a sessão TCP não era estabelecida?**

Porque havia uma inconsistência no endereço do vizinho configurado em um dos roteadores.

**3. Por que o endereço estava incorreto?**

O endereço configurado como `neighbor` não correspondia ao endereço utilizado pela interface diretamente conectada.

**4. Por que isso impedia o BGP?**

O BGP utiliza TCP/179 e precisa alcançar o endereço configurado do vizinho para estabelecer a sessão.

**5. Qual foi a causa raiz?**

Endereço de vizinhança BGP configurado incorretamente.

### Ações Realizadas

Foi verificado o estado da sessão:

```bash
show ip bgp summary
```

Em seguida foi analisada a configuração:

```bash
show running-config | section router bgp
```

Também foi validada a conectividade IP entre os roteadores.

Após identificar a divergência entre o endereço real e o endereço configurado no `neighbor`, a configuração foi corrigida.

A sessão foi novamente validada através de:

```bash
show ip bgp summary
```

O estado esperado passou a ser:

```text
Established
```

### Considerações Finais

O incidente reforçou a importância de separar problemas de:

1. conectividade IP;
2. TCP/179;
3. configuração BGP;
4. troca de rotas.

Uma sessão BGP `Idle` ou `Active` não deve ser analisada isoladamente. A conectividade entre os endpoints precisa ser validada antes da análise da política de roteamento.
