# RCA — Falha no Failover WAN

> **Tipo:** Incidente reproduzido em laboratório
> **Área:** Redundância / WAN / pfSense

## Problema identificado

Durante a simulação de indisponibilidade do link WAN principal, o tráfego não realizou o failover conforme esperado para o próximo gateway disponível.

### 5 Porquês

**1. Por que o tráfego não realizou o failover?**

Porque o pfSense continuava considerando o gateway principal como operacional.

**2. Por que o gateway continuava sendo considerado operacional?**

Porque o mecanismo de monitoramento configurado para o gateway não detectava corretamente a falha simulada.

**3. Por que a falha não era detectada?**

O monitoramento dependia de um destino que continuava alcançável através de outro caminho.

**4. Por que isso era um problema?**

A disponibilidade do destino não representava necessariamente a disponibilidade do próprio caminho WAN monitorado.

**5. Qual foi a causa raiz?**

Critério inadequado de monitoramento para determinar a disponibilidade do gateway.

### Ações Realizadas

Foi analisado o grupo:

```text
GW_JANUS
```

Foram verificados:

* status dos gateways;
* prioridades;
* monitoramento;
* tabela de rotas;
* conectividade antes da falha;
* comportamento durante a falha;
* recuperação do link.

A indisponibilidade do enlace foi simulada e o comportamento do gateway foi acompanhado.

O monitoramento foi ajustado para utilizar um destino capaz de representar melhor a disponibilidade do caminho correspondente.

Após a correção, o teste foi repetido.

### Considerações Finais

Failover não deve ser validado apenas verificando se existem múltiplos links configurados.

É necessário validar:

* detecção da falha;
* decisão do gateway;
* alteração da rota;
* continuidade do tráfego;
* retorno ao caminho preferencial.

O teste demonstrou a diferença entre possuir redundância configurada e possuir redundância efetivamente validada.
