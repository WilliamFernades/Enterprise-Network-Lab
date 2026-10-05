# NAT e Port Forward

## Objetivo

Este documento descreve a configuração de NAT realizada no pfSense da Matriz para publicação de um site de vendas hospedado na rede interna.

O objetivo foi permitir que usuários externos acessassem o serviço através do endereço WAN do firewall, mantendo o servidor protegido na rede interna.

```text
                  INTERNET
                      |
                      | HTTP/HTTPS
                      v
              +---------------+
              |    pfSense    |
              |     MATRIZ    |
              +-------+-------+
                      |
                Port Forward
                      |
                      v
              +---------------+
              | Web Server     |
              | Site de vendas |
              +---------------+
```

---

## 1. Cenário

O servidor responsável pelo site de vendas está localizado na rede interna da Matriz.

O pfSense atua como gateway e firewall de borda, recebendo as conexões externas e realizando a tradução do endereço/porta para o servidor interno.

Fluxo:

```text
Cliente externo
      |
      | conexão ao endereço WAN
      v
pfSense Matriz
      |
      | DNAT / Port Forward
      v
Servidor Web
      |
      v
Site de vendas
```

---

## 2. Port Forward

Foi criada uma regra de Port Forward no pfSense da Matriz para encaminhar as conexões recebidas na interface WAN para o servidor responsável pelo site.

A regra considera:

| Parâmetro        | Configuração                     |
| ---------------- | -------------------------------- |
| Interface        | WAN                              |
| Protocolo        | TCP                              |
| Porta externa    | Porta utilizada pelo serviço Web |
| Destino          | Endereço WAN do pfSense          |
| Redirecionamento | Servidor Web interno             |
| Porta interna    | Porta do serviço Web             |

A publicação foi realizada de forma específica para o serviço necessário, evitando a exposição de portas adicionais do servidor.

---

## 3. Relação entre NAT e Firewall

O Port Forward não deve ser analisado isoladamente.

O fluxo envolve:

```text
Internet
   |
   v
WAN pfSense
   |
   v
Firewall
   |
   v
NAT / Port Forward
   |
   v
Servidor Web
```

O NAT realiza a tradução do destino da conexão.

A política de firewall determina se o tráfego recebido pode ser encaminhado.

Dessa forma, a publicação do serviço não significa que o servidor interno esteja diretamente exposto à Internet.

---

## 4. Segurança

A regra foi criada com escopo específico para o serviço publicado.

Foram considerados:

* Interface de entrada;
* Protocolo;
* Porta publicada;
* Endereço do servidor;
* Porta do serviço;
* Necessidade do serviço;
* Restrição de portas não utilizadas.

O objetivo é evitar uma publicação excessivamente permissiva, como encaminhar todas as portas do servidor.

---

## 5. Validação

Após a configuração, o acesso ao site foi testado a partir de uma origem externa à rede interna.

Validação realizada:

```text
Cliente externo
      |
      | TCP
      v
Endereço WAN
      |
      v
pfSense
      |
      | Port Forward
      v
Servidor Web
      |
      v
HTTP/HTTPS
      |
      v
Site de vendas
```

O resultado esperado foi confirmado:

```text
Conexão externa
       |
       v
    SUCESSO
       |
       v
Site de vendas acessível
```

Além da disponibilidade do site, a validação confirmou que o encaminhamento estava chegando ao servidor interno correto.

---

## 6. Troubleshooting

Em caso de falha no acesso externo, a análise pode ser realizada por camadas:

```text
1. Serviço Web
       ↓
2. Servidor interno
       ↓
3. Gateway
       ↓
4. Regra de NAT
       ↓
5. Regra de Firewall
       ↓
6. Interface WAN
       ↓
7. Conectividade externa
```

Pontos analisados:

* Servidor Web ativo;
* Porta do serviço em listening;
* Conectividade entre pfSense e servidor;
* Regra de Port Forward;
* Regra de firewall associada;
* Logs do firewall;
* Estados das conexões;
* Acesso utilizando uma origem externa.

---

## 7. Resultado

O Port Forward foi configurado e validado com sucesso no pfSense da Matriz.

O site de vendas passou a ser acessível externamente através do firewall, enquanto o servidor permanece localizado na rede interna.

A implementação demonstra a utilização de NAT/DNAT para publicação controlada de serviços, associada às políticas de firewall e segmentação da infraestrutura.
