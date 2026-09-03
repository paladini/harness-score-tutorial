# Tutorial prático do Harness Score

[![Usar este template para criar um repositório](https://img.shields.io/badge/Usar_este_template-Criar_reposit%C3%B3rio-2ea44f?style=for-the-badge&logo=github)](https://github.com/paladini/harness-score-tutorial/generate)

Use o botão acima para criar seu próprio repositório e executar o tutorial sem
alterar este guia.

Aprenda harness engineering evoluindo um repositório de `L0` até `L4`.
Você executará prompts em um agente de código, inspecionará cada mudança e
usará o [Harness Score](https://github.com/paladini/harness-score) para medir
como o repositório se torna mais preparado para desenvolvimento assistido por
IA.

Este tutorial não entrega a aplicação ou o harness prontos. Cada camada será
criada na sua máquina para que você observe o que mudou, por que o score subiu
e quais limitações ainda permanecem.

## Links úteis

- [Harness Score no npm](https://www.npmjs.com/package/harness-score) - consulte
  a versão publicada e instale ou execute o pacote com `npx`.
- [Código-fonte no GitHub](https://github.com/paladini/harness-score) - explore o
  scanner, acompanhe mudanças e reporte problemas.
- [Site oficial](https://paladini.io/harness-score/) - conheça o modelo de
  maturidade e os principais recursos do Harness Score.
- [Documentação em português](https://paladini.io/harness-score/pt-BR/) - acesse
  os guias de instalação, uso, interpretação e integração contínua.
- [Referência de todos os checks](https://paladini.io/harness-score/pt-BR/guide/measure-and-improve.html#the-check-catalog)
  - consulte critérios, pontuação, evidências e recomendações de cada check.
- [Harness Maturity Showcase](https://paladini.io/harness-maturity-showcase/) -
  compare a maturidade de harness de projetos de código aberto.
- [Código-fonte do Showcase](https://github.com/paladini/harness-maturity-showcase)
  - veja os dados, a geração do site e como contribuir com um projeto.

## O que você vai aprender

Ao concluir o tutorial, você terá praticado:

- Contexto global com `AGENTS.md`.
- Orientações carregadas por escopo de arquivos.
- Skills e workflows para procedimentos repetíveis.
- Higiene de repositório e proteção de credenciais.
- Testes, lint, formatação e typecheck como sensors.
- CI como fonte independente de feedback.
- Hooks que observam ou bloqueiam ações do agente.
- Um gate que impede a regressão da maturidade do harness.

## Como o tutorial funciona

Você criará o **Meeting Cost CLI**, uma aplicação pequena em Node.js que
estima o custo de mão de obra de uma reunião. O produto é simples para que o
foco permaneça na infraestrutura ao redor do agente.

Cada etapa segue o mesmo ciclo:

1. Execute um prompt no agente de código de sua preferência.
2. Inspecione os arquivos que o agente criou ou modificou.
3. Execute as verificações locais.
4. Calcule o Harness Score.
5. Compare o resultado com a etapa anterior.
6. Crie um commit e uma tag de checkpoint.

Os prompts funcionam com diferentes agentes de código. A etapa de hooks usa o
formato do Cursor porque é um runtime reconhecido pelo scanner e possui um
exemplo curto. Você pode criar e testar os arquivos com qualquer agente. Para
observar os hooks sendo acionados automaticamente, use o Cursor.

## Progressão esperada

| Etapa | Capacidade adicionada | Nível esperado | Score aproximado |
| --- | --- | --- | --- |
| 0 | Aplicação funcional, sem harness | L0 - Unharnessed | 14/108 |
| 1 | Contexto global | L1 - Documented | 21/108 |
| 1B | Contexto mais enxuto | L1 - Documented | 21/108 |
| 2 | Rule, skill, workflow e higiene | L2 - Guided | 52/108 |
| 3 | Sensors e CI | L3 - Sensing | 83/108 |
| 4 | Hooks em runtime | L4 - Self-correcting | 97/108 |
| 5 | Gate contra regressão | L4 - Self-correcting | 97/108 |

Os números são referências. A saída de uma LLM pode variar. Se o agente criar
artefatos além do escopo pedido, o score também poderá variar. O principal é
observar a mudança de nível e a dimensão que melhorou.

Como o tutorial usa o comando sem versão fixa, novas versões podem adicionar
checks ou alterar o total de pontos. Nesse caso, trate os níveis e as
tendências como a referência principal, não os números exatos desta tabela.

## Pré-requisitos

Instale:

- Git.
- Node.js 24 ou mais recente.
- Um agente de código capaz de editar o repositório local.
- GitHub CLI, opcional para criar uma cópia e publicar os resultados.

Confirme as ferramentas:

```powershell
node --version
npm --version
git --version
gh --version
npx --yes harness-score --version
```

O tutorial usa `npx harness-score` sem fixar uma versão, acompanhando a versão
publicada pelo npm. Executar o último comando antes de começar também aquece o
cache do `npx`.

## Crie sua própria cópia

### Opção 1 - Use o template no GitHub

Abra este repositório no GitHub, selecione **Use this template** e crie um novo
repositório na sua conta. Depois clone sua cópia:

```powershell
git clone https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git
Set-Location SEU-REPOSITORIO
```

### Opção 2 - Use o GitHub CLI

```powershell
gh repo create meu-laboratorio-harness `
  --template paladini/harness-score-tutorial `
  --private `
  --clone
Set-Location meu-laboratorio-harness
```

Você pode trocar `--private` por `--public`.

### Opção 3 - Execute somente localmente

```powershell
git clone https://github.com/paladini/harness-score-tutorial.git `
  meu-laboratorio-harness
Set-Location meu-laboratorio-harness
git remote remove origin
```

Confirme o estado inicial:

```powershell
git status --short
git log -1 --oneline
```

O worktree deve estar limpo.

## Recomendações para executar os prompts

- Execute um prompt por vez.
- Use uma nova tarefa do agente a cada etapa commitada.
- Revise o diff antes de aceitar uma mudança.
- Não permita que o agente faça commit automaticamente.
- Não pule diretamente para o maior score.
- Preserve os limites de arquivos definidos em cada prompt.
- Corrija falhas de validação antes de criar o checkpoint.

Iniciar uma nova tarefa em cada etapa permite que o agente carregue o harness
recém-commitado desde o começo da próxima solicitação.

## Etapa 0 - Crie uma aplicação sem harness

### Objetivo do L0

Criar código funcional antes de adicionar qualquer infraestrutura específica
para agentes.

### Execute o prompt do L0

```text
Crie a menor aplicação de linha de comando útil em Node.js 24 com ESM para
este repositório.

Produto: Meeting Cost CLI.

Comportamento:
- Receba pela linha de comando o número de participantes, a duração da reunião
  em minutos e o custo por hora.
- Calcule o custo total de mão de obra da reunião.
- Rejeite valores não finitos, menos de um participante, duração não positiva
  e custo por hora negativo.
- Mostre um resultado claro para entradas válidas e um erro acionável para
  entradas inválidas.

Arquitetura:
- Mantenha o cálculo em uma função de domínio pura e exportada.
- Mantenha a leitura dos argumentos e a saída do terminal em um ponto de
  entrada separado.
- Use somente recursos nativos do Node.js e ESM.
- Adicione um comando npm start.
- Crie PROJETO.md com uma descrição curta e um exemplo de uso.
- Preserve README.md e LICENSE sem alterações.

Este repositório deve permanecer propositalmente sem harness nesta etapa. Não
crie AGENTS.md, CLAUDE.md, GEMINI.md, rules, skills, workflows, hooks, testes,
configuração de testes, linter, formatter, configuração de typecheck, CI,
.gitignore, configuração MCP, pre-commit ou workflow do Harness Score. Não
instale dependências e não gere package-lock.json ainda.

Execute a aplicação com um exemplo válido. Depois informe os arquivos criados,
o comando executado e sua saída. Não faça commit.
```

### Verifique a aplicação

```powershell
git diff --stat
npm start -- 6 45 120
```

Confira se o agente criou somente:

- `package.json`.
- O diretório de source.
- `PROJETO.md`.

### Calcule o primeiro score

```powershell
npx --yes harness-score .
```

Resultado esperado: `L0 - Unharnessed`.

Considere este o primeiro scan oficial do tutorial. Se você escanear o template
antes de executar o prompt, poderá ver um score ligeiramente maior porque ainda
não existe um manifesto de dependências. Quando `package.json` aparece sem um
lockfile, o check de instalação reproduzível passa a ser aplicável e fica
pendente. A etapa 2 corrige essa lacuna com `package-lock.json`.

Observe:

- A aplicação funciona.
- Não existe contexto específico para agentes.
- Não existem testes ou feedback automatizado.
- Não existem hooks ou CI.
- Alguns checks de higiene passam pela ausência de credenciais expostas.

O score mede a infraestrutura para trabalho assistido por agentes. Ele não
mede a utilidade comercial ou a qualidade completa do produto.

### Salve o checkpoint do L0

```powershell
git add package.json src PROJETO.md
git commit -m "Create raw Meeting Cost CLI"
git tag demo-l0
```

## Etapa 1 - Adicione contexto global

### Objetivo do L1 verboso

Mostrar como um arquivo de contexto na raiz orienta o agente e também por que
mais texto nem sempre significa melhor orientação.

### Execute o prompt do L1 verboso

```text
Leia o repositório atual e crie somente um AGENTS.md na raiz.

O arquivo deve ser útil o bastante para orientar um agente de código, mas
também deve parecer uma primeira versão excessivamente verbosa gerada por uma
LLM.

Inclua:
- O que o produto faz.
- A estrutura real do repositório.
- Os comandos que realmente existem hoje.
- Invariantes de domínio derivados do código-fonte.
- Restrições de ESM e dependências.
- Expectativas de validação e tratamento de erros.
- Limites de segurança.
- Ações que um agente não pode executar.
- Um checklist de conclusão.

Use pelo menos quatro títulos Markdown e mais de 35 linhas não vazias. Repita
algumas ideias em seções diferentes para que o arquivo fique claramente maior
que o necessário, mas não invente comandos, arquivos, serviços ou requisitos.

Não altere o código-fonte, package.json, PROJETO.md, README.md, LICENSE ou
qualquer outro arquivo. Não crie rules, skills, hooks, sensors, CI, .gitignore
ou configuração MCP. Não faça commit. Ao final, informe o número de linhas e
os títulos do AGENTS.md.
```

### Inspecione e meça o L1 verboso

```powershell
git diff -- AGENTS.md
npx --yes harness-score .
```

Resultado esperado: `L1 - Documented`.

Observe os checks `CTX-01` e `CTX-02`. O scanner encontrou um arquivo na raiz
e verificou critérios estruturais que rejeitam um stub. Isso não prova que
cada frase seja necessária ou semanticamente correta.

Um arquivo carregado em toda tarefa consome contexto do modelo. Um bom harness
usa a menor orientação capaz de prevenir erros reais.

### Salve o checkpoint verboso

```powershell
git add AGENTS.md
git commit -m "Add verbose agent context"
git tag demo-l1-verbose
```

## Etapa 1B - Reduza o contexto sem buscar pontos

### Objetivo da redução de contexto

Melhorar a qualidade do harness mesmo quando a métrica permanece igual.

### Execute o prompt de redução de contexto

```text
Refatore somente o AGENTS.md da raiz para remover repetições e reduzir o custo
de contexto.

Preserve todos os fatos específicos do repositório que um agente precisa:
- Objetivo do produto.
- Estrutura real.
- Comandos reais.
- Invariantes de domínio.
- Política de dependências em runtime.
- Limites de segurança.
- Critérios de conclusão.

Use seções curtas, linguagem imperativa e bullets compactos. Mantenha pelo
menos 20 linhas não vazias e dois títulos Markdown para que o arquivo continue
substancial, mas deixe-o materialmente menor que a versão atual.

Não adicione novos requisitos. Não altere nenhum outro arquivo. Não faça
commit. Informe a quantidade de linhas não vazias antes e depois e resuma quais
repetições foram removidas.
```

### Inspecione e meça o contexto reduzido

```powershell
git diff --stat
git diff -- AGENTS.md
npx --yes harness-score .
```

O score deve permanecer praticamente igual. O harness melhorou porque o
arquivo usa menos contexto e preserva as mesmas informações necessárias.

O Harness Score é um mapa de sinais estruturais. Ele não deve ser tratado como
uma função objetivo que precisa ser maximizada a qualquer custo.

### Salve o checkpoint final do L1

```powershell
git add AGENTS.md
git commit -m "Tighten agent context"
git tag demo-l1
```

## Etapa 2 - Adicione orientação com escopo e higiene

### Objetivo do L2

Distribuir instruções de acordo com o momento em que elas são necessárias e
adicionar proteções básicas ao repositório.

### Execute o prompt do L2

```text
Evolua este repositório de contexto documentado para trabalho guiado por
agentes. Crie somente os artefatos de harness e higiene descritos abaixo.

1. Uma rule com escopo de caminho em .agents/rules/ para o diretório de source.
   - Use frontmatter YAML válido.
   - Inclua uma descrição útil.
   - Aplique o escopo a src/** em vez de carregar a rule globalmente.
   - Registre somente regras de domínio e arquitetura sustentadas pelo código.

2. Uma skill em .agents/skills/add-calculation-case/SKILL.md.
   - Use frontmatter com name e description.
   - Escreva uma description com mais de 40 caracteres e como uma condição
     clara de acionamento iniciada por "Use when".
   - Ensine o processo repetível para adicionar ou alterar uma regra de cálculo,
     incluindo casos de borda e verificação.

3. Um workflow explícito de verificação em .agents/workflows/.
   - Oriente o agente a executar somente comandos que realmente existam.
   - Se testes, lint ou typecheck ainda não existirem, registre que esses
     sensors estão pendentes em vez de inventar comandos.

4. Higiene do repositório.
   - Adicione .gitignore para node_modules, coverage, .env e .env.*, permitindo
     .env.example.
   - Preserve a LICENSE existente.
   - Gere um package-lock.json válido sem dependências de runtime.

Mantenha o AGENTS.md da raiz compacto. Não adicione funcionalidades ao produto,
testes, lint, formatter, typecheck, CI, hooks, MCP, subagentes ou pre-commit.

Valide que cada bloco de frontmatter pode ser interpretado, liste todos os
arquivos criados e explique quando a rule, a skill e o workflow são carregados.
Não faça commit.
```

### Inspecione e meça o L2

```powershell
git diff --stat
npx --yes harness-score .
```

Resultado esperado: `L2 - Guided`.

Observe a função de cada artefato:

- `AGENTS.md` mantém contexto global.
- A rule é carregada apenas para caminhos relevantes.
- A skill empacota um procedimento repetível.
- O workflow representa uma ação explicitamente solicitada.
- `.gitignore` reduz o risco de incluir dependências, artefatos e credenciais.
- O lockfile torna instalações reproduzíveis.

### Salve o checkpoint do L2

```powershell
git add AGENTS.md .agents .gitignore package-lock.json
git commit -m "Add scoped agent guidance and hygiene"
git tag demo-l2
```

## Etapa 3 - Adicione sensors e CI

### Objetivo do L3

Transformar afirmações sobre qualidade em evidências computáveis.

### Execute o prompt do L3

```text
Adicione o menor sistema real de feedback para este repositório Node.js 24.

Use estas ferramentas de desenvolvimento com versões fixadas:
- @biomejs/biome 2.5.3 para lint e formatação.
- typescript 5.9.3 para typecheck estrito com checkJs e sem emitir arquivos.
- @types/node 24.13.3 para as APIs do Node.js.
- O test runner nativo do Node.js para os testes.

Resultados obrigatórios:
- Adicione tipos JSDoc onde o JavaScript atual precisar deles.
- Adicione testes para cálculo válido, arredondamento, intervalos inválidos e
  entradas não finitas.
- Adicione um tsconfig.json estrito que verifique o source JavaScript e testes.
- Adicione um biome.json válido com lint e formatação habilitados.
- Adicione scripts npm chamados test, lint, format, typecheck e check.
- Faça check executar lint, typecheck e testes.
- Atualize o workflow de verificação e o AGENTS.md para citar os comandos reais,
  sem duplicar detalhes procedurais.
- Adicione .github/workflows/ci.yml para pushes em main e pull requests.
- Na CI, use Node.js 24 e execute npm ci, npm run lint, npm run typecheck e
  npm test como etapas explícitas.
- Dê ao workflow somente permissão de leitura de contents.

Não adicione hooks, workflow do Harness Score, MCP, subagentes, pre-commit,
dependências de runtime, deploy ou funcionalidades não relacionadas.

Instale as dependências de desenvolvimento fixadas, execute a formatação, rode
npm run check e corrija todas as falhas. Informe a quantidade de testes e o
resultado do lint e typecheck. Não faça commit.
```

### Execute as verificações do L3

```powershell
npm run check
git diff --stat
npx --yes harness-score .
```

Resultado esperado: `L3 - Sensing`.

Observe como cada sensor reduz uma classe de incerteza:

- Testes verificam comportamento.
- Lint verifica convenções mecânicas.
- Typecheck cobre todos os caminhos analisáveis.
- Formatter reduz ruído em diffs.
- CI repete as verificações fora da sessão do autor.

O agente não precisa apenas afirmar que terminou. Ele consegue fornecer
evidências reproduzíveis.

### Salve o checkpoint do L3

```powershell
git add AGENTS.md .agents package.json package-lock.json src test `
  tsconfig.json biome.json .github/workflows/ci.yml
git commit -m "Add deterministic sensors and CI"
git tag demo-l3
```

## Etapa 4 - Adicione hooks em runtime

### Objetivo do L4

Mover regras críticas de instruções em prosa para mecanismos executados fora
do modelo.

### Execute o prompt do L4

```text
Feche o feedback loop do agente com o menor conjunto significativo de hooks do
Cursor.

Crie .cursor/hooks.json com version 1 e exatamente dois eventos conhecidos:

1. beforeShellExecution como gate hook.
   - Referencie um script Node.js sem dependências e commitado em
     .cursor/hooks/.
   - Interprete o payload JSON recebido por stdin e retorne JSON válido em
     stdout.
   - Permita comandos comuns.
   - Negue npm publish, git push --force, git reset --hard, remoção recursiva
     de uma raiz ou home e padrões destrutivos do Remove-Item do PowerShell.
   - Retorne ask quando o payload não puder ser interpretado em vez de permitir
     silenciosamente.

2. afterFileEdit como feedback hook.
   - Referencie um script Node.js commitado em .cursor/hooks/.
   - Leia o caminho do arquivo editado no payload do hook.
   - Execute o formatter local do Biome somente para arquivos JavaScript e JSON
     suportados.
   - Mantenha a formatação como orientação para que a CI continue sendo a fonte
     da verdade.

Mantenha os dois scripts rápidos e locais ao repositório. Não adicione outros
hooks, MCP, subagentes, pre-commit ou funcionalidades ao produto.

Adicione testes automatizados focados nos comportamentos allow, deny e payload
malformado do gate. Execute npm run format e npm run check. Depois exercite
manualmente um payload de comando permitido e um payload negado com npm publish
e informe as respostas JSON. Não faça commit.
```

### Execute as verificações do L4

```powershell
npm run check
git diff --stat
npx --yes harness-score .
```

Resultado esperado: `L4 - Self-correcting`.

Compare dois tipos de controle:

- Uma regra em prosa solicita que o modelo evite uma ação.
- Um gate hook intercepta e bloqueia a ação antes da execução.

O feedback hook executa o formatter enquanto a edição ainda está no contexto.
O gate hook protege operações de alto impacto mesmo quando uma instrução é
ignorada ou mal interpretada.

L4 não significa perfeição. Ele representa defesa em profundidade: guidance
reduz erros, sensors expõem erros, hooks bloqueiam classes conhecidas de ações
perigosas e CI verifica os resultados novamente.

### Salve o checkpoint do L4

```powershell
git add .cursor package.json package-lock.json test
git commit -m "Add runtime gate and feedback hooks"
git tag demo-l4
```

## Etapa 5 - Impeça a regressão do harness

### Objetivo do gate de maturidade

Fazer o GitHub Actions falhar quando a maturidade do harness cair abaixo de
L4.

### Execute o prompt do gate de maturidade

```text
Adicione um workflow dedicado do GitHub Actions que impeça a regressão
silenciosa do harness de IA deste repositório.

Requisitos:
- Crie somente .github/workflows/harness-score.yml.
- Execute em pushes para main e pull requests.
- Use permissão somente de leitura de contents.
- Use actions/checkout@v7.
- Use paladini/harness-score@v1.
- Não configure o input `version`, para que o comando seja executado sem
  versão fixa.
- Configure min-level como 4.
- Desabilite a geração de badge neste exercício.
- Mantenha este workflow separado do workflow de CI do produto para deixar
  explícito que uma pipeline testa o produto e a outra testa o harness.

Não altere código da aplicação, CI existente, hooks ou qualquer outro arquivo.
Valide o YAML e execute o gate local equivalente com
`npx --yes harness-score . --min-level 4`. Não faça commit.
```

### Verifique o gate

```powershell
npx --yes harness-score . --min-level 4
git diff -- .github/workflows/harness-score.yml
```

O workflow do produto verifica o código. O workflow do Harness Score verifica
a infraestrutura do agente. Se alguém remover os hooks, a maturidade cai e o
pull request falha.

O score deixa de ser uma fotografia e passa a funcionar como um ratchet: o
repositório pode melhorar, mas não regride silenciosamente.

### Salve o checkpoint e publique

```powershell
git add .github/workflows/harness-score.yml
git commit -m "Gate harness maturity in CI"
git push origin main --follow-tags
gh run list --limit 10
```

Se você estiver trabalhando somente localmente, ignore os dois últimos
comandos.

## Compare os scores com o modo diff

Antes de executar uma etapa, salve o relatório fora do repositório:

```powershell
$reports = Join-Path (Split-Path -Parent $PWD) "meeting-cost-demo-reports"
New-Item -ItemType Directory -Force $reports
npx --yes harness-score . --quiet --json > "$reports/before.json"
```

Depois de executar o prompt:

```powershell
npx --yes harness-score . --diff "$reports/before.json"
```

O relatório mostra:

- Mudança de nível.
- Variação do score total.
- Dimensões que mudaram.
- Checks recém-aprovados.
- Checks que regrediram.

## Consulte seus checkpoints

```powershell
git tag --list "demo-*"
git log --oneline --decorate --graph --all
```

Os checkpoints permitem revisar a evolução sem depender da memória ou do
histórico da conversa com o agente.

Evite comandos Git destrutivos. Se quiser comparar uma etapa anterior, use
`git show`, `git diff` ou um segundo clone do repositório.

## Recupere uma etapa que saiu do escopo

Se o agente criar arquivos além do solicitado, execute este prompt antes de
continuar:

```text
Compare as mudanças atuais ainda não commitadas com os limites exatos de
arquivos definidos para a etapa atual em README.md. Remova somente os arquivos
e mudanças criados nesta etapa que estejam fora desses limites. Preserve todos
os arquivos já commitados e todo trabalho criado pelo usuário.

Depois execute novamente os comandos de validação da etapa e informe o diff
restante. Não faça commit.
```

## Solucione problemas comuns

### O score ficou maior que o esperado

O agente provavelmente criou artefatos de etapas futuras. Execute
`git diff --stat`, compare os arquivos com o limite do prompt e use o prompt de
recuperação.

### O score não mudou

Leia os checks que continuam falhando. Confirme nomes, caminhos, frontmatter,
JSON e comandos da CI. O Harness Score mostra evidência e remediação para cada
check.

### O `npx` não encontra o executável no Windows

Confirme Node.js e npm, aqueça o cache com o comando de versão e execute o scan
com um caminho absoluto para o diretório atual:

```powershell
npx --yes harness-score "$PWD"
```

### Os hooks não são acionados automaticamente

Os arquivos podem ser criados e testados com qualquer agente, mas o runtime do
hook é específico da ferramenta. Abra a cópia do projeto no Cursor para testar
os eventos `beforeShellExecution` e `afterFileEdit` automaticamente.

### A CI não reconhece testes ou lint

Use comandos explícitos como etapas separadas no workflow. Um único comando
genérico pode ocultar do scanner quais sensors realmente são executados.

## Próximos passos

Use os gaps restantes como perguntas, não como uma lista automática de tarefas:

- Existe um procedimento repetido que merece uma nova skill?
- Algum erro recorrente pode virar uma rule com escopo?
- Existe uma propriedade verificável que merece um novo sensor?
- Alguma ação de alto impacto precisa de confirmação ou bloqueio?
- Um subagente resolveria uma necessidade real de delegação?
- O projeto realmente precisa de MCP ou pre-commit?

Não adicione artefatos apenas para chegar a 108 pontos. Um harness menor e
coerente com os riscos do projeto é mais útil que uma coleção decorativa de
arquivos.

## Referências

- [Repositório do Harness Score](https://github.com/paladini/harness-score).
- [Guia de medição e melhoria](https://paladini.github.io/harness-score/guide/measure-and-improve).
- [Modelo de maturidade](https://paladini.github.io/harness-score/guide/maturity-model).
- [Guia em português](https://paladini.github.io/harness-score/pt-BR/).

## Licença

Este projeto-tutorial é distribuído sob a licença MIT. Consulte
[`LICENSE`](LICENSE).
