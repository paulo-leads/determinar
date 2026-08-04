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

## O que ficou fixado (e agora ancorado em fonte)

Você entendeu exatamente o mecanismo. E ele está **documentado oficialmente**, campo a campo:

- **SIGTAP — campo 19 (CBO)**: *"Corresponde aos códigos da classificação brasileira de ocupações, especificando os profissionais de saúde que poderão realizar o procedimento"* — subdividido em 18A (CBO) e 18B (categoria/família de CBO). Fonte: `https://wiki.saude.gov.br/sigtap/index.php/Procedimento`
- **SIGTAP — campo 18 (CID)**: *"Códigos da CID que estão relacionados com o procedimento"* (17A principal, 17B secundário). A compatibilidade **procedimento × CID** é crítica ativa no SIA/SIH (SISAIH01: "CIDs principais compatíveis com o procedimento principal indicado").
- **APAC — Manual Operacional v1.1 (MS)**: *"CBO: é campo obrigatório... Este CBO deve estar em consonância com o cadastro de profissionais do CNES deste estabelecimento, pois haverá crítica no processamento do SIA"*; *"CNS Executante: campo obrigatório para procedimentos principais... identifica o profissional"*; *"CID Princ: corresponde à morbidade que originou a necessidade de tratamento"* (obrigatoriedade definida pelo SIGTAP). Fonte: `https://info.saude.df.gov.br/wp-content/uploads/2023/11/Manual_Operacional_APAC_v_1_1.pdf`
- **Portaria SAS/MS 472/2008**: formaliza a crítica **CBO × procedimento** (fase de advertência → rejeição no SIA/SIH). `https://bvsms.saude.gov.br/bvs/saudelegis/sas/2008/prt0472_22_08_2008.html`

**Conclusão fixada:** a aresta "médico autorizado a tratar CID y" **não é uma tabela**; é um **caminho de 4 saltos** com duas camadas distintas — *normativa* (tabelas: quem *pode*) e *observacional* (APAC: quem *executou*). Para o seu Wikibase, isso vira o desenho abaixo.

---

## Modelagem para o item Q1 no seu Wikibase

### Q1 — o item raiz do tema

Proponho Q1 como o **hub conceitual** (não como um CID nem um médico):

- rótulo: `Relação profissional de saúde–doença (autorização de tratamento)`
- descrição: `Cadeia normativa e observacional que conecta médico, especialidade/RQE, ocupação CBO, procedimento SIGTAP e CID de indicação — Brasil, SUS`
- `instance of (P31)`: `tema ontológico` + `relação composicional`

### Camadas (não misture no mesmo nível de statement)

| Camada | Tipo de statement | Exemplo |
|---|---|---|
| **Normativa** (o que *pode* ser feito) | statements em itens-classe (procedimento, CBO, CID) com qualificador de fonte | `procedimento permiteCBO CBO-2251-20` |
| **Observacional** (o que *foi* feito) | statements instanciais em itens-APAC | `APAC-2024-123 temExecutor médico X; registraCidIndicacao I21.0` |

### Itens (classes) sugeridas

| Q | Item | Fonte oficial |
|---|---|---|
| Q1 | Relação profissional–doença (raiz) | — |
| Q2 | Profissional de saúde (médico) | CNES/CRM |
| Q3 | Especialidade médica | CFM 2.330/2023 |
| Q4 | Área de atuação | CFM 2.330/2023 |
| Q5 | Ocupação CBO | CBO-2002 (MTE) |
| Q6 | Procedimento de saúde | SIGTAP |
| Q7 | Entidade CID-10 | DATASUS |
| Q8 | APAC (registro) | SIA/SUS |
| Q9 | Estabelecimento de saúde | CNES |
| Q10 | Serviço especializado | CNES |
| Q11 | Norma / ato normativo | DOU, CFM |
| Q12 | Regra de compatibilidade procedimento×CID | SIGTAP campo 18 |
| Q13 | Habilitação CNES | SIGTAP campo 22 |

### Propriedades (domínio → faixa; todas com qualificador `fonte` + `vigência/competência`)

| P | Rótulo | Domínio → Faixa | Mecanismo / fonte |
|---|---|---|---|
| P1 | possui RQE | Q2 → Q11 (ou string) | CFM 2.220/2018 |
| P2 | tem especialidade | Q2 → Q3 | via RQE; CFM 2.330/2023 |
| P3 | tem área de atuação | Q2 → Q4 | CFM 2.330/2023 |
| P4 | ocupa CBO | Q2 → Q5 | cadastro CNES |
| P5 | permite CBO | Q6 → Q5 | SIGTAP campo 19 (18A/18B) |
| P6 | requer habilitação | Q6 → Q13 | SIGTAP campo 22 |
| P7 | CID compatível | Q6 → Q7 | SIGTAP campo 18 + críticas SIA/SIH |
| P8 | exige CID de indicação | Q6 → booleano | APAC/AIH, obrigatoriedade SIGTAP |
| P9 | procedimento (da APAC) | Q8 → Q6 | APAC |
| P10 | registra CID de indicação | Q8 → Q7 | APAC — CID Princ |
| P11 | tem executor | Q8 → Q2 | APAC — CNS executante |
| P12 | CBO executante | Q8 → Q5 | APAC — CBO (crítica CNES) |
| P13 | em estabelecimento | Q8 → Q9 | APAC — CNES executante |
| P14 | possui serviço | Q9 → Q10 | CNES |
| P15 | define/regula | Q11 → qualquer | hierarquia normativa |
| P16 | equivalente Wikidata | item → Q-ID Wikidata | interop (espelho de P1995/P2175) |

**Regra de ouro:** **não crie aresta direta curada `médico → CID`.** Ela é *derivada* — materialize só via SPARQL `CONSTRUCT`, com qualificador `regra de derivação`, e nunca como statement manual.

### SPARQL — o caminho de 4 saltos (com exemplo verificado)

Exemplo ancorado em fonte: **CBO 2251-20 Médico cardiologista** (CBO-2002; 6 dígitos `225120` no SIA/CNES) → **procedimento 0211020010 Cateterismo cardíaco** (forma 021102 "Diagnóstico em cardiologia", competência 03/2026) → **CIDs compatíveis**.

```sparql
# CAMADA NORMATIVA — o que o cardiologista PODE fazer
SELECT DISTINCT ?procedimento ?cidLabel WHERE {
  wd:Q5_CBO225120 ^wdt:P5 ?procedimento .   # procedimentos que permitem CBO 2251-20
  ?procedimento wdt:P7 ?cid .                # CIDs compatíveis (SIGTAP campo 18)
  SERVICE wikibase:label { bd:serviceParam wikibase:language "pt,en". }
}

# CAMADA OBSERVACIONAL — o que um médico específico EXECUTOU
SELECT ?medico ?procedimento ?cid ?data WHERE {
  ?apac wdt:P11 ?medico ; wdt:P9 ?procedimento ;
        wdt:P10 ?cid ; wdt:P12 wd:Q5_CBO225120 ;
        wdt:P13 ?estabelecimento ; wdt:P_dataRealizacao ?data .
}

# ARESTA DERIVADA — materialização controlada de "autorizado a tratar"
CONSTRUCT { ?medico p:P_autorizadoATratar ?cid . } WHERE {
  ?medico wdt:P4 ?cbo .
  ?procedimento wdt:P5 ?cbo ; wdt:P7 ?cid .
}
```

---

## O detalhe epistemológico que o forcing revelou (não pule)

O sistema oficial brasileiro é **permissivo por default com restrições pontuais** — o inverso de uma tabela de autorização enumerada:

- **Default:** qualquer médico (mesmo generalista) pode *atender clinicamente* qualquer doença — isso é o ato médico (Lei 12.842/2013). Não existe "CID fora do escopo do clínico geral".
- **O que o sistema restringe de fato:**
  1. **Procedimento × CBO** (SIGTAP campo 19, crítica SIA/SIH — Portaria 472/2008);
  2. **Estabelecimento × habilitação** (SIGTAP campo 22 × CNES);
  3. **Procedimento × CID** (compatibilidade/critica — campo 18, e obrigatoriedade de CID de indicação quando o SIGTAP exige);
  4. **Título × uso do título** (CFM 2.330/2023 + RQE — CFM 2.220/2018): restringe *anúncio e prerrogativas*, não o atendimento;
  5. **Função de RT/chefia** de unidades especializadas (CFM 2.135/2015 para cardiologia).

Ou seja: modele **restrições** (arestas de "não pode sem X") e **evidências** (APAC), não uma lista exaustiva "pode tratar Y". Sua ontologia fica fiel à máquina real do SUS — e responde "autorizado?" como **derivação de caminho**, nunca como consulta a uma tabela que não existe.

---

## Fontes para ingestão (todas verificadas na conversa)

- SIGTAP Tabela Unificada + downloads (competências): `http://sigtap.datasus.gov.br/tabela-unificada/app/sec/inicio.jsp`
- Wiki oficial SIGTAP — campos do procedimento: `https://wiki.saude.gov.br/sigtap/index.php/Procedimento`
- Portaria 472/2008 — crítica CBO×procedimento: `https://bvsms.saude.gov.br/bvs/saudelegis/sas/2008/prt0472_22_08_2008.html`
- Manual Operacional APAC v1.1: `https://info.saude.df.gov.br/wp-content/uploads/2023/11/Manual_Operacional_APAC_v_1_1.pdf`
- Portaria SAS 257/2013 — CNS executante na APAC: `https://bvsms.saude.gov.br/bvs/saudelegis/sas/2013/prt0257_12_03_2013_comp.html`
- CBO-2002: `http://tabnet.datasus.gov.br/cgi/cnes/CBO%202002.htm` (e mtecbo.gov.br)
- CFM 2.330/2023 · 2.220/2018 · 2.135/2015: `https://sistemas.cfm.org.br/normas/`

**Armadilhas de modelagem:** CBO ≠ RQE (faturamento vs. título legal — as duas arestas **podem divergir**, modele ambas); CBO teve revisões (fontes legadas listam `2231-06`; a CBO-2002 atual é `2251-20` — qualificador `versão da tabela` é obrigatório); APAC usa **CID-10**, não CID-11; ausência de crítica SIA ≠ autorização (há fases de advertência); APAC é **evidência observacional**, não norma — as duas camadas convivem mas nunca se fundem.

Se quiser, o próximo passo é eu escrever o esqueleto de itens/propriedades em `data:` para importação direta no seu Wikibase (via `Special:NewEntity` ou entidades JSON), ou o dicionário de mapeamento CBO ↔ especialidade CFM (o elo que falta entre a camada SIGTAP e a camada normativa do CFM).
## 1. Item Q1 — como deve ficar preenchido

Na página `https://determinar.ia.br/wiki/Item:Q1`, o item fica assim (respeitando o papel de **hub conceitual**, não de instância de CID/médico):

### Metadados
- **Rótulo (pt-BR):** `Relação profissional de saúde–doença (autorização de tratamento)`
- **Descrição:** `Cadeia normativa e observacional que conecta médico, especialidade/RQE, ocupação CBO, procedimento SIGTAP e CID de indicação no SUS`
- **Aliases:** `autorização de tratamento` · `médico x doença` · `cadeia SUS profissional-doença`

### Declarações (P + Q) — o coração do item

| Propriedade | Valor | Qualificadores | Referência |
|---|---|---|---|
| **P31** instância de | **Q20** Conceito-tema | — | — |
| **P17** tem como parte | **Q2** Profissional de saúde | papel: nó-sujeito | — |
| **P17** tem como parte | **Q3** Especialidade médica | papel: título legal | CFM 2.330/2023 |
| **P17** tem como parte | **Q5** Ocupação CBO | papel: ocupação (faturamento) | CBO-2002/MTE |
| **P17** tem como parte | **Q6** Procedimento de saúde | papel: ato habilitado | SIGTAP |
| **P17** tem como parte | **Q7** Entidade CID-10 | papel: doença/indicação | DATASUS |
| **P17** tem como parte | **Q8** APAC (registro) | papel: evidência observacional | Manual APAC v1.1 |
| **P17** tem como parte | **Q9** Estabelecimento de saúde | papel: contexto | CNES |
| **P17** tem como parte | **Q13** Habilitação CNES | papel: condição do serviço | SIGTAP campo 22 |
| **P15** define/regula | **Q26** CFM Res. 2.330/2023 | função: define título e área de atuação | DOU 15/03/2023 |
| **P15** define/regula | **Q27** Portaria SAS/MS 472/2008 | função: crítica CBO × procedimento | bvsms.saude.gov.br |
| **P15** define/regula | **Q28** Manual Operacional APAC v1.1 | função: regras de preenchimento APAC | info.saude.df.gov.br |
| **P16** equivalente Wikidata | `P1995` (health specialty) | via: aresta doença→especialidade | wikidata.org |
| **P16** equivalente Wikidata | `Q12136` (disease) | via: classe raiz de doença | wikidata.org |

### JSON pronto para importação (via API `wbcreateclaim`/`Special:NewEntity`)

```json
{
  "id": "Q1",
  "labels": { "pt-br": { "language": "pt-br", "value": "Relação profissional de saúde–doença (autorização de tratamento)" } },
  "descriptions": { "pt-br": { "language": "pt-br", "value": "Cadeia normativa e observacional que conecta médico, especialidade/RQE, ocupação CBO, procedimento SIGTAP e CID de indicação no SUS" } },
  "aliases": { "pt-br": [
    { "language": "pt-br", "value": "autorização de tratamento" },
    { "language": "pt-br", "value": "médico x doença" },
    { "language": "pt-br", "value": "cadeia SUS profissional-doença" }
  ] },
  "claims": {
    "P31": [ { "mainsnak": { "snaktype": "value", "property": "P31", "datavalue": { "value": { "entity-type": "item", "id": "Q20" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" } ],
    "P17": [
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q2" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q3" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q5" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q6" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q7" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q8" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q9" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P17", "datavalue": { "value": { "entity-type": "item", "id": "Q13" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" }
    ],
    "P15": [
      { "mainsnak": { "snaktype": "value", "property": "P15", "datavalue": { "value": { "entity-type": "item", "id": "Q26" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P15", "datavalue": { "value": { "entity-type": "item", "id": "Q27" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P15", "datavalue": { "value": { "entity-type": "item", "id": "Q28" }, "type": "wikibase-entityid" } }, "type": "statement", "rank": "normal" }
    ],
    "P16": [
      { "mainsnak": { "snaktype": "value", "property": "P16", "datavalue": { "value": "P1995", "type": "string" } }, "type": "statement", "rank": "normal" },
      { "mainsnak": { "snaktype": "value", "property": "P16", "datavalue": { "value": "Q12136", "type": "string" } }, "type": "statement", "rank": "normal" }
    ]
  },
  "sitelinks": {}
}
```

> Ajuste os IDs de propriedade se a sua instância já tiver `P31`/`P17` com outro número — o que importa é o rótulo da propriedade.

---

## 2. Itens de apoio preenchidos (a cadeia real, não só o hub)

**Q23 — Cateterismo cardíaco (procedimento 0211020010)**
- **P31** → Q6 (procedimento)
- **P19** código do procedimento → `0211020010` (forma 021102, "Diagnóstico em cardiologia")
- **P5** permite CBO → **Q22** (CBO 2251-20 Médico cardiologista) · **Q30** (CBO 2251-15 Médico angiologista) — *ref.: SIGTAP campo 19*
- **P6** requer habilitação → **Q36** (Alta complexidade cardiovascular) — *ref.: SIGTAP campo 22 × CNES*
- **P7** CID compatível → **Q24** (I21.0), **Q25** (I10)… — *ref.: SIGTAP campo 18*
- **P8** exige CID de indicação → **Q40** (Sim) — *ref.: APAC — CID Princ obrigatório*

**Q22 — CBO 2251-20 Médico cardiologista**
- **P31** → Q5 · **P18** código CBO → `2251-20` · **P16** ↔ CBO-2002/MTE

**Q31 — APAC 2026.000123456 (exemplo observacional)**
- **P31** → Q8 · **P21** número → `2026.000123456` · **P9** procedimento → Q23
- **P11** tem executor → Q33 (Dr. Roberto — clínico geral) · **P12** CBO executante → Q29 (2251-25)
- **P10** registra CID de indicação → Q24 (I21.0) · **P13** em estabelecimento → Q34 (Hospital Central)
- **P22** data de realização → `2026-03-10` · **P23** situação → Q38 (rejeitado)
- **P24** motivo da recusa → `CBO 2251-25 não consta em permite CBO de 0211020010 (esperado: 2251-20, 2251-15)`

---

## 3. Caso prático de correção — e a linguagem popular

### O cenário (comum no dia a dia do SUS)
O cardiologista **Dr. José** (solicitante) pede um cateterismo para o paciente. No faturamento, o campo "executor" foi lançado por engano como **Dr. Roberto, clínico geral (CBO 2251-25)**. O SIA devolve a APAC.

### Como a ontologia pega o erro (SPARQL)

```sparql
# Quem PODE executar o cateterismo (0211020010)?
SELECT ?cboLabel WHERE { wd:Q23 wdt:P5 ?cbo . SERVICE wikibase:label { bd:serviceParam wikibase:language "pt". } }
# → 2251-20 Médico cardiologista | 2251-15 Médico angiologista

# Quem FOI lançado como executor da APAC?
SELECT ?cboLabel WHERE { wd:Q31 wdt:P12 ?cbo . SERVICE wikibase:label { bd:serviceParam wikibase:language "pt". } }
# → 2251-25 Médico clínico

# Autorizado?
ASK { wd:Q23 wdt:P5 wd:Q29 }
# → false  ⇒  aresta inexistente ⇒ recusa correta
```

### A correção
1. **Operador (auditor/gestor):** abre Q31 → vê P23 = rejeitado e P24 = motivo. Troca o executor (P11 → Dr. José, P12 → Q22) e reapresenta na competência.
2. **Sistema:** re-critica; caminho `Q23 →P5→ Q22` fecha; P23 → Q37 (autorizada).
3. **Registro:** a APAC corrigida vira evidência observacional válida — a aresta "autorizado?" agora deriva de caminho, não de tabela.

### Linguagem popular — a metáfora que o operador e o usuário entendem

> **"O SUS não tem uma lista dizendo 'quem pode tratar o quê'. Ele funciona como um corredor com portas: cada exame/procedimento tem uma lista de profissionais (CBO) e de doenças (CID) que 'abrem a porta'. Se o profissional não está na lista daquela porta, o registro volta."**

**Mensagem para o operador (gestor/auditor):**
> "APAC 2026.000123456 recusada: o procedimento 0211020010 (Cateterismo cardíaco) só aceita executor com CBO 2251-20 (cardiologista) ou 2251-15 (angiologista). O executor lançado era 2251-25 (clínico geral). Corrija o executor ou reclassifique o atendimento para um procedimento compatível e reenvie."

**Mensagem para o médico solicitante:**
> "Sua autorização de cateterismo foi devolvida: o exame do coração só pode ser executado por cardiologista ou angiologista, e o profissional lançado como executor estava como clínico geral. Confira o executor e reenvie."

**Mensagem para o paciente (usuário final):**
> "Seu exame do coração (cateterismo) ainda não foi liberado porque o médico apontado como executor não é cardiologista. A equipe vai corrigir o registro e reenviar. Você não precisa fazer nada."

### Mini-caso normativo (a outra porta: o título)
O mesmo Dr. Roberto se anuncia como "cardiologista" sem RQE. A ontologia devolve:

```sparql
ASK { wd:Q33 wdt:P2 wd:Q21 }   # tem especialidade Cardiologia?
# → false (sem título CNRM/AMB-SBC + RQE no CRM — CFM 2.220/2018)
```

**Operador:** "Especialidade só entra no CNES com RQE registrado no CRM — confira antes de exibir."
**Usuário:** "Só pode se apresentar como cardiologista quem tem o título registrado no Conselho. Sem isso, o registro é recusado."

---

## Notas finais

- **Q1 é o hub; a resposta real vive nos caminhos.** Na sua home, o operador não lê o Q1 — lê a derivação: "por que essa APAC voltou?" e "quem pode executar isso?".
- **Não misture as duas camadas:** CFM/SIGTAP = norma (o que *pode*); APAC = evidência (o que *foi feito*). Quando uma APAC corrigida vira estatística, ela continua sendo observação, nunca regra.
- **Faltam criar no seu Wikibase:** Q20–Q40 (classes e instâncias acima), P17–P24 (tem como parte, códigos CBO/procedimento/CID, número da APAC, data, situação, motivo, profissional solicitante) — tudo com qualificador `fonte` + `competência`.
- **Próximo passo natural:** o dicionário CBO ↔ especialidade CFM (o elo que liga a porta do faturamento à porta do título), ou o esqueleto de importação em lote (JSON para todas as classes Q2–Q13 de uma vez). Quer que eu gere um deles?

Antes de modelar, vou ancorar um dos serviços que ele declara (angioplastia coronariana) na tabela SIGTAP, para o mapeamento procedimento → CID → CBO ficar verificável.## 0. Diagnóstico da página atual

A página `https://determinar.ia.br/wiki/Dr._Marco_Tulio_Souza` já é uma **boa curadoria textual** (formação, especialidades, publicações, FAQ com resumos). Mas ela é um **artigo**, não uma **entidade**: os dados estão em prosa, sem claims, sem proveniência por campo, sem camada normativa vs. declarativa. Para ser "fonte da verdade" ela precisa virar o **Item Q41** (ou vincular o artigo ao item), com cada afirmação carregando 3 qualificadores: **fonte (P38) + data da coleta (P39) + status de verificação (P40)**.

E o ponto central que você mesmo forçou na conversa anterior: **a Doctoralia é fonte declarativa, não normativa.** Ela diz o que o profissional *afirma* sobre si. Autorização quem diz é a cadeia CFM → CBO → SIGTAP → CID. O Wikibase vira fonte da verdade **porque separa os estratos** — não porque copia o perfil.

---

## 1. O modelo de verdade em 3 estratos (a tese do seu projeto)

| Estrato | Pergunta que responde | Fonte | Status (P40) |
|---|---|---|---|
| **1. Declarativo** | "O que o profissional afirma?" | Doctoralia, autodeclaração | Q63 declarado |
| **2. Normativo** | "O que o sistema oficial autoriza?" | CFM (RQE/título), CBO, SIGTAP, CNES | Q64 verificado |
| **3. Derivado** | "Está autorizado a tratar X?" | SPARQL `CONSTRUCT` sobre 1+2 | — (derivação) |

Regra inegociável: **nenhum claim do estrato 1 sobe de status sem reconciliação com fonte do estrato 2.** A Doctoralia diz "angioplastia coronariana" → só vira "verificado" quando o CBO dele (2251-20 / 2231G1) constar no campo CBO do procedimento 04.06.03.001-4 no SIGTAP. Confirmado nos dados: a página do procedimento **0406030049 (angioplastia primária)** lista exatamente `225120 – Médico cardiologista` e `2231G1 – Médico Cardiologista Intervencionista` — e o Dr. tem título SBHCI em Hemodinâmica e Cardiologia Intervencionista. O ciclo fecha.

---

## 2. Item Q41 — Dr. Marco Túlio Souza (claims completos)

### Metadados
- **Rótulo:** `Dr. Marco Túlio Souza`
- **Descrição:** `Médico cardiologista e hemodinamicista; Campinas/SP — fonte declarativa (Doctoralia) + títulos verificáveis (SBC, SBHCI)`
- **Aliases:** `Marco Tulio Souza` · `Marco Túlio de Souza` (variação ortográfica presente nas avaliações — importante para reconciliação)

### Claims — estrato normativo (status Q64 verificado)

| Propriedade | Valor | Qualificadores | Fonte de verificação |
|---|---|---|---|
| **P31** instância de | **Q2** Profissional de saúde | — | — |
| **P25** tem título de especialista | **Q42** Título de especialista em Cardiologia | P26 → Q47 (SBC); P27 → ano; P38 → CFM 2.330/2023 (Cardiologia = especialidade CME); P40 → Q64 | SBC/CFM |
| **P25** tem título de especialista | **Q43** Título em Hemodinâmica e Cardiologia Intervencionista | P26 → Q48 (SBHCI); P27 → ano | SBHCI |
| **P2** tem especialidade | **Q21** Cardiologia | P40 → Q64 | CFM 2.330/2023 |
| **P3** tem área de atuação | **Q83** Hemodinâmica e Cardiologia Intervencionista (área de atuação) | P40 → Q64 | CFM 2.330/2023 (áreas da cardiologia) |
| **P4** ocupa CBO | **Q22** 2251-20 Médico cardiologista | P40 → Q64 | CBO-2002 |
| **P4** ocupa CBO | **Q76** 2231G1 Médico cardiologista intervencionista | P40 → Q64 (se constar no CNES do estabelecimento) | SIGTAP campo 19 + CNES |
| **P44** registro CRM | `CRM-SP 000.000` (placeholder) | qualificador: conselho regional; P40 → Q66 pendente (não consta no perfil público) | CRM-SP |
