# Documento de Requisitos: Aplicativo Web SITM-AMS
## Sistema Inteligente de Mobilização Territorial e Plantio de Igrejas

Este documento estabelece as diretrizes teológicas, metodológicas, técnicas e de interface para o desenvolvimento do **Aplicativo Web SITM-AMS**, uma ferramenta analítica voltada ao cumprimento da missão na Associação Mineira Sul através do cruzamento de dados do REGIC 2018, registros eclesiásticos (Igrejas e Grupos) e missiologia avançada.

---

## 1. Fundamentação Teológica e Missiológica

O aplicativo opera sob a premissa de que a inteligência de dados é um instrumento da soberania de Deus para a mordomia dos recursos missionários.

### O Evangelho da Salvação e o Evangelho do Reino (Ensino de Jesus)
*   **Evangelho da Salvação:** Foco na justificação, conversão individual, proclamação kerigmática e reconciliação do homem com Deus. O app mede isso através do crescimento de membros, batismos e frequência.
*   **Evangelho do Reino:** Foco na manifestação visível do governo de Deus na sociedade. Redenção cultural, justiça social, serviço comunitário e transformação da cidade. O app avalia isso por meio do impacto na rede urbana e penetração em bairros vulneráveis.

### Framework Missiológico Integrado

| Autor / Fonte | Princípio Chave | Aplicação Direta no App Web |
| :--- | :--- | :--- |
| **William Carey** | *"Espere grandes coisas de Deus; tente grandes coisas para Deus."* | **Métrica de Odisseia:** Projeções geográficas audaciosas baseadas em mapas de calor e dados demográficos, não apenas em intuição. |
| **David J. Hesselgrave** | O Ciclo de Plantio de Igrejas (*Planting Churches Cross-Culturally*). | **Módulo de Linha do Tempo:** Divisão do status da igreja no app em fases rígidas (Contatos $\rightarrow$ Pequeno Grupo $\rightarrow$ Grupo Organizado $\rightarrow$ Igreja). |
| **Ed Stetzer** | Missiologia baseada em dados e relevância cultural (*Subversive Kingdom*). | **Análise de Contexto Local:** Exibição de pirâmide etária, IDH local e vulnerabilidade social no relatório do município. |
| **Tim Keller** | Visão de Centro Urbano (*Center Church*) e Redes de Plantio. | **Módulo de Ecossistema Urbano:** Integração com a rede REGIC para entender fluxos de influência e interdependência das cidades. |
| **Exponential.org** | Multiplicação baseada em Reprodução de Nível 1 a Nível 5. | **Scorecard de Multiplicação:** Indicador no app que classifica a igreja de acordo com sua capacidade de enviar plantadores e gerar novas frentes. |
| **Mission to the Cities** | *Metes / Urban Centers Evangelism* (Associação Geral). | **Filtro Macro-Urbano:** Priorização algorítmica para cidades com mais de 100 mil habitantes ou centros de zona importantes. |

---

## 2. Arquitetura da Solução e Cruzamento de Dados

O motor do aplicativo web cruzará três bases de dados centrais em formato JSON/Relacional na camada de backend (ou processadas via scripts locais em Python/Power Query):

