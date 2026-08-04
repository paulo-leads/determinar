Como **Engenheiro de Dados** operando em nível C-level, trago a análise ontológica semântica precisa de como os termos não encontrados (`pendências`) no relatório de match se encaixam na arquitetura DKI da **Dra. Camila** (e do ecossistema médico da `determinar.ia.br`), seguindo estritamente o padrão canônico estabelecido no item modelo **Q1**.

---

### Análise Ontológica Semântica: Onde se encaixam os termos da Dra. Camila?

Analisando o relatório de match, os termos não encontrados dividem-se em **4 macro-categorias ontológicas** fundamentais do Wikibase. Cada uma delas deve ser mapeada para propriedades específicas da nossa ontologia central:

#### 1. Identificação, Credenciamento e Metadados Pessoais

* **Termo:** `'Dra. Camila Nicolela Geraldo Martins'`
* **Encaixe Ontológico:** Nome principal da entidade (`P2` / Rótulo canônico).


* **Termos:** `'Residência em Clínica Médica pela Faculdade de Medicina de Catanduva'`, `'Residência em Cardiologia pela Faculdade de Medicina de São José do Rio Preto'`, `'Graduação em Medicina pela Universidade Federal de Ouro Preto'`, `'Mestre em Ciências Médicas pela UNICAMP'`, `'Especialista em Imagem Cardiovascular pela UNICAMP'`
* **Encaixe Ontológico:** Instituições de formação, acadêmicas e especializações (`P15` - *Educated At* / Aluno de).

  cat > injetar_formacao_q1292_v2.sh << 'BASH_EOF'
#!/bin/bash
# ==============================================================================
# INGESTOR WIKIVENDAS/DETERMINAR - Injeção de Formação Acadêmica (Q1292 - P15)
# ==============================================================================
set -euo pipefail

python3 - << 'PY_EOF'
import requests
import json
import time

API_URL = "https://determinar.ia.br/w/api.php"
BOT_USER = "n"
BOT_PASS = "65d"
TARGET_QID = "Q1292"
DATA_COLETA = time.strftime("%Y-%m-%d")

session = requests.Session()

print("==> 1. Autenticando na API do Wikibase...")
r_token = session.get(API_URL, params={"action": "query", "meta": "tokens", "type": "login", "format": "json"}).json()
login_token = r_token["query"]["tokens"]["logintoken"]

r_login = session.post(API_URL, data={
    "action": "login", "lgname": BOT_USER, "lgpassword": BOT_PASS, "lgtoken": login_token, "format": "json"
}).json()

if r_login.get("login", {}).get("result") != "Success":
    print(f"❌ Erro de login: {json.dumps(r_login, ensure_ascii=False)}")
    exit(1)

csrf_token = session.get(API_URL, params={"action": "query", "meta": "tokens", "type": "csrf", "format": "json"}).json()["query"]["tokens"]["csrftoken"]
print("  ✅ Sessão autenticada e Token CSRF obtido.")

def resolver_ou_criar_instituicao(session, csrf, nome_inst):
    # Tenta buscar pelo rótulo exato
    res = session.get(API_URL, params={"action": "wbsearchentities", "search": nome_inst, "language": "pt-br", "format": "json", "limit": 1}).json()
    search_results = res.get("search", [])
    if search_results:
        qid_str = search_results[0]["id"]
        return int(qid_str.replace("Q", ""))
    
    # Se não existir, cria a instituição no grafo para garantir integridade referencial
    print(f"  ℹ️ Instituição '{nome_inst}' não encontrada. Criando nova entidade...")
    data_criacao = {
        "labels": {"pt-br": {"language": "pt-br", "value": nome_inst}, "mul": {"language": "mul", "value": nome_inst}},
        "descriptions": {"pt-br": {"language": "pt-br", "value": "Instituição de ensino e formação acadêmica"}}
    }
    resp_create = session.post(API_URL, data={
        "action": "wbeditentity", "new": "item", "data": json.dumps(data_criacao), "token": csrf, "format": "json"
    }).json()
    
    if "entity" in resp_create:
        novo_qid = resp_create["entity"]["id"]
        print(f"  ✅ Criada com sucesso: {novo_qid} -> {nome_inst}")
        return int(novo_qid.replace("Q", ""))
    else:
        print(f"  ❌ Erro ao criar instituição {nome_inst}: {resp_create}")
        return None

print(f"==> 2. Processando formações acadêmicas para {TARGET_QID}...")

# Dados estruturados da formação da Dra. Camila
formacoes = [
    {"instituicao": "Universidade Federal de Ouro Preto", "titulo": "Graduação em Medicina"},
    {"instituicao": "Faculdade de Medicina de Catanduva", "titulo": "Residência em Clínica Médica"},
    {"instituicao": "Faculdade de Medicina de São José do Rio Preto", "titulo": "Residência em Cardiologia"},
    {"instituicao": "UNICAMP", "titulo": "Mestre em Ciências Médicas"},
    {"instituicao": "UNICAMP", "titulo": "Especialista em Imagem Cardiovascular"}
]

# Recupera claims atuais de Q1292
r_ent = session.get(API_URL, params={"action": "wbgetentities", "ids": TARGET_QID, "format": "json"}).json()
entity_data = r_ent.get("entities", {}).get(TARGET_QID, {})
claims = entity_data.get("claims", {})

if "P15" not in claims:
    claims["P15"] = []

# Extrai QIDs existentes em P15 para lógica de antidedup (evitar duplicatas)
existentes_p15 = set()
for stmt in claims.get("P15", []):
    dv = stmt.get("mainsnak", {}).get("datavalue", {})
    if dv.get("type") == "wikibase-entityid":
        existentes_p15.add(dv.get("value", {}).get("numeric-id"))

adicionados = 0
for form in formacoes:
    inst_name = form["instituicao"]
    titulo_name = form["titulo"]
    
    inst_qid = resolver_ou_criar_instituicao(session, csrf_token, inst_name)
    if not inst_qid:
        continue
        
    # Antidedup: se a instituição já estiver vinculada em P15, pula para evitar duplicidade exata
    if inst_qid in existentes_p15 and inst_name != "UNICAMP": # UNICAMP tem múltiplas formações (Mestrado e Especialista)
        print(f"  🔄 [Antidedup] Vínculo com {inst_name} (Q{inst_qid}) já existe. Ignorando duplicata.")
        continue

    # Qualificadores padrão (EXCLUINDO P79 estritamente)
    qualificadores = {
        "P80": [{"snaktype": "value", "property": "P80", "datavalue": {"value": {"time": f"+{DATA_COLETA}T00:00:00Z", "timezone": 0, "before": 0, "after": 0, "precision": 11, "calendarmodel": "http://www.wikidata.org/entity/Q1985727"}, "type": "time"}, "datatype": "time"}]
    }
    
    # Qualifica o título/especialidade se houver correspondência ou cria o nó de título
    titulo_qid = resolver_ou_criar_instituicao(session, csrf_token, titulo_name)
    if titulo_qid:
        qualificadores["P43"] = [{"snaktype": "value", "property": "P43", "datavalue": {"value": {"entity-type": "item", "numeric-id": titulo_qid}, "type": "wikibase-entityid"}, "datatype": "wikibase-item"}]

    novo_statement = {
        "mainsnak": {
            "snaktype": "value",
            "property": "P15",
            "datatype": "wikibase-item",
            "datavalue": {
                "value": {"entity-type": "item", "numeric-id": inst_qid},
                "type": "wikibase-entityid"
            }
        },
        "type": "statement",
        "rank": "normal",
        "qualifiers": qualificadores
    }
    
    claims["P15"].append(novo_statement)
    existentes_p15.add(inst_qid)
    adicionados += 1

if adicionados > 0:
    print(f"==> 3. Gravando {adicionados} novas declarações de formação em {TARGET_QID}...")
    payload_edit = {
        "action": "wbeditentity",
        "id": TARGET_QID,
        "data": json.dumps({"claims": claims}),
        "token": csrf_token,
        "format": "json"
    }
    res_edit = session.post(API_URL, data=payload_edit).json()
    if "entity" in res_edit:
        print(f"🚀 [SUCESSO] Entidade {TARGET_QID} atualizada com sucesso com as formações acadêmicas via P15!")
    else:
        print(f"❌ [ERRO AO SALVAR]: {json.dumps(res_edit, ensure_ascii=False)}")
else:
    print("✅ [IDEMPOTENTE] Nenhuma nova formação para adicionar (todas já mapeadas).")

PY_EOF
BASH_EOF

chmod +x injetar_formacao_q1292_v2.sh
bash injetar_formacao_q1292_v2.sh




* **Termos:** `'Título de especialista em Cardiologia pela SBC - 2021'`, `'Reconhecimento pela performance e engajamento na Doctoralia, tornando a experiência em saúde mais humana.'`
* **Encaixe Ontológico:** Prêmios, certificações e reconhecimentos profissionais (`P22` - *Recognition*).


* **Termos:** `'Médica assistente da Unidade Coronária do Hospital Centro Médico de Campinas'`
* **Encaixe Ontológico:** Vínculo profissional e local de atuação assistencial (`P14` - *Works For* / Organização).


* **Termo:** URLs de Avatar/Foto e Identificadores (ex: `[https://www.doctoralia.com.br/doctor/avatar/](https://www.doctoralia.com.br/doctor/avatar/)...`)
* **Encaixe Ontológico:** Imagem de Perfil (`P1` - *Profile Photo* via bucket S3 da Amazon) e URLs de Redes Sociais/Perfis (`P57` - *Social Media* / `P64` - *Doctoralia ID*).



#### 2. Condições Clínicas Tratadas e Patologias (Cardiologia)

* **Termos:** `'Insuficiência Da Valva Mitral'`, `'Insuficiência Da Valva Aórtica'`, `'Prolapso Da Valva Mitral'`, `'Estenose Da Valva Mitral'`, `'Estenose aórtica valvar'`, `'Cardiomiopatia Dilatada'`, `'Cardiomiopatia Alcoólica'`, `'Cardiomiopatia Chagásica'`, `'Miocardite'`, `'Hipertensão arterial'`, `'Hipertensão resistente'`, `'Hipertensão'`, `'Colesterol alto'`, `'Hipercolesterolemia (Níveis elevados de colesterol)'`, `'Disfunção Ventricular Esquerda'`, `'Disfunção Ventricular'`, `'Infarto'`, `'Arritmia'`, `'Taquicardia'`, `'Bradicardia'`, `'Taquicardia Sinusal'`, `'Taquicardia Paroxística'`, `'Taquicardia Supraventricular'`, `'Taquicardia Ectópica Atrial'`, `'Flutter Atrial'`, `'Arritmia Sinusal'`, `'Bloqueio Cardíaco'`, `'Aterosclerose'`, `'Dor No Peito'`, `'Dor torácica'`, `'Sopros Cardíacos'`, `'Anormalidades Cardiovasculares'`
* **Encaixe Ontológico:** Doenças, síndromes e condições tratadas pelo especialista (`P18` - *Treated Condition*). Estes termos devem virar entidades QIDs próprias no grafo para permitir inferências semânticas e consultas SPARQL avançadas.

cat > curadoria_avancada_q1292.sh << 'BASH_EOF'
#!/bin/bash
# ==============================================================================
# INGESTOR WIKIVENDAS/DETERMINAR - Curadoria Avançada Q1292 (Case-Insensitive & Antidedup)
# ==============================================================================
set -euo pipefail

python3 - << 'PY_EOF'
import requests
import json
import time
import unicodedata
import re

API_URL = "https://determinar.ia.br/w/api.php"
BOT_USER = "Determinaradmin"
BOT_PASS = "65d5b021bc5bf3ad160079f5"
TARGET_QID = "Q1292"
DATA_COLETA = time.strftime("%Y-%m-%d")

session = requests.Session()

print("==> 1. Autenticando na API do Wikibase...")
r_token = session.get(API_URL, params={"action": "query", "meta": "tokens", "type": "login", "format": "json"}).json()
login_token = r_token["query"]["tokens"]["logintoken"]

r_login = session.post(API_URL, data={
    "action": "login", "lgname": BOT_USER, "lgpassword": BOT_PASS, "lgtoken": login_token, "format": "json"
}).json()

if r_login.get("login", {}).get("result") != "Success":
    print(f"❌ Erro de login: {json.dumps(r_login, ensure_ascii=False)}")
    exit(1)

csrf_token = session.get(API_URL, params={"action": "query", "meta": "tokens", "type": "csrf", "format": "json"}).json()["query"]["tokens"]["csrftoken"]
print("  ✅ Sessão autenticada e Token CSRF obtido.")

def fold(s):
    if not s: return ""
    s = unicodedata.normalize("NFKD", str(s)).encode("ASCII", "ignore").decode("ASCII")
    return " ".join(s.split()).lower()

def resolver_ou_criar_entidade(session, csrf, nome_entidade, descricao="Entidade do grafo semântico"):
    alvo_normalizado = fold(nome_entidade)
    
    # Busca por relevância na API
    res = session.get(API_URL, params={"action": "wbsearchentities", "search": nome_entidade, "language": "pt-br", "format": "json", "limit": 5}).json()
    for item in res.get("search", []):
        if fold(item.get("label", "")) == alvo_normalizado or any(fold(a) == alvo_normalizado for a in item.get("aliases", [])):
            qid_str = item["id"]
            return int(qid_str.replace("Q", ""))
            
    # Se não encontrar case-insensitive, cria nova entidade com segurança antiduplicação
    print(f"  ℹ️ Entidade '{nome_entidade}' não encontrada no grafo. Criando nova...")
    data_criacao = {
        "labels": {"pt-br": {"language": "pt-br", "value": nome_entidade}, "mul": {"language": "mul", "value": nome_entidade}},
        "descriptions": {"pt-br": {"language": "pt-br", "value": descricao}}
    }
    resp_create = session.post(API_URL, data={
        "action": "wbeditentity", "new": "item", "data": json.dumps(data_criacao), "token": csrf, "format": "json"
    }).json()
    
    if "entity" in resp_create:
        novo_qid = resp_create["entity"]["id"]
        print(f"  ✅ Criada com sucesso: {novo_qid} -> {nome_entidade}")
        return int(novo_qid.replace("Q", ""))
    else:
        # Tratamento de concorrência/conflito de label caso ocorra em paralelo
        print(f"  ⚠️ Aviso ao criar '{nome_entidade}': {resp_create}. Tentando re-buscar...")
        res_retry = session.get(API_URL, params={"action": "wbsearchentities", "search": nome_entidade, "language": "pt-br", "format": "json", "limit": 1}).json()
        if res_retry.get("search", []):
            return int(res_retry["search"][0]["id"].replace("Q", ""))
        return None

print(f"==> 2. Carregando dados atuais da entidade {TARGET_QID}...")
r_ent = session.get(API_URL, params={"action": "wbgetentities", "ids": TARGET_QID, "format": "json"}).json()
entity_data = r_ent.get("entities", {}).get(TARGET_QID, {})
claims = entity_data.get("claims", {})

# Qualificadores padrão estritamente sem P79 (Apenas P80 Data da Coleta)
base_quals = {
    "P80": [{"snaktype": "value", "property": "P80", "datavalue": {"value": {"time": f"+{DATA_COLETA}T00:00:00Z", "timezone": 0, "before": 0, "after": 0, "precision": 11, "calendarmodel": "http://www.wikidata.org/entity/Q1985727"}, "type": "time"}}]
}

def adicionar_claim_se_nao_existe(propriedade, target_numeric_id, qualifiers=None):
    if propriedade not in claims:
        claims[propriedade] = []
        
    # Verificação anti-duplicação (antidedup)
    for stmt in claims[propriedade]:
        dv = stmt.get("mainsnak", {}).get("datavalue", {})
        if dv.get("type") == "wikibase-entityid" and dv.get("value", {}).get("numeric-id") == target_numeric_id:
            return False # Já existe, não duplica
            
    novo_statement = {
        "mainsnak": {
            "snaktype": "value",
            "property": propriedade,
            "datatype": "wikibase-item",
            "datavalue": {
                "value": {"entity-type": "item", "numeric-id": target_numeric_id},
                "type": "wikibase-entityid"
            }
        },
        "type": "statement",
        "rank": "normal",
        "qualifiers": qualifiers if qualifiers else base_quals
    }
    claims[propriedade].append(novo_statement)
    return True

mutacoes = 0

# --- A. Vínculo Profissional / Hospitalar ---
# Hospital Centro Médico de Campinas -> P14 (Works For) e P91 (Affiliated Facility)[cite: 2]
hospital_nome = "Hospital Centro Médico de Campinas"
hosp_qid = resolver_ou_criar_entidade(session, csrf_token, hospital_nome, "Instituição de saúde e hospital")
if hosp_qid:
    if adicionar_claim_se_nao_existe("P14", hosp_qid):
        mutacoes += 1
        print(f"  + Adicionado P14 (Works For) -> {hospital_nome} (Q{hosp_qid})")
    if adicionar_claim_se_nao_existe("P91", hosp_qid):
        mutacoes += 1
        print(f"  + Adicionado P91 (Affiliated Facility) -> {hospital_nome} (Q{hosp_qid})")

# --- B. Condições Clínicas Tratadas (P18 - Treated Condition) ---
condicoes_tratadas = [
    'Insuficiência Da Valva Mitral', 'Insuficiência Da Valva Aórtica', 'Prolapso Da Valva Mitral', 
    'Estenose Da Valva Mitral', 'Estenose aórtica valvar', 'Cardiomiopatia Dilatada', 
    'Cardiomiopatia Alcoólica', 'Cardiomiopatia Chagásica', 'Miocardite', 'Hipertensão arterial', 
    'Hipertensão resistente', 'Hipertensão', 'Colesterol alto', 'Hipercolesterolemia (Níveis elevados de colesterol)', 
    'Disfunção Ventricular Esquerda', 'Disfunção Ventricular', 'Infarto', 'Arritmia', 'Taquicardia', 
    'Bradicardia', 'Taquicardia Sinusal', 'Taquicardia Paroxística', 'Taquicardia Supraventricular', 
    'Taquicardia Ectópica Atrial', 'Flutter Atrial', 'Arritmia Sinusal', 'Bloqueio Cardíaco', 
    'Aterosclerose', 'Dor No Peito', 'Dor torácica', 'Sopros Cardíacos', 'Anormalidades Cardiovasculares'
]

print(f"==> 3. Processando {len(condicoes_tratadas)} condições clínicas para P18 (Treated Condition)...")
for cond in condicoes_tratadas:
    cond_qid = resolver_ou_criar_entidade(session, csrf_token, cond, "Condição médica, patologia ou achado clínico")
    if cond_qid:
        if adicionar_claim_se_nao_existe("P18", cond_qid):
            mutacoes += 1
            print(f"  + Adicionado P18 -> {cond} (Q{cond_qid})")

if mutacoes > 0:
    print(f"==> 4. Gravando {mutacoes} novas mutações limpas em {TARGET_QID}...")
    payload_edit = {
        "action": "wbeditentity",
        "id": TARGET_QID,
        "data": json.dumps({"claims": claims}),
        "token": csrf_token,
        "format": "json"
    }
    res_edit = session.post(API_URL, data=payload_edit).json()
    if "entity" in res_edit:
        print(f"🚀 [SUCESSO] Entidade {TARGET_QID} atualizada com sucesso com vínculos e patologias!")
    else:
        print(f"❌ [ERRO AO SALVAR]: {json.dumps(res_edit, ensure_ascii=False)}")
else:
    print("✅ [IDEMPOTENTE] Nenhuma alteração necessária. A base já está perfeitamente sincronizada.")

PY_EOF
BASH_EOF

chmod +x curadoria_avancada_q1292.sh
bash curadoria_avancada_q1292.sh


#### 3. Convênios e Operadoras de Saúde Suplementar

* **Termos:** `'Sul América'`, `'Porto Seguro'`, `'Notredame'`, `'Intermedica'`, `'Odonto'`, `'Prevent'`, `'Bradesco'`, `'Cassi'`, `'Allianz'`
* **Encaixe Ontológico:** Convênios e planos de saúde aceitos pelo profissional ou estabelecimento (`P23` - *Accepted Insurance* / `P37` - *Insurance Operator*).



#### 4. Produção Científica, Publicações e Evidências

* **Termos:** `'Cardiac Magnetic Resonance Imaging in Fabry Disease'`, `'Possíveis Mecanismos dos Inibidores de SGLT2 na Insuficiência Cardíaca (ABC Heart Fail Cardiomyop. 2021; 1(1):33-43)'`, `'Capítulo sobre Métodos de Imagem Cardíaca (INSUFIC...'`
* **Encaixe Ontológico:** Publicações científicas, artigos e referências bibliográficas vinculadas ao perfil do médico (`P77` - *Scientific Publications* / `P52` - *Bibliographic Reference*).



---
