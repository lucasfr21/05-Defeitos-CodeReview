# 🐛 Bug Reports — Parte A


**Dupla:** Lucas Freitas Reis + 223790 + Gabriel de Oliveira Corrêa + 226653
**Data da exploração:** 23/04/2026
**Navegador usado:** Chrome 121
**Sistema operacional:** Windows 11

---

## BUG-001

**Título:** Campo não é limpo após adicionar tarefa, dados anteriores persistem no forms.

**Severidade:** Alta
**Justificativa da severidade:** O usuário é forçado a apagar manualmente cada campo antes de criar uma nova tarefa, o que torna o fluxo principal da aplicação inutilizável de forma fluida.

**Prioridade:** P2
**Justificativa da prioridade:** A função central da aplicação é adicionar tarefas; qualquer fricção nesse caminho afeta todos os usuários em todas as sessões.

**Ambiente:**
- Navegador: Chrome
- Sistema Operacional: Windows 11
- Versão da aplicação: TarefaQS v1.0.0

**Passos para reprodução:**
1. coloque um título + categoria + prazo + prioridade
2. Clique no botão de "Adcionar tarefa"


**Resultado esperado:**
Após adicionar a tarefa com sucesso, todos os campos do formulário devem ser limpos automaticamente, prontos para uma nova entrada.

**Resultado obtido:**
Os campos permanecem preenchidos com os dados da tarefa anterior. O usuário precisa clicar em "Limpar campos" manualmente ou apagar cada campo individualmente.

**Evidência:**
<img width="1906" height="961" alt="image" src="https://github.com/user-attachments/assets/9d28a056-8267-4efc-94d1-52d08c8413b2" />


**Sugestão de causa raiz (opcional):**


---

## BUG-002

**Título:** Contador de "Pendentes" exibe valor incorreto — tarefas de prioridade 5 são contadas em dobro

**Severidade:** Média
**Justificativa da severidade:** O dado exibido no painel de estatísticas é falso, o que prejudica a confiança do usuário nas informações apresentadas pela aplicação.

**Prioridade:** P3
**Justificativa da prioridade:** Não bloqueia a criação ou conclusão de tarefas, mas gera confusão sobre o estado real das pendências do usuário.

**Ambiente:**
- Navegador: Chrome
- Sistema Operacional: Windows 11
- Versão da aplicação: TarefaQS v1.0.0

**Passos para reprodução:**
1. Abrir a aplicação
2.Adicionar 3 tarefas com prioridade 5 e deixá-las pendentes
3.Adicionar 1 tarefa com prioridade 1 e deixá-la pendente
4.Observar o valor exibido no card "Pendentes"


**Resultado esperado:** O contador "Pendentes" deve exibir 4 (total de tarefas não concluídas).

**Resultado obtido:** O contador exibe 7 — as 3 tarefas de prioridade 5 são somadas novamente ao total, como se fossem tarefas extras.

**Evidência:** <img width="1897" height="947" alt="image" src="https://github.com/user-attachments/assets/b543a0d6-14d3-4267-8507-ad12c44f1ea7" />


**Sugestão de causa raiz (opcional):** 

---

## BUG-003

**Título:** Formulário permite criar tarefa com título vazio — tarefa em branco aparece na lista

**Severidade:** Média
**Justificativa da severidade:** Dados inválidos são persistidos na lista de tarefas, comprometendo a integridade do conteúdo e gerando itens sem significado para o usuário.

**Prioridade:** P2
**Justificativa da prioridade:** Qualquer usuário pode reproduzir acidentalmente ao pressionar Enter ou "Adicionar tarefa" sem preencher o título, resultando em lixo na lista.

**Ambiente:**
- Navegador: Chrome
- Sistema Operacional: Windows 11
- Versão da aplicação: TarefaQS v1.0.0

**Passos para reprodução:**
1.Abrir a aplicação
2.Deixar o campo "Título" completamente em branco
3.Selecionar qualquer categoria (ex.: Estudo)
4.Clicar em "Adicionar tarefa" (ou pressionar Enter)
5.Observar a lista de tarefas


**Resultado esperado:** A aplicação deve exibir uma mensagem de erro informando que o título é obrigatório e não adicionar nenhuma tarefa à lista.

**Resultado obtido:** Uma tarefa com título vazio é adicionada à lista sem nenhuma mensagem de aviso. O item aparece na lista com o espaço do título em branco.

**Evidência:** <img width="1864" height="926" alt="image" src="https://github.com/user-attachments/assets/ca02ce3a-e8bf-4583-9c84-9c5d95240ef2" />


**Sugestão de causa raiz (opcional):**

---

<!-- Para reports adicionais, copie o bloco acima trocando o número. -->

---

## ✅ Critérios de qualidade do bug report
*(Use para conferir antes de entregar)*

- [✅] Título descritivo — outra pessoa entende o problema só pelo título?
- [✅] Passos são **numerados** e **reproduzíveis** por terceiros?
- [✅] Há **pelo menos uma evidência** (screenshot, GIF ou log)?
- [✅] Severidade tem **justificativa explícita**?
- [✅] Prioridade tem **justificativa explícita**?
- [✅] Ambiente inclui **navegador + SO**?
- [✅] "Esperado vs. Obtido" deixa o gap claro?

## ✅ Checklist de qualidade dos reports

Antes de submeter, confirme em cada report:

- [✅] Título é específico e acionável (não `"Não funciona"`).
- [✅] Passos estão **numerados** e são reproduzíveis por terceiros.
- [✅] Há **pelo menos uma evidência** por report (imagem, GIF ou log).
- [✅] Severidade tem **justificativa explícita**.
- [✅] Prioridade tem **justificativa explícita**.
- [✅] Ambiente inclui **navegador + SO**.
- [✅] "Esperado × Obtido" deixa a diferença clara.
- [✅] Os 3 defeitos reportados cobrem **categorias diferentes**
      (funcional, UX, validação, persistência, etc.)
