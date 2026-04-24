# 📦 Relatório Final

> **Atividade:** Bug Report Profissional + Code Review Guiado
> **Curso:** Qualidade de Software
> **Professor:** Prof. Claudio Nunes

---

## 👥 Identificação da dupla

| Nome completo | RA | GitHub |
|---|---|---|
| Lucas Freitas Reis | 223790 | lucasfr21 |
| Gabriel de Oliveira Corrêa | 226653 | gaadrelnablunt-oss |

**Ambiente de testes:** [Descreva brevemente o setup — ex: Chrome 121 no Windows 11, GitHub Pages do fork, editor web do GitHub]

---

## 📋 Sumário

- [Parte A — Bug Reports](#parte-a--bug-reports)
- [Parte B — Code Review](#parte-b--code-review)
- [Reflexão final](#-reflexão-final)
- [Declarações](#-declarações)

---

## Parte A — Bug Reports

> **Substitua este bloco de citação pelo conteúdo copiado integralmente do seu arquivo `parte-a-bug-report/template-bug-report.md`.**
> Preservando todos os campos, incluindo a Matriz de Prioridade x Severidade, Passos para Reprodução e Evidências.
> Mínimo: 3 defeitos.

---

## Parte B — Code Review

> **Substitua este bloco de citação pelo conteúdo copiado integralmente do seu arquivo `parte-b-code-review/formulario-code-review.md`.**
> Preservando os rótulos, linhas e sugestões de correção.
> Mínimo: 6 findings.

### Resumo

| # | Linha | Dimensão | Rótulo | Severidade |
|---|-------|----------|--------|------------|
| 1 |   12    |     Segurança     |    blocker     |     Crítica       |
| 2 | 16 - 31, 33 - 39      |    Erros       |    major     |        Alta    |
| 3 |    5, 22   |     Erros      |     major    |       Média      |
| 4 |    41–108   |     Complexidade      |    major     |     Alta        |
| 5 |  41–108 e 110–152     |     Padrões      |    major     |     Média        |
| 6 |   34    |     Legibilidade     |    nit     |       Baixa      |

### Findings detalhadas

### Finding #1

**📍 Linha(s):** 12
**🏷 Rótulo:** blocker
**📂 Dimensão:** Segurança
**⚠️ Severidade:** Crítica

**🐛 Problema:** A função buscarUsuarioPorNome monta a query SQL concatenando diretamente o parâmetro nome recebido do chamador, sem qualquer sanitização. Um atacante pode injetar SQL arbitrário — por exemplo, passando ' OR '1'='1 como nome — e obter todos os registros da tabela, ou pior, executar comandos destrutivos.

**💡 Sugestão de correção:** Substituir a concatenação por query parametrizada, que é suportada por qualquer driver SQL moderno.

```javascript
// async function buscarUsuarioPorNome(nome) {
  return db.executarQuery(
    'SELECT * FROM usuarios WHERE nome = ?',
    [nome]
  );
}
```


---

### Finding #2

**📍 Linha(s):** 16–31, 33–39
**🏷 Rótulo:** major
**📂 Dimensão:** Erros
**⚠️ Severidade:** Alta

**🐛 Problema:** As funções assíncronas cadastrarUsuario e atualizarEmail fazem await em operações de banco de dados sem nenhum bloco try/catch. Se db.insert, db.buscarPorId ou db.atualizar lançarem uma exceção (ex.: timeout, constraint violation, registro não encontrado), a Promise rejeitada vai se propagar sem tratamento, podendo derrubar o processo ou retornar uma resposta 500 não controlada para o cliente.

**💡 Sugestão de correção:** Envolver o corpo de cada função com try/catch e tratar o erro de forma explícita.

```javascript
async function cadastrarUsuario(dados) {
  try {
    const salt = 10;
    const senhaHash = await crypto.hash(dados.senha, salt);
    const usuario = {
      nome: dados.nome,
      email: dados.email,
      tipo: dados.tipo,
      senha: senhaHash,
      ativo: true,
      criadoEm: new Date().toISOString()
    };
    await db.insert('usuarios', usuario);
    logger.info('Usuario cadastrado: ' + dados.email);
    return usuario;
  } catch (erro) {
    logger.error('Erro ao cadastrar usuario: ' + erro.message);
    throw erro;
  }
}
```

Finding #3
📍 Linha(s): 5, 22
🏷 Rótulo: major
📂 Dimensão: Erros
⚠️ Severidade: Média
🐛 Problema:
A constante TIPOS_VALIDOS é declarada na linha 5 mas nunca é usada para validar o campo dados.tipo em cadastrarUsuario. Qualquer string arbitrária (ex.: "admin", "superuser") é aceita e inserida diretamente no banco, podendo causar inconsistências e brechas de autorização.
💡 Sugestão de correção:
Adicionar validação do tipo antes de inserir o usuário.

```javascript
async function cadastrarUsuario(dados) {
  if (!TIPOS_VALIDOS.includes(dados.tipo)) {
    throw new Error(`Tipo inválido: ${dados.tipo}. Esperado: ${TIPOS_VALIDOS.join(', ')}`);
  }
```


### Finding #4

📍 Linha(s): 41–108
🏷 Rótulo: major
📂 Dimensão: Complexidade
⚠️ Severidade: Alta
🐛 Problema:
A função calcularLimiteEmprestimo possui complexidade ciclomática acima de 15, com 6 níveis de aninhamento de if/else. Isso torna o código extremamente difícil de ler, testar e manter — cada novo requisito de negócio exige navegar por toda a árvore para encontrar o ponto certo de alteração. Também há uma inconsistência lógica: a verificação de bloqueadoAte (linha 103) é feita depois de todo o cálculo, ao invés de ser o primeiro guard-clause.
💡 Sugestão de correção:
Extrair as regras por tipo de usuário em funções auxiliares e usar guard-clauses no início.

```javascript
function calcularLimiteEmprestimo(usuario) {
  if (usuario.bloqueadoAte && new Date(usuario.bloqueadoAte) > new Date()) {
    return 0;
  }
  if (usuario.tipo === 'professor') return _limiteProfessor(usuario);
  if (usuario.tipo === 'aluno') return _limiteAluno(usuario);
  return 5;
}

function _limiteProfessor(usuario) {
  const base = usuario.tempoCasaEmDias > 365 ? { sem: 20, poucos: 15, muitos: 3 }
                                              : { sem: 10, poucos: 7,  muitos: 2 };
  if (usuario.atrasos === 0) return base.sem;
  if (usuario.atrasos < 3)  return base.poucos;
  if (usuario.multaPendente) return 1;
  return base.muitos;
}
```

---

### Finding #5

📍 Linha(s): 41–108 e 110–152
🏷 Rótulo: major
📂 Dimensão: Padrões
⚠️ Severidade: Média
🐛 Problema:
calcularLimiteEmprestimo e calcularLimiteComSuspensao duplicam toda a lógica de negócio de cálculo de limite por tipo de usuário. Qualquer mudança de regra (ex.: alterar o limite de professor com > 365 dias e 0 atrasos) precisa ser feita em dois lugares, com alto risco de divergência entre as funções ao longo do tempo.
💡 Sugestão de correção:
calcularLimiteComSuspensao deve delegar o cálculo base para calcularLimiteEmprestimo e apenas acrescentar sua verificação adicional.

```javascript
function calcularLimiteComSuspensao(usuario) {
  if (usuario.suspenso) return 0;
  return calcularLimiteEmprestimo(usuario);
}
```

---

### Finding #6

📍 Linha(s): 34
🏷 Rótulo: nit
📂 Dimensão: Legibilidade
⚠️ Severidade: Baixa
🐛 Problema:
A variável u na função atualizarEmail é um nome sem significado. Em um code review ou depuração, não é imediatamente claro que u representa um objeto usuário completo retornado do banco, com todos os seus campos.
💡 Sugestão de correção:
Renomear para usuario, mantendo consistência com o restante do código.

```javascript
async function atualizarEmail(id, novoEmail) {
  const usuario = await db.buscarPorId('usuarios', id);
  usuario.email = novoEmail;
  await db.atualizar('usuarios', id, usuario);
  logger.info('Email atualizado: ' + novoEmail);
  return usuario;
}
```

---

## 💭 Reflexão final

A atividade nos mostrou que defeitos reais raramente aparecem sozinhos: o mesmo descuido que gera um SQL Injection (ausência de validação de entrada) é o mesmo que permite cadastrar um usuário com tipo inválido. Percebemos também que código que "funciona" pode esconder problemas sérios de manutenção e segurança que só ficam visíveis quando lido com atenção sistemática, dimensão por dimensão. A experiência de escrever bug reports e findings como se fossem comentários de PR reais tornou o processo mais concreto e nos fez pensar não só em "o que está errado" mas em "como comunicar isso de forma útil para quem vai corrigir".

**Qual dimensão do checklist foi mais difícil aplicar? Por quê?**

A dimensão de Complexidade foi a mais desafiadora. Ao ler calcularLimiteEmprestimo, é fácil se perder no aninhamento e achar que o código "funciona", sem perceber que a profundidade da árvore de decisão já ultrapassa qualquer limite razoável de manutenibilidade. Identificar que a complexidade ciclomática estava acima de 15 exigiu contar deliberadamente os caminhos independentes de execução, algo que não é intuitivo em uma primeira leitura.

**O que vocês fariam diferente se revisassem o código novamente?**

Começaríamos pelos pontos de entrada e saída de dados externos (parâmetros de funções públicas) antes de analisar a lógica interna, o que teria nos levado ao SQL Injection e à ausência de validação de tipo logo no início. Também usaríamos uma ferramenta de análise estática (como ESLint com regras de complexidade) para detectar automaticamente funções com ciclomática elevada, ao invés de identificar isso apenas pela leitura visual.

---

## 📣 Declarações


### Uso de IA como parceiro de trabalho

- [ ] Não usamos IA nesta atividade.
- [ ] Usamos IA para esclarecer conceitos teóricos.
- [✔] Usamos IA para revisar a redação dos bug reports.
- [✔] Usamos IA para discutir se um achado era ou não um defeito.
- [ ] Uso específico: [descreva]

### Declaração de autoria

Declaramos que este relatório é de autoria da dupla, que exploramos
pessoalmente a aplicação da Parte A e lemos o código da Parte B. As
findings aqui registradas representam nosso próprio julgamento
técnico.
