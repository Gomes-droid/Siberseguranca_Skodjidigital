# Sessão 03 — Hardening de Redes Linux e Configuração de Firewalls

**Curso:** Reskilling — Skodji Digital
**Módulo:** Linux e Cibersegurança
**Objetivo de Aprendizagem:** OA3 · Aplicar
**Formador:** Péricles Borges
**Ambiente:** KillerCoda Ubuntu Playground / TryHackMe — Network Security Essentials
**Data de execução:** `DD/MM/AAAA`

---

## 1. Objetivo do laboratório

Aplicar uma política defensiva estrita de firewall num servidor Linux, combinando
**UFW** (gestão simplificada) e **iptables** (controlo granular), de forma a impedir
acessos não autorizados a serviços críticos.

A política pretendida assenta em três decisões:

1. **Default Deny** no tráfego de entrada — nada entra, exceto o explicitamente autorizado.
2. **Exceção mínima** — apenas SSH (22/tcp), por ser o único acesso administrativo necessário.
3. **Bloqueio dirigido** de um IP identificado como malicioso, com descarte silencioso (DROP).

---

## 2. Estado inicial do sistema

Verificação do estado da firewall antes de qualquer alteração.

```bash
sudo ufw status
```

**Output:**

```text
[COLAR AQUI O OUTPUT]
```

> **Observação:** _[Indicar se a firewall estava `inactive` ou `active`, e se existiam regras prévias.]_

---

## 3. Configuração aplicada

### 3.1. Políticas por defeito

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**Output:**

```text
[COLAR AQUI O OUTPUT]
```

### 3.2. Exceção para acesso SSH

```bash
sudo ufw allow 22/tcp
```

**Output:**

```text
[COLAR AQUI O OUTPUT]
```

> **Nota de procedimento:** a regra de SSH é criada **antes** da ativação da firewall,
> para evitar a perda do próprio acesso administrativo à máquina remota.

### 3.3. Bloqueio de IP malicioso (iptables)

```bash
sudo iptables -A INPUT -s 203.0.113.50 -j DROP
```

| Componente | Função |
|---|---|
| `-A INPUT` | Acrescenta a regra ao fim da chain de tráfego de entrada |
| `-s 203.0.113.50` | Aplica-se à origem indicada (bloco RFC 5737, reservado a documentação) |
| `-j DROP` | Descarta o pacote silenciosamente, sem resposta ao emissor |

### 3.4. Persistência das regras

```bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

**Output:**

```text
[COLAR AQUI O OUTPUT]
```

> As regras do iptables residem em memória e não sobrevivem a um reinício.
> A exportação para `/etc/iptables/rules.v4` garante a sua reposição no arranque.

---

## 4. Evidências — validação final

### 4.1. Regras UFW ativas

```bash
sudo ufw status verbose
```

```text
[COLAR AQUI O OUTPUT COMPLETO]
```

### 4.2. Listagem das chains iptables com contadores

```bash
sudo iptables -L -v
```

```text
[COLAR AQUI O OUTPUT COMPLETO]
```

### 4.3. Capturas de ecrã

| # | Descrição | Ficheiro |
|---|---|---|
| 1 | Estado do UFW após configuração | `capturas/ufw-status-verbose.png` |
| 2 | Listagem iptables com contadores | `capturas/iptables-l-v.png` |

<!-- Inserir as imagens com: ![Descrição](capturas/nome-do-ficheiro.png) -->

---

## 5. Explicação da política aplicada

### O que está bloqueado

_[Preencher com base no output obtido.]_

- Todo o tráfego de entrada não explicitamente autorizado.
- Tráfego proveniente do IP `203.0.113.50`, descartado sem resposta.

### O que está permitido

- Ligações SSH na porta `22/tcp`.
- Todo o tráfego de saída originado no servidor.

### Fundamentação

**Porquê Default Deny.** Uma política de negação por defeito aplica o *princípio do menor
privilégio*: em vez de bloquear apenas as ameaças conhecidas, autoriza-se exclusivamente o
que é necessário. Serviços novos, portas esquecidas ou vulnerabilidades ainda não conhecidas
ficam automaticamente fechados, sem necessidade de enumeração.

**Porquê permitir a saída.** O vetor de ataque relevante neste cenário é de fora para dentro.
O servidor mantém necessidades legítimas de tráfego de saída (atualizações, resolução DNS,
sincronização horária), pelo que a restrição de saída não se justifica neste âmbito.

**Porquê DROP e não REJECT.** O `DROP` não devolve qualquer resposta ao emissor, ao contrário
do `REJECT`, que responde com uma mensagem ICMP. Em face pública, o silêncio reduz a
informação disponível ao atacante durante a fase de reconhecimento: um scan não consegue
distinguir entre porta filtrada, máquina inexistente e pacote perdido. O custo é um
diagnóstico interno mais difícil, aceitável neste contexto perimetral.

**Porquê apenas a porta 22.** É o único serviço requerido para administração remota do
servidor. Qualquer outra porta aberta aumentaria a superfície de ataque sem contrapartida
funcional.

---

## 6. Dificuldades encontradas e resolução

_[Preencher. Exemplos possíveis: diretoria `/etc/iptables` inexistente, resolvida com
`sudo mkdir -p /etc/iptables`; expiração da sessão do ambiente de laboratório; etc.]_

---

## 7. Conclusões e aprendizagens

- **UFW e iptables não são alternativas concorrentes** — o UFW é uma camada de gestão
  simplificada que escreve regras iptables por trás. O subsistema que efetivamente filtra
  os pacotes é o **Netfilter**, no kernel.
- As **chains** determinam o âmbito de aplicação de uma regra: `INPUT` (tráfego destinado ao
  host), `OUTPUT` (originado no host), `FORWARD` (encaminhado através do host).
- A escolha entre **DROP e REJECT** é uma decisão de postura defensiva, não uma questão
  técnica indiferente.
- **Regras não persistidas perdem-se no reinício** — a persistência é parte integrante da
  configuração, não um passo opcional.

---

## 8. Ligação à Sessão 2

A análise forense da sessão anterior identificou o IP responsável pelas tentativas de
autenticação falhadas. O bloqueio aplicado neste laboratório representa a **resposta
defensiva** a essa deteção: identificação da origem nos logs → contenção na firewall.

---

## Referências

- Laboratório da Sessão 3 — Skodji Digital, Percurso Reskilling
- [KillerCoda — Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)
- [TryHackMe — Network Security Essentials](https://tryhackme.com/room/networksecurityessentials)
- Documentação: `man ufw`, `man iptables`
