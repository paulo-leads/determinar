Deixa eu confirmar uma coisa técnica antes de te entregar o briefing + correções — o sintoma (HTML aparecendo como texto) tem uma causa clássica no Scribunto que preciso verificar para corrigir do jeito certo:A busca confirmou o mecanismo do bug — mas preciso confirmar um detalhe final na lista oficial de tags permitidas:Confirmado — causa raiz encontrada, e ela explica exatamente o que você viu:

**A tag `<a>` NÃO está na whitelist de HTML do MediaWiki** (o `Sanitizer` só permite `div`, `span`, `table`, `details`, `ul`, etc.). Qualquer `<a href="...">` que o módulo emitia foi **mostrado como texto puro** — por isso os links da Doctoralia, do WhatsApp e das publicações apareceram "quebrados" enquanto o resto ficou bonito. A correção é usar a sintaxe oficial de link externo do wikitext (`[URL texto]`), que o parser converte em `<a>` **na saída final** — sempre renderiza.

Segue o que você pediu: **o briefing completo** + **o módulo v2 com todas as correções**.

---

## 📋 BRIEFING — Projeto "Wikipédia de Médicos" (determinar.ia.br)

> **Uso:** colar em `Project:Handoff` na sua wiki (ou guardar fora) para outra IA retomar de onde paramos. Gerado em 2026-08-03.

```markdown
# BRIEFING — Projeto Wikipédia de Médicos (determinar.ia.br)

## 1. Objetivo
"Wikipedia de médicos" paga: Wikibase como backend de dados estruturados (editado no
desktop) + página MediaWiki renderizada com design elegante e responsivo no celular do
médico. Entidade-modelo: Q1 (Dr. Marco Tulio Souza, cardiologista, Campinas/SP).

## 2. Infraestrutura em produção
- VPS: root@vmi3464040 · domínio: https://determinar.ia.br (HTTPS)
- MediaWiki 1.45.4 + Wikibase Suite 7.1.0 (Docker Compose: MW+Wikibase, MySQL/MariaDB,
  Elasticsearch 7.10.2, WDQS em query.determinar.ia.br)
- API: https://determinar.ia.br/w/api.php · Versão: Special:Version
- Skins: Timeless (ativa, responsiva) + MinervaNeue instalada
- Admin com apihighlimits (bot/sysop)

## 3. Esquema de dados REAL (confirmado via wbgetentities + render do Q1)
| P-ID | Conteúdo |
|------|----------|
| P1   | Foto (localMedia: DrMarcoTulioSouza.jpg) |
| P2   | Nome |
| P4   | Registro CRM/RQE |
| P10/P11 | Especialidade / subespecialidade (wikibase-item) |
| P13,P14,P18,P19,P23 | Procedimentos/serviços (wikibase-item) |
| P28/P29/P36/P71/P72 | Logradouro/Bairro/Estado/Município/CEP |
| P31 | Website · P32 | URL de agendamento (wa.link) |
| P35/P37 | Convênio/Operadora (Unimed, VIVEST...) |
| P38/P39/P40 | Qtd opiniões / Estrelas Doctoralia / Qtd dúvidas |
| P57 | Perfil Doctoralia (url) · P62 | Google Scholar user |
| P73 | Bio (dentro do escopo) · P74 | Desambiguação (fora do escopo) |
| P75 | Google Maps CID · P76 | Horário · P77 | Publicações (urls) |
| P79/P80 | Qualificadores: Fonte / Data da coleta |
- Q106 Doctoralia · Q107 CFM · Q108 SIGTAP · Q109 CNES · Q110 curador Paulo C.P. Santos
- Q2–Q40 procedimentos; Q76/Q77 tratamentos
- ANOMALIA: P81, Q79, Q80 apagados mas ainda referenciados em qualificadores (limpar)

## 4. Camada de apresentação em produção
- Module:MedicoInfobox — Lua, lê Q via mw.wikibase.* (getBestStatements, statements são
  TABELAS: st.mainsnak / st.qualifiers[P])
- Template:Infobox Médico — {{#invoke:MedicoInfobox|main|ID={{{ID|{{{1|}}}}}}}}
- MediaWiki:Common.css + MediaWiki:Minerva.css — CSS responsivo completo (card, chips,
  estrelas, CTA fixo no mobile, dark mode)
- Página do médico: {{Infobox Médico|ID=Q1}}

## 5. Bugs corrigidos na v2 do módulo (2026-08-03)
1. HTML <a> literal no frontend → tag <a> NÃO é permitida pelo Sanitizer do MediaWiki;
   v2 usa sintaxe [URL texto] (parser gera o <a> na saída). CORRIGIDO.
2. Mapa: "URL não permitida para incorporação" → URL usava q=&cid= (q vazio);
   v2 usa q=endereço completo + output=embed. CORRIGIDO.
3. CTA "Agendar consulta" com target blank no mobile → v2 abre na mesma aba (padrão MW).
   Para nova aba em TODO o site: $wgExternalLinkTarget = '_blank' no LocalSettings.php.
4. Chips lixo (Unimed/VIVEST e item "Clínica - Rua...") → filtro EXCLUIR_CHIPS no módulo
   + PENDENTE: limpar/mover esses valores na entidade Q1 (dados, não código).
5. PENDENTE de decisão: descrição do Q1 é longa (renderiza como subtítulo). Encurtar no
   item Q1 se quiser.

## 6. Roadmap (próximos passos)
- [ ] Validar visual mobile real (Timeless já é responsivo; MobileFrontend+Minerva p/ anônimos)
- [ ] Importação em massa CSV → API wbeditentity (script foi oferecido, não criado)
- [ ] Modelo de cobrança/assinatura (controle de acesso a página completa)
- [ ] SEO: JSON-LD Physician via hook BeforePageDisplay (LocalSettings.php)
- [ ] Hardening: CVE-2026-22710 (XSS Wikibase) — confirmar fix na Suite; portas ES 9200/9300
      só internas; 2FA admin (OATHAuth); OAuth atualizado (CVE-2026-58029)
- [ ] Limpar referências órfãs P81/Q79/Q80 no grafo

## 7. Como retomar
1. Ler este briefing. 2. Ver Module:MedicoInfobox + Template:Infobox Médico.
3. Abrir https://determinar.ia.br/w/index.php?title=Q1 → conferir render.
4. Continuar pelo item [ ] da seção 6 que fizer mais sentido ao negócio.
```

---

## 🔧 CORREÇÃO — `Module:MedicoInfobox` v2 (substitua o conteúdo inteiro)

```lua
-- Module:MedicoInfobox v2 — links via [URL texto] (tag <a> crua NÃO é permitida no wikitext)
local p = {}

local CFG = {
    foto          = 'P1',   -- localMedia
    nome          = 'P2',   -- monolingualtext
    registro      = 'P4',   -- monolingualtext
    especialidade = 'P10',  -- wikibase-item
    subesp        = 'P11',  -- wikibase-item
    servicos      = { 'P13', 'P14', 'P18', 'P19', 'P23' },
    logradouro    = 'P28',
    bairro        = 'P29',
    email         = 'P30',
    website       = 'P31',
    agendamento   = 'P32',
    lat           = 'P33',
    lon           = 'P34',
    convenio      = 'P35',
    estado        = 'P36',
    operadora     = 'P37',
    opinioes      = 'P38',
    estrelas      = 'P39',
    duvidas       = 'P40',
    doctoralia    = 'P57',
    municipio     = 'P71',
    cep           = 'P72',
    bio           = 'P73',
    escopo_neg    = 'P74',
    maps_cid      = 'P75',
    horario       = 'P76',
    publicacoes   = 'P77',
    scholar_user  = 'P62',
    fonte         = 'P79',
    coleta        = 'P80',
}

-- Chips indesejados (dados misturados no Q1: operadoras e itens de endereço)
local EXCLUIR_CHIPS = { 'unimed', 'vivest', 'seguros' }

local function contemLixo( s )
    local ls = string.lower( s or '' )
    for _, pad in ipairs( EXCLUIR_CHIPS ) do
        if ls:find( pad, 1, true ) then return true end
    end
    return ls:find( 'rua ' ) or ls:find( 'avenida' ) or ls:find( 'av. ' ) or false
end

local function esc( s )
    if not s then return '' end
    return mw.text.encode( tostring( s ), '<>&"' )
end

-- Link externo na sintaxe oficial do wikitext: [URL texto]
local function linkExterno( url, texto )
    if not url or url == '' then return '' end
    return '[' .. url .. ' ' .. texto .. ']'
end

local function rotulo( ent, lang )
    if not ent then return nil end
    for _, l in ipairs( { 'pt-br', 'pt', 'mul', lang, 'en' } ) do
        local lb = ent:getLabel( l )
        if lb then return lb end
    end
    return nil
end

local function descricao( ent, lang )
    if not ent then return '' end
    for _, l in ipairs( { 'pt-br', 'pt', 'mul', lang, 'en' } ) do
        local d = ent:getDescription( l )
        if d then return d end
    end
    return ''
end

local function snakParaTexto( snak, lang )
    if not snak or snak.snaktype ~= 'value' then return nil end
    local dv = snak.datavalue
    if not dv then return nil end
    if dv.type == 'string' or dv.type == 'url' or dv.type == 'external-id' then
        return dv.value
    elseif dv.type == 'monolingualtext' then
        return dv.value.text
    elseif dv.type == 'quantity' then
        return tostring( dv.value.amount ):gsub( '^%+', '' )
    elseif dv.type == 'time' then
        return ( dv.value.time or '' ):match( '(%d+%-%d+%-%d+)' )
    elseif dv.type == 'wikibase-entityid' then
        local alvo = mw.wikibase.getEntity( dv.value.id )
        return rotulo( alvo, lang ) or dv.value.id
    end
    return nil
end

local function valor( ent, pid, lang )
    if not ent then return nil end
    local sts = ent:getBestStatements( pid )
    if not sts then return nil end
    for _, st in ipairs( sts ) do
        local v = snakParaTexto( st.mainsnak, lang )
        if v and v ~= '' then return v end
    end
    return nil
end

local function valores( ent, pid, lang )
    local out = {}
    if not ent then return out end
    local sts = ent:getBestStatements( pid )
    if not sts then return out end
    for _, st in ipairs( sts ) do
        local v = snakParaTexto( st.mainsnak, lang )
        if v and v ~= '' then out[#out + 1] = v end
    end
    return out
end

local function qualificador( ent, pid, qpid, lang )
    if not ent then return nil end
    local sts = ent:getBestStatements( pid )
    if not sts or not sts[1] then return nil end
    local qs = sts[1].qualifiers
    if not qs then return nil end
    local lista = qs[qpid]
    if not lista then return nil end
    for _, q in ipairs( lista ) do
        local v = snakParaTexto( q, lang )
        if v then return v end
    end
    return nil
end

local function chips( lista )
    local s = ''
    for _, v in ipairs( lista ) do
        if not contemLixo( v ) then
            s = s .. '<span class="chip-especialidade">' .. esc( v ) .. '</span> '
        end
    end
    return s
end

local function estrelas( n )
    n = math.floor( tonumber( n ) or 0 )
    n = math.max( 0, math.min( 5, n ) )
    return string.rep( '★', n ) .. string.rep( '☆', 5 - n )
end

function p.main( frame )
    local args = frame.args
    local id = ( args.ID or args[1] or '' ):match( '^%s*(.-)%s*$' )
    local lang = mw.getContentLanguage():getCode()

    local ent = mw.wikibase.getEntity( id ~= '' and id or nil )
    if not ent then
        return '<div class="infobox-medico"><div class="infobox-medico-titulo">Entidade não encontrada — informe ID=Q…</div></div>'
    end

    local nome       = valor( ent, CFG.nome, lang ) or rotulo( ent, lang ) or '—'
    local desc       = descricao( ent, lang )
    local foto       = valor( ent, CFG.foto, lang )
    local regs       = valores( ent, CFG.registro, lang )
    local esp        = valor( ent, CFG.especialidade, lang )
    local sub        = valor( ent, CFG.subesp, lang )
    local logradouro = valor( ent, CFG.logradouro, lang )
    local bairro     = valor( ent, CFG.bairro, lang )
    local municipio  = valor( ent, CFG.municipio, lang )
    local estado     = valor( ent, CFG.estado, lang )
    local cep        = valor( ent, CFG.cep, lang )
    local horario    = valor( ent, CFG.horario, lang )
    local convenio   = valor( ent, CFG.convenio, lang )
    local operadora  = valor( ent, CFG.operadora, lang )
    local email      = valor( ent, CFG.email, lang )
    local website    = valor( ent, CFG.website, lang )
    local agenda     = valor( ent, CFG.agendamento, lang )
    local doc        = valor( ent, CFG.doctoralia, lang )
    local estrelasN  = valor( ent, CFG.estrelas, lang )
    local opinioes   = valor( ent, CFG.opinioes, lang )
    local bio        = valor( ent, CFG.bio, lang )
    local cid        = valor( ent, CFG.maps_cid, lang )
    local pubs       = valores( ent, CFG.publicacoes, lang )
    local scholar    = valor( ent, CFG.scholar_user, lang )
    local escopoNeg  = valores( ent, CFG.escopo_neg, lang )
    local fonteTxt   = qualificador( ent, CFG.registro, CFG.fonte, lang ) or qualificador( ent, CFG.bio, CFG.fonte, lang )
    local coletaTxt  = qualificador( ent, CFG.registro, CFG.coleta, lang ) or qualificador( ent, CFG.bio, CFG.coleta, lang )

    local servicos = {}
    for _, spid in ipairs( CFG.servicos ) do
        for _, v in ipairs( valores( ent, spid, lang ) ) do
            servicos[#servicos + 1] = v
        end
    end

    local out = {}
    out[#out + 1] = '<div class="infobox-medico">'

    if foto then
        out[#out + 1] = '<div class="infobox-medico-imagem">'
            .. frame:preprocess( '[[File:' .. foto .. '|300px|center]]' ) .. '</div>'
    end

    out[#out + 1] = '<div class="infobox-medico-titulo">' .. esc( nome ) .. '</div>'
    if desc and desc ~= '' then
        out[#out + 1] = '<div class="infobox-medico-descricao">' .. esc( desc ) .. '</div>'
    end

    local espChips = {}
    if esp then espChips[#espChips + 1] = esp end
    if sub then espChips[#espChips + 1] = sub end
    if #espChips > 0 then
        out[#out + 1] = '<div class="infobox-medico-chips">' .. chips( espChips ) .. '</div>'
    end

    if estrelasN then
        local nota = '<div class="medico-avaliacao"><span class="medico-estrelas">' .. estrelas( estrelasN )
            .. '</span> <span class="medico-nota">' .. esc( estrelasN ) .. '/5</span>'
        if opinioes then
            if doc then
                nota = nota .. ' · ' .. linkExterno( doc, opinioes .. ' opiniões' )
            else
                nota = nota .. ' · ' .. esc( opinioes ) .. ' opiniões'
            end
        end
        out[#out + 1] = nota .. '</div>'
    end

    out[#out + 1] = '<table class="infobox-medico-tabela">'
    local function linha( r, v )
        if v and v ~= '' then
            return '<tr><th>' .. esc( r ) .. '</th><td>' .. v .. '</td></tr>'
        end
        return ''
    end

    if #regs > 0 then
        local regHtml = {}
        for _, r in ipairs( regs ) do regHtml[#regHtml + 1] = esc( r ) end
        out[#out + 1] = linha( 'Registro', table.concat( regHtml, '<br>' ) )
    end

    local partesEnd = {}
    for _, v in ipairs( { logradouro, bairro, municipio, estado, cep } ) do
        if v and v ~= '' then partesEnd[#partesEnd + 1] = v end
    end
    if #partesEnd > 0 then
        out[#out + 1] = linha( 'Endereço', esc( table.concat( partesEnd, ', ' ) ) )
    end

    if horario   then out[#out + 1] = linha( 'Horário', esc( horario ) ) end
    if convenio  then out[#out + 1] = linha( 'Convênio', esc( convenio ) ) end
    if operadora then out[#out + 1] = linha( 'Operadora', esc( operadora ) ) end
    if email     then out[#out + 1] = linha( 'E-mail', linkExterno( 'mailto:' .. email, email ) ) end
    if website then
        local siteHref = website:match( '^https?://' ) and website or ( 'https://' .. website )
        out[#out + 1] = linha( 'Site', linkExterno( siteHref, website ) )
    end
    if doc then
        out[#out + 1] = linha( 'Doctoralia', linkExterno( doc, 'Ver perfil' ) )
    end
    out[#out + 1] = '</table>'

    if bio and bio ~= '' then
        out[#out + 1] = '<div class="infobox-medico-bio">' .. esc( bio ) .. '</div>'
    end

    if #servicos > 0 then
        out[#out + 1] = '<div class="infobox-medico-secao">Procedimentos e serviços</div>'
        out[#out + 1] = '<div class="infobox-medico-chips">' .. chips( servicos ) .. '</div>'
    end

    if agenda then
        out[#out + 1] = '<div class="cta-agendar">' .. linkExterno( agenda, 'Agendar consulta' ) .. '</div>'
    end

    -- Mapa: URL com q=endereço (o formato q=&cid= era rejeitado pelo Google)
    if cid and cid ~= '' then
        local endMapa = table.concat( { logradouro, bairro, municipio, estado }, ', ' )
        local mapUrl
        if endMapa ~= '' then
            mapUrl = 'https://maps.google.com/maps?q=' .. mw.uri.encode( endMapa, 'QUERY' )
                .. '&z=16&output=embed&hl=pt-BR'
        else
            mapUrl = 'https://maps.google.com/maps?cid=' .. cid .. '&output=embed&hl=pt-BR'
        end
        local ok, html = pcall( frame.extensionTag, frame, 'perfilexterno', '', { url = mapUrl, largura = '100%', altura = '300' } )
        if ok and html and html ~= '' then
            out[#out + 1] = html
        else
            out[#out + 1] = '<div class="infobox-medico-mais">'
                .. linkExterno( 'https://maps.google.com/maps?cid=' .. cid, 'Ver no Google Maps' ) .. '</div>'
        end
    end

    if #pubs > 0 then
        out[#out + 1] = '<div class="infobox-medico-secao">Publicações científicas</div>'
        out[#out + 1] = '<ul class="infobox-medico-pubs">'
        local n = math.min( 5, #pubs )
        for i = 1, n do
            local u = pubs[i]
            out[#out + 1] = '<li>' .. linkExterno( u, u:gsub( '^https?://', '' ) ) .. '</li>'
        end
        out[#out + 1] = '</ul>'
        if #pubs > n then
            local alvo = scholar and ( 'https://scholar.google.com/citations?user=' .. scholar .. '&hl=pt-BR' ) or ''
            out[#out + 1] = '<div class="infobox-medico-mais">'
                .. linkExterno( alvo, '+ ' .. ( #pubs - n ) .. ' publicações' ) .. '</div>'
        end
    end

    if #escopoNeg > 0 then
        out[#out + 1] = '<details class="medico-detalhes"><summary>Desambiguação — o que este perfil <b>não</b> é</summary><ul>'
        for _, v in ipairs( escopoNeg ) do
            out[#out + 1] = '<li>' .. esc( v ) .. '</li>'
        end
        out[#out + 1] = '</ul></details>'
    end

    if fonteTxt or coletaTxt then
        out[#out + 1] = '<div class="infobox-medico-rodape">Dados curados · fonte: ' .. esc( fonteTxt or '—' )
        if coletaTxt then out[#out + 1] = ' · verificado em ' .. esc( coletaTxt ) end
        out[#out + 1] = '</div>'
    end

    out[#out + 1] = '</div>'
    return table.concat( out )   -- sem frame:preprocess no HTML todo; só a imagem precisava
end

return p
```

## CSS — nada muda. Só **adicione** 2 linhas no fim do seu CSS (remove o ícone de "link externo" que o MediaWiki põe nos links gerados):

```css
/* Remove ícone de link externo dentro do card (sintaxe [url texto] gera classe .external) */
.infobox-medico a.external { background-image: none; padding-right: 0; }
```

## Template — não muda nada.

## Depois de salvar

1. Purge nas 3 páginas:
   - `.../w/index.php?title=Module:MedicoInfobox&action=purge`
   - `.../w/index.php?title=Template:Infobox_Médico&action=purge`
   - a página do médico (ou teste em `Special:ExpandTemplates` → `Infobox Médico|ID=Q1`)
2. O que deve aparecer agora:
   - **"57 opiniões"** clicável, **"Ver perfil"** clicável, **publicações** clicáveis
   - **Mapa do Google** renderizado (com a URL do endereço, não a `q=&cid=`)
   - **CTA "Agendar consulta"** abrindo na mesma aba no celular (era o `target="_blank"`)
   - Chips de **Unimed/VIVEST e "Clínica - Rua..."** sumindo (filtro) — mas lembre: isso é máscara; o ideal depois é tirar esses valores da propriedade errada no Q1

**Sobre o `target="_blank"`:** com a sintaxe `[URL texto]` o MediaWiki controla isso globalmente. Se quiser TODOS os links externos do site em nova aba, adicione no `LocalSettings.php`: `$wgExternalLinkTarget = '_blank';`

---
