# Ingestão Base — Paulo Leads

Script de ingestão semântica que injeta na Wikibase a malha de estabelecimentos
de saúde de Campinas e vincula os médicos cardiologistas a esses locais.

- **CNPJ** do estabelecimento → propriedade `P7`
- **CNES** do estabelecimento → propriedade `P92`
- **Razão Social** → descrição do item do estabelecimento
- **Médico** → entidade única no grafo
- **CBO** do médico → qualificador `P88` da propriedade de vínculo `P91`

---

## O mantra

> **Toda entidade no grafo (médico ou estabelecimento) é única. Antes de criar,
> sempre pergunte se ela já existe — e nunca use um identificador composto que
> misture a entidade com o contexto onde ela aparece.**

Isso existe porque foi exatamente o oposto disso que causou a falha original em produção.

### O bug histórico (por que esse mantra existe)

A primeira versão do script deduplicava o médico assim:

```python
med_key = f"{nome_med}_{cbo}_{unkey}"   # ERRADO
```

`unkey` é o identificador do **estabelecimento**. Como um cardiologista pode
atuar em várias clínicas, a mesma pessoa gerava chaves diferentes a cada
estabelecimento novo — o script nunca reconhecia que o médico já existia e
tentava **criar o item de novo**, com o mesmo rótulo. A Wikibase rejeitava
corretamente com:

```
wikibase-validator-label-with-description-conflict
Item Q813 already has label "CARLOS ROBERTO FERNANDES" associated with
language code pt-br, using the same description text.
```

O bug tinha ainda uma segunda camada: mesmo corrigindo a chave, não existia
nenhuma rotina para **adicionar um vínculo novo a um médico já existente** — o
script só sabia criar, nunca vincular incrementalmente. Um médico em 5
clínicas diferentes deveria gerar 1 item + 5 claims `P91`; o bug fazia (na
melhor das hipóteses) 1 item e descartava as outras 4 aparições, ou (na pior)
tentava criar 5 itens e falhava em 4.

---

## A lógica corrigida

Para **cada entidade** (médico OU estabelecimento), nessa ordem estrita:

1. **Cache local primeiro.** Rápido, sem chamada de rede. Se achou o QID, pula
   direto pro passo 3 — proibido criar.
2. **Fallback via API (`wbsearchentities`) antes de desistir e criar.** O
   cache local pode estar desatualizado; a Wikibase é a fonte de verdade. Se
   achou, registra no cache e pula pro passo 3.
3. **"Já existe" nunca é "não fazer nada".** É "trocar de operação": em vez de
   `wbeditentity(new=item)`, vira `wbcreateclaim` (`P91`) + `wbsetqualifier`
   (`P88`) no item que já existe.
4. **Só cria (`wbeditentity(new=item)`) se não achou em nenhuma das duas
   camadas acima.**

```
┌─────────────────────┐
│ Nome/CNES da entidade│
└──────────┬───────────┘
           │
           ▼
   ┌───────────────┐   sim    ┌─────────────────────────┐
   │ Está no cache? ├─────────► usa o QID do cache        │
   └───────┬────────┘         └────────────┬─────────────┘
           │ não                            │
           ▼                                │
   ┌────────────────────┐  sim              │
   │ wbsearchentities    ├──────────────────►│
   │ encontra o item?    │  (atualiza cache) │
   └───────┬─────────────┘                  │
           │ não                            │
           ▼                                ▼
   ┌────────────────┐            ┌──────────────────────┐
   │ wbeditentity    │            │ wbcreateclaim (P91)  │
   │ (new=item)      │            │ + wbsetqualifier(P88)│
   │ + registra cache│            │  no item existente   │
   └─────────────────┘            └───────────────────────┘
```

Um vínculo médico↔estabelecimento é **sempre incremental**: cada aparição do
mesmo médico em outra clínica é uma nova claim `P91` no mesmo item — nunca um
item novo, nunca uma sobrescrita.

---

## Estrutura dos dados de origem

`campinas_cardiologistas_nomes_reais.json`:

```json
{
  "municipio": "Campinas (350950)",
  "total_estabelecimentos": 197,
  "estabelecimentos": {
    "3509509937838": {
      "cnes": "9937838",
      "nome": "CLINICA CR FERNANDES",
      "razao_social": "CLINICA CRR FERNANDES LTDA",
      "cnpj": "29188670000165",
      "cardiologistas": [
        {"nome": "CARLOS ROBERTO FERNANDES", "cbo": "225120"}
      ]
    }
  }
}
```

## Arquivos de cache local

- `mapa_campinas_gerado.json` — `{"estabelecimentos": {unkey: QID}, "profissionais": {NOME_NORMALIZADO: QID}}`
- `mapa_nos_gerados.json` — QIDs dos itens de CBO já criados em fase anterior (`{"CBO": {codigo_cbo: QID}}`)

A chave de `profissionais` é o **nome normalizado** (sem acento, uppercase,
espaços colapsados) — nunca combinado com CBO ou estabelecimento.

---

## Configuração

Credenciais via variável de ambiente (nunca hardcoded no script):

```bash
export WIKI_BOT_USER='SeuUsuarioBot'
export WIKI_BOT_PASS='sua_senha_ou_bot_password'
python3 injetar_campinas_blindado_fixed.py
```

Se a conta exigir 2FA ou a instância bloquear login normal via API, gere uma
credencial em `Special:BotPasswords` e use o formato `Usuario@NomeDoBot` como
`WIKI_BOT_USER`.

O script confirma logo no início que a sessão está realmente autenticada
(`action=query&meta=userinfo`), não apenas que o login retornou `Success` —
sessão "anônima" mesmo após login bem-sucedido é a causa clássica de erros
`badtoken` em toda escrita.

---

## Script

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
injetar_campinas_blindado_fixed.py

Correção do script original (injetar_campinas_blindado.py).

BUG ENCONTRADO:
  A chave de deduplicação do médico era:
      med_key = f"{nome_med}_{cbo}_{unkey}"
  Isso inclui o ESTABELECIMENTO (unkey) na chave. Resultado: o mesmo médico
  aparecendo em clínicas diferentes gerava chaves diferentes, então o script
  nunca reconhecia que o médico já existia -- e tentava CRIAR o item de novo,
  causando o erro 'wikibase-validator-label-with-description-conflict'
  (ex: Q813 'CARLOS ROBERTO FERNANDES', que aparece em 3+ estabelecimentos
  no JSON de origem).

  Bug secundário: mesmo corrigindo a chave, não havia rotina para adicionar
  um NOVO vínculo (P91 + qualificador P88) a um médico já existente -- o
  script só criava, nunca vinculava incrementalmente.

CORREÇÃO:
  1. Chave de dedup do médico agora é só o nome normalizado (maiúsculo,
     sem acento, espaços colapsados) -- um médico é UMA entidade única no
     grafo, como o cenário original pedia.
  2. Fallback via wbsearchentities antes de criar (cobre caso do mapa local
     estar desatualizado em relação à Wikibase).
  3. Quando o médico já existe (via cache OU via API), o script executa
     EXCLUSIVAMENTE wbcreateclaim (P91) + wbsetqualifier (P88) no item
     existente -- nunca wbeditentity(new=item) de novo.
"""
import os
import sys
import json
import re
import time
import unicodedata

import requests

API_URL = "https://determinar.ia.br/w/api.php"
BOT_USER = os.environ.get("WIKI_BOT_USER", "ADM")
BOT_PASS = os.environ.get("WIKI_BOT_PASS", "")
# ^ recomendo fortemente nunca deixar a senha em texto plano no código-fonte.

SESSION = requests.Session()
SESSION.headers.update({"User-Agent": "determinar-ia-ingestor/1.0"})

MAPA_LOCAL_CAMPINAS = "/opt/determinar-tools/mapa_campinas_gerado.json"
MAPA_GLOBAL_M1 = "/opt/determinar-tools/mapa_nos_gerados.json"

P_CNPJ = "P7"
P_CNES = "P92"
P_VINCULO = "P91"
P_CBO = "P88"


# ---------------------------------------------------------------------------
# Auxiliares
# ---------------------------------------------------------------------------
def normalizar(texto: str) -> str:
    """Chave de dedup robusta: sem acento, sem espaço duplicado, uppercase."""
    texto = (texto or "").strip()
    texto = unicodedata.normalize("NFKD", texto).encode("ASCII", "ignore").decode("ASCII")
    texto = " ".join(texto.split())
    return texto.upper()


def wikibase_login():
    r = SESSION.get(API_URL, params={
        "action": "query", "meta": "tokens", "type": "login", "format": "json"
    }).json()
    token = r["query"]["tokens"]["logintoken"]
    res = SESSION.post(API_URL, data={
        "action": "login", "lgname": BOT_USER, "lgpassword": BOT_PASS,
        "lgtoken": token, "format": "json"
    }).json()
    if res.get("login", {}).get("result") != "Success":
        print(f"[ERRO] Falha no login da Wikibase: {res}")
        sys.exit(1)
    print("[OK] Login efetuado com sucesso na Wikibase.")

    # Diagnóstico: confirma que a sessão é reconhecida como logada de verdade
    # (evita o problema clássico de token 'anônimo' mesmo com login Success).
    userinfo = SESSION.get(API_URL, params={
        "action": "query", "meta": "userinfo", "format": "json"
    }).json().get("query", {}).get("userinfo", {})
    if userinfo.get("id", 0) == 0:
        print(f"[ERRO] Sessão reconhecida como ANÔNIMA mesmo após login (userinfo={userinfo}). "
              f"Verifique se a conta precisa de bot password (Special:BotPasswords).")
        sys.exit(1)
    print(f"[OK] Sessão autenticada como usuário id={userinfo.get('id')} ({userinfo.get('name')}).")


def get_csrf():
    r = SESSION.get(API_URL, params={"action": "query", "meta": "tokens", "type": "csrf", "format": "json"}).json()
    return r["query"]["tokens"]["csrftoken"]


def formatar_cnpj(cnpj_raw):
    digitos = re.sub(r'\D', '', str(cnpj_raw))
    if len(digitos) == 14:
        return f"{digitos[:2]}.{digitos[2:5]}.{digitos[5:8]}/{digitos[8:12]}-{digitos[12:]}"
    return None


def extrair_qid(val):
    """Garante que retorna uma string limpa no formato Q..."""
    if isinstance(val, dict):
        val = val.get("id") or val.get("entity-id") or ""
    return str(val).strip()


def carregar_json_seguro(path):
    if os.path.exists(path):
        try:
            with open(path, "r", encoding="utf-8") as f:
                return json.load(f)
        except Exception:
            return {}
    return {}


def salvar_mapa(mapa):
    with open(MAPA_LOCAL_CAMPINAS, "w", encoding="utf-8") as f:
        json.dump(mapa, f, indent=2, ensure_ascii=False)


def buscar_item_por_nome(token, nome):
    """Fallback: pergunta à própria API se já existe item com esse rótulo
    exato em pt-br, independente do que diz o mapa local."""
    r = SESSION.get(API_URL, params={
        "action": "wbsearchentities",
        "search": nome,
        "language": "pt-br",
        "type": "item",
        "format": "json",
        "limit": 5,
    }).json()
    for hit in r.get("search", []):
        if normalizar(hit.get("label", "")) == normalizar(nome):
            return hit["id"]
    return None


def criar_item(token, payload_data):
    payload = {
        "action": "wbeditentity",
        "new": "item",
        "data": json.dumps(payload_data),
        "token": token,
        "format": "json"
    }
    res = SESSION.post(API_URL, data=payload).json()
    if "entity" in res:
        return res["entity"]["id"]
    print(f"[AVISO] Erro ao criar item: {res}")
    return None


def adicionar_vinculo_medico(token, qid_medico, estab_numeric_id, cbo_qid):
    """Adiciona uma NOVA claim P91 (-> estabelecimento) com qualificador P88
    (CBO) em um item de médico JÁ EXISTENTE. Nunca cria item aqui -- essa é
    a rotina que faltava no script original."""
    claim_payload = {
        "action": "wbcreateclaim",
        "format": "json",
        "token": token,
        "entity": qid_medico,
        "property": P_VINCULO,
        "snaktype": "value",
        "value": json.dumps({"entity-type": "item", "numeric-id": estab_numeric_id}),
    }
    res = SESSION.post(API_URL, data=claim_payload).json()
    if "claim" not in res:
        print(f"[AVISO] Erro ao criar claim P91 em {qid_medico}: {res}")
        return None
    claim_id = res["claim"]["id"]

    if cbo_qid and cbo_qid.startswith("Q"):
        qualifier_payload = {
            "action": "wbsetqualifier",
            "format": "json",
            "token": token,
            "claim": claim_id,
            "property": P_CBO,
            "snaktype": "value",
            "value": json.dumps({"entity-type": "item", "numeric-id": int(cbo_qid.replace("Q", ""))}),
        }
        res = SESSION.post(API_URL, data=qualifier_payload).json()
        if "error" in res:
            print(f"[AVISO] Erro ao adicionar qualificador P88 na claim {claim_id}: {res}")

    return claim_id


# ---------------------------------------------------------------------------
# Main
# ---------------------------------------------------------------------------
def main():
    json_path = "campinas_cardiologistas_nomes_reais.json"
    if not os.path.exists(json_path):
        print(f"[ERRO] O arquivo {json_path} não foi encontrado na pasta atual.")
        return

    dados_campinas = carregar_json_seguro(json_path)
    mapa_m1 = carregar_json_seguro(MAPA_GLOBAL_M1)  # Q-IDs dos CBOs (Fase 3)

    mapa_campinas = carregar_json_seguro(MAPA_LOCAL_CAMPINAS)
    if not isinstance(mapa_campinas, dict):
        mapa_campinas = {}
    if "estabelecimentos" not in mapa_campinas:
        mapa_campinas["estabelecimentos"] = {}
    # profissionais é indexado por NOME NORMALIZADO, não por "{nome}_{cbo}_{unkey}"
    if "profissionais" not in mapa_campinas:
        mapa_campinas["profissionais"] = {}

    total_estabs = dados_campinas.get('total_estabelecimentos', 0)
    print(f"==> Iniciando ingestão blindada em {total_estabs} estabelecimentos...")

    wikibase_login()
    token = get_csrf()

    total_vinculos = 0
    total_medicos_criados = 0
    total_medicos_vinculados = 0

    for unkey, info in dados_campinas.get("estabelecimentos", {}).items():
        cnes = info.get("cnes", "")
        nome_estab = info.get("nome", "").strip()
        razao_social = info.get("razao_social", "").strip()
        cnpj_raw = info.get("cnpj", "")

        # --- Estabelecimento (dedup por unkey/CNES) ---
        claims_estab = {}
        cnpj_fmt = formatar_cnpj(cnpj_raw)
        if cnpj_fmt:
            claims_estab["P7"] = [{
                "mainsnak": {"snaktype": "value", "property": "P7",
                             "datavalue": {"value": cnpj_fmt, "type": "string"}},
                "type": "statement", "rank": "normal"
            }]
        if cnes:
            claims_estab["P92"] = [{
                "mainsnak": {"snaktype": "value", "property": "P92",
                             "datavalue": {"value": str(cnes), "type": "string"}},
                "type": "statement", "rank": "normal"
            }]

        q_estab_raw = mapa_campinas["estabelecimentos"].get(unkey)
        q_estab = extrair_qid(q_estab_raw) if q_estab_raw else None

        if not q_estab:
            q_estab = buscar_item_por_nome(token, nome_estab)
            if q_estab:
                mapa_campinas["estabelecimentos"][unkey] = q_estab
                salvar_mapa(mapa_campinas)
                print(f"[ESTABELECIMENTO] '{nome_estab}' já existia na Wikibase -> {q_estab} (mapa atualizado)")

        if not q_estab:
            payload_estab = {
                "labels": {
                    "mul": {"language": "mul", "value": nome_estab},
                    "pt-br": {"language": "pt-br", "value": nome_estab},
                    "en": {"language": "en", "value": nome_estab}
                },
                "descriptions": {
                    "pt-br": {"language": "pt-br",
                              "value": f"Razão Social: {razao_social} - Estabelecimento de saúde em Campinas/SP"},
                    "en": {"language": "en", "value": "Healthcare establishment in Campinas, Brazil"}
                },
                "claims": claims_estab
            }
            q_estab = criar_item(token, payload_estab)
            if q_estab:
                mapa_campinas["estabelecimentos"][unkey] = q_estab
                salvar_mapa(mapa_campinas)
                print(f"[ESTABELECIMENTO] {nome_estab} (CNES: {cnes}) -> {q_estab} (criado)")
            time.sleep(0.3)

        if not q_estab:
            continue

        estab_numeric_id = int(q_estab.replace("Q", ""))

        # --- Profissionais: dedup por NOME (não mais por nome+cbo+estab) ---
        for med in info.get("cardiologistas", []):
            nome_med = med.get("nome", "").strip()
            cbo = med.get("cbo", "").strip()
            med_key = normalizar(nome_med)  # <-- A CORREÇÃO CENTRAL

            cbo_qid_raw = mapa_m1.get("CBO", {}).get(cbo)
            cbo_qid = extrair_qid(cbo_qid_raw) if cbo_qid_raw else None

            q_med_raw = mapa_campinas["profissionais"].get(med_key)
            q_med = extrair_qid(q_med_raw) if q_med_raw else None

            if not q_med:
                q_med = buscar_item_por_nome(token, nome_med)
                if q_med:
                    mapa_campinas["profissionais"][med_key] = q_med
                    salvar_mapa(mapa_campinas)
                    print(f"  [PROFISSIONAL] '{nome_med}' já existia na Wikibase -> {q_med} (mapa atualizado)")

            if not q_med:
                # Não existe em lugar nenhum -> cria (só a PRIMEIRA vez que
                # esse médico aparece em qualquer estabelecimento)
                claims_med = {
                    "P91": [{
                        "mainsnak": {
                            "snaktype": "value", "property": "P91",
                            "datavalue": {
                                "value": {"entity-type": "item", "numeric-id": estab_numeric_id},
                                "type": "wikibase-entityid"
                            }
                        },
                        "type": "statement", "rank": "normal",
                        "qualifiers": (
                            {"P88": [{
                                "snaktype": "value", "property": "P88",
                                "datavalue": {
                                    "value": {"entity-type": "item", "numeric-id": int(cbo_qid.replace("Q", ""))},
                                    "type": "wikibase-entityid"
                                }
                            }]} if cbo_qid and cbo_qid.startswith("Q") else {}
                        )
                    }]
                }
                payload_med = {
                    "labels": {
                        "mul": {"language": "mul", "value": nome_med},
                        "pt-br": {"language": "pt-br", "value": nome_med},
                        "en": {"language": "en", "value": nome_med}
                    },
                    "descriptions": {
                        "pt-br": {"language": "pt-br", "value": "Médico Cardiologista em Campinas/SP"},
                        "en": {"language": "en", "value": "Cardiologist physician in Campinas, Brazil"}
                    },
                    "claims": claims_med
                }
                q_med = criar_item(token, payload_med)
                if q_med:
                    mapa_campinas["profissionais"][med_key] = q_med
                    salvar_mapa(mapa_campinas)
                    total_medicos_criados += 1
                    total_vinculos += 1
                    print(f"  [PROFISSIONAL] {nome_med} -> {q_med} [CRIADO + vinculado a {q_estab} | CBO: {cbo_qid}]")
                time.sleep(0.3)

            else:
                # Médico já existe (achado no cache OU via API) -> SÓ adiciona
                # um novo vínculo P91+P88, nunca recria o item.
                claim_id = adicionar_vinculo_medico(token, q_med, estab_numeric_id, cbo_qid)
                if claim_id:
                    total_medicos_vinculados += 1
                    total_vinculos += 1
                    print(f"  [PROFISSIONAL] {nome_med} ({q_med}) -> VINCULADO a {q_estab} | CBO: {cbo_qid}")
                time.sleep(0.3)

    print(f"==> Ingestão semântica finalizada. "
          f"{total_medicos_criados} médicos criados, "
          f"{total_medicos_vinculados} vínculos adicionados a médicos existentes, "
          f"{total_vinculos} vínculos totais.")


if __name__ == "__main__":
    main()
```

---

## Uso

```bash
export WIKI_BOT_USER='SeuUsuarioBot'
export WIKI_BOT_PASS='sua_senha_ou_bot_password'
python3 injetar_campinas_blindado_fixed.py
```

Requer, na mesma pasta:
- `campinas_cardiologistas_nomes_reais.json` (dados de origem)
- `mapa_campinas_gerado.json` (cache de QIDs — criado automaticamente se não existir)
- `mapa_nos_gerados.json` (QIDs dos CBOs, gerados em etapa anterior)

## Checklist antes de rodar em produção de novo

- [ ] Credenciais fora do código-fonte (variável de ambiente, nunca commitado)
- [ ] `mapa_campinas_gerado.json` sem chaves órfãs do formato antigo (`nome_cbo_unkey`)
- [ ] Confirmar que `userinfo.id != 0` no log antes de deixar rodar os 197 estabelecimentos inteiros
- [ ] Rodar primeiro com um subconjunto pequeno do JSON de origem pra validar antes do lote completo
