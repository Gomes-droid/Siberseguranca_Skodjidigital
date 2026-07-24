[README.md](https://github.com/user-attachments/files/30347379/README.md)
# Sessão 04 — Gestão Segura de Acessos Remotos SSH em Linux

**Curso:** Reskilling — Skodji Digital
**Módulo:** Linux e Cibersegurança
**Objetivo de Aprendizagem:** OA4 · Aplicar
**Formador:** Péricles Borges
**Ambiente:** TryHackMe — Linux Strength Training / KillerCoda Ubuntu Playground
**Data de execução:** `DD/MM/AAAA`

---

## 1. Contexto e objetivo

Proteger o canal de gestão remota do servidor Ubuntu, **eliminando a autenticação por
password** e migrando para autenticação criptográfica por par de chaves.

A postura final pretendida:

| Diretiva | Valor | Ameaça que neutraliza |
|---|---|---|
| `PermitRootLogin` | `no` | Ataque direto à conta com privilégios totais |
| `PasswordAuthentication` | `no` | Ataques de dicionário e força bruta |
| `Port` | `2222` | Ruído de varrimento automatizado à porta 22 |

---

## 2. Resumo executivo

> _Preencher após a execução._

| Elemento | Resultado |
|---|---|
| **Algoritmo da chave gerada** | `Ed25519` |
| **Utilizador de teste criado** | `[PREENCHER]` |
| **Porta SSH final** | `2222` |
| **Autenticação por password** | Desativada |
| **Login root direto** | Bloqueado |
| **Validação de sintaxe (`sshd -t`)** | `[sem erros / erros corrigidos]` |
| **Acesso por chave confirmado** | `[sim / não]` |

---

## 3. Preparação — criação do utilizador de teste

```bash
sudo adduser <utilizador>
```

**Output (excerto):**

```text
[COLAR AQUI]
```

> O hardening é aplicado sobre uma conta não privilegiada. O acesso administrativo
> passa a fazer-se por `sudo` após a autenticação, e não por login direto como `root`.

---

## 4. Ciclo de vida da chave

### 4.1. Geração do par de chaves

```bash
ssh-keygen -t ed25519
```

**Output:**

```text
[COLAR AQUI O OUTPUT — REMOVER O FINGERPRINT SE PREFERIDO]
```

Ficheiros produzidos em `~/.ssh/`:

| Ficheiro | Tipo | Partilhável |
|---|---|---|
| `id_ed25519` | Chave **privada** | ❌ Nunca |
| `id_ed25519.pub` | Chave **pública** | ✅ Sim |

**Justificação do algoritmo.** O Ed25519 assenta em criptografia de curva elíptica,
atingindo um nível de segurança equivalente a RSA-3072 com chaves de apenas 256 bits.
É mais rápido a gerar e a verificar, e produz chaves substancialmente mais compactas.
O RSA mantém-se apenas como opção de compatibilidade com sistemas legados.

### 4.2. Distribuição da chave pública

```bash
ssh-copy-id <utilizador>@<IP_DO_SERVIDOR>
```

**Output:**

```text
[COLAR AQUI]
```

O comando acrescenta a chave pública ao ficheiro `~/.ssh/authorized_keys` da conta no
servidor — a lista de chaves autorizadas a autenticar-se nessa conta.

### 4.3. Verificação das permissões

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/authorized_keys
ls -la ~/.ssh/
```

**Output:**

```text
[COLAR AQUI]
```

| Permissão | Notação | Significado |
|---|---|---|
| `700` | `rwx------` | Apenas o dono lê, escreve e acede à diretoria |
| `600` | `rw-------` | Apenas o dono lê e escreve o ficheiro |

> O OpenSSH **recusa-se a utilizar** chaves com permissões demasiado abertas. Se outro
> utilizador pudesse escrever em `authorized_keys`, poderia acrescentar a sua própria
> chave e obter acesso à conta.

### 4.4. Teste do acesso por chave — **antes** de qualquer alteração

```bash
ssh -i ~/.ssh/id_ed25519 <utilizador>@<IP>
```

**Output:**

```text
[COLAR AQUI — EVIDÊNCIA DE LOGIN BEM-SUCEDIDO]
```

> **Passo obrigatório.** A autenticação por password só é desativada depois de confirmado
> que o acesso por chave funciona.

---

## 5. Hardening do `sshd_config`

### 5.1. Ficheiro editado

```bash
sudo nano /etc/ssh/sshd_config
```

> `sshd_config` (com `d`) é a configuração do **daemon** — o serviço no servidor.
> `ssh_config` (sem `d`) é a configuração do **cliente**. São ficheiros distintos.

### 5.2. Linhas modificadas

```text
PermitRootLogin no
PasswordAuthentication no
Port 2222
```

### 5.3. Fundamentação de cada diretiva

**`PermitRootLogin no`** — impede a autenticação direta na conta `root` via SSH. Um
atacante deixa de conhecer antecipadamente metade das credenciais. O acesso privilegiado
passa a exigir dois passos: autenticação com conta nominal e posterior elevação por `sudo`.
Ganha-se ainda rastreabilidade: cada ação privilegiada fica associada a um utilizador
concreto nos logs de autenticação.

**`PasswordAuthentication no`** — elimina integralmente o vetor de ataque por dicionário.
Sem password aceite pelo serviço, tentativas repetidas de adivinhação tornam-se inúteis,
independentemente do volume. Apenas a posse da chave privada correspondente permite o acesso.

**`Port 2222`** — reduz o ruído gerado por varrimento automatizado, que incide
maioritariamente na porta 22. Trata-se de uma medida de **redução de ruído**, não de uma
defesa criptográfica: um varrimento completo (`nmap -p-`) localiza o serviço na nova porta.
O benefício real é a legibilidade dos logs, que passam a evidenciar tentativas dirigidas.

### 5.4. Cópia limpa das secções relevantes do `sshd_config`

> Sem chaves privadas, endereços internos sensíveis ou credenciais.

```text
[COLAR AQUI AS SECÇÕES RELEVANTES DO FICHEIRO]
```

---

## 6. Validação e aplicação

### 6.1. Validação de sintaxe

```bash
sudo sshd -t
```

**Output:**

```text
[COLAR AQUI — A AUSÊNCIA DE OUTPUT INDICA SINTAXE VÁLIDA]
```

> Passo crítico. Um erro de sintaxe impede o arranque do `sshd`; se a sessão de recurso
> já tiver sido encerrada, o acesso remoto ao servidor é perdido de forma irrecuperável.

### 6.2. Atualização da regra de firewall

```bash
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp
sudo ufw status verbose
```

**Output:**

```text
[COLAR AQUI]
```

> **Dependência da Sessão 03.** A política aplicada nessa sessão autorizava exclusivamente
> a porta `22/tcp`. Alterar a porta do SSH sem atualizar a firewall resultaria em bloqueio
> do próprio acesso administrativo. A nova porta é aberta **antes** de a antiga ser removida.

### 6.3. Reinício do serviço

```bash
sudo systemctl restart sshd
sudo systemctl status sshd
```

**Output:**

```text
[COLAR AQUI]
```

> O reinício do serviço **não encerra as sessões já estabelecidas**. A sessão original
> foi mantida aberta como via de recurso durante todo o procedimento.

---

## 7. Teste final — evidência de acesso por chave na nova porta

```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 <utilizador>@<IP>
```

**Output (critério de entrega):**

```text
[COLAR AQUI A EVIDÊNCIA COMPLETA DO LOGIN BEM-SUCEDIDO]
```

### Verificação nos logs de autenticação

```bash
sudo grep "Accepted publickey" /var/log/auth.log | tail -5
```

```text
[COLAR AQUI]
```

> A presença de `Accepted publickey` — em substituição de `Accepted password` — confirma
> que a migração para autenticação criptográfica foi efetivamente aplicada. É exatamente
> a distinção analisada no laboratório da Sessão 02.

---

## 8. Evidências — capturas de ecrã

| # | Descrição | Ficheiro |
|---|---|---|
| 1 | Geração do par de chaves Ed25519 | `capturas/ssh-keygen.png` |
| 2 | Linhas modificadas do `sshd_config` | `capturas/sshd-config.png` |
| 3 | Validação de sintaxe com `sshd -t` | `capturas/sshd-t.png` |
| 4 | Login por chave na porta 2222 | `capturas/login-chave-2222.png` |

<!-- Inserir com: ![Descrição](capturas/nome-do-ficheiro.png) -->

---

## 9. Sequência de segurança adotada

A ordem de execução foi definida para evitar perda de acesso ao servidor:

| # | Passo | Razão |
|---|---|---|
| 1 | Gerar par de chaves | — |
| 2 | Distribuir chave pública (`ssh-copy-id`) | Ainda possível por password |
| 3 | **Testar login por chave** | Confirmar antes de remover a alternativa |
| 4 | Abrir a nova porta na firewall | Evitar bloqueio pelo UFW |
| 5 | Editar `sshd_config` | — |
| 6 | **Validar sintaxe (`sshd -t`)** | Um erro impede o arranque do serviço |
| 7 | Reiniciar o serviço, mantendo a sessão aberta | Sessão de recurso para correção |
| 8 | Testar em terminal novo | Confirmação independente |
| 9 | Encerrar a sessão original | Apenas após confirmação |

---

## 10. Dificuldades encontradas e resolução

_[Preencher. Exemplos possíveis: recusa da chave por permissões incorretas; erro de sintaxe
detetado por `sshd -t`; bloqueio pela regra de firewall da sessão anterior; nome do serviço
`ssh` em vez de `sshd` consoante a distribuição.]_

---

## 11. Conclusões e aprendizagens

- A **criptografia assimétrica** resolve o problema logístico da criptografia simétrica: as
  duas chaves nunca precisam de se encontrar, e a privada nunca circula na rede.
- A autenticação SSH por chave funciona por **desafio-resposta** — o servidor envia um valor
  aleatório, o cliente assina-o com a chave privada e o servidor verifica a assinatura com a
  pública. Nenhuma password é transmitida.
- **Ed25519 supera RSA** em velocidade e compacidade para segurança equivalente ou superior.
  RSA mantém-se apenas por compatibilidade com sistemas antigos.
- A mudança de porta é **segurança por obscuridade**: reduz ruído automatizado, mas não
  constitui defesa contra um adversário dedicado.
- O **maior risco do hardening é o próprio operador**. A validação prévia (`sshd -t`) e a
  manutenção de uma sessão de recurso são o que separa uma configuração aplicada de um
  servidor inacessível.

---

## 12. Ligação às sessões anteriores

| Sessão | Relação |
|---|---|
| **01 — Reconhecimento** | O `nmap -sV` deteta a versão do OpenSSH exposta; após a alteração, o serviço deixa de responder na porta 22. |
| **02 — Análise de logs** | O ataque investigado explorava autenticação SSH por password — exatamente o vetor eliminado nesta sessão. |
| **03 — Firewall** | A regra `allow 22/tcp` teve de ser substituída por `allow 2222/tcp`, sob pena de bloqueio do acesso. |

➡️ [Sessão 03 — Hardening de Redes Linux e Configuração de Firewalls](../sessao-03/README.md)

---

## Referências

- Laboratório da Sessão 4 — Skodji Digital, Percurso Reskilling
- [TryHackMe — Linux Strength Training](https://tryhackme.com/room/linuxstrengthtraining)
- [KillerCoda — Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)
- Documentação: `man ssh-keygen`, `man ssh-copy-id`, `man sshd_config`, `man systemctl`
