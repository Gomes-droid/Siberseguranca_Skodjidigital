# Sessão 01 — Introdução ao Linux para Segurança e Comandos de Rede

**Curso:** Reskilling — Skodji Digital
**Módulo:** Linux e Cibersegurança
**Objetivo de Aprendizagem:** OA1 · Analisar
**Formador:** Péricles Borges
**Ambiente:** KillerCoda Ubuntu Playground / TryHackMe — Further Nmap
**Data de execução:** `DD/MM/AAAA`

---

## 1. Contexto e objetivo

Mapeamento e análise da **superfície de exposição** de um servidor alvo na rede local,
assumindo o papel de auditor de sistemas.

O trabalho segue três fases:

1. **Reconhecimento local** — identificar a interface de rede e os serviços em escuta no
   próprio ambiente.
2. **Reconhecimento remoto** — mapear um alvo com o Nmap.
3. **Documentação** — registar portas, serviços e versões detetadas.

---

## 2. Resumo executivo

> _Preencher após a execução._

| Elemento | Resultado |
|---|---|
| **IP do ambiente local** | `[PREENCHER]` |
| **Interface principal** | `[PREENCHER — ex.: eth0]` |
| **Portas em escuta localmente** | `[PREENCHER]` |
| **IP do alvo (TryHackMe)** | `[PREENCHER]` |
| **Nº de portas abertas no alvo** | `[PREENCHER]` |
| **Serviço mais exposto** | `[PREENCHER]` |

---

## 3. Reconhecimento do ambiente local (KillerCoda)

### 3.1. Identificação da interface e endereço IP

```bash
ip a
```

**Output:**

```text
[COLAR AQUI O OUTPUT COMPLETO]
```

**Leitura:**

| Elemento | Valor | Significado |
|---|---|---|
| Interface principal | `[ex.: eth0]` | Interface de rede ativa |
| Endereço IPv4 (`inet`) | `[ex.: 10.0.2.15]` | Endereço da máquina na rede |
| Máscara | `[ex.: /24]` | Dimensão da rede — `[N]` endereços possíveis |
| Loopback | `127.0.0.1` (`lo`) | Interface interna, não é rede real |

### 3.2. Listagem das portas em escuta

```bash
ss -tuln
```

**Output:**

```text
[COLAR AQUI O OUTPUT COMPLETO]
```

Decomposição dos switches:

| Switch | Função |
|---|---|
| `-t` | Protocolo TCP |
| `-u` | Protocolo UDP |
| `-l` | Apenas sockets em escuta (*listening*) |
| `-n` | Saída numérica — evita resolução de nomes de serviço |

**Portas identificadas:**

| Porta | Protocolo | Endereço de escuta | Alcance | Serviço presumido |
|---|---|---|---|---|
| `[22]` | `[tcp]` | `[0.0.0.0]` | Todas as interfaces | `[ssh]` |
| | | | | |

> **Nota de análise:** um socket em `0.0.0.0` escuta em **todas** as interfaces e é
> alcançável a partir da rede; um socket em `127.0.0.1` escuta apenas localmente. Esta
> distinção determina a exposição real do serviço.

---

## 4. Reconhecimento do alvo remoto (TryHackMe — Further Nmap)

### 4.1. Comando executado

```bash
nmap -sV -sC <IP_DO_ALVO>
```

| Flag | Função |
|---|---|
| `-sV` | Deteção de versão dos serviços em execução |
| `-sC` | Execução dos scripts NSE do conjunto por defeito |

### 4.2. Output do scan

```text
[COLAR AQUI O OUTPUT COMPLETO DO NMAP]
```

### 4.3. Portas, serviços e versões detetadas

_Critério de entrega principal — preencher a partir do output acima._

| Porta | Estado | Serviço | Versão detetada |
|---|---|---|---|
| `[22/tcp]` | `[open]` | `[ssh]` | `[OpenSSH X.Y]` |
| `[80/tcp]` | `[open]` | `[http]` | `[Apache X.Y.Z]` |
| | | | |

**Total de portas abertas identificadas:** `[N]`

### 4.4. Achados dos scripts NSE (`-sC`)

_[Registar informação adicional revelada pelos scripts: títulos de páginas web, métodos
HTTP permitidos, certificados, banners, configurações permissivas, etc.]_

```text
[COLAR AQUI OS EXCERTOS RELEVANTES]
```

---

## 5. Evidências — capturas de ecrã

| # | Descrição | Ficheiro |
|---|---|---|
| 1 | Output de `ip a` no ambiente local | `capturas/ip-a.png` |
| 2 | Output de `ss -tuln` no ambiente local | `capturas/ss-tuln.png` |
| 3 | Resultado do scan Nmap ao alvo | `capturas/nmap-sv-sc.png` |

<!-- Inserir com: ![Descrição](capturas/nome-do-ficheiro.png) -->

---

## 6. Análise da superfície de exposição

**Superfície identificada.** _[Descrever em prosa: o alvo expõe N serviços, dos quais X são
de acesso remoto e Y são serviços web.]_

**Serviços de maior risco.** _[Justificar. Considerar: serviços de administração remota
expostos publicamente, versões antigas com vulnerabilidades conhecidas, serviços que não
deveriam estar acessíveis a partir do exterior.]_

**Relevância da deteção de versões.** As vulnerabilidades documentadas são específicas de
versões concretas. A identificação exata (`-sV`) é o que permite, do lado defensivo,
verificar se o software está atualizado — e, do lado ofensivo, procurar exploits aplicáveis.

**Valor acrescentado dos scripts NSE.** _[Indicar que informação os scripts revelaram que o
scan de portas por si só não daria.]_

---

## 7. Considerações éticas e legais

O varrimento de portas contra sistemas sem autorização expressa é ilegal na generalidade
das jurisdições. Todo o trabalho documentado foi executado em ambientes de laboratório
concebidos para o efeito — **KillerCoda** e **TryHackMe** — onde a autorização é explícita
e o alvo é fornecido pela própria plataforma.

---

## 8. Dificuldades encontradas e resolução

_[Preencher. Exemplos possíveis: expiração da máquina do TryHackMe durante o scan;
necessidade de estabelecer ligação VPN; duração elevada do scan com `-sV`.]_

---

## 9. Conclusões e aprendizagens

- A arquitetura Linux organiza-se em camadas — **kernel**, **shell** e **user space**. O
  kernel é o único que acede ao hardware; tudo o resto pede-lhe autorização.
- A **CLI** impõe-se em auditoria por três razões concretas: os servidores raramente têm
  ambiente gráfico, os comandos são automatizáveis e deixam um histórico auditável.
- O reconhecimento começa **de dentro para fora**: `ip a` e `ss -tuln` caracterizam o
  próprio ambiente antes de qualquer scan externo.
- Uma porta aberta não é, por si só, um problema. O que importa é **que serviço**, **que
  versão** e **a quem está acessível**.

---

## 10. Ligação às sessões seguintes

| Sessão | Relação |
|---|---|
| **02 — Análise de logs** | Os serviços aqui mapeados (nomeadamente SSH) são os que geram os registos de autenticação analisados forensicamente. |
| **03 — Firewalls** | O `ss -tuln` mostra o que está aberto; o UFW decide o que fica alcançável a partir do exterior. |

➡️ [Sessão 02 — Auditoria de Sistemas Linux e Análise Avançada de Logs](../sessao-02/README.md)

---

## Referências

- Laboratório da Sessão 1 — Skodji Digital, Percurso Reskilling
- [KillerCoda — Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)
- [TryHackMe — Further Nmap](https://tryhackme.com/room/furthernmap)
- Documentação: `man ip`, `man ss`, `man nmap`


# Siberseguran-a_Skodjidigital

# Laboratório — Sessão_1

<img width="1598" height="1002" alt="Tarefas" src="https://github.com/user-attachments/assets/944ecbeb-7393-42f9-bbca-a1536d5711a8" />



---



## ss -tuln

---

<img width="961" height="363" alt="nmap -sV -sC " src="https://github.com/user-attachments/assets/bedca684-1d13-4f45-9e5d-3752157733da" />



---




<img width="966" height="883" alt="ss -tuln" src="https://github.com/user-attachments/assets/7a09b397-8815-4166-b376-25a540500a02" />



---



## nmap -sV -sC 10.130.151.190

---


<img width="1916" height="905" alt="nmap -sV -sC 10 130 151 190" src="https://github.com/user-attachments/assets/8079a347-a8ec-4b3b-b28c-e06f80d4e758" />



---

# Laboratório — Sessão 2


---

## cd /var/log/

<img width="1912" height="893" alt="image" src="https://github.com/user-attachments/assets/232ada5f-d507-42ae-b5f5-e19ae68a5096" />

<img width="1142" height="285" alt="image" src="https://github.com/user-attachments/assets/92d9acf5-4a47-4c96-87c9-8e7db1d096ae" />

---

## grep "Failed password" auth.log

<img width="1125" height="405" alt="image" src="https://github.com/user-attachments/assets/28c1a7bf-a127-471b-b4b5-98ecece14e12" />

---

## log com CMD
---

<img width="911" height="1033" alt="image" src="https://github.com/user-attachments/assets/a673c408-466c-4727-b9fc-f1d8a13d1e8a" />

---

## cd /var/log/
## grep "Failed password" auth.log
## grep "Failed password" auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
## grep -E "Accepted password|Accepted publickey" auth.log
<img width="1042" height="353" alt="image" src="https://github.com/user-attachments/assets/dda3ce18-8e73-464d-8f7f-4fca65b18ff1" />

---
