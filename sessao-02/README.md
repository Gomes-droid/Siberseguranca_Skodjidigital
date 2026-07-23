[README.md](https://github.com/user-attachments/files/30308858/README.md)

# Sessão 02 — Auditoria de Sistemas Linux e Análise Avançada de Logs

**Curso:** Reskilling — Skodji Digital
**Módulo:** Linux e Cibersegurança
**Objetivo de Aprendizagem:** OA2 · Avaliar
**Formador:** Péricles Borges
**Ambiente:** TryHackMe — Intro to Logs / Linux Server Forensics
**Data de execução:** `DD/MM/AAAA`

---

## 1. Contexto e objetivo

Um servidor da infraestrutura foi alvo de conexões anómalas. O objetivo deste laboratório
é atuar como **analista forense** e determinar, a partir dos registos de autenticação
(`/var/log/auth.log`), três factos:

1. Qual a **origem** do ataque.
2. Se o atacante obteve **sucesso**.
3. **Quando** e sobre que **conta** ocorreu o comprometimento.

---

## 2. Resumo executivo

> _Preencher após a análise. Esta secção deve responder ao caso em 3–4 linhas, para quem
> só lê o topo do relatório._

| Elemento | Resultado |
|---|---|
| **IP do atacante** | `[PREENCHER]` |
| **Utilizador afetado** | `[PREENCHER]` |
| **Timestamp do comprometimento** | `[PREENCHER]` |
| **Nº de tentativas falhadas** | `[PREENCHER]` |
| **Método de autenticação bem-sucedido** | `[password / publickey]` |
| **Classificação do ataque** | `[PREENCHER — ex.: força bruta contra SSH]` |

---

## 3. Metodologia

### 3.1. Acesso à máquina comprometida

```bash
ssh fred@<IP-DA-MAQUINA-ALVO>
```

### 3.2. Navegação para a diretoria de logs

```bash
cd /var/log/
```

O ficheiro `auth.log` regista todos os eventos de autenticação do sistema — tentativas de
login bem-sucedidas e falhadas, uso de `sudo` e sessões SSH.

### 3.3. Isolamento das tentativas falhadas

```bash
grep "Failed password" auth.log
```

**Output (excerto):**

```text
[COLAR AQUI 3 A 5 LINHAS REPRESENTATIVAS]
```

> **Observação:** _[Indicar o volume aproximado de linhas e o padrão observado —
> tentativas repetidas em intervalos curtos, utilizadores testados, etc.]_

### 3.4. Contagem e ordenação dos IPs de origem

```bash
grep "Failed password" auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
```

**Output:**

```text
[COLAR AQUI O OUTPUT]
```

Decomposição da pipeline:

| Etapa | Função |
|---|---|
| `grep "Failed password"` | Filtra apenas as linhas de autenticação falhada |
| `awk '{print $11}'` | Extrai o campo correspondente ao IP de origem |
| `sort` | Agrupa entradas idênticas (requisito do passo seguinte) |
| `uniq -c` | Elimina duplicados consecutivos e conta as ocorrências |
| `sort -nr` | Ordena numericamente por contagem, do maior para o menor |

> **Nota:** o índice do campo (`$11`) depende do formato do log. Foi validado por inspeção
> de uma linha completa antes da extração.

### 3.5. Verificação de acessos bem-sucedidos

```bash
grep -E "Accepted password|Accepted publickey" auth.log
```

**Output:**

```text
[COLAR AQUI O OUTPUT]
```

Aqui o carácter `|` funciona como operador **OU** de expressão regular (ativado por `-E`),
e não como *pipe* — distinção relevante face aos comandos anteriores.

---

## 4. Evidências

### 4.1. Excertos relevantes do `auth.log`

**Tentativas falhadas (fase de força bruta):**

```text
[COLAR AQUI]
```

**Autenticação bem-sucedida (momento do comprometimento):**

```text
[COLAR AQUI]
```

### 4.2. Capturas de ecrã

| # | Descrição | Ficheiro |
|---|---|---|
| 1 | Contagem de IPs por tentativas falhadas | `capturas/ips-tentativas-falhadas.png` |
| 2 | Linha de autenticação aceite | `capturas/acesso-bem-sucedido.png` |

<!-- Inserir com: ![Descrição](capturas/nome-do-ficheiro.png) -->

---

## 5. Linha temporal do ataque

_Preencher com os timestamps reais extraídos do log._

| Momento | Evento | Evidência |
|---|---|---|
| `HH:MM:SS` | Primeira tentativa falhada registada a partir do IP `[X]` | `Failed password` |
| `HH:MM:SS` | Pico de tentativas — `[N]` falhas em `[intervalo]` | `Failed password` |
| `HH:MM:SS` | **Autenticação aceite** para o utilizador `[Y]` | `Accepted password` |
| `HH:MM:SS` | _[Atividade pós-comprometimento, se observada]_ | `[evidência]` |

**Narrativa:** _[Descrever em prosa curta: o IP X iniciou tentativas sistemáticas de
autenticação às HH:MM, acumulando N falhas; às HH:MM obteve acesso com sucesso à conta Y.]_

---

## 6. Análise e interpretação

**Padrão identificado.** _[Justificar a classificação do ataque. Um volume elevado de
falhas concentradas num intervalo curto, a partir de uma única origem, é característico
de um ataque de força bruta automatizado contra o serviço SSH.]_

**Indicador de sucesso.** A transição de `Failed password` para `Accepted password` no
mesmo IP de origem confirma o comprometimento — não se trata apenas de tentativa.

**Superfície explorada.** O serviço SSH exposto na porta 22, com autenticação por password,
permitiu a repetição ilimitada de tentativas.

---

## 7. Recomendações de mitigação

_Preencher/ajustar consoante os achados._

1. **Bloqueio da origem** — regra de firewall descartando tráfego do IP identificado
   (aplicado na Sessão 03).
2. **Autenticação por chave** — desativar autenticação SSH por password em favor de pares
   de chaves pública/privada.
3. **Limitação de tentativas** — implementar `fail2ban` ou equivalente para banir
   automaticamente origens com falhas repetidas.
4. **Rotação de credenciais** — alterar a password da conta comprometida e auditar
   atividade posterior ao acesso.
5. **Redução de exposição** — restringir o acesso SSH a origens conhecidas.

---

## 8. Dificuldades encontradas e resolução

_[Preencher. Exemplos possíveis: identificação do índice correto do campo no `awk`;
expiração da sessão do laboratório; permissões de leitura no `auth.log`.]_

---

## 9. Conclusões e aprendizagens

- O `auth.log` é a fonte primária para investigar acessos indevidos num sistema Linux:
  regista tanto falhas como sucessos de autenticação.
- O padrão **filtrar → extrair coluna → ordenar → contar → ordenar pela contagem** é uma
  pipeline reutilizável para qualquer análise de frequência em logs.
- O **pipe (`|`)** encadeia comandos passando a saída de um como entrada do seguinte;
  dentro de aspas com `grep -E`, o mesmo carácter significa **OU** — contextos distintos.
- A conclusão forense exige **correlação**, não apenas contagem: o IP mais frequente só é
  relevante quando associado a uma autenticação aceite.

---

## 10. Ligação à Sessão 03

O IP identificado nesta análise é o alvo do bloqueio aplicado no laboratório de
**hardening de firewall** da sessão seguinte: deteção → contenção.

➡️ [Sessão 03 — Hardening de Redes Linux e Configuração de Firewalls](../sessao-03/README.md)

---

## Referências

- Laboratório da Sessão 2 — Skodji Digital, Percurso Reskilling
- [TryHackMe — Intro to Logs](https://tryhackme.com/room/introtologs)
- [TryHackMe — Linux Server Forensics](https://tryhackme.com/room/linuxserverforensics)
- Documentação: `man grep`, `man awk`, `man sort`, `man uniq`
