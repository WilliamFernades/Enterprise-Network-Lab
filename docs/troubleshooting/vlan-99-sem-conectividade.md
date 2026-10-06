# RCA — VLAN 99 sem conectividade

> **Tipo:** Incidente reproduzido em laboratório
> **Área:** Switching / VLAN / Trunking
> **Equipamentos:** ArubaOS-CX / pfSense

## Problema identificado

O switch ArubaOS-CX não conseguia alcançar o endereço `172.16.99.1`, configurado no pfSense para a VLAN 99 de gerenciamento.

O SVI do Aruba utilizava `172.16.99.2/30`.

### 5 Porquês

**1. Por que o Aruba não conseguia pingar `172.16.99.1`?**

Porque os pacotes da VLAN 99 não estavam chegando corretamente ao pfSense.

**2. Por que os pacotes não estavam chegando corretamente?**

Porque havia uma inconsistência na forma como a VLAN 99 estava sendo transportada entre o Aruba e o pfSense.

**3. Por que havia essa inconsistência?**

O enlace estava configurado como trunk no Aruba, com a VLAN 99 permitida e transportada de forma tagueada.

**4. Por que isso causava o problema?**

Porque o lado do pfSense precisava tratar a VLAN 99 como uma VLAN 802.1Q sobre a interface física correspondente.

**5. Qual foi a causa raiz?**

Incompatibilidade entre o modelo de transporte da VLAN 99 no trunk e a configuração da interface correspondente no pfSense.

### Ações Realizadas

Foram verificadas:

```bash
show running-config interface 1/1/1
show interface 1/1/1
show vlan
```

Foi confirmado que:

* a interface estava `up`;
* a VLAN 99 estava permitida;
* não existiam erros físicos;
* a VLAN 99 estava associada ao enlace;
* o Aruba utilizava `172.16.99.2/30`.

O enlace foi ajustado para que a VLAN 99 fosse transportada corretamente entre os equipamentos.

Após a correção:

```bash
ping 172.16.99.1
```

foi utilizado para validar a comunicação.

### Considerações Finais

O incidente demonstrou que a existência da VLAN no switch não garante sua conectividade ponta a ponta.

A análise precisou considerar simultaneamente:

* estado físico;
* VLAN;
* trunk;
* tagging 802.1Q;
* SVI;
* configuração do equipamento remoto.

O diagnóstico foi conduzido da camada física até a camada de rede.
