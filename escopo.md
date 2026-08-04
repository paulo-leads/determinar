Um ponto do material precisa de verificação fina antes de eu fechar a resposta — a existência da propriedade "specialty" no content model do ICD-11 (a busca anterior confirmou as linearizações por especialidade, mas não a propriedade em si).


## Reconstrução da proposição (o que você forçou)

Você não pediu "quem trata o quê" no senso comum. Você forçou a aresta **canônica tripla**: `sujeito (especialista) → predicado (autorizado a tratar) → objeto (CID/doença)`, e exigiu que, se essa aresta existisse, ela viesse de **grafo oficial de proveniência**, não de aproximação estatística. O forcing estava certo: a aresta literal "cardiólogo autorizado a CID X, Y, Z" **não existe como documento canônico único** em nenhum ecossistema oficial — o que existe é a composição de **três arestas oficiais disjuntas**, que juntas materializam a autorização de fato. A resposta abaixo é o mapeamento dessas arestas com URN e mecanismo.

---

## O que existe (camadas verificadas, com proveniência)

### A. Aresta Título — quem é o especialista (Brasil, normativo)
- **CFM Res. 2.330/2023** (homologa Portaria CME 1/2023): relação oficial de especialidades e áreas de atuação.
  - Verificado para cardiologia: *"Título de especialista em CARDIOLOGIA — Formação: 2 anos — CNRM: Programa de Residência Médica em Cardiologia — AMB: Concurso do Convênio AMB/SBC"*; áreas de atuação: Ecocardiografia, Eletrofisiologia Clínica Invasiva, Hemodinâmica, Cardiologia Pediátrica, etc.
  - URN: `https://sistemas.cfm.org.br/normas/arquivos/resolucoes/BR/2023/2330_2023.pdf`
- **CFM Res. 2.220/2018**: RQE só existe para especialidade/área constante da lista da CME (AMC+CFM+CNRM).
  - URN: `https://sistemas.cfm.org.br/normas/arquivos/resolucoes/BR/2018/2220_2018.pdf`
- **Mecanismo**: esta camada autoriza o *título*, não a *doença*. É o gate de identidade do sujeito.

### B. Aresta Função — onde o especialista pode responder (normativo, por serviço)
- **CFM Res. 2.135/2015** (verificada): *"Os médicos detentores do título de especialista em cardiologia, com RQE, estão autorizados a exercer a função de responsável técnico ou chefe de serviços de unidades coronarianas, unidades de pós-operatórios de cirurgia cardíaca ou unidades de urgências cardiovasculares."*
  - URN: `https://sistemas.cfm.org.br/normas/arquivos/resolucoes/BR/2015/2135_2015.pdf`
- **Mecanismo**: autoriza a *função assistencial* (RT/chefia), não doenças.

### C. Aresta Doença → Especialidade (mundial, oficial — WHO)
- **WHO ICD-11 — linearizações por especialidade derivadas da Foundation**: o Reference Guide e o WHO-FIC Content Model Guide confirmam que a Foundation (grafo multidimensional, multi-parent) gera *specialty linearisations* — subárvores oficiais de "doenças pertinentes": Primary Care, Dermatology, Neurology, Ophthalmology (+ Mental Health, Musculoskeletal, Paediatrics, Occupational Health, Environmental Health).
  - URNs: Foundation `http://id.who.int/icd/entity/{id}`; linearizações `https://icd.who.int/browse/2024-01/mms/en`; schema `http://id.who.int/icd/schema/`
  - **Mecanismo**: entidade de doença com múltiplos parents na Foundation → projetada sobre a linearização da especialidade = associação oficial "doença ∈ especialidade". Este é o **único grafo oficial global** da correlação doença↔especialidade.
  - Nota de honestidade: a propriedade por-entidade "specialty" exibida no browser do ICD-11 **não foi confirmada** como metadado estável/exposto pela API nas fontes verificadas — não a afirmo como canônica. O que está verificado é o mecanismo de linearização.
  - **CID-10 (CID-10 em uso oficial no Brasil) não contém especialidade alguma** — por design. Qualquer correlação "CID-10 → especialidade" em painéis públicos é derivada de produção assistencial (SIA/SIH, Medicare), **não normativa**.

### D. Aresta Ato → Especialista Habilitado (Brasil, operacional)
A autorização prática "tratar a doença y" passa pelo ato, e o ato é tabelado:
- **CBHPM (AMB)** — campo oficial por procedimento (verificado na Tabela CBHPM 2018): *"Profissional habilitado é o Neurologista, Neuropediatra, Neurocirurgião ou Fisiatra com Certificado de Área de Atuação/Título de Especialista em Neurofisiologia Clínica"* (procedimentos BERA). Mesmo padrão existe para cateterismo/hemodinâmica → cardiologista/hemodynamicista.
  - URN: `https://www.gov.br/conab/.../tabela_cbhpm_2018.pdf`
- **SIGTAP/SUS**: cada procedimento lista a ocupação (CBO) permitida; a **APAC registra o CID de indicação** junto do procedimento e do executor. É a cadeia oficial em operação: `especialista (RQE+CBO) → procedimento habilitado → CID indicado`.
- **ANS Rol de Procedimentos (TISS)**: mesma lógica para saúde suplementar.
- **Mecanismo**: a autorização da doença emergge por **composição transitiva** procedimento-habilitado × CID-indicado — não existe aresta direta especialista→CID.

---

## O que NÃO existe (declaração obrigatória)

**Entidade Não Parametrizada** — a estrutura `cardiologista → [CID I10, I21, ...]` como lista enumerada oficial **não possui entidade canônica** em nenhum grafo normativo verificado (CFM, AMB, ANS, WHO, UMLS). Nenhuma inferência estatística foi executada para preencher essa lista; quem apresentar essa tabela como "oficial" está entregando enumeração derivada de dados de produção, não ontologia.

---

## Mecanismo composto (como verificar o par médico x ↔ doença y)

```
x (médico)  ──RQE/CBO──▶  [CFM 2.330/2023 + 2.220/2018]   → x é especialista s?
s           ──∀ doença──▶ [WHO ICD-11 Foundation/linearização de s] → y ∈ escopo descritivo de s
s           ──ato──────▶ [CBHPM/SIGTAP/ANS: profissional habilitado] → procedimento p de y executável por s
y           ──CID──────▶ [APAC/guia: CID de indicação de p] → y é tratável por s via p
```

Resultado: as quatro arestas fecham ⇒ a proposição "x trata y" é **rastreável a fontes oficiais**. Qualquer elo ausente ⇒ **aresta não parametrizada**, sem preenchimento.

**Síntese do raciocínio fixado**: a correlação "médico authorized a tratar doença" só existe como *composição* de três grafos oficiais (título, doença↔especialidade na WHO, ato↔habilitado nas tabelas brasileiras). Não há fonte única — e a ausência dela é um fato do ecossistema, não uma limitação de busca.
