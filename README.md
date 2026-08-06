# Previsão de Puxada de Postes (D+30 / D+60)

Aplicação web para prever a quantidade de puxada de postes junto a fornecedores, com horizonte de
30 e 60 dias. Todo o processamento — leitura da planilha, treinamento do modelo e geração da
previsão — roda **inteiramente no navegador de quem usa**. Nenhum dado é enviado para nenhum
servidor: é só um arquivo HTML.

## O que a aplicação faz

- Treina um modelo de árvore de decisão (Gradient Boosting) usando o histórico de estoque, demanda
  e puxadas, com variáveis explicativas escolhidas pelo usuário e hiperparâmetros ajustáveis.
- Compara o resultado com um modelo mais simples de suavização exponencial (Holt), para validar se
  a árvore realmente agrega valor.
- Segrega a previsão por fornecedor (conforme o percentual de contrato de cada um) e por classe de
  SKU.
- Aponta os 10 combos de empresa/regional/SKU/fornecedor com maior erro de previsão no histórico,
  para revisão manual.
- Exporta tudo em Excel (.xlsx) e CSV.

## Como hospedar no GitHub Pages

1. Crie um repositório novo no GitHub (pode ser privado, se sua organização tiver GitHub
   Enterprise/Team com Pages privado; num plano gratuito, o Pages só funciona com repositório
   público — veja a nota de privacidade abaixo).
2. Suba os arquivos deste pacote (`index.html` e este `README.md`) para a raiz do repositório.
3. Vá em **Settings → Pages**, em "Source" escolha a branch `main` e a pasta `/ (root)`, e salve.
4. Em 1–2 minutos o GitHub mostra o link (algo como
   `https://SEU-USUARIO.github.io/NOME-DO-REPOSITORIO/`). É esse link que você compartilha com os
   outros usuários.
5. Qualquer atualização futura: basta subir um novo `index.html` para o mesmo repositório
   (substituindo o arquivo) — o link continua o mesmo.

### Nota sobre privacidade

O código do app (a página em si) fica visível a quem acessa o link — e, se o repositório for
público, visível a qualquer pessoa no GitHub. Isso não expõe dados de ninguém, porque a aplicação
não guarda nem transmite as planilhas que os usuários sobem — cada pessoa processa o próprio
arquivo, localmente, no navegador dela. Mas se a sua empresa não quiser nem o *código* público,
duas opções:
- usar GitHub Pages privado (disponível em planos GitHub Team/Enterprise), ou
- não hospedar via Pages e só distribuir o arquivo `index.html` diretamente (por e-mail, Teams,
  SharePoint etc.) — quem receber abre com dois cliques, sem precisar de internet nem instalar nada.

## Como usar

1. Abra o link (ou o arquivo `index.html` direto no navegador).
2. Suba o Excel com a base atualizada (mesma query de sempre). As abas obrigatórias são
   `f_stockout` e `f_relatorio_cartas`; `f_contratos` e `d_class_poste` são opcionais — se não
   estiverem no arquivo, as seções de fornecedor e classe simplesmente não aparecem.
3. Escolha as variáveis explicativas e, se quiser, ajuste os hiperparâmetros (ou use os padrões já
   validados).
4. Clique em "Treinar modelo e gerar previsão". O treino roda no navegador e leva cerca de 1 a 2
   minutos.
5. Veja os resultados na tela ou baixe o Excel/CSV completo.

## Estrutura de dados esperada

As abas devem ter exatamente esses nomes e essas colunas (a ordem das colunas não importa, mas o
nome de cada uma precisa bater; letras maiúsculas/minúsculas e acentos importam).

### `f_stockout` (obrigatória)
data_ref, cod_empresa, cod_regional, Codigo Material, Descrição_do_Material, Consumo_M0,
Demanda_M0, Demanda_M1, Demanda_M2, Demanda_M3, CMM_3, CMM_6, CMM_12, Estoque_N3,
Compra_em_aberto

Uma linha por combinação de empresa/regional/SKU, por mês (snapshot de fim de mês).

### `f_relatorio_cartas` (obrigatória)
data_ref, cod_empresa, cod_regional, cod_mat, desc_mat, qtd_puxada

Histórico real de puxada por empresa/regional/SKU/mês — é o que o modelo aprende a prever.

### `f_contratos` (opcional — ativa a segregação por fornecedor)
cod_empresa, cod_regional_contrato_compra, codmat, dscmat, codcdr, nomecdr,
porcentagem_contrato.1

Uma linha por fornecedor de cada combo empresa/regional/SKU, com o percentual de contrato dele
(0 a 1). Se a soma dos percentuais de um combo for menor que 1, a diferença é sinalizada como
"não contratada" na previsão, sem alterar o total.

### `d_class_poste` (opcional — ativa a segregação por classe)
codmat, dscmat, Classificação

Uma linha por SKU, com a classe (ex.: Circular, Tipo B, Especiais).

## Limitações

- O treino roda no navegador da pessoa que está usando — em bases muito maiores que a atual
  (dezenas de milhares de linhas), o tempo de treino pode aumentar bastante.
- É um modelo estatístico sobre o histórico disponível; não substitui o julgamento de quem
  conhece o contexto de cada fornecedor e regional.
- Testado nos navegadores modernos baseados em Chromium (Chrome, Edge) e Firefox.
