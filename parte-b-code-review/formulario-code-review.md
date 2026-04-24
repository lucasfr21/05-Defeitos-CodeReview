# 🔎 Formulário — Parte B

> Preencha uma seção por finding. O mínimo esperado é **6 findings**.

**Dupla:** [Lucas Freitas Reis + 223790] + [Gabriel de Oliveira Corrêa + 226653]
**Data da revisão:** [23/04/2026]

---

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

## ✅ Checklist final

- [✅] Há pelo menos 6 findings preenchidas
- [✅] Cada finding cita linha, dimensão, rótulo e severidade
- [✅] As sugestões são concretas e acionáveis
- [✅] Pelo menos uma finding cobre segurança
- [✅] Pelo menos uma finding cobre complexidade
