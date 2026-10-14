# Figo × UFC — Ideias, orçamento e tempo

Plataforma da equipe para organizar ideias de campanha, acompanhar orçamento e carga por fase, cadastrar públicos-alvo e consultar o SWOT/PESTEL do UFC.

É um arquivo único (`index.html`), sem instalação e sem build. Funciona em computador e celular.

## Dois modos de uso

| Modo | Como ativar | O que acontece |
| --- | --- | --- |
| **Local** (padrão) | Não precisa fazer nada | Os dados ficam só no navegador de cada pessoa. Serve para testar. |
| **Equipe, em tempo real** | Colar a URL de um Firebase Realtime Database em `CONFIG` | Todo mundo vê e edita os mesmos dados, e as mudanças aparecem na hora, sem recarregar. |

---

## 1. Colocar o site no ar com o GitHub Pages

Tudo pelo navegador, sem terminal.

1. Entre em [github.com](https://github.com) (ou crie uma conta gratuita).
2. Clique em **+** (canto superior direito) → **New repository**. Dê um nome, por exemplo `figo-ufc`, deixe **Public** e clique em **Create repository**.
3. Na página do repositório, clique em **uploading an existing file**, arraste o `index.html` (e este `README.md`) e clique em **Commit changes**. O `index.html` precisa ficar na raiz do repositório.
4. Vá em **Settings → Pages**. Em **Build and deployment**, escolha **Deploy from a branch**, selecione a branch **main** e a pasta **/ (root)**, e clique em **Save**.
5. Espere cerca de 1 a 2 minutos. O endereço do site aparece na mesma tela e tem este formato:
   `https://SEU-USUARIO.github.io/figo-ufc/`

Pronto: é esse link que você passa para a equipe. Nesse ponto ele está no modo local.

---

## 2. Ligar o compartilhamento em tempo real (Firebase)

O GitHub Pages só hospeda arquivos; ele não guarda dados. Para a equipe enxergar as mesmas informações, o app usa o **Firebase Realtime Database**, que tem plano gratuito.

1. Acesse o [console do Firebase](https://console.firebase.google.com) com uma conta Google e clique em **Add project** (adicionar projeto). Dê um nome e conclua; o Google Analytics pode ficar desativado.
2. No menu lateral, abra **Build → Realtime Database** e clique em **Create Database**. Escolha a localização mais próxima e, nas regras de segurança, escolha **Start in locked mode** (modo bloqueado).
3. Abra a aba **Rules**, apague o conteúdo e cole as regras abaixo. Clique em **Publish**.

   ```json
   {
     "rules": {
       "figoUfc": {
         ".read": true,
         ".write": true
       }
     }
   }
   ```

   Por que não usar o "test mode"? Ele libera tudo por apenas 30 dias e depois bloqueia o banco, o que derrubaria o app no meio do projeto.

4. Na aba **Data**, copie a URL do banco, que aparece no topo. Ela parece com `https://seu-projeto-default-rtdb.firebaseio.com` (em algumas regiões termina em `firebasedatabase.app`).
5. No GitHub, abra o `index.html`, clique no ícone de lápis (**Edit**), procure este trecho perto do início do código e cole a URL entre as aspas:

   ```js
   firebaseDatabaseUrl: "",
   ```

   Fica assim:

   ```js
   firebaseDatabaseUrl: "https://seu-projeto-default-rtdb.firebaseio.com",
   ```

   Clique em **Commit changes**.
6. Espere 1 a 2 minutos, recarregue o site e olhe o rodapé do menu lateral. Deve aparecer **"Ao vivo — mudanças aparecem para toda a equipe"**. Abra o link em dois navegadores para ver uma mudança aparecer no outro sem recarregar.

Na primeira vez, o app grava no banco o SWOT e o PESTEL do UFC; depois disso eles passam a ser editáveis por todos.

### Importante sobre acesso e privacidade

- **O site é público.** Mesmo com o repositório privado, o GitHub Pages publica a página para quem tiver o link. Como a URL do banco fica dentro do `index.html`, **qualquer pessoa que abrir o site consegue ler e editar os dados**.
- Para um trabalho de faculdade com dados fictícios isso costuma ser aceitável. **Não coloque informações pessoais ou sensíveis** (dados reais de pessoas, contratos, etc.).
- Se precisar de acesso restrito (login da equipe), isso exige autenticação no Firebase e não está incluído nesta versão.
- Se quiser, troque `basePath` (no `CONFIG`) por um nome difícil de adivinhar, como `figoUfc-k8x2q`, e use o mesmo nome nas regras do passo 3. Isso dificulta o acesso por acaso, mas não é uma proteção de verdade.

---

## 3. Dia a dia

- **Backup:** o botão **Baixar backup (JSON)**, no rodapé do menu, salva uma cópia de todos os dados. Vale baixar antes de entregas importantes.
- **Conflitos:** cada ideia, público e grupo do SWOT/PESTEL é gravado separadamente. Se duas pessoas editarem itens diferentes ao mesmo tempo, nada se perde. Se editarem exatamente o mesmo item, vale a última gravação.
- **Quem está digitando não é interrompido:** atualizações de outras pessoas esperam o campo perder o foco.

### Atalhos e navegação

- **Celular:** o menu fica atrás do botão com três linhas (hambúrguer), no canto superior esquerdo. Ele fecha ao escolher uma tela, ao tocar fora dele ou ao apertar Esc.
- **Enter** envia os formulários de ideia, de público e de itens do SWOT/PESTEL. **Esc** fecha formulários que ainda não têm nada digitado.
- **Excluir** é imediato, mas aparece um aviso com **Desfazer** por 9 segundos.
- O custo aceita `12000`, `12.000` ou `12.000,50`.
- A análise de usabilidade (heurísticas de Nielsen e navegação) está em `ANALISE_UX.md`.

## 4. Importar públicos-alvo de um arquivo

Na tela **Públicos-alvo** há uma área de upload no topo. Arraste um arquivo para ela ou clique em **escolha no computador**.

- **Formatos aceitos:** PDF, Word (.docx), TXT, Markdown, JSON e CSV.
- **Como o app lê o texto:** ele procura títulos como `Público-alvo: nome do público` para separar os públicos e itens como `Faixa etária: …`, `Interesses: …`, `Oportunidade`, `Objetivo estratégico` e `Ponto de atenção` para preencher os campos. Todos os campos do formulário de público-alvo são reconhecidos pelo nome. O que não tem campo correspondente vai para **Outras informações**.
- **Prévia antes de importar:** o app mostra os públicos encontrados e o que foi reconhecido em cada um. Você marca quais importar e depois pode editar cada público normalmente. Públicos com o mesmo nome de um já cadastrado vêm desmarcados.
- **CSV:** pode ter uma linha por público (colunas com o nome dos campos) ou duas colunas `Critério` e `Preenchimento` para um único público.
- **JSON:** aceita o backup baixado pelo próprio app (só os públicos são importados) ou uma lista de públicos.
- **PDF e Word precisam de internet:** o app carrega os leitores desses formatos da internet na primeira vez que você usa. PDFs escaneados (só imagem) não têm texto para ler.

## 5. Mudar valores fixos

No `index.html`, procure no início do código JavaScript:

- `TOTAL_BUDGET = 70000;` — orçamento total (hoje US$ 70.000).
- `CATEGORIES` — categorias de orçamento e a meta de cada uma em `pct` (% do total).
- `PHASES` — fases de implementação.

Para editar, use o ícone de lápis no GitHub e faça **Commit changes**. O site atualiza em 1 a 2 minutos.

## 6. Problemas comuns

| Sintoma | O que verificar |
| --- | --- |
| "Não foi possível conectar ao banco de dados da equipe" | A URL em `firebaseDatabaseUrl` está correta, sem espaços e sem barra no fim? As regras foram publicadas? |
| Fica em "Conectando…" ou "Sem conexão" | Internet, bloqueadores de anúncio/privacidade que barrem `firebaseio.com`, e se o nome em `basePath` é o mesmo das regras. |
| Cada pessoa vê dados diferentes | O site está no modo local: `firebaseDatabaseUrl` ainda está vazio no arquivo publicado. |
| A mudança no GitHub não apareceu | O GitHub Pages leva 1 a 2 minutos; recarregue com Ctrl+F5. |
