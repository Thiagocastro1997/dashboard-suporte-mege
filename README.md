# Dashboard de Suporte MEGE

Dashboard interativo standalone para análise de **4.218 tickets de suporte ao aluno** — cursos preparatórios para Magistratura, Ministério Público e Defensoria Pública.

---

## Como usar

Abra o arquivo `dashboard_mege.html` diretamente no navegador. Não requer servidor, instalação ou internet após o carregamento inicial (exceto Google Fonts).

Para regenerar a partir de um novo arquivo de dados:

```bash
pip install openpyxl
python gen_dashboard.py
```

---

## Processo de Investigação

### 1. Leitura e diagnóstico dos dados

O arquivo fonte (`tickets_etiquetados_mege_completo.xlsx`) continha **4.626 linhas** e **22 colunas**. A primeira etapa foi inspecionar os campos disponíveis:

```
id, titulo, descricao, data_criacao, data_atualizacao, reposta, status,
resolvido, data_resposta, tipo_suporte, turma, id_aluno, nome_aluno,
email_aluno, coodenador_id_responsavel, coodenador_responsavel,
tipo_solicitacao, material_referenciado, carreira, materia, confianca,
nota_ambiguidade
```

Campos etiquetados por IA: `tipo_solicitacao`, `material_referenciado`, `carreira`, `materia`, `confianca`, `nota_ambiguidade`.

### 2. Derivação de campos

Dois campos importantes não existiam diretamente e precisaram ser derivados:

**`fase`** — extraída do campo `turma` via regex + matching de keywords:

| Padrão no nome da turma | Fase derivada |
|---|---|
| "2ª Fase", "2 Fase" | 2ª Fase |
| "Pré-Edital", "Pre-Edital" | Pré-Edital |
| "Clube" | Clube |
| "Pós-Edital" | Pós-Edital |
| "Até Passar", "Ate Passar" | Até Passar |
| "1ª Fase" | 1ª Fase |
| "Reta Final" | Reta Final |
| (demais) | Geral |

**`dia`, `mes`, `semana`, `dow`** — extraídos de `data_criacao` via `datetime.strftime`.

**`turma_clean`** — nome da turma com conteúdo entre parênteses removido (`re.sub(r'\s*\([^)]*\)', ...)`) e truncado em 50 caracteres.

### 3. Descoberta de duplicatas

Durante a análise, identificou-se que o banco de dados continha registros duplicados pelo campo `id`. A investigação revelou:

| Métrica | Valor |
|---|---|
| Total de linhas brutas | 4.626 |
| IDs únicos | 4.218 |
| IDs duplicados (aparecem 2x ou mais) | 398 |
| Linhas redundantes removidas | 408 |

Destaques:
- 1 ID aparecia **4 vezes**
- 8 IDs apareciam **3 vezes**
- 389 IDs apareciam **2 vezes**

**Solução aplicada:** deduplicação por `id` no momento da leitura, mantendo apenas a primeira ocorrência (`seen_ids = set()`).

### 4. Descoberta do campo `tipo_suporte`

A investigação identificou que, além das etiquetas geradas por IA (`tipo_solicitacao`), existia um campo de classificação **original do sistema** (`tipo_suporte`) com categorias mais granulares e diferentes:

| Categoria original | Volume |
|---|---|
| Conteúdo / Material de Apoio | 1.653 |
| Coordenação Acadêmica/Institucional | 1.176 |
| Assuntos Administrativos | 579 |
| Simulados de provas Objetivas | 459 |
| Perguntas das avaliações | 391 |
| Financeiro / Atendimento | 287 |
| MEGE Informativos | 81 |

Esse campo foi adicionado como filtro independente em todas as abas do dashboard, permitindo cruzar a classificação humana original com a classificação gerada por IA.

### 5. Investigação sobre "aulas ao vivo" e "SOS"

Investigação específica por keywords em títulos e descrições:

- `"ao vivo"` — 27 ocorrências reais (slides de aulas, links de meet, etc.)
- `"sos"` — 296 matches, mas apenas ~3 eram o uso real da palavra "SOS" como urgência; os demais eram falsos positivos em palavras como "cursos", "processos"
- `"plantão"` — 6 ocorrências
- `"aula específica"` — 4 ocorrências

Conclusão: não há volume suficiente para uma categoria dedicada. Esses tickets estão distribuídos organicamente nos tipos existentes.

---

## Arquitetura do Dashboard

### Estrutura de dados embarcada

O HTML carrega dois arrays JSON compactos embutidos diretamente no `<script>`:

**`FD` — Filter Data** (usado por Abas 1, 2 e 3)
```
[data, tipo_solicitacao, carreira, fase, materia, material_referenciado,
 turma, status, mes, semana, dow, tipo_suporte]
```

**`DD` — Detail Data** (usado pela Aba 4 — Explorador)
```
[data, titulo, tipo_solicitacao, carreira, fase, materia, material_referenciado,
 turma, status, coordenador, confianca, descricao, resposta, nota_ambiguidade,
 mes, tipo_suporte, id]
```

Tamanho final: **~4 MB** (limite definido: 6 MB).

### Abas

| Aba | Conteúdo |
|---|---|
| Visão Geral | KPIs, timeline (dia/semana/mês), ranking de tipos com pareto, top turmas clicáveis, volume por dia da semana, materiais |
| Natureza | Cards por fase, heatmap Matéria × Material, tabela Tipo × Matéria, insights automáticos |
| Por Carreira | Pills de seleção, barras agrupadas por carreira, backlog, insights |
| Explorador | Busca em tempo real, 7 filtros combinados, tabela paginada (30/página), painel lateral com ticket completo + botão copiar ID |

### Stack

- HTML + CSS + Vanilla JS (sem frameworks)
- [Chart.js 4.4.1](https://www.chartjs.org/) via CDN — apenas para timeline e gráficos agrupados
- Google Fonts — Inter + Syne
- Geração: Python 3 + openpyxl

---

## Filtros disponíveis

**Aba Visão Geral:** Carreira · Fase · Turma (top 30) · Mês · Tipo de Suporte

**Explorador:** Busca no título · Tipo de Solicitação (IA) · Carreira · Fase · Matéria · Status · Tipo de Suporte

---

## Observações técnicas

- Dados dos alunos (nome, e-mail, CPF) **não estão incluídos** no repositório — o `.gitignore` bloqueia arquivos `.xlsx`
- Descrições e respostas truncadas em **800 caracteres** cada para controle de tamanho
- Nomes de turmas sanitizados: parênteses removidos, máximo 50 caracteres
- Confiança da classificação IA exibida como badge colorido: verde ≥ 90%, amarelo ≥ 70%, vermelho < 70%
