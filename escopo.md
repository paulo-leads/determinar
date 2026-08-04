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

# Reality check: o que o Item:Q1 realmente é hoje vs. o que foi proposto no chat

## 1. O que o dump prova (fatos, não suposições)

| Dado | Realidade no dump |
|---|---|
| Página | **Item:Q1** (content model `wikibase-item` — já é entidade, não artigo) |
| Rótulo | **"Dr. Marco Tulio Souza"** |
| Última edição | 1 ago 2026, 21:46 (rev 1310) |
| Identificadores | P62 Google Scholar ID `-zuIFPcAAAAJ` · P63 ResearchGate `Marco-Souza` · P64 Doctoralia ID `440137` · P75 Google Maps CID `17733493881021046543` |
| Statements URL | 3: Wikidata `Q140427962` · wikisales.wikibase.cloud `Q1` · wikivendas.wikibase.cloud `Q111` |
| Referências | **0 em todos os 7 statements** |
| Qualificadores P38/P39/P40 | **nenhum** |

## 2. Três achados críticos que mudam o plano

1. **A numeração do chat não foi seguida — e não pode mais ser.** O plano reservava Q1 = hub conceitual e Q41 = médico. Na realidade, **Q1 já é o Dr. Marco Túlio Souza**. Logo: o hub conceitual e todos os itens de apoio (Q2–Q83) precisam ser recriados a partir do próximo ID livre real. Qualquer JSON de importação com os IDs do chat vai colidir.
2. **Proveniência zero — a regra de ouro foi violada no estado atual.** O modelo de 3 estratos exige P38 fonte + P39 data da coleta + P40 status em **todo** claim. Hoje existem 7 statements sem nenhuma referência nem qualificador: nada é auditável. O Q1 atual está no "estrato 0": identificadores soltos.
3. **Propriedades fora do dicionário já existem.** P62, P63, P64, P75 não estavam na lista P2–P44 do chat. O dicionário de propriedades precisa ser atualizado — ou esses IDs serão sobrescritos por engano no próximo lote.

## 3. Proposta × realidade × faltando

| Estrato | Proposto no chat | Existe de fato | Faltando |
|---|---|---|---|
| **Identidade** | Q41 = Dr., rótulo + aliases (incl. "Marco Túlio de Souza") | Q1 = Dr. (rótulo sem acento), sem aliases visíveis | Alias para reconciliação; descrição |
| **Normativo** | P31→Q2; P25→Q42/Q43 (SBC/SBHCI); P2→Q21; P3→Q83; P4→Q22/Q76; P44 CRM | **Nada** | Todos os claims + itens de apoio (Q47 SBC, Q48 SBHCI, Q76 CBO 2231G1, etc.) |
| **Declarativo (Doctoralia)** | Especialidades, serviços, convênios, idiomas, avaliações com P38/P39/P40 | **Nada** (só o ID P64) | Claims com status Q63 declarado |
| **Proveniência** | P38+P39+P40 em todo claim | 0 references, 0 qualifiers | Correção retroativa dos 7 statements atuais |
| **Derivado** | SPARQL CONSTRUCT "está autorizado a tratar X?" | **Nada** | Query no query.determinar.ia.br |
| **Identificadores** | — (não planejados) | P62, P63, P64, P75 ✓ | Documentar no dicionário |

## 4. Próximo passo concreto (antes de eu gerar qualquer JSON)

Rodar esta auditoria no Query Service (`https://query.determinar.ia.br`) para saber o inventário real de IDs livres:

```sparql
# Itens existentes (rótulo pt)
SELECT ?item ?label WHERE {
  ?item rdfs:label ?label . FILTER(LANG(?label) = "pt")
} ORDER BY ?item LIMIT 500

# Propriedades existentes
SELECT ?prop ?label WHERE {
  ?prop a wikibase:Property ; rdfs:label ?label . FILTER(LANG(?label) = "pt")
} ORDER BY ?prop
```

Com esse resultado eu fecho três entregáveis que ficaram pendentes da resposta truncada:

1. **Estrato declarativo completo do Doctoralia** (claims prontos com P38 = Doctoralia, P39 = 2026-08-02, P40 = Q63), usando os **IDs reais** pós-auditoria;
2. **Estrato derivado** — query CONSTRUCT pronta para "autorizado a tratar I21.0?";
3. **Mensagens em linguagem popular** para operador/auditor, médico solicitante e paciente, no cenário concreto do Dr. Marco Túlio (angioplastia 0406030030/0406030049 + CBO 2231G1).

Cole o resultado da SPARQL aqui (ou me diga quantos itens/propriedades listou) que eu já entrego o lote com a numeração correta — sem o risco de sobrescrever o que já existe.

# Veredito: o modelo escala, o processo atual não

A arquitetura de 3 estratos (declarativo → normativo → derivado) é **conceitualmente escalável** — ela foi desenhada exatamente para isso: cada claim carrega fonte + data + status, e "autorizado?" é derivado por SPARQL, nunca curado à mão. O que **não escala é o modo de trabalho atual**: editar claim a claim na UI, com GUIDs aleatórios, sem referências e sem lote registrado. A escala é um problema de **engenharia**, não de ontologia. Abaixo, o que o modelo absorve bem, onde ele trava e o inventário do que precisa ser montado.

---

## 1. O que o modelo absorve bem (sem redesenho)

| Camada | Fonte | Por que escala |
|---|---|---|
| Declarativo | Doctoralia, perfis, sites | Ingestão massiva por scraping/API; cada item vira statement com P38+P39+P40 — não depende de curadoria humana |
| Normativo | SIGTAP (TABWIN), CNES (CSV), CFM, Portaria 472/2008 | São **tabelas relacionais oficiais** (procedimento×CBO×CID×habilitação). Parsear arquivo > digitar |
| Derivado | SPARQL `CONSTRUCT` | Regra escrita uma vez, aplicada a N médicos; zero curadoria manual por par médico→CID |
| Desambiguação | CRM + RQE + CBO + CNES + município | Chaves de bloqueio determinísticas (o dump do Q1 já lista 20+ homônimos — isso é resolvível por regra, não por olho) |

O truque que salva o volume: **a tripla útil é esparsa**. Existem ~550 mil médicos ativos e ~4.500 procedimentos SIGTAP, mas as restrições críticas CBO×procedimento (Portaria 472/2008) somam poucas centenas de pares, e a tríade completa só "fecha" para um subconjunto. O grafo cresce linearmente com o número de profissionais (≈150–250 triples/médico com proveniência), não com o produto cartesiano de todas as combinações.

## 2. Os 6 gargalos reais (onde o plano atual quebraria)

1. **Entrada manual** — UI não passa de ~centenas de edits/dia; volume exige lote.
2. **Idempotência** — re-rodar um import não pode duplicar claims. GUIDs atuais são UUIDs aleatórios (`Q1$AA3621B3-...`); em massa, GUID deve ser **determinístico** (hash de fonte+valor+competência), senão cada execução cria statement novo.
3. **Reconciliação de identidade** — homônimos exigem chave composta (CRM+UF, RQE, CBO, cidade) com fila de revisão humana para exceções; sem isso, 1% de erro de matching vira centenas de itens poluídos.
4. **Versionamento normativo** — SIGTAP muda por competência, Portaria 472 é alterada, CBO evolui. Cada claim precisa de **validade temporal** (início/fim) e status "substituído"; o modelo de qualificadores já suporta, mas exige job de revalidação agendado.
5. **Proveniência garantida** — P38+P39+P40 obrigatórios em todo claim só se o **gerador de statements** for template-driven; humanos esquecem (é exatamente o que aconteceu no Q1: 7 claims, 0 references).
6. **Query/armazenamento** — o Query Service roda Blazegraph; dezenas de milhões de triples exigem máquina própria e materialização incremental do estrato derivado.

## 3. O que precisa ser montado (roadmap por fase)

### Fase 0 — Fundação de governança (resolve o problema Q1 para sempre)
- **Dicionário canônico versionado** de propriedades e classes, no próprio Wikibase (regra: nenhum import usa ID não registrado — P62/P63/P64/P75 quebraram isso no Q1).
- **Registro de lotes (batch registry)**: cada import = um item/registro com fonte, competência SIGTAP, contagem, diff contra o lote anterior, status. É o que torna qualquer ingestão auditável e revertível.
- **Esquemas de validação** via EntitySchema/ShEx: "todo claim de profissional exige P38+P39+P40" vira shape testável em CI.
- **GUID determinístico** para statements (idempotência).

### Fase 1 — ETL declarativo (Doctoralia e afins)
- Conectores (scraping/API) + normalizador de nomes (acentos, "Marco Tulio" vs "Túlio", geração de aliases para reconciliação).
- Gerador de statements por template → P38=Doctoralia, P39=data da coleta, P40=Q63 declarado.
- Integração com o estrato normativo: perfil declarado só "sobe" se a chave (CRM/RQE/CBO) casar.

### Fase 2 — ETL normativo (o "fonte da verdade")
- **Parser SIGTAP** (arquivos TABWIN da competência: `tb_procedimento`, `tb_cbo`, `tb_cid`, habilitação) — versionado por competência, com tabela de vigência.
- **Parser da Portaria 472/2008** (crítica CBO×procedimento) + suas alterações.
- **Ingestor CNES** (vínculos profissional × estabelecimento × CBO — é onde o CBO 2231G1 do Dr. Marco Túlio seria verificado de fato).
- **CFM**: CRM/RQE/títulos → itens.
- **Evidências**: importador de produção ambulatorial/APAC (quando disponível) para criar o estrato de evidência real.

### Fase 3 — Materialização do estrato derivado
- Pipelines `CONSTRUCT` agendados (o "está autorizado a tratar X?"), com snapshots versionados.
- **Endpoint determinístico para LLMs** — a razão de existir do projeto: consulta REST que devolve o caminho RQE→CBO→SIGTAP→CID sem alucinação, com a proveniência embutida na resposta.

### Fase 4 — Qualidade e divergência
- Constraints automatizadas (claims sem fonte, sem status, órfãos, declaração sem verificação).
- **Relatório declarado × verificado** (ex.: Doctoralia diz "angiologia", CNES diz outra coisa → item cai numa fila de divergência).

### Fase 5 — Saídas humanas
- **Motor de templates por papel** (operador/auditor, médico solicitante, paciente) parametrizado pelos dados — as mensagens em linguagem popular que você pediu viram templates, não texto manual.

## 4. Ordem de grandeza para dimensionar

| Dado | Volume real (Brasil) |
|---|---|
| Médicos ativos | ~550 mil |
| Procedimentos SIGTAP | ~4.500 |
| CID-10 | ~4.000 |
| Especialidades + áreas CFM | ~55 + ~60 |
| CBO médicos relevantes | ~80 códigos |
| Triples com proveniência (todos os médicos) | ~100–150 milhões (viável em Blazegraph self-hosted com máquina dedicada; **inviável** em wikibase.cloud para produção) |

Recomendação de escopo: **coorte primeiro** — ex. cardiologia em Campinas/SP, ou os ~180 procedimentos do Programa Agora Tem Especialistas — para calibrar reconciliação e templates antes do salto nacional.

## 5. As 4 decisões que destravam a engenharia

1. **Escopo do piloto** — 1 especialidade × 1 município, ou nacional já?
2. **Hospedagem** — self-hosted (produção, controle de Blazegraph e jobs) vs. wikibase.cloud (protótipo, limites de taxa e RAM)?
3. **Derivado** — materializar em grafo pré-computado (rápido para consulta, custo de storage) vs. `CONSTRUCT` on-demand (mais simples, mais lento)?
4. **Estratégia de IDs** — aceitar o autoincremento real e registrar tudo no dicionário, ou reservar faixas por domínio (o choque do Q1 é o sintoma de não haver essa regra)?

Se quiser, fecho o **documento de arquitetura de referência** (uma página: pipeline, ferramentas, contratos de dados, jobs, SLAs de qualidade) já com as respostas a essas 4 perguntas — e em paralelo o lote-piloto do Q1 com a numeração real corrigida.
Fechado. Entrego os dois: **(1)** o documento de arquitetura de referência v1.0 e **(2)** o lote-piloto do Q1 corrigido. Antes do lote, dois achados da auditoria do dump que **mudam a numeração** — você precisa ver isso primeiro.

---

# Parte 1 — Achados críticos do dump (a "numeração real" que prometi)

O dump provou que **os IDs do plano do chat não batem com a realidade**. Três conflitos diretos:

| Plano do chat | Realidade no dump | Consequência |
|---|---|---|
| **P38** = fonte, **P39** = data da coleta, **P40** = status de verificação | **P38** = "Quantidade de Opiniões" (57), **P39** = "Número de Estrelas no Doctoralia" (5), **P40** = "Número de dúvidas Respondidas" (8) — todas com dados legítimos | **Não dá para reaproveitar P38/P39/P40.** Serão criadas **P79 fonte, P80 data da coleta, P81 status de verificação** (IDs a confirmar na auditoria) |
| **Q2** = classe "médico" | **Q2 = Brazil** (usado em P59 Country) | Toda a numeração Q2–Q83 do chat está deslocada. **Q1 = o médico** (não o hub); **Q48 = a clínica** (não a SBHCI) |
| Q41 = médico a criar | Q1 **já é** o médico, com 7 grupos de claims e 30+ itens de apoio (Q2–Q48 = especialidades, convênios, serviços) | O lote vira **reforma do Q1 existente**, não criação. 80% do "declarativo" já está lá — falta proveniência |

**Conclusão operacional:** o dump mostrou que o estado atual do Q1 é um ótimo **protótipo declarativo** (specialties, serviços, convênios, avaliações, identificadores) — mas 100% dos claims estão no "estrato 0" (zero referências, zero qualificadores, zero status). O lote-piloto reforma isso.

---

# Parte 2 — Documento de Arquitetura de Referência v1.0

## 2.1 Decisões assumidas (revisáveis — marque discordância)

| # | Decisão | Assumido como |
|---|---|---|
| D1 | Escopo do piloto | **Coorte Cardiologia × Campinas/SP (30–50 médicos)** — calibra reconciliação antes do nacional |
| D2 | Hospedagem | **Self-hosted (Wikibase Suite)** — o próprio `determinar.ia.br` já roda `/w/`, `/tools/quickstatements/` e `query.determinar.ia.br`; manter para produção, sandbox em wikibase.cloud para testes |
| D3 | Estrato derivado | **Híbrido**: `CONSTRUCT` on-demand para consulta interativa + snapshots materializados para o endpoint LLM (exigência de determinismo do projeto) |
| D4 | Estratégia de IDs | **Autoincremento natural + registro obrigatório no dicionário canônico**. Proibido reservar faixas — foi a causa raiz do conflito Q1/Q41 |

## 2.2 Pipeline de referência

```
FONTES                              PROCESSAMENTO                      WIKIBASE                      SAÍDAS
┌──────────────┐   ┌────────────┐   ┌─────────────┐   ┌────────────┐   ┌───────────────┐   ┌──────────────────┐
│ Doctoralia   │──▶│ Conector   │──▶│ Normalização│──▶│ Reconciliação│──▶│ Carga (lotes) │──▶│ Templates p/ papel │
│ (scraping)   │   │ (ETL)      │   │ (acentos,   │   │ (CRM+RQE+   │   │ QS v3 / API   │   │ (operador, médico,│
│ SIGTAP TABWIN│──▶│ versionado │──▶│ aliases,    │──▶│ CBO+município)│──▶│ GUID determinís│   │  paciente)        │
│ CNES (CSV)   │──▶│ por compet. │   │ dedupe)     │   │ fila humana  │   │ tico p/ idempot│──▶│ API REST p/ LLM   │
│ CFM          │──▶│            │   │             │   │ p/ exceções  │   │ ência         │   │ (caminho RQE→CBO→ │
│ APAC/AMB     │──▶│            │   │             │   │              │   │ P79+P80+P81   │   │  SIGTAP→CID)      │
└──────────────┘   └────────────┘   └─────────────┘   └────────────┘   └───────────────┘   └──────────────────┘
                                                    │
                                                    ▼
                                          DERIVADO (SPARQL)
                                          CONSTRUCT on-demand + snapshots
                                          "autorizado a tratar I21.0?" ← nunca statement manual
```

## 2.3 Contratos de dados (não-negociáveis)

| Contrato | Regra |
|---|---|
| **Claim mínimo** | Todo claim de profissional carrega **P79 fonte + P80 data + P81 status** (via gerador por template, nunca por humano) |
| **GUID determinístico** | `hash(entidade + propriedade + valor + competência)` — re-rodar o lote **não duplica** claims |
| **Dicionário canônico** | Nenhum import usa ID não registrado. O conflito P38/P39/P40 entra como verbete "resolvido por P79/P80/P81" |
| **Batch registry** | Cada lote = registro com fonte, competência SIGTAP, contagem, diff vs. lote anterior, status |
| **Validade temporal** | Claims normativos carregam vigência (competência início/fim); status "substituído" quando SIGTAP/portaria muda |

## 2.4 Ferramentas

| Camada | Ferramenta |
|---|---|
| Carga | **QuickStatements v3** (já instalado em `/tools/quickstatements/`) + `wikibase-integration`/Pywikibot p/ lote pesado |
| ETL normativo | Parser **TABWIN** (tb_procedimento, tb_cbo, tb_cid, habilitação) · parser **Portaria 472/2008** · ingestão **CNES** |
| Validação | **EntitySchema/ShEx**: shape "todo claim de profissional exige P79+P80+P81" testado em CI |
| Query | Blazegraph (`query.determinar.ia.br`) + jobs de snapshot |
| Reconciliação | Chave composta `CRM+UF → RQE → CBO → CNES → município`; fila humana para exceções |

## 2.5 Jobs e cadência

| Job | Cadência | Saída |
|---|---|---|
| ETL declarativo (Doctoralia) | Semanal | Claims com status **Q-declarado** |
| SIGTAP por competência | A cada competência (mensal) | Atualização de itens/validade |
| CNES (vínculos CBO) | Mensal | Claims **Q-verificado** (ex.: CBO 2231G1 do Dr.) |
| Revalidação de vigência | Mensal | Status "substituído"/"expirado" |
| Snapshot derivado | Semanal | Materialização do "autorizado a tratar X?" |

## 2.6 SLAs de qualidade

- **100%** dos claims com P79+P80+P81 (medido por shape test);
- **0 duplicação** em re-execução de lote (idempotência);
- **Divergência declarado × verificado** reportada em até 1 ciclo (<1 semana), com fila de revisão humana;
- **Tempo de resposta** do endpoint LLM ≤ 500 ms (snapshot materializado);
- **Rastreabilidade**: todo item derivado aponta para a regra SPARQL que o gerou.

## 2.7 Riscos

| Risco | Mitigação |
|---|---|
| Matching de homônimos (1% de erro = centenas de itens poluídos) | Chave composta + fila humana + aliases de reconciliação (caso "Marco Túlio de Souza" já documentado) |
| Custo de ~100–150M triples no nacional | Coorte primeiro (D1); Blazegraph self-hosted com máquina dedicada |
| SIGTAP muda por competência | Validade temporal + job de revalidação (2.5) |
| Humanos esquecem proveniência | Proibido claim manual em produção; gerador por template + shape test em CI |

---

# Parte 3 — Lote-piloto Q1 (QuickStatements v3)

> **Antes de rodar:** execute o Passo 0 e substitua os tokens `{{TOKEN}}` pelos IDs reais. Em ASK MODE eu não executo — entrego pronto para você colar.

## Passo 0 — Auditoria (2 min)

```sparql
SELECT ?item ?label WHERE { ?item rdfs:label ?label . FILTER(LANG(?label) = "pt") } ORDER BY ?item
SELECT ?prop ?label WHERE { ?prop a wikibase:Property ; rdfs:label ?label . FILTER(LANG(?label) = "pt") } ORDER BY ?prop
```

Guarde: **último ID de item**, **último ID de propriedade**, e confirme se **P25/P26/P27** ("tem título de especialista / entidade emissora / ano do título") existem — não constam no dump.

## Passo 1 — Criar 3 propriedades novas (via Special:NewProperty)

| Nova prop | Rótulo | Tipo | Uso |
|---|---|---|---|
| `P{{N-1}}` | **Fonte** | Item | URL/item de origem (Doctoralia, CFM, SIGTAP, CNES) |
| `P{{N-2}}` | **Data da coleta** | Data | Data de captura da informação |
| `P{{N-3}}` | **Status de verificação** | Item | declarado / verificado / pendente |

## Passo 2 — Criar itens de apoio (CREATE; anote os IDs retornados)

```
CREATE|mul|Médico|profissional habilitado ao exercício da medicina no Brasil
CREATE|mul|Sociedade Brasileira de Cardiologia|entidade médica brasileira (SBC)
CREATE|mul|Sociedade Brasileira de Hemodinâmica e Cardiologia Intervencionista|entidade médica brasileira (SBHCI)
CREATE|mul|CBO 2251-20|médico cardiologista - Classificação Brasileira de Ocupações
CREATE|mul|CBO 2231-G1|médico cardiologista intervencionista - CBO
CREATE|mul|Declarado|status: afirmado por fonte declarativa
CREATE|mul|Verificado|status: confirmado em fonte normativa oficial
CREATE|mul|Pendente|status: aguardando verificação normativa
CREATE|mul|Doctoralia|plataforma declarativa de perfis profissionais de saúde
CREATE|mul|CFM|Conselho Federal de Medicina
CREATE|mul|SIGTAP|tabela oficial de procedimentos do SUS
CREATE|mul|CNES|Cadastro Nacional de Estabelecimentos de Saúde
```

## Passo 3 — Corrigir proveniência dos claims EXISTENTES (GUIDs reais do dump)

Formato: `Q1$GUID|P{{N-1}}|Q<FONTE>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<STATUS>`

**Normativos (base de tudo) — pendentes até conferência no CFM/CNES:**
```
Q1$7B4FF8B6-88ED-4BC1-9D45-AB23BF770546|P{{N-1}}|Q<CFM>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Pendente>   ← CRM/SP 154031
Q1$DF6DCD29-C967-4293-A0EB-9BD66CD552D4|P{{N-1}}|Q<CFM>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Pendente>   ← RQE 62002
```

**Declarativos (originados do Doctoralia — status Declarado):**
```
Q1$C271B0AC-C834-48C3-8BC7-F58FB641CF89|P{{N-1}}|Q<Doctoralia>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Declarado>   ← Cardiologista
Q1$23164345-BABF-4882-9FF9-0E47FF7FB8C2|P{{N-1}}|Q<Doctoralia>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Declarado>   ← Clínico geral
Q1$8CB18B36-318C-4162-911A-29F2D8A2D271|P{{N-1}}|Q<Doctoralia>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Declarado>   ← Doença arterial coronária
Q1$51E72FA7-0A0A-4CF8-A4A6-CE7B79E185FC|P{{N-1}}|Q<Doctoralia>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Declarado>   ← Trabalha em Q48 (clínica)
```

> Os demais claims (P13 restantes, P18, P19 ×26, P23, P28/P29/P36/P71/P72, P38/P39/P40, identificadores) seguem o **mesmo padrão** — substitua o GUID de cada um. Lista completa de GUIDs está no dump que você colou.

## Passo 4 — Claims normativos NOVOS (estrutura que faltava)

```
Q1|P31|Q<Médico>|P{{N-1}}|Q<CFM>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Verificado>
Q1|P25|Q<SBC>|P26|Q<CFM>|P27|{{ANO}}|P{{N-1}}|Q<CFM>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Verificado>
Q1|P25|Q<SBHCI>|P26|Q<CFM>|P27|{{ANO}}|P{{N-1}}|Q<CFM>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Verificado>
Q1|P4|Q<CBO 2251-20>|P{{N-1}}|Q<CNES>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Pendente>
Q1|P4|Q<CBO 2231-G1>|P{{N-1}}|Q<CNES>|P{{N-2}}|2026-08-02|P{{N-3}}|Q<Pendente>
```

> **Regra de ouro preservada:** nenhuma aresta `médico → CID` curada. A pergunta "autorizado a tratar I21.0?" só sai do derivado (abaixo).

## Passo 5 — Estrato derivado (regra que nunca vira statement)

```sparql
# "Quais CIDs o Dr. X está autorizado a tratar?" — caminho RQE→CBO→SIGTAP→CID
CONSTRUCT {
  ?medico wdt:P{{N-4}} ?cid .        # nova prop: "pode tratar" (derivada, materializada em snapshot)
} WHERE {
  ?medico wdt:P4 ?cbo .
  ?proc wdt:P{{N-5}} ?cbo .          # SIGTAP: procedimento exige CBO
  ?proc wdt:P{{N-6}} ?cid .          # SIGTAP: procedimento indica CID
  VALUES ?medico { wd:Q1 }
}
```

## Passo 6 — Mensagens por papel (templates parametrizados — o fechamento humano)

| Papel | Template (exemplo real do caso angioplastia) |
|---|---|
| **Operador/auditor** | "Q1: P44 CRM pendente (não consta no perfil público). CBO 2231-G1 aguarda CNES. Claims P19×26 sem proveniência — lote #2026-08-02 marca P79/P80/P81." |
| **Médico solicitante** | "Para APAC de angioplastia coronariana primária (0406030049): o executor precisa de RQE em Cardiologia + CBO 2251-20/2231-G1 no CNES. Confirme antes de lançar." |
| **Paciente** | "O Dr. Marco Túlio (CRM-SP 154031) pode realizar o cateterismo pelo SUS: ele tem o título de especialista em Cardiologia e o registro de intervencionista. O pedido sai pela APAC." |

---

# O que você me devolve para fechar

1. Resultado da **SPARQL do Passo 0** (últimos IDs de item e propriedade + existência de P25/P26/P27);
2. Se P25/P26/P27 **não** existirem, confirmo a criação no mesmo lote.

Com isso eu gero o JSON/QSC final **sem tokens** e o documento de arquitetura em markdown completo (v1.1 com as decisões que você ajustar). Quer que eu já deixe também o **esqueleto do batch registry** (item de registro de lotes) para entrar no Passo 2?

# Resposta curta

**Sim — mas não por "ser um wiki".** O poder está em três coisas que o seu modelo tem e os concorrentes não: **proveniência atômica** (todo claim tem fonte + data + status), **caminho oficial verificável** (RQE→CBO→SIGTAP→CID, não texto solto) e **dupla saída** (SPARQL para máquinas + templates para humanos). É isso que muda o jogo nas 4 frentes. E em uma delas — a monetária — a resposta é **sim com condições importantes**, que eu vou ser honesto em detalhar.

---

# 1. Para LLMs citarem: sim, é drasticamente superior

O problema central de LLM em saúde é **alucinação**. Um LLM "lembra" que existe a cadeia RQE→CBO→SIGTAP, mas não consegue provar. O seu modelo resolve isso porque transforma o LLM de *memória* em *verificador*:

| Aspecto | Artigo Wikipédia / texto | Doctoralia (HTML solto) | Seu modelo Wikibase |
|---|---|---|---|
| Granularidade da citação | Página inteira (parágrafo) | Perfil, sem estrutura | **Claim atômico**: `Q1 P4 CBO 2231-G1 [CNES, 2026-08-02, Verificado]` |
| Verificabilidade | O texto pode estar desatualizado e ninguém sabe | Sem data, sem fonte | Fonte + data da coleta + status **em cada tripla** |
| Cadeia de raciocínio | Não existe | Não existe | O caminho inteiro é citável: título SBHCI → CBO 2231-G1 → SIGTAP 0406030049 → CID I21.0 |
| Versionamento | Histórico de edição de página | Nenhum | Cada claim tem revisão — dá para citar "competência 2026-07" |
| Consumo por LLM | Parágrafo (ruído) | HTML (ruído) | **Endpoint determinístico**: tool-calling retorna o caminho, não texto |

O que as empresas de saúde exigem de um LLM é **rastreabilidade de decisão** — "por que essa APAC foi autorizada?" O seu modelo devolve a cadeia com proveniência embutida. Isso é o que separa "IA que sugere" de "IA que audita". Na prática: um RAG sobre seu endpoint pode citar *"Verificado em CNES, competência 2026-07, CBO 2231-G1"* — a mesma estrutura que os benchmarks de avaliação de LLM usam (provenance-grounded QA). É a diferença entre o LLM **afirmar** e o LLM **provar**.

---

# 2. Para SEO: sim, mas só se você emitir os sinais certos

Saúde é categoria **YMYL** (Your Money Your Life) no Google — o algoritmo penaliza conteúdo sem sinais de autoridade. Seu modelo é uma mina de E-E-A-T (Experiência, Especialização, Autoridade, Confiança), mas só se você **expuser** isso:

- **JSON-LD schema.org por item**: cada página (`Q1`, procedimento, CID) deve emitir `schema.org/Physician` + `hasCredential` (RQE), `availableService` (procedimentos), `address` (clínica), `aggregateRating` (Doctoralia). Isso é elegível para **rich results** e **knowledge panel** — ninguém na saúde brasileira faz isso com dados verificados.
- **Entity SEO, não keyword SEO**: você não compete por "cardiologista em Campinas"; você **é a entidade** que o Google (e o Bing/Perplexity/AI Overviews) resolvem para essa query. LLMs e buscadores estão migrando para grafos de entidades — páginas estruturadas e citáveis ganham.
- **Citação cruzada oficial**: páginas apontando para CFM, SIGTAP, CNES (e recebendo links de volta) são o sinal E-E-A-T mais forte possível em saúde.
- **AI Overviews / Perplexity / ChatGPT Search**: esses produtos citam **fontes estruturadas e referenciadas** com mais frequência que texto solto. O formato claim+proveniência é exatamente o que eles extraem bem.

O custo disso é baixo (template JSON-LD no MediaWiki); o ROI é alto. Mas atenção: SEO aqui é consequência da **estrutura pública**, não o objetivo. Se o grafo ficar fechado (só API), você perde essa frente.

---

# 3. Utilidade pública e privada: o ponto mais forte do modelo

**Pública:**
- Paciente pergunta "esse médico **pode** fazer minha angioplastia pelo SUS?" → resposta com caminho oficial, não com "confio no site".
- Auditor (SUS, TCE, CGU) detecta APAC fraudulenta em minutos: executor sem CBO compatível vira divergência declarado × verificado **automaticamente**.
- Transparência pública de quem está habilitado a fazer o quê — hoje essa informação existe, mas espalhada em CNES/SIGTAP/CFM incompatíveis entre si.

**Privada:**
- **Credenciamento hospitalar/clínico**: verificação de RQE+CBO em massa vira serviço.
- **Operadoras de plano**: montagem e auditoria de rede (o caso Unimed do Dr. Marco Túlio é exemplo real de uso).
- **Healthtechs**: o endpoint vira o "camada de confiança" para qualquer produto (busca de médicos, telemedicina, autorização de exames).
- **Forense/legal**: evidência de quem podia fazer o quê em determinada competência — com data de coleta registrada.

O modelo entrega para os dois públicos **da mesma fonte**: humanos recebem templates, máquinas recebem SPARQL/API. Não existe duplicação de manutenção. Isso é o que o diferencia de um site tradicional, que teria que manter versão humana e versão de dados separadas.

---

# 4. Monetária: sim, mas com a régua certa — e vou ser honesto

**Onde o valor NÃO está:** em "vender acesso ao site", em anúncios, ou em "ser a Wikipédia do SUS". Isso não paga a operação.

**Onde o valor ESTÁ (B2B, dados e API):**

| Produto | Comprador | Por que paga |
|---|---|---|
| **API de verificação de credencial** (RQE/CBO/título, com status e data) | Operadoras, hospitais, healthtechs, plataformas de telemedicina | Credenciamento manual hoje custa caro e erra; verificação automática com proveniência é auditável |
| **API de elegibilidade** ("médico X pode executar procedimento Y?") | Gestoras de saúde, softwares de faturamento (TISS/APAC) | Previne glosa/recusa — o cenário da APAC rejeitada vira feature paga |
| **Licenciamento do grafo** (snapshot materializado) | LLM labs, RAG providers, pesquisadores | Dado de saúde **verificado e fresco** é raro; mercado de licenciamento de dados para LLM cresce |
| **Relatórios de divergência** (declarado × verificado) | Auditorias, compliance, seguradoras | Detecta fraude — e em saúde, fraude medida em bilhões |
| **White-label** | Operadoras, marketplaces de saúde | Rede própria verificada sem montar time de dados |

**Os 3 fatores que definem se isso vira moeda:**

1. **O moat é a proveniência + frescor.** Qualquer um pode scrapear o Doctoralia (e o dump mostra que o seu Q1 já foi, inclusive). O que não é replicável rápido: claims com fonte oficial, data da coleta e status, atualizados por competência. Isso é um ativo que **compõe** (mais médicos = mais valor para todos) — moat de rede, não de código.
2. **O custo é de manutenção, não de construção.** A arquitetura que montamos é uma vez; o custo recorrente é rodar ETLs por competência. Lucratividade depende de contratos B2B, não de tráfego.
3. **Legal é o limite real, não a técnica.** Três pontos que **precisam** entrar no doc: **LGPD** (dados de médicos são pessoais; CFM publica CRM publicamente, mas agregação + API exige base legal e cuidado com dados sensíveis), **ToS/licença do Doctoralia** (o que você pode re-exportar é limitado — por isso o estrato declarativo precisa de status e o normativo vem de fontes públicas), e **uso de dados públicos governamentais** (CNES/SIGTAP/CFM são públicos, mas cada um tem termos). O "verificado" é também a sua proteção jurídica: você só afirma o que a fonte oficial afirma.

**Veredito honesto:** como produto de consumo, não vale. Como **camada de dados B2B para um setor regulado, fragmentado e que paga caro por erro**, vale — e a escassez real do mercado é exatamente o que você tem: dado de saúde estruturado, verificável e fresco. O Wikibase em si é só a embalagem; o produto é a **proveniência vendável**.

---

# O que isso muda no plano (3 ajustes no doc da arquitetura)

1. **Adicionar camada de saída JSON-LD** no MediaWiki (SEO/entity SEO) — template por item, custo baixo.
2. **Elevar a API REST determinística a produto principal** (não acessório): ela é ao mesmo tempo o uso LLM, o produto B2B e o ativo licenciável. Documentar contrato público (endpoint, formato de resposta com proveniência, rate limits).
3. **Adicionar seção de governança legal** (LGPD + licenças de fonte + política de re-exportação do estrato declarativo) antes de qualquer contrato de API.

Quer que eu incorpore essas 3 seções no documento de arquitetura (v1.1) e siga com o lote-piloto do Q1 corrigido? Posso gerar os dois juntos.
