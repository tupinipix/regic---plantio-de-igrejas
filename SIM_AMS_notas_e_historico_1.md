# SIM — Sistema de Inteligência Missionária (AMS)
### Notas técnicas, histórico de alterações e pendências futuras

> Arquivo de arquivamento gerado a partir das sessões de desenvolvimento do app `SIM_AMS.html`. Serve como registro do que já foi feito e como ponto de partida para os próximos ajustes — não precisa ser lido do início ao fim; use o índice abaixo para ir direto ao que interessa.

---

## 1. O que é o app

O SIM é um aplicativo web de página única (SPA — *single page application*), pensado para uso da liderança da **Associação Mineira Sul (AMS)** no planejamento estratégico da missão. Ele cruza dois tipos de dado:

- **Base eclesiástica da AMS** — planilha fornecida com 287 registros (152 igrejas, 135 grupos/PGs), distribuídos em 44 distritos e 6 regiões pastorais.
- **REGIC 2018 (IBGE)** — o estudo *Regiões de Influência das Cidades*, que classifica a hierarquia urbana e a rede de influência entre os 853 municípios de Minas Gerais.

A partir desse cruzamento, o app gera indicadores de cobertura territorial, prioriza municípios sem presença adventista, simula cenários de ação missionária e disponibiliza um assistente de perguntas e respostas — tudo rodando localmente no navegador, sem enviar dados para fora, exceto a consulta de população em tempo real à API pública do IBGE.

### Telas do app
Dashboard (Visão Geral, Mapa, Vazios e Prioridades, Polos e Irradiação, Grupos em Expansão, Simulador Estratégico) · Inteligência territorial (Pesquisar Município, Índices Estratégicos, Qualidade dos Dados) · Apoio à decisão (Assistente Missionário, Sobre o SIM).

---

## 2. Arquitetura técnica

```
sim-ams/
├── template.html          # casca HTML + todo o CSS (design system) + placeholders __X__
├── app.js                 # toda a lógica do app (~2.100 linhas) — router, telas, dados
├── churches.json          # 287 registros eclesiásticos (igrejas + grupos)
├── municipios_master.json # os 853 municípios de MG + REGIC + status + prioridade
├── polos.json             # municípios com presença, classificados por tipologia de polo
├── regioes.json / distritos.json  # agregados por região pastoral / distrito
├── summary.json           # números agregados usados no Dashboard
├── regic_mg.json          # REGIC bruto (fonte para municipios_master.json)
├── build_data.py          # lê os .xlsx originais → churches.json + regic_mg.json
├── build_master2.py       # cruza churches.json + regic_mg.json → municipios_master.json + summary.json
├── build_polos.py         # gera polos.json, regioes.json, distritos.json
├── add_regic_dependents.py # calcula municípios dependentes/satélites por polo (roda depois do build_master2.py)
├── build_final.py         # NOVO (sessão 7) — monta o SIM_AMS.html a partir de template.html + app.js + os 6 JSONs
└── SIM_AMS.html            # ARQUIVO FINAL — template.html com os placeholders substituídos
```

**Ordem de build** (sempre que `app.js`, `template.html` ou algum `.json` mudar):
```
build_master2.py → add_regic_dependents.py → build_polos.py → build_final.py
```
> ⚠️ Rodar `build_master2.py` sozinho e esquecer o `add_regic_dependents.py` **apaga** os campos `regic_dependentes_total`, `regic_dependentes_sem_presenca` e `regic_dependentes_lista` de `municipios_master.json` (ver seção 4, item de 20/ago). Sempre rodar os três em sequência, e só então `build_final.py`.

O arquivo final `SIM_AMS.html` é gerado por `build_final.py` (script Python, sessão 7 — antes esse passo era feito manualmente/ad hoc; agora está formalizado e versionado) que lê `template.html` e substitui os marcadores `__SUMMARY_JSON__`, `__MUNICIPIOS_JSON__`, `__CHURCHES_JSON__`, `__POLOS_JSON__`, `__REGIOES_JSON__`, `__DISTRITOS_JSON__` e `__APP_JS__` pelo conteúdo real — o resultado é um único HTML autossuficiente (~910 KB), sem dependências externas além de Leaflet e Chart.js via CDN (com fallback em cascata cdnjs → jsDelivr → unpkg).

### Números atuais da base (após as correções desta sessão)
- 853 municípios de MG · 307 no escopo de influência da AMS · 166 com presença adventista (141 vazios no escopo).
- 24.608 membros na base eclesiástica (planilha de geolocalização) · 152 igrejas · 135 grupos/PGs.
- **Números oficiais da Secretaria AMS** (Relatório "Inteligência Territorial para a Missão", Comissão Diretiva Plenária 2026), mostrados lado a lado com o número acima desde a sessão 7, nunca fundidos num só: **24.607 membros ativos** e **15.715 membros frequentes**.
- Prioridades: A = 4 · B = 4 · C = 1 · D = 132.

---

## 3. Histórico de alterações — o que já foi feito

### Sessão 1 — correções de interface e Simulador
- **Bug do Assistente Missionário**: o chat não reagia a cliques — corrigido.
- **Sobre o SIM**: citações/referências deixaram de aparecer em destaque (agora dentro de `<details>` recolhido por padrão — "Referências metodológicas").
- **Campo de pergunta do Assistente**: reposicionado para logo abaixo da saudação; respostas passaram a aparecer abaixo do campo (antes da correção, layout empurrava tudo para baixo).
- **Simulador Estratégico**: resultado da simulação passou a aparecer **abaixo** do formulário (não mais lado a lado) + botão de **exportar/imprimir em PDF**.
- **Sugestões por tipo de ação**: cada uma das opções de "tipo de ação" passou a gerar sugestões de **curto, médio e longo prazo** distintas (antes, o resultado era genérico e igual para todas as opções), fundamentadas no conteúdo de "Sobre o SIM" (Ciclo de Plantio de Hesselgrave, Missão em diferentes níveis, Referências Metodológicas). Cada bloco termina com o aviso de que são **apenas sugestões**, ajustáveis pela liderança da AMS.

### Sessão 2 — integração com a API do IBGE
- Implementada consulta **em tempo real** à população de cada município via `servicodados.ibge.gov.br`, com duas vias em cascata: **API de Pesquisas** (tentada primeiro) → **API de Dados Agregados/SIDRA** (agregado 6579, variável 9324) como reserva confiável.
- **Atualização automática**: a cada abertura do app, os dados de população dos ~307 municípios do escopo são buscados em segundo plano (sem exigir clique do usuário), usada no indicador "Habitantes por adventista" do Dashboard e no Perfil do Município / Simulador.
- Erros de rede são **classificados e mostrados** ao usuário (timeout, CORS/rede, HTTP, formato inesperado) — nunca escondidos nem substituídos por um número antigo.
- **Decisão registrada**: foi recusado o uso de scripts Python/`curl` para contornar as restrições de busca de URL da plataforma, mesmo a pedido explícito — a integração foi feita pelas vias de navegador padrão (`fetch`), que é o comportamento correto e sustentável a longo prazo.

### Sessão 3 — referências de Ellen G. White
A partir de 4 PDFs (*Serviço Cristão*, *Ministério para as Cidades* — digitalizado, exigiu OCR em português —, *Evangelismo*, *Beneficência Social*):
- Nova linha em **Referências Metodológicas** (Sobre o SIM) para Ellen G. White — princípio-chave: **"Método de Jesus Cristo"** (relacionar → servir → conquistar confiança → convidar).
- Simulador — **"tipo de ação"**: adicionada **"Formação de centros de influência adventista"**; renomeada **"Plantio de congregação"** → **"Formação do núcleo base para plantio de Igreja"**.
- Sugestões de curto/médio/longo prazo **ampliadas em todas as 7 opções** de tipo de ação, com conteúdo extraído dos livros (Método de Cristo, "igreja colmeia", trabalho pessoal/casa em casa, Movimento Dorcas, missionários de sustento próprio, missão urbana como escola de obreiros).

### Sessão 4 — correção de bug crítico de rede (IBGE)
- **Sintoma**: em certos ambientes de pré-visualização (iframe em sandbox que retransmite `fetch` via `postMessage` entre janelas), as chamadas ao IBGE falhavam com `DataCloneError: ... AbortSignal object could not be cloned`, disfarçado de erro "formato" do IBGE.
- **Causa**: o app usava `AbortController`/`AbortSignal` para timeout, e esse objeto não pode ser clonado por `postMessage`.
- **Correção**: timeout reimplementado com `Promise.race` (cronômetro correndo em paralelo à chamada), sem depender de `AbortSignal` — elimina o problema em qualquer ambiente, sem perder o comportamento de timeout.

### Sessão 5 — auditoria funcional completa
Leitura de todo o código-fonte + testes automatizados navegando por todas as 11 telas. Dois bugs reais confirmados e corrigidos:
1. **Links de "Polo de referência" quebrados**: em 44 municípios, o `polo_nome` do REGIC não corresponde a um município individual de MG (cidade de outro estado, ex. Vitória da Conquista, Brasília; ou um "Arranjo Populacional" — agrupamento de municípios). O link clicável não levava a lugar nenhum. Corrigido: só é clicável quando o polo existe de fato na base; caso contrário aparece como texto com uma legenda explicando o motivo.
2. **Total de membros subcontado em 27 pessoas** (24.581 em vez de 24.608): o cálculo de `membros_total` somava só os registros já associados a um município, excluindo silenciosamente 1 grupo (Areias, 27 membros) sem cidade geocodificada — mesmo esse registro aparecendo corretamente nas outras contagens. Corrigido em `build_master2.py` para somar todos os 287 registros.

### Sessão 6 — glossário de termos do REGIC
- Qualquer sigla/termo técnico do REGIC (Metrópole, Capital Regional, Centro Sub-Regional, Centro de Zona, Centro Local, Arranjo Populacional, Polo de referência, município satélite, Prioridade A–D, ICTA, ISP, ICHU, IPMT, IDMP, ILAP, REGIC, IBGE) agora tem sublinhado pontilhado e, ao passar o mouse (desktop) ou tocar/clicar (mobile), abre uma caixa de explicação clean/moderna (cantos arredondados, sombra leve, botão de fechar no toque).
- Aplicado nos selos de hierarquia e prioridade (toda a base do app), cabeçalhos de tabela (Vazios, Polos, Índices), KPIs do Dashboard e Perfil do Município, títulos dos 6 índices estratégicos.
- Cuidado técnico: nas tabelas com cabeçalho clicável para ordenar (Vazios, Polos), um clique no termo do glossário **não** dispara mais a reordenação por engano (`isGtermClick()` guarda os dois handlers de clique).

### Sessão 7 — leitura do relatório oficial da Secretaria, correção da razão membros×habitantes, bug real de IBGE encontrado e corrigido
Ponto de partida: os 6 arquivos já usados nas sessões anteriores foram reenviados (confirmado por `md5sum` — nenhum dado novo), mais um PDF de 37 páginas até então não lido, *"Inteligência Territorial para a Missão — A AMS e a Rede de Influência Urbana"* (Relatório Secretaria AMS, Comissão Diretiva Plenária 2026). Pedido em 3 partes: (a) depurar/tornar mais funcional o app; (b) corrigir o resultado estatístico da relação membros×habitantes por município, cruzando IBGE × base eclesiástica; (c) este arquivo.

**O que o PDF revelou (achado central desta sessão):** a Secretaria reporta oficialmente **24.607 membros ativos** e **15.715 membros frequentes** — dois números que não existiam em lugar nenhum do app, que expõe apenas um único total (24.608, somado da planilha de geolocalização). A diferença entre 24.608 e 24.607 é de 1 membro — investigada a fundo (soma bruta do `.xlsx` recontada linha a linha, sem `NaN`, sem duplicata, sem linha de rodapé) e concluída como **normal**: são dois extratos independentes, de datas-base diferentes, mantidos por fontes diferentes (planilha de geolocalização vs. Tesouraria/Secretaria). **Decisão de design**: não forçar esses números a coincidir — isso seria maquiar dado, o oposto do princípio "dados conflitantes devem ser mostrados, nunca escondidos" que já guiava o app. Em vez disso:
- Novo bloco informativo na Início (`#membrosOficialBox`) mostra os três números lado a lado, com fonte e explicação da diferença.
- A razão "Habitantes por adventista" (Início) ganhou uma **segunda leitura**, calculada sobre os 15.715 membros frequentes — mais conservadora, mais alinhada ao "impacto real" que o próprio relatório contrasta com a leitura otimista de olhar só o total de membros.
- `summary.json` ganhou os campos `membros_ativos_oficial`, `membros_frequentes_oficial`, `membros_oficial_fonte`, `membros_oficial_data_ref` (constantes, não recalculadas — ver comentário em `build_master2.py`).

**A lacuna funcional de fato corrigida (o "bug" da relação membros×habitantes):** no Perfil do Município, "Membros" e "População (IBGE)" sempre foram dois números soltos, lado a lado — em nenhum lugar do app essa razão *por município* era de fato calculada. Corrigido: novo KPI **"Habitantes por membro (neste município)"**, com 3 comportamentos deliberados:
1. Membros > 0 e população resolvida → `1 membro adventista para cada X habitantes` + `% da população`.
2. Membros = 0 (município ainda sem presença) → mensagem explícita "sem membros cadastrados — razão não se aplica", nunca uma divisão por zero disfarçada.
3. População do IBGE indisponível → mensagem explícita de indisponibilidade, nunca um traço mudo sem explicação.

**Bug real encontrado durante a auditoria (não estava nos objetivos, mas se qualifica como "corrigir e tornar mais funcional"):** `fetchPopulacaoIBGE(codmun)` guardava, ao falhar, o sentinela `'error'` (uma *string*) no cache de população. Numa consulta *seguinte* para o mesmo código — reabrir o mesmo município, ou visitar Perfil depois do Simulador — a função devolvia essa string como se fosse um resultado válido (strings não-vazias são *truthy* em JS), e quem chamava tentava ler `.valor` dela → `undefined` → tela renderizava `"—"` habitantes e **"NaN%"** da população, em vez da mensagem clara de indisponibilidade. Corrigido normalizando qualquer valor de cache que não seja uma Promise nem um objeto de resultado válido (`'valor' in cached`) para `null`. Verificado com Playwright (Chromium headless, `fetch()` mockado no nível do JS para simular sucesso/falha de forma determinística, já que este ambiente de desenvolvimento não tem acesso de rede a `servicodados.ibge.gov.br`): os 3 comportamentos acima confirmados batendo exatamente com os números esperados (ex.: Pouso Alegre, 809 membros, população mockada 143.967 → "178" e "0,56%", conferindo à mão).

**Auditoria mobile/web mais ampla:** varredura headless (Playwright/Chromium) nas 6 telas principais, em viewport desktop (1440×900) e mobile (390×844) — sem overflow horizontal em nenhuma combinação, sem `pageerror` de JavaScript. Bibliotecas externas (Leaflet, Chart.js) ficam indisponíveis neste ambiente de desenvolvimento específico por bloqueio de rede do sandbox — o fallback gracioso já existente (seção 4) foi confirmado funcionando corretamente, mostrando aviso amigável em vez de tela quebrada.

**Build**: novo `build_final.py` (ver seção 2) formaliza a montagem do `SIM_AMS.html`, que antes era um passo manual/ad hoc.

**Segundo bug real, encontrado pelo usuário logo após a entrega (mesma sessão):** o console mostrava `IBGE lote indisponível · formato · Failed to fetch · url testada: ...` — rótulo enganoso, porque "formato" deveria significar "o IBGE respondeu, mas em formato inesperado", e "Failed to fetch" é o oposto disso: nenhuma resposta chegou. Causa: `ibgeClassifyError()` decidia a categoria só com `err instanceof TypeError`, e em certos ambientes onde esta página é aberta (ex.: um iframe/preview em sandbox que relay-a o `fetch` entre janelas — mesma classe de ambiente por trás do bug de `DataCloneError` da Sessão 4) o erro que chega ao `catch` pode ser reconstruído em outro *realm* de JavaScript: `.name` e `.message` sobrevivem, mas a cadeia de protótipos não bate mais com `TypeError` local, e o `instanceof` falha silenciosamente. Corrigido acrescentando dois fallbacks — `err.name === 'TypeError'` e um teste da mensagem (`/failed to fetch|networkerror|load failed|.../i`) — para além do `instanceof`. Depois da correção, o mesmo cenário classifica corretamente como `rede-ou-cors`, e tanto o console quanto o texto "por quê o IBGE não respondeu?" na tela passam a dizer a coisa certa: falha de rede/CORS, não formato de resposta. **Importante**: mesmo com o rótulo corrigido, se o app estiver rodando num ambiente que não alcança `servicodados.ibge.gov.br` (rede corporativa restrita, sandbox sem acesso à internet, `file://` local em alguns navegadores), a população continuará indisponível — o app já lida bem com isso (mostra o aviso, nunca trava ou inventa número), mas não há como o app "resolver" uma rede que não chega ao IBGE a partir de onde ele está rodando.

**Causa raiz confirmada nesta sessão**: o usuário testou e confirmou que abre o `SIM_AMS.html` **direto do disco** (`file://...`), não por um servidor. Esse é exatamente o cenário mais comum de "100% das consultas falham identicamente com `Failed to fetch`": uma página `file://` manda `Origin: null` para o servidor, e isso costuma ser bloqueado (pelo navegador e/ou pela própria API) mesmo com internet normal e mesmo a API sendo pública — nada a ver com o computador ou a rede do usuário estarem com problema. Como é a causa mais provável e o app não tem como contornar isso a partir do JavaScript da própria página (não dá para forjar o cabeçalho `Origin`), acrescentei uma **detecção automática de `location.protocol === 'file:'`** em `ibgeDiagnosticHTML()`: quando a página está aberta assim, o diagnóstico ("por quê o IBGE não respondeu?") agora inclui uma nota específica explicando a causa e o passo a passo para resolver — servir o arquivo por um servidor local (`python3 -m http.server` na pasta + acessar via `http://localhost:8000/...`) ou hospedar em qualquer servidor web. Verificado end-to-end com Playwright abrindo o arquivo via `file://` de verdade (não simulado): o texto novo aparece corretamente no modal de diagnóstico.

**Terceiro bug real, também encontrado pelo usuário (mesma sessão) — este era o culpado de fato pelo "IBGE não responde" mesmo já usando `http://localhost:8000`:** depois de servir o arquivo por um servidor local (eliminando de vez a hipótese `file://`), o problema continuou: "Habitantes por adventista" seguia mostrando 0/307 municípios. Pedi ao usuário para clicar em "Testar este link" no diagnóstico — teste que isola se o problema é CORS/rede (o navegador nem chega a receber resposta) ou o próprio IBGE recusando o pedido. Resultado: o link, aberto direto no navegador, devolveu uma resposta HTTP **real e válida** — `{"statusCode":500,"message":"Internal server error"}` — para uma URL de lote com 27 códigos de município. Ou seja: não era CORS, não era rede — o servidor do IBGE respondeu, e respondeu com erro.

Causa raiz: a API de Dados Agregados/SIDRA do IBGE, quando recebe um pedido em lote (`localidades=N6[cod1|cod2|...|codN]`), tem um comportamento conhecido em que **um único código de município problemático em algum ponto do lote pode derrubar o lote inteiro com HTTP 500** — mesmo que os outros ~39 códigos do mesmo lote sejam perfeitamente válidos. O app, antes desta correção, tratava qualquer falha de lote (`runChunk`) como "todo o lote indisponível", descartando os dados de todos os municípios daquele lote — inclusive os que o IBGE teria respondido normalmente se pedidos sozinhos ou em outro agrupamento.

**Correção**: reescrita a função `runChunk()` dentro de `fetchPopulacaoIBGEBatch()` para, em vez de desistir do lote inteiro na primeira falha, **bissectá-lo recursivamente**: divide o lote ao meio e tenta cada metade separadamente (sequencial, não em paralelo, para não multiplicar a carga simultânea sobre uma API pública); se uma metade falhar de novo, bisecta de novo — até restar um único código, que aí sim é isolado e marcado como indisponível (só ele, não os demais). Nova função auxiliar `bisectChunk()`. Efeito prático: um único município "podre" no meio de um lote de 40 deixa de contaminar os outros 39 — o app agora recupera o máximo de dados possível e isola exatamente o município problemático, em vez de reportar 0 para um lote inteiro que só tinha 1 código ruim.

**Verificação**: teste automatizado (Playwright, Chromium headless, servido por `http://localhost:8791`) simulando com precisão o cenário relatado pelo usuário — `window.fetch` mockado para que **qualquer** lote contendo o código real de Pouso Alegre (`3152501`) devolva `HTTP 500` (a mesma mensagem de erro do IBGE real), enquanto todos os outros 306 códigos do teste respondem normalmente. Clicando no botão real "Calcular" da tela (não uma chamada interna simulada, o caminho de produção completo): resultado final **306/307 municípios recuperados**, com apenas Pouso Alegre (o código deliberadamente "podre") isolado e reportado como indisponível — confirmado tanto pelo texto do botão (`"IBGE respondeu 306/307 municípios"`) quanto pelos logs do console, que mostram a bisecção acontecendo em cascata (lote de 40 falha → bisecta em 20 → 10 → 5 → 2 → isola o `3152501` sozinho) até isolar exatamente o único código problemático.

Limite conhecido desta correção: se o próprio IBGE estiver com uma instabilidade mais ampla (não um único código ruim, mas o serviço inteiro degradado), a bisecção não pode inventar dados que o IBGE não consegue fornecer — nesse caso o app continuará mostrando os municípios afetados como indisponíveis, agora de forma mais granular (só os realmente afetados, não o lote inteiro).

### Sessão 8 — tooltips de definição nas tipologias de Polo e nos estágios de Grupo
Pedido do usuário: em "Polos e Irradiação", ao passar o cursor sobre cada tipologia ("Polo consolidado", "Polo subaproveitado", "Polo de alta pressão missionária", "Polo de apoio distrital"), mostrar uma descrição — hoje essas definições só existiam escondidas atrás do link "Metodologia →" (modal). O mesmo pedido para "Grupos em Expansão", dentro do filtro "Todos os estágios" ("Grupo avançado (60+ membros)", "Grupo em consolidação (30–59)", "Grupo em formação (<30)").

Implementado com `title` (tooltip nativo do navegador, funciona em desktop e a maioria dos navegadores mobile via toque longo) em **todo lugar onde o termo aparece como opção/rótulo**, não só no filtro pedido — para não deixar a mesma etiqueta com tooltip num lugar e sem no outro:
- Cada `<option>` dos filtros `#poloTipoFilter` e `#grpBucket` (o pedido original).
- Os cartões de KPI no topo de cada tela (`Polos e Irradiação` e `Grupos em Expansão`), que repetem os mesmos rótulos.
- O badge de tipologia/estágio em cada linha das tabelas (`poloTable` e `grpTable`).

**Ajuste pedido logo em seguida pelo usuário**: a primeira versão usava o atributo `title` nativo do navegador (tooltip do sistema operacional) — funcional, mas visualmente diferente do padrão já usado no resto do app. O usuário pediu para seguir a mesma formatação já vista em "Vazios e Prioridades" ao passar o mouse sobre "Hierarquia REGIC", "Polo de referência" e "Prioridade": ali, o app já tem um **motor de dica próprio** (glossário `.gterm`/`GLOSSARY`/`initGlossaryTooltip()`, seção 21) — um balão estilizado, que também funciona por toque/foco em telas mobile, reaproveitado em várias telas do app. Refeito para usar exatamente esse mesmo mecanismo, no mesmo padrão de outras telas (`G('chave','Rótulo')`, cabeçalho geral + termo específico em cada valor):
- Cadastradas 9 novas entradas no glossário central (`GLOSSARY`): `tipologiapolo` (conceito geral) + `polo_consolidado`, `polo_altapressao`, `polo_subaproveitado`, `polo_apoiodistrital` (uma por tipologia); `estagiogrupo` (conceito geral) + `grupo_avancado`, `grupo_consolidacao`, `grupo_formacao` (uma por estágio) — os textos de Polo reaproveitam a mesma redação já usada no modal "Metodologia →", para não haver duas versões divergentes do mesmo critério circulando no app.
- Em "Polos e Irradiação": cabeçalho "Tipologia" da tabela, cada rótulo nos cartões de KPI e o badge de tipologia em cada linha da tabela passaram a usar `G(...)`, exatamente como o cabeçalho "Prioridade" e o badge A/B/C/D já fazem em "Vazios e Prioridades".
- Em "Grupos em Expansão": mesma coisa para o cabeçalho "Estágio (proxy)", os cartões de KPI e o badge de estágio em cada linha.
- Os `<select>` de filtro (`#poloTipoFilter`, `#grpBucket`) voltaram a ter apenas texto simples nas opções, sem `title` — porque o `<select id="vazHier">` de "Vazios e Prioridades" (a referência de formatação pedida) também não tem dica em suas opções: um `<option>` nativo de navegador não pode conter HTML, então não é possível encaixar o balão estilizado ali; manter os dois filtros consistentes entre si era mais fiel ao padrão pedido do que inventar um comportamento novo só para estes dois selects.

Verificado com Playwright: ao passar o mouse sobre cada termo (cartões de KPI, cabeçalho de coluna, badge de linha), o balão `#gtermTip` mostra o título e a descrição corretos, nas duas telas, sem erros de página.

---

## 4. Decisões de design que vale lembrar

- **Nunca substituir dado ausente ou incerto silenciosamente** — princípio repetido em várias telas ("dados conflitantes ou ausentes devem ser mostrados, nunca substituídos silenciosamente"). Isso guiou, por exemplo, o tratamento de erro do IBGE (mostra diagnóstico, nunca reaproveita valor antigo), a correção do bug de membros_total (sessão 5), e — sessão 7 — a decisão de **não forçar** os 24.608 (base eclesiástica) e 24.607 (Secretaria, oficial) a coincidirem: dois extratos de fontes e datas diferentes são mostrados lado a lado, com fonte, em vez de um dos dois ser silenciosamente descartado ou "ajustado" para bater com o outro.
- **Algoritmos não substituem liderança** — todo resultado do Simulador e do Assistente vem acompanhado do aviso de que são sugestões ajustáveis, nunca uma prescrição.
- **Um único arquivo HTML autossuficiente** é o formato de entrega — decisão deliberada para facilitar o compartilhamento (basta enviar o arquivo, sem instalar nada).
- **Fallback gracioso sempre que uma biblioteca externa (Leaflet, Chart.js) não carrega** — nunca trava a tela, mostra aviso claro.

---

## 5. Pendências e possíveis ajustes futuros

Itens que já estão sinalizados no próprio app como limitação conhecida, ou que surgiram como ideia natural durante o desenvolvimento — nenhum é urgente, mas ficam registrados para quando a AMS quiser evoluir o sistema:

### Dados ainda não integrados (sinalizados nas telas "Índices" e "Qualidade dos Dados")
- **Distância real / tempo de deslocamento até o polo** (exigiria dado de malha viária ou uma API de rotas).
- **Indicadores educacionais (INEP) e de saúde (CNES/DATASUS)** por município.
- **IDH local e vulnerabilidade social** (Atlas do Desenvolvimento Humano) — ainda não pesado na fórmula do IPMT.
- **População (IBGE) como peso na fórmula de prioridade (IPMT)** — hoje já é consultada em tempo real e exibida, mas a pontuação de prioridade ainda considera só hierarquia REGIC + dependência de polo, não o tamanho populacional.
- **Geocodificação pendente de 4 registros eclesiásticos** (Areias, Pedra Dourada, Porto Firme, Seritinga) — coordenadas placeholder (0,0), sinalizados na tela "Qualidade dos Dados".
- **Dados de frequência, liderança e tempo de organização dos grupos** — a tela "Grupos em Expansão" hoje usa nº de membros como proxy de maturidade, na ausência desses dados de campo.

### Possíveis melhorias de produto (não solicitadas ainda, apenas ideias registradas)
- Completar a leitura OCR das últimas páginas de *Ministério para as Cidades* (chegou a ~128/160 páginas processadas) para eventualmente aprofundar ainda mais as referências de Ellen White nas sugestões do Simulador, se a AMS quiser.
- Estender o glossário de termos a outras áreas do app onde siglas técnicas ainda aparecem sem explicação (por ex. dentro das respostas do Assistente e nos textos de "Como foi calculado?").
- No Perfil do Município, ao clicar em "Ver no mapa", hoje o app apenas navega para a tela de Mapa sem centralizar/destacar automaticamente o município selecionado — poderia ser melhorado para abrir o mapa já focado nele.
- Avaliar se cidades-polo situadas fora de Minas Gerais (as 44 exceções tratadas na correção de links quebrados) merecem, no futuro, um tratamento mais rico do que "texto explicativo sem link" — por exemplo, se a AMS decidir formalmente considerar articulação interestadual.
- **População (IBGE) como peso na fórmula de prioridade (IPMT)** segue pendente (já era um item registrado antes da sessão 7) — agora que o Perfil do Município também expõe a razão local habitantes/membro, esse dado está mais visível, mas ainda não entra na pontuação do IPMT.

### Novidades do relatório da Secretaria (PDF, sessão 7) ainda não incorporadas ao app — decisão de escopo, não bug
O relatório "Inteligência Territorial para a Missão" traz um **segundo conjunto de 5 indicadores** (Índice de Influência Missionária, Índice de Cobertura Territorial, Índice de Vulnerabilidade Missionária, Índice de Continuidade Territorial, Índice de Oportunidade Missionária) — nomes e enfoque diferentes dos 6 índices que o app já implementa (ICTA, ISP, ICHU, IPMT, IDMP, ILAP, definidos no documento-mestre, Seção 24). O PDF não detalha as fórmulas desses 5 (só o "o que mede" em linguagem executiva), e a relação entre os dois conjuntos não é 1-para-1 óbvia — por isso **não foram implementados nesta sessão**: exigiria antes uma conversa com a AMS sobre se são (a) o mesmo conteúdo com nomes diferentes, (b) uma evolução que deveria substituir os 6 atuais, ou (c) indicadores genuinamente novos a acrescentar. Fica registrado como o item mais substancial para uma próxima sessão, se a AMS confirmar a direção.

Também do PDF, uma lista de municípios citados como exemplos que servem de **checkpoint de QA** contra as telas Vazios/Polos do app (todos já deveriam aparecer coerentemente nessas telas, dado que usam a mesma base REGIC): polos "que precisam ser fortalecidos" — Varginha, Pouso Alegre, Barbacena, Ubá, Viçosa, Ponte Nova, São João del-Rei, Santa Rita do Sapucaí; "vazios estratégicos" — Antônio Carlos, Ijaci, Soledade de Minas, Pirapetinga, Alfredo Vasconcelos; eixo Juiz de Fora — Santos Dumont, Matias Barbosa, Bicas, Lima Duarte, Rio Novo; caso Baependi (posição sensível no eixo São Lourenço–Caxambu–Baependi–Carmo de Minas–Soledade de Minas) e caso Leopoldina (ciclo Presença→Irradiação→Continuidade→Inteligência, com Argirita/Laranjal/Recreio como dependentes).

### Manutenção de dados
- Sempre que a planilha eclesiástica (`IGREJAS E GRUPOS AMS.xlsx`) for atualizada, repetir o pipeline completo de build (seção 2, agora terminando em `build_final.py`) e reconferir o `summary.json` resultante contra a soma bruta de `churches.json`, para não reintroduzir o tipo de discrepância corrigido na Sessão 5.
- Se o REGIC tiver uma nova edição publicada pelo IBGE (a atual é 2018), os arquivos `regic_mg.json` / `build_data.py` precisarão ser atualizados a partir da nova planilha oficial.
- Os números oficiais da Secretaria (`membros_ativos_oficial`, `membros_frequentes_oficial` em `summary.json`) são constantes fixas, sourced do PDF desta sessão — se a AMS publicar um relatório mais recente, atualizar esses 4 campos manualmente em `build_master2.py` (não são recalculados por nenhum pipeline).

---

## 6. Onde encontrar cada coisa no código (mapa rápido para retomar o trabalho)

| O que você quer mexer | Onde está |
|---|---|
| Cores, fontes, layout geral (CSS) | `template.html`, dentro de `<style>` |
| Glossário de termos (textos das dicas) | `app.js`, objeto `GLOSSARY` (seção 2-B) |
| Sugestões do Simulador por tipo de ação | `app.js`, objeto `ACTION_PLAYBOOK` (seção 13-B) |
| Fatores de projeção do Simulador | `app.js`, função `runSimulation()`, objeto `actionFactor` |
| Referências Metodológicas (tabela Sobre) | `app.js`, função `viewSobre()` |
| Lógica de recomendação por município | `app.js`, função `recommendFor()` |
| Integração com a API do IBGE | `app.js`, seção "0-B. IBGE" (funções `ibgeFetchJSON`, `fetchPopulacaoIBGE`, etc.) |
| Cálculo de prioridade/hierarquia dos municípios | `build_master2.py` |
| Números do Dashboard (summary.json) | `build_master2.py`, bloco `summary = {...}` no final |
| Números oficiais da Secretaria (membros ativos/frequentes) | `build_master2.py`, constantes perto do fim + `app.js`, bloco `#membrosOficialBox` em `viewInicio()` |
| Razão "habitantes por membro" por município (Perfil) | `app.js`, `loadMunicipioPopulacao()` e o KPI `#kpiHabPorMembroLocal` em `renderPerfilMunicipio()` |
| Razão "habitantes por adventista" (Início, agregada) | `app.js`, `renderHabAdvResult()` / `refreshHabAdvData()` |
| Montagem do SIM_AMS.html final | `build_final.py` (novo, sessão 7) |

---

*Documento vivo — reflete o estado do app ao final da Sessão 7 (24/08/2026). Este é o mesmo arquivo desde a Sessão 6; a partir de agora, continue editando-o em vez de gerar um novo Markdown a cada sessão, para manter um único histórico contínuo.*
