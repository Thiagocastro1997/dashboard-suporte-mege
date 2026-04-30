# Dashboard de Suporte MEGE

Dashboard interativo standalone para análise de tickets de suporte ao aluno — cursos preparatórios para Magistratura, Ministério Público e Defensoria Pública.

## Como usar

Abra o arquivo `dashboard_mege.html` diretamente no navegador. Não requer servidor, instalação ou internet após o carregamento inicial.

## Funcionalidades

- **Visão Geral** — KPIs, evolução temporal, ranking de tipos, pareto, turmas clicáveis, material referenciado
- **Natureza** — Cards por fase, heatmap Matéria × Material, insights automáticos
- **Por Carreira** — Comparativo entre carreiras, backlog, matérias
- **Explorador** — Busca e filtros em tempo real, painel lateral com detalhes completos do ticket

## Filtros disponíveis

Carreira · Fase · Turma · Mês · Tipo de Suporte (original) · Tipo de Solicitação (IA) · Matéria · Status

## Stack

HTML + CSS + Vanilla JS · Chart.js 4.4 · Google Fonts (Inter + Syne)

## Geração

O dashboard é gerado pelo script `gen_dashboard.py` a partir do arquivo de dados (não incluso no repositório por conter dados pessoais de alunos).

```bash
pip install openpyxl
python gen_dashboard.py
```
