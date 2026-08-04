cat > import_medico.sh << 'BASH_EOF'
#!/bin/bash
# ==============================================================================
# INGESTOR WIKIVENDAS/DETERMINAR - DKI (Production Ready - No P79 & Amazon S3)
# ==============================================================================
set -euo pipefail

ARQUIVO_JSON=${1:-}

if [ -z "$ARQUIVO_JSON" ] || [ ! -f "$ARQUIVO_JSON" ]; then
    echo "❌ ERRO: Arquivo JSON não informado ou não encontrado."
    echo "Uso: bash import_medico.sh template.json"
    exit 1
fi

python3 - "$ARQUIVO_JSON" << 'PY_EOF'
import sys
import os
import json
import requests
import re
import unicodedata
import time
import hashlib

ARQUIVO = sys.argv[1]
ARQUIVO_RELATORIO = "relatorio_itens_pendentes.txt"

# --- Configurações Hardcoded ---
API_URL = "https://determinar.ia.br/w/api.php"
SPARQL_ENDPOINT = "https://determinar.ia.br/query/sparql"
BOT_USER = "_____________________admin"
BOT_PASS = "65d5b021bc________________"

DATA_COLETA = time.strftime("%Y-%m-%d")
AMAZON_S3_BASE = "https://pixel-p1.s3.sa-east-1.amazonaws.com"

# --- ONTOLOGIA ABSOLUTA (Datatypes do Wikibase validados) ---
ONTOLOGIA = {
    "P1": "localMedia", "P2": "monolingualtext", "P3": "monolingualtext", "P4": "monolingualtext",
    "P5": "monolingualtext", "P6": "wikibase-item", "P7": "string", "P8": "monolingualtext",
    "P9": "monolingualtext", "P10": "wikibase-item", "P11": "wikibase-item", "P12": "wikibase-item",
    "P13": "wikibase-item", "P14": "wikibase-item", "P15": "wikibase-item", "P16": "wikibase-item",
    "P17": "wikibase-item", "P18": "wikibase-item", "P19": "wikibase-item", "P20": "wikibase-item",
    "P21": "wikibase-item", "P22": "wikibase-item", "P23": "wikibase-item", "P24": "monolingualtext",
    "P26": "monolingualtext", "P28": "monolingualtext", "P29": "monolingualtext", "P30": "string",
    "P31": "url", "P32": "url", "P33": "string", "P34": "string", "P35": "monolingualtext",
    "P36": "monolingualtext", "P37": "string", "P38": "quantity", "P39": "quantity",
    "P40": "quantity", "P41": "wikibase-item", "P42": "wikibase-item", "P43": "wikibase-item",
    "P44": "wikibase-item", "P45": "string", "P46": "url", "P47": "string", "P48": "wikibase-item",
    "P49": "time", "P50": "wikibase-item", "P51": "monolingualtext", "P52": "wikibase-item",
    "P53": "url", "P54": "string", "P55": "string", "P56": "monolingualtext", "P57": "url",
    "P59": "wikibase-item", "P60": "quantity", "P61": "external-id", "P62": "external-id",
    "P63": "external-id", "P64": "external-id", "P65": "external-id", "P66": "string",
    "P67": "string", "P68": "url", "P69": "url", "P70": "string", "P71": "wikibase-item",
    "P72": "string", "P73": "string", "P74": "string", "P75": "external-id", "P76": "string",
    "P77": "url", "P78": "url", "P80": "time", "P86": "string",
    "P87": "string", "P88": "wikibase-item", "P89": "wikibase-item", "P90": "wikibase-item",
    "P91": "wikibase-item", "P92": "string"
}

log_pendentes = []

def fold(s):
    if not s: return ""
    s = unicodedata.normalize("NFKD", str(s)).encode("ASCII", "ignore").decode("ASCII")
    return " ".join(s.split()).lower()

print(f"==> 1. Carregando dados de {ARQUIVO}...")
with open(ARQUIVO, "r", encoding="utf-8") as f:
    dados_brutos = json.load(f)

props_para_importar = {}
if "claims" in dados_brutos: props_para_importar.update(dados_brutos["claims"])
if "normativos_fora_de_escopo" in dados_brutos: props_para_importar.update(dados_brutos["normativos_fora_de_escopo"])

# Sanitização rigorosa: remove qualquer chave inválida ou vazia (como P---)
props_para_importar = {k: v for k, v in props_para_importar.items() if re.match(r'^P\d+$', k)}

doctoralia_id = str(props_para_importar.get("P64", {}).get("value", "")).strip()

nome_busca = ""
lista_p3 = props_para_importar.get("P3", {}).get("lista", [])
if lista_p3:
    for n in lista_p3:
        if not re.match(r'^(Dr\.|Dra\.|Prof\.)', str(n), re.IGNORECASE):
            nome_busca = str(n); break
    if not nome_busca: nome_busca = str(lista_p3[-1])

if not nome_busca:
    val_p2 = props_para_importar.get("P2", {}).get("value", "")
    nome_busca = val_p2[0] if isinstance(val_p2, list) else val_p2

# --- Sessão HTTP ---
SESSION = requests.Session()
login_token = SESSION.get(API_URL, params={"action": "query", "meta": "tokens", "type": "login", "format": "json"}).json()["query"]["tokens"]["logintoken"]
r_login = SESSION.post(API_URL, data={"action": "login", "lgname": BOT_USER, "lgpassword": BOT_PASS, "lgtoken": login_token, "format": "json"}).json()
if r_login.get("login", {}).get("result") != "Success":
    print(f"❌ ERRO LOGIN: {json.dumps(r_login)}")
    sys.exit(1)
csrf_token = SESSION.get(API_URL, params={"action": "query", "meta": "tokens", "type": "csrf", "format": "json"}).json()["query"]["tokens"]["csrftoken"]
print("  ✅ Sessão autenticada.")

# --- Reconciliação por Rótulos e Aliases ---
def reconciliar_por_rotulos_aliases(nome_alvo):
    if not nome_alvo: return None
    alvo_dobrado = fold(nome_alvo)
    r_busca = SESSION.get(API_URL, params={"action": "wbsearchentities", "search": nome_alvo, "language": "pt-br", "format": "json", "limit": 10}).json()
    for item in r_busca.get("search", []):
        qid = item.get("id")
        label = item.get("label", "")
        aliases = item.get("aliases", [])
        if fold(label) == alvo_dobrado or any(fold(a) == alvo_dobrado for a in aliases):
            return qid
    return None

q_id_medico = reconciliar_por_rotulos_aliases(nome_busca)

if not q_id_medico:
    print(f"❌ ERRO CRÍTICO: Entidade para '{nome_busca}' não encontrada[cite: 1]. Operação abortada.")
    sys.exit(1)

print(f"==> 2. Status Entidade: ENCONTRADO -> {q_id_medico} (Modo UPDATE estrito)")

claims_existentes = {}
r_ent = SESSION.get(API_URL, params={"action": "wbgetentities", "ids": q_id_medico, "format": "json"}).json()
if "entities" in r_ent and q_id_medico in r_ent["entities"]:
    claims_existentes = r_ent["entities"][q_id_medico].get("claims", {})

def existe_propriedade(prop_id, valor_novo, r_type):
    if prop_id not in claims_existentes: return False
    for stmt in claims_existentes[prop_id]:
        dv = stmt.get("mainsnak", {}).get("datavalue", {})
        if r_type in ["string", "url", "external-id", "localMedia"]:
            if dv.get("value") == valor_novo: return True
        elif r_type == "monolingualtext":
            if dv.get("value", {}).get("text") == valor_novo: return True
        elif r_type == "quantity":
            if str(dv.get("value", {}).get("amount", "")) in (valor_novo, f"+{valor_novo}"): return True
        elif r_type == "time":
            if str(dv.get("value", {}).get("time", "")) == str(valor_novo): return True
        elif r_type == "wikibase-item":
            if str(dv.get("value", {}).get("numeric-id")) == str(valor_novo): return True
    return False

cache_itens, cache_imagens = {}, {}

def resolver_qid(t_str):
    t_str = str(t_str).strip()
    if re.match(r'^Q\d+$', t_str, re.IGNORECASE): return int(t_str.upper().replace("Q", ""))
    if t_str in cache_itens: return cache_itens[t_str]
    res = SESSION.get(API_URL, params={"action": "wbsearchentities", "search": t_str, "language": "pt-br", "format": "json", "limit": 3}).json()
    for item in res.get("search", []):
        if fold(item.get("label", "")) == fold(t_str):
            cache_itens[t_str] = int(item["id"].replace("Q", ""))
            return cache_itens[t_str]
    return None

def gerar_nome_arquivo_foto(nome, url):
    limpo = re.sub(r'^(Dr\.|Dra\.|Prof\.)\s*', '', nome, flags=re.IGNORECASE)
    nfkd = unicodedata.normalize("NFKD", limpo)
    sem_acento = "".join([c for c in nfkd if not unicodedata.combining(c)])
    alpha_num = re.sub(r'[^a-zA-Z0-9\s]', '', sem_acento)
    palavras = alpha_num.split()
    slug_base = "".join([p.capitalize() for p in palavras]) or "MedicoPerfil"
    if not slug_base.startswith("Dr"): slug_base = "Dr" + slug_base
    ext = os.path.splitext(url.split("?")[0])[1].lstrip(".").lower()
    if ext not in ("jpg", "jpeg", "png", "webp"): ext = "jpg"
    return f"{slug_base}.{ext}"

def baixar_e_upar_amazon_imagem(url_relativa, nome_medico):
    url_limpa = url_relativa.strip()
    if url_limpa.startswith("http"):
        if "amazonaws.com" in url_limpa:
            url_final = url_limpa
        else:
            url_final = AMAZON_S3_BASE + "/" + url_limpa.lstrip("/")
    else:
        url_final = AMAZON_S3_BASE + "/" + url_limpa.lstrip("/")

    if url_final in cache_imagens: return cache_imagens[url_final]
    try:
        resp = requests.get(url_final, timeout=15)
        resp.raise_for_status()
        nome_arquivo = gerar_nome_arquivo_foto(nome_medico, url_final)
        r_check = SESSION.get(API_URL, params={"action": "query", "titles": f"File:{nome_arquivo}", "prop": "imageinfo", "format": "json"}).json()
        if "imageinfo" not in list(r_check.get("query", {}).get("pages", {}).values())[0]:
            SESSION.post(API_URL, data={
                "action": "upload", "filename": nome_arquivo, "token": csrf_token,
                "ignorewarnings": 1, "comment": f"Upload Amazon S3 ({DATA_COLETA})", "format": "json"
            }, files={"file": (nome_arquivo, resp.content)})
        cache_imagens[url_final] = nome_arquivo
        return nome_arquivo
    except Exception as e:
        log_pendentes.append(f"[P1] Falha ao baixar do Amazon S3 ('{url_final}'): {e}")
        return None

# --- Qualificadores padrão (EXCLUINDO P79) ---
base_quals = {
    "P80": [{"snaktype": "value", "property": "P80", "datavalue": {"value": {"time": f"+{DATA_COLETA}T00:00:00Z", "timezone": 0, "before": 0, "after": 0, "precision": 11, "calendarmodel": "http://www.wikidata.org/entity/Q1985727"}, "type": "time"}}]
}

claims_payload = {}
print("==> 3. Construindo Payload sem P79 e integrando com Amazon S3...")

for prop, info in props_para_importar.items():
    if prop not in ONTOLOGIA:
        continue
    real_datatype = ONTOLOGIA[prop]
    lista = info.get("lista", [])
    if not lista:
        if info.get("value") and str(info.get("value")).lower() != "pendente": lista = [info.get("value")]
        else: continue

    snaks = []
    for val in lista:
        if str(val).lower() == "pendente" or not val: continue
        datavalue, comparador = None, None

        if prop in {"P1", "P45", "P47"}:
            nome_arquivo = baixar_e_upar_amazon_imagem(str(val), nome_busca)
            if not nome_arquivo: continue
            comparador = nome_arquivo
            datavalue = {"value": nome_arquivo, "type": "string"}
        
        elif real_datatype == "url":
            url_str = str(val).strip()
            if not re.match(r'^https?://', url_str, re.IGNORECASE): continue
            comparador = url_str
            datavalue = {"value": comparador, "type": "string"}

        elif real_datatype == "time":
            val_str = str(val).strip()
            time_val = f"+{val_str}T00:00:00Z" if re.match(r'^\d{4}-\d{2}-\d{2}$', val_str) else (val_str if val_str.startswith("+") else f"+{val_str}")
            comparador = time_val
            datavalue = {"value": {"time": time_val, "timezone": 0, "before": 0, "after": 0, "precision": 11, "calendarmodel": "http://www.wikidata.org/entity/Q1985727"}, "type": "time"}

        elif real_datatype == "monolingualtext":
            comparador = f"Q:{val.get('pergunta','')} A:{val.get('resposta','')}" if isinstance(val, dict) else str(val).strip()
            if len(comparador) > 1650: comparador = comparador[:1647] + "..."
            datavalue = {"value": {"text": comparador, "language": "pt-br"}, "type": "monolingualtext"}
        
        elif real_datatype == "quantity":
            num = str(val).replace(",", ".")
            if not num.startswith("+") and not num.startswith("-"): num = "+" + num
            comparador = num
            datavalue = {"value": {"amount": comparador, "unit": "1"}, "type": "quantity"}
        
        elif real_datatype == "wikibase-item":
            qid_num = resolver_qid(val)
            if qid_num:
                comparador = str(qid_num)
                datavalue = {"value": {"entity-type": "item", "numeric-id": qid_num}, "type": "wikibase-entityid"}
            else:
                log_pendentes.append(f"[{prop}] Item inexistente no Grafo: '{val}'[cite: 1].")
                continue
        else:
            comparador = str(val).strip()
            datavalue = {"value": comparador, "type": "string"}

        if datavalue and not existe_propriedade(prop, comparador, real_datatype):
            snaks.append({
                "mainsnak": {"snaktype": "value", "property": prop, "datatype": real_datatype, "datavalue": datavalue},
                "type": "statement", "rank": "normal", "qualifiers": base_quals
            })

    if snaks: claims_payload[prop] = snaks

if log_pendentes:
    with open(ARQUIVO_RELATORIO, "a", encoding="utf-8") as rf:
        rf.write(f"\n--- Relatório de Itens Inexistentes / Pendências ({nome_busca} - {DATA_COLETA}) ---\n")
        for lp in log_pendentes: rf.write(f"  {lp}\n")
    print(f"  📝 {len(log_pendentes)} itens inexistentes listados em {ARQUIVO_RELATORIO}[cite: 1].")

if not claims_payload:
    print("  ✅ [IDEMPOTENTE] Item atualizado. Nenhuma mutação necessária.")
    sys.exit(0)

print(f"==> 4. Gravando alterações (ID: {q_id_medico}) sem P79...")
nome_label = props_para_importar.get("P2", {}).get("value", nome_busca)
nome_label = nome_label[0] if isinstance(nome_label, list) else nome_label

payload_edit = {
    "action": "wbeditentity",
    "id": q_id_medico,
    "data": json.dumps({
        "labels": {"pt-br": {"language": "pt-br", "value": str(nome_label)}},
        "claims": claims_payload
    }),
    "token": csrf_token,
    "format": "json"
}

res = SESSION.post(API_URL, data=payload_edit).json()

if "entity" in res:
    print(f"🚀 [SUCESSO] Entidade atualizada com sucesso: {res['entity']['id']} | Nome: {nome_label}")
else:
    print(f"❌ [ERRO AO SALVAR]: {json.dumps(res, indent=2, ensure_ascii=False)}")
    sys.exit(1)
PY_EOF

chmod +x import_medico.sh
echo "✅ Script configurado com sucesso e permissões aplicadas!"
BASH_EOF

bash import_medico.sh template.json
