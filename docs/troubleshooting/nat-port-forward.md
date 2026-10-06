# RCA — NAT Port Forward sem acesso externo

> **Tipo:** Incidente reproduzido em laboratório
> **Área:** NAT / Firewall / Publicação de serviços
> **Equipamento:** pfSense Matriz

## Problema identificado

Um servidor interno utilizado para representar o site comercial não estava acessível através do endereço WAN publicado pelo pfSense.

O Port Forward havia sido criado para encaminhar o tráfego externo ao servidor interno.

### 5 Porquês

**1. Por que o site não estava acessível externamente?**

Porque o tráfego recebido na interface WAN não estava chegando ao serviço interno.

**2. Por que o tráfego não chegava ao servidor?**

Porque o fluxo dependia tanto da tradução NAT quanto da regra correspondente de firewall.

**3. Por que o firewall poderia bloquear o tráfego?**

O Port Forward precisava estar associado à permissão correspondente para o tráfego encaminhado.

**4. Como foi descartada uma falha do servidor?**

Foi realizado teste diretamente a partir da rede interna para verificar se o serviço estava disponível no endereço e porta originais.

**5. Qual foi a causa raiz?**

Inconsistência na publicação do serviço entre a regra de Port Forward e a autorização correspondente no firewall.

### Ações Realizadas

Foram verificados:

* endereço do servidor;
* porta TCP publicada;
* Port Forward;
* regra automática/manual de firewall;
* interface WAN;
* conectividade interna com o servidor;
* teste externo.

A validação foi dividida em duas etapas:

**Teste interno:**

```text
Cliente interno → Servidor
```

**Teste externo:**

```text
Cliente externo → WAN pfSense → NAT → Servidor
```

Após o ajuste da regra, o acesso externo foi novamente testado.

### Considerações Finais

A publicação de um serviço não deve ser analisada somente pela existência do Port Forward.

O fluxo completo precisa ser validado:

```text
Internet
   ↓
WAN
   ↓
Firewall
   ↓
DNAT / Port Forward
   ↓
Servidor
   ↓
Serviço
```

A abordagem por etapas permitiu determinar se a falha estava no serviço, na conectividade interna, no NAT ou no firewall.
