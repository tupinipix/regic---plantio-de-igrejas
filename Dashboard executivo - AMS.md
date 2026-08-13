# Dashboard Executivo da AMS
## SITM-AMS — Sistema de Inteligência Territorial Missionária

### Finalidade
Transformar dados territoriais, eclesiásticos e de influência urbana em um painel executivo de tomada de decisão para a presidência, secretaria, tesouraria, ministérios e comissão diretiva da **Associação Mineira Sul**.

> ⚠️ **Diretriz Estratégica:** O dashboard não deve funcionar apenas como painel descritivo. Ele deve funcionar como sistema de priorização missionária, monitor de capilaridade territorial e simulador de expansão institucional.

---

## 1. Arquitetura Geral do Dashboard

O painel executivo deve ser organizado em **6 módulos principais**:
1. Visão Geral do Campo
2. Mapa Territorial Missionário
3. Vazios e Prioridades
4. Polos Adventistas e Irradiação Regional
5. Grupos em Organização e Expansão
6. Simulador Estratégico de Decisão

### Filtros Globais
* Ano
* Região/Polo
* Distrito pastoral
* Status da presença
* Hierarquia REGIC
* Região de influência
* Faixa populacional
* Prioridade missionária

---

## 2. Tela 1 — Visão Geral do Campo

> **Objetivo:** Oferecer, em poucos segundos, uma leitura executiva do território da AMS.

### KPIs Principais
* Total de municípios da AMS
* Municípios com presença organizada
* Municípios sem presença
* Municípios com grupo em organização
* População total do território
* População em municípios sem presença
* Membros totais
* Habitantes por adventista
* Cobertura missionária municipal (%)
* Cobertura missionária populacional (%)

### Visualizações
* **Mapa-síntese da AMS:** com presença, grupos em organização e vazios.
* **Gráfico de rosca:** por status da presença.
* **Gráfico de barras:** por faixa populacional e status.
* **Gráfico de colunas:** por hierarquia REGIC.

### Perguntas Respondidas
* Qual é o nível geral de cobertura do campo?
* Onde estão os principais vazios?
* A AMS já cobre os municípios mais importantes da rede urbana?

---

## 3. Tela 2 — Mapa Territorial Missionário

> **Objetivo:** Mostrar se a rede adventista acompanha ou não a rede urbana funcional do território.

### Camadas do Mapa
* Limites municipais da AMS
* Status da presença adventista
* Hierarquia REGIC
* Região de influência principal
* Arranjos populacionais
* Polos adventistas
* Grupos em organização
* Eixos prioritários

### Recursos Visuais
* **Cores:** por status da presença.
* **Contorno:** por hierarquia REGIC.
* **Linhas de influência:** entre município e cidade-polo.
* **Bolhas:** proporcionais ao número de membros.

### Filtros Específicos
* Exibir apenas vazios missionários
* Exibir apenas municípios com mais de X habitantes
* Exibir apenas centros de zona ou superiores
* Exibir apenas eixo/polo selecionado

---

## 4. Tela 3 — Vazios e Prioridades

> **Objetivo:** Converter municípios sem presença em uma agenda objetiva de decisão.

### Estrutura Principal
Tabela-radar dos municípios sem presença.

#### Colunas Recomendadas
1. Município
2. População
3. Hierarquia REGIC
4. Região de influência
5. Arranjo populacional
6. Distância ao polo adventista mais próximo
7. Tempo estimado de acesso
8. Existência de fluxo para cidade com presença
9. Classificação territorial
10. Índice de Prioridade Missionária Territorial (IPMT)
11. Recomendação executiva

### Classificação Sugerida
* **Prioridade A:** Vazio estratégico
* **Prioridade B:** Vazio intermediário
* **Prioridade C:** Vazio dependente de polo
* **Prioridade D:** Vazio residual

### Visualizações
* Ranking dos 20 vazios mais prioritários.
* **Matriz 2x2:** Centralidade territorial x Ausência/Cobertura adventista.
* Mapa de calor por região de influência.

---

## 5. Tela 4 — Polos Adventistas e Irradiação Regional

> **Objetivo:** Avaliar a capacidade missionária dos municípios onde a AMS já tem presença.

### KPIs por Polo
* Membros no município
* Habitantes por adventista
* Municípios satélites vinculados
* População potencial da área de influência
* Quantidade de municípios sem presença ligados ao polo
* Distância média dos satélites
* Presença de grupo nos satélites
* Saldo missionário recente

### Visualizações
* Ranking dos polos adventistas
* Diagrama em rede
* Radar de desempenho do polo

### Tipologia dos Polos
* Polo consolidado
* Polo de alta pressão missionária
* Polo subaproveitado
* Polo de apoio distrital

---

## 6. Tela 5 — Grupos em Organização e Expansão

> **Objetivo:** Monitorar a transição entre ausência, grupo e igreja organizada.

### Status Recomendados
* Sem presença -> Frente missionária -> Grupo em formação -> Grupo pronto para organização -> Grupo organizado -> Igreja consolidada

### Indicadores
* Número de membros
* Frequência média
* Estabilidade da liderança
* Origem dos membros
* Potencial local vs dependência do polo
* Papel territorial após organização

### Visualizações
* Linha do tempo de organização por município
* Mapa dos grupos em expansão
* Tabela de prontidão para organização

---

## 7. Tela 6 — Simulador Estratégico de Decisão

> **Objetivo:** Permitir que a liderança teste cenários antes de votar ou investir.

* **Pergunta Central:** Se a AMS organizar presença em determinado município, qual será o ganho territorial e missionário?

### Entradas do Simulador
* Município-alvo
* **Tipo de ação:** Grupo, igreja, reforço de polo
* Polo de apoio
* **Horizonte temporal:** 1, 3, 5 anos

### Saídas Esperadas
* População diretamente coberta
* População indiretamente influenciada
* Municípios satélites beneficiados
* Redução do vazio territorial
* Melhoria na continuidade da rede adventista
* Impacto no IPMT regional
* Ganho estimado em cobertura por hierarquia urbana

---

## 8. KPIs Inéditos Centrais

1. **Índice de Prioridade Missionária Territorial (IPMT)**
   * *Combina:* Centralidade REGIC, população, ausência/presença frágil, descontinuidade territorial e potencial de irradiação.
2. **Índice de Continuidade Territorial Adventista (ICTA)**
   * Mede o quanto a presença adventista forma uma malha territorial contínua.
3. **Índice de Dependência Missionária de Polo (IDMP)**
   * Mostra quantos municípios sem presença dependem funcionalmente de um mesmo polo adventista.
4. **Índice de Cobertura por Hierarquia Urbana (ICHU)**
   * Mostra o percentual de cobertura adventista em cada nível da hierarquia REGIC.
5. **Índice de Lacuna em Arranjos Populacionais (ILAP)**
   * Identifica vazios in áreas urbanas funcionalmente integradas.
6. **Índice de Subaproveitamento de Polo (ISP)**
   * Detecta polos com grande posição territorial, mas baixo efeito missionário regional.

---

## 9. Governança e Uso Institucional

* **Presidência:** Expansão territorial, redivisão pastoral e definição de polos prioritários.
* **Secretaria:** Evolução de grupos, consolidação da presença e análise de cobertura municipal.
* **Tesouraria:** Alocação de recursos por prioridade territorial e apoio a polos de maior irradiação.
* **Ministério Pessoal / Evangelismo:** Escolha de cidades-alvo, definição de eixos missionários e monitoramento de frentes abertas.
* **Comissão Diretiva:** Votar organização de grupos, decidir implantação em vazios estratégicos e comparar cenários de investimento missionário.

---

## 10. Estrutura Técnica Recomendada

### Base Mínima
* Municípios AMS
* Presença adventista
* População / Membros / Hab. por adventista
* Status eclesiástico
* Hierarquia REGIC / Região de influência / Arranjos populacionais

### Base Ideal Futura
* Tempo de deslocamento
* Fluxo intermunicipal de membros
* Batismos por origem territorial
* Presença de instituições adventistas
* Séries temporais anuais
* Georreferenciamento das igrejas

### Ferramentas Possíveis
* **Power BI:** Para comissão executiva.
* **QGIS:** Para análise territorial avançada.
* **Excel / Power Query:** Como base de integração.

---

## 11. Página Final — Agenda Executiva

### Prioridades Imediatas
* Quais municípios devem entrar em pauta na próxima comissão?
* Quais grupos estão prontos para organização?
* Quais polos precisam reforço?

### Alertas Territoriais
* Regiões com excesso de dependência de um único polo.
* Blocos populacionais ainda sem presença.
* Áreas onde a presença existe, mas é muito frágil.

### Próximas Ações Sugeridas
* Organizar grupo
* Fortalecer polo
* Abrir frente missionária
* Revisar distrito pastoral
* Produzir estudo local complementar

---

## 12. Princípio Final

> 💡 **Nota de Encerramento:** O dashboard executivo da AMS não deve ser um painel sobre “onde há igrejas”. Ele deve mostrar como a missão se distribui no território, como a presença adventista interage com a rede urbana real e onde a Igreja deve agir primeiro para maximizar capilaridade, continuidade e irradiação regional.
