# RCA — VPN UP sem tráfego

> **Tipo:** Incidente reproduzido em laboratório
> **Área:** VPN / Firewall / IPsec
> **Equipamentos:** pfSense Matriz / pfSense Filial

## Problema identificado

O túnel IPsec entre Matriz e Filial apresentava estado operacional ativo, porém hosts das redes internas não conseguiam se comunicar através do túnel.

### 5 Porquês

**1. Por que os hosts não conseguiam se comunicar?**

Porque o tráfego das redes internas estava sendo bloqueado.

**2. Por que estava sendo bloqueado?**

Porque o estabelecimento do túnel IPsec não implicava automaticamente autorização do tráfego nas regras do firewall.

**3. Por que não havia autorização?**

As regras necessárias para permitir o tráfego entre determinadas redes internas não estavam presentes ou não correspondiam aos seletores configurados.

**4. Por que isso não foi percebido inicialmente?**

Porque o estado `UP` do túnel foi interpretado como evidência de que a comunicação estava funcionando.

**5. Qual foi a causa raiz?**

Ausência/inconsistência das regras de firewall necessárias para permitir o tráfego através do IPsec.

### Ações Realizadas

Foi verificado:

* status do túnel;
* Phase 1;
* Phase 2;
* redes locais;
* redes remotas;
* regras da interface IPsec;
* regras das interfaces internas.

Foram realizados testes entre as redes:

```text
172.16.20.0/28
        ↕
172.16.21.0/29
```

e demais redes previstas no túnel.

As regras necessárias foram ajustadas para permitir somente os fluxos previstos.

Após a alteração, foram repetidos os testes de conectividade.

### Considerações Finais

O incidente demonstrou que:

**VPN estabelecida ≠ tráfego permitido.**

A validação correta de uma VPN deve considerar:

1. negociação criptográfica;
2. Phase 1;
3. Phase 2;
4. seletores;
5. roteamento;
6. firewall;
7. teste efetivo entre hosts.

Esse processo evita concluir que uma VPN está funcional apenas porque o túnel aparece como `UP`.
