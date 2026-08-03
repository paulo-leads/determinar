# O que está acontecendo (3 causas — nenhuma é VPS)

**1. O Template mostra a mesma coisa porque ele *sempre* mostra o wikitext.** A página `Template:Infobox_Médico` exibe o código-fonte (`{{#invoke:...}}`), que **não mudou** — quem muda são o `Module:` e o CSS. O render de verdade você vê em `.../w/index.php?title=Q1`.

**2. `<details>` e `<summary>` NÃO estão na lista de tags permitidas do Sanitizer do MediaWiki.** Por isso o FAQ mostrou `</summary>` cru e a desambiguação ficou feia: o parser não reconhece essas tags e as trata como texto. O v3 usava `<details>` — por isso pareceu que "nada funcionou". O v3.1 abaixo usa **somente tags permitidas** (`div`, `span`, `ul`, `li`, `b`).

**3. CSS:** `MediaWiki:Common.css` vale para **todos os skins** (Timeless, Minerva, Vector). `MediaWiki:Minerva.css` só vale no Minerva. Se você testou no desktop (Timeless) editando só o Minerva.css, nada carrega. E **sem `action=purge` o MediaWiki serve CSS/HTML em cache** — parece que não salvou.

---

# 1) Module:MedicoInfobox v3.1 — substitua o conteúdo inteiro

```lua
-- Module:MedicoInfobox v3.1
-- FAQ (P56) e Desambiguação (P74) em divs (tags permitidas pelo Sanitizer)
-- Mapa tenta CID primeiro (comprovado funcionando na sua wiki), depois endereço
local p = {}

local CFG = {
    foto          = 'P1',
    nome          = 'P2',
    registro      = 'P4',
    especialidade = 'P10',
    subesp        = 'P11',
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
    faq           = 'P56',
    doctoralia    = 'P57',
    scholar_user  = 'P62',
    municipio     = 'P71',
    cep           = 'P72',
    bio           = 'P73',
    escopo_neg    = 'P74',
    maps_cid      = 'P75',
    horario       = 'P76',
    publicacoes   = 'P77',
    fonte         = 'P79',
    coleta        = 'P80',
}

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

local function linkExterno( url, texto )
    if not url or url == '' then return esc( texto or '' ) end
    return '[' .. url .. ' ' .. ( texto or url ) .. ']'
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

-- Divide "Q: pergunta?A: resposta" em pares { q, a }
local function parseFAQ( texto )
    local itens = {}
    if not texto then return itens end
    if not texto:find( 'Q:', 1, true ) then
        local t = texto:gsub( '^%s+', '' ):gsub( '%s+$', '' )
        if t ~= '' then itens[#itens + 1] = { q = '', a = t } end
        return itens
    end
    local pos = 1
    while true do
        local qs = texto:find( 'Q:', pos, true )
        if not qs then break end
        local qe = texto:find( 'A:', qs + 2, true )
        if not qe then break end
        local pergunta = texto:sub( qs + 2, qe - 1 ):gsub( '^%s+', '' ):gsub( '%s+$', '' )
        local ae = texto:find( 'Q:', qe + 2, true ) or ( #texto + 1 )
        local resposta  = texto:sub( qe + 2, ae - 1 ):gsub( '^%s+', '' ):gsub( '%s+$', '' )
        if pergunta ~= '' then itens[#itens + 1] = { q = pergunta, a = resposta } end
        pos = ae
    end
    return itens
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

-- Tenta o embed; devolve nil se a extensão rejeitar (cai no link estilizado)
local function embedMapa( frame, url, largura, altura )
    local ok, html = pcall( frame.extensionTag, frame, 'perfilexterno', '', {
        url = url, largura = largura, altura = altura
    } )
    if ok and html and html ~= ''
        and not html:find( 'não permitida', 1, true )
        and not html:find( 'permitida', 1, true ) then
        return html
    end
    return nil
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

    -- FAQ: uma declaração P56 = um par Q&A (varios Q:...?A:... numa string tambem funciona)
    local faqs = {}
    for _, t in ipairs( valores( ent, CFG.faq, lang ) ) do
        for _, par in ipairs( parseFAQ( t ) ) do
            faqs[#faqs + 1] = par
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

    -- Mapa: CID PRIMEIRO (comprovado funcionando na sua wiki), depois endereço
    if cid and cid ~= '' or logradouro then
        local endMapa = table.concat( { logradouro, bairro, municipio, estado }, ', ' )
        local candidatas = {}
        if cid and cid ~= '' then
            candidatas[#candidatas + 1] = 'https://maps.google.com/maps?cid=' .. cid .. '&output=embed&hl=pt-BR'
            candidatas[#candidatas + 1] = 'https://www.google.com/maps?cid=' .. cid .. '&output=embed&hl=pt-BR'
        end
        if endMapa ~= '' then
            candidatas[#candidatas + 1] = 'https://maps.google.com/maps?q=' .. mw.uri.encode( endMapa, 'QUERY' )
                .. '&z=16&output=embed&hl=pt-BR'
            candidatas[#candidatas + 1] = 'https://www.google.com/maps?q=' .. mw.uri.encode( endMapa, 'QUERY' )
                .. '&z=16&output=embed&hl=pt-BR'
        end
        local htmlMapa = nil
        for _, u in ipairs( candidatas ) do
            htmlMapa = embedMapa( frame, u, '100%', '300' )
            if htmlMapa then break end
        end
        if htmlMapa then
            out[#out + 1] = '<div class="medico-mapa">' .. htmlMapa .. '</div>'
        else
            local alvo
            if cid and cid ~= '' then
                alvo = 'https://maps.google.com/maps?cid=' .. cid
            else
                alvo = 'https://maps.google.com/maps?q=' .. mw.uri.encode( endMapa, 'QUERY' )
            end
            out[#out + 1] = '<div class="medico-mapa-link">'
                .. linkExterno( alvo, 'Ver localização no Google Maps' ) .. '</div>'
        end
    end

    if #pubs > 0 then
        out[#out + 1] = '<div class="infobox-medico-secao">Publicações científicas</div>'
        out[#out + 1] = '<ul class="infobox-medico-pubs">'
        local n = math.min( 5, #pubs )
        for i = 1, n do
            local u = pubs[i]
            out[#out + 1] = '<li>' .. linkExterno( u, u:gsub( '^https?://', '' ):gsub( '^www%.', '' ) ) .. '</li>'
        end
        out[#out + 1] = '</ul>'
        if #pubs > n then
            local resto = #pubs - n
            local alvo = scholar and ( 'https://scholar.google.com/citations?user=' .. scholar .. '&hl=pt-BR' ) or ''
            if alvo ~= '' then
                out[#out + 1] = '<div class="medico-mais-pubs">'
                    .. linkExterno( alvo, '+ ' .. resto .. ' publicações no Google Scholar' ) .. '</div>'
            else
                out[#out + 1] = '<div class="medico-mais-pubs">+ ' .. resto .. ' publicações científicas</div>'
            end
        end
    end

    -- Desambiguação (div, tag permitida) — fica ANTES do FAQ
    if #escopoNeg > 0 then
        out[#out + 1] = '<div class="medico-desamb">'
        out[#out + 1] = '<div class="medico-desamb-titulo">Desambiguação — o que este perfil <b>não</b> é</div>'
        out[#out + 1] = '<ul class="medico-desamb-lista">'
        for _, v in ipairs( escopoNeg ) do
            out[#out + 1] = '<li>' .. esc( v ) .. '</li>'
        end
        out[#out + 1] = '</ul></div>'
    end

    -- FAQ logo abaixo da desambiguação
    if #faqs > 0 then
        out[#out + 1] = '<div class="infobox-medico-secao">Perguntas Frequentes</div>'
        for _, par in ipairs( faqs ) do
            out[#out + 1] = '<div class="medico-faq">'
            if par.q ~= '' then
                out[#out + 1] = '<div class="medico-faq-q">' .. esc( par.q ) .. '</div>'
            end
            out[#out + 1] = '<div class="medico-faq-a">' .. esc( par.a ) .. '</div>'
            out[#out + 1] = '</div>'
        end
    end

    if fonteTxt or coletaTxt then
        out[#out + 1] = '<div class="infobox-medico-rodape">Dados curados · fonte: ' .. esc( fonteTxt or '—' )
        if coletaTxt then out[#out + 1] = ' · verificado em ' .. esc( coletaTxt ) end
        out[#out + 1] = '</div>'
    end

    out[#out + 1] = '</div>'
    return table.concat( out )
end

return p
```

---

# 2) CSS — substitua o conteúdo inteiro de MediaWiki:Common.css (vale p/ todos os skins)

E **copie o mesmo conteúdo** para MediaWiki:Minerva.css (não custa nada).

```css
/* ===== Infobox Médico — v3.1 ===== */
.infobox-medico {
  max-width: 520px; margin: 0 auto;
  background: #ffffff; border: 1px solid #e5e7eb;
  border-radius: 16px; padding: 20px;
  box-shadow: 0 2px 12px rgba(0,0,0,.06);
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
}
.infobox-medico-imagem { text-align: center; margin-bottom: 12px; }
.infobox-medico-imagem img { border-radius: 12px; }
.infobox-medico-titulo { font-size: 1.45rem; font-weight: 800; color: #111827; line-height: 1.25; }
.infobox-medico-descricao { color: #6b7280; font-size: .95rem; margin-top: 2px; }
.infobox-medico-chips { margin-top: 10px; }
.chip-especialidade {
  display: inline-block; background: #eef2ff; color: #3730a3;
  border: 1px solid #c7d2fe; border-radius: 999px;
  padding: 3px 12px; margin: 3px 4px 0 0; font-size: .78rem; font-weight: 600;
}
.medico-avaliacao { margin-top: 12px; font-size: .92rem; color: #374151; }
.medico-estrelas { color: #f59e0b; letter-spacing: 2px; }
.medico-nota { font-weight: 700; }
.infobox-medico-tabela { width: 100%; border-collapse: collapse; margin-top: 12px; }
.infobox-medico-tabela th {
  text-align: left; vertical-align: top; color: #6b7280;
  font-weight: 600; font-size: .82rem; text-transform: uppercase;
  padding: 6px 10px 6px 0; width: 110px; border-bottom: 1px solid #f3f4f6;
}
.infobox-medico-tabela td { padding: 6px 0; font-size: .92rem; color: #111827; border-bottom: 1px solid #f3f4f6; }
.infobox-medico-bio { margin-top: 14px; color: #374151; font-size: .95rem; line-height: 1.6; }
.infobox-medico-secao {
  margin-top: 18px; font-size: .78rem; font-weight: 700; text-transform: uppercase;
  letter-spacing: .05em; color: #6b7280; border-bottom: 2px solid #e5e7eb; padding-bottom: 6px;
}
.infobox-medico-pubs { margin: 8px 0 0; padding-left: 20px; font-size: .9rem; }
.infobox-medico-pubs li { margin: 4px 0; }
.infobox-medico-rodape { margin-top: 16px; font-size: .75rem; color: #9ca3af; border-top: 1px dashed #e5e7eb; padding-top: 10px; }
.infobox-medico a.external { background-image: none; padding-right: 0; }

/* CTA — azul de verdade (vence Timeless e Minerva) */
.cta-agendar { margin: 18px 0 6px; text-align: center; }
.cta-agendar a.external,
.cta-agendar a,
.content .cta-agendar a.external {
  display: block !important;
  background: #0d6efd !important;
  color: #ffffff !important;
  padding: 14px 18px !important;
  border-radius: 14px !important;
  font-weight: 700 !important;
  font-size: 1.05rem !important;
  text-decoration: none !important;
  box-shadow: 0 4px 14px rgba(13,110,253,.35) !important;
}
.cta-agendar a.external:hover,
.cta-agendar a:hover { background: #0a58ca !important; color: #ffffff !important; }

/* Desambiguação — card âmbar */
.medico-desamb {
  margin: 18px 0 0; border: 1px solid #e5e7eb; border-left: 4px solid #f59e0b;
  border-radius: 10px; background: #fffbeb; overflow: hidden;
}
.medico-desamb-titulo {
  padding: 10px 14px; font-size: .9rem; color: #92400e; font-weight: 600;
  cursor: pointer; user-select: none;
}
.medico-desamb-titulo::after { content: " ▴"; }
.medico-desamb.fechado .medico-desamb-titulo::after { content: " ▾"; }
.medico-desamb-lista { margin: 0; padding: 4px 16px 12px 34px; color: #78350f; font-size: .85rem; line-height: 1.5; }
.medico-desamb.fechado .medico-desamb-lista { display: none; }

/* FAQ — cards abaixo da desambiguação */
.medico-faq {
  margin: 10px 0 0; border: 1px solid #e5e7eb; border-radius: 10px;
  background: #f9fafb; overflow: hidden;
}
.medico-faq-q {
  padding: 12px 14px; font-weight: 600; font-size: .92rem; color: #111827;
  cursor: pointer; user-select: none;
}
.medico-faq-q::before { content: "❓ "; }
.medico-faq-q::after { content: " ▴"; color: #9ca3af; }
.medico-faq.fechado .medico-faq-q::after { content: " ▾"; }
.medico-faq-a {
  padding: 10px 14px; color: #374151; font-size: .88rem; line-height: 1.55;
  border-top: 1px dashed #d1d5db;
}
.medico-faq-a::before { content: "R: "; font-weight: 600; color: #0d6efd; }
.medico-faq.fechado .medico-faq-a { display: none; }

/* Mapa / publicações */
.medico-mapa { margin: 16px 0 0; border-radius: 12px; overflow: hidden; box-shadow: 0 2px 10px rgba(0,0,0,.08); }
.medico-mapa iframe { display: block; border: 0; width: 100%; }
.medico-mapa-link { margin: 14px 0 0; }
.medico-mais-pubs { margin-top: 10px; font-size: .9rem; font-weight: 600; }

/* Dark mode */
html.skin-theme-clientpref-night .infobox-medico { background: #111827; border-color: #374151; }
html.skin-theme-clientpref-night .infobox-medico-titulo { color: #f3f4f6; }
html.skin-theme-clientpref-night .medico-desamb { background: #2a2414; border-color: #444; }
html.skin-theme-clientpref-night .medico-desamb-titulo { color: #fbbf24; }
html.skin-theme-clientpref-night .medico-desamb-lista { color: #d6c8a8; }
html.skin-theme-clientpref-night .medico-faq { background: #1f2937; border-color: #374151; }
html.skin-theme-clientpref-night .medico-faq-q { color: #f3f4f6; }
html.skin-theme-clientpref-night .medico-faq-a { color: #d1d5db; }
```

---

# 3) JS opcional — MediaWiki:Common.js (accordion suave)

Sem JS o conteúdo fica **todo visível** (nada quebra); com JS, clicar na pergunta/desambiguação recolhe/expande:

```js
$( function () {
	document.querySelectorAll( '.medico-desamb-titulo, .medico-faq-q' ).forEach( function ( el ) {
		el.addEventListener( 'click', function () {
			var bloco = el.closest( '.medico-desamb, .medico-faq' );
			if ( bloco ) bloco.classList.toggle( 'fechado' );
		} );
	} );
} );
```

---

# 4) Mapa — CID primeiro, sem tocar no VPS

Você tem razão: se uma página antiga exibia o mapa com `cid=...`, a extensão **aceita o formato CID** — o problema era o v2/v3 tentar só a URL de endereço. O v3.1 tenta nesta ordem: **CID (`maps.google.com`) → CID (`www.google.com`) → endereço → link estilizado**. Nada de VPS.

Se ainda assim falhar, abra a página antiga que funcionava, clique com o botão direito no iframe → "Inspecionar" e me mande o `src` exato — eu faço o v3.2 usar exatamente esse formato.

---

# 5) Ordem exata de verificação (a parte que você pulou: purge)

1. Salve o Module v3.1 (página `Module:MedicoInfobox`) e os 2 CSS.
2. **Purge obrigatório** (se não fizer, tudo continua em cache e "parece que não salvou"):
   ```
   /w/index.php?title=Module:MedicoInfobox&action=purge
   /w/index.php?title=MediaWiki:Common.css&action=purge
   /w/index.php?title=MediaWiki:Minerva.css&action=purge
   /w/index.php?title=MediaWiki:Common.js&action=purge
   /w/index.php?title=Q1&action=purge
   ```
3. Recarregue **`/w/index.php?title=Q1`** (não a página do Template) com **Ctrl+Shift+R** (hard refresh).
4. Confirme que o módulo novo está ativo: abra `/w/index.php?title=Module:MedicoInfobox&action=raw` e procure por `parseFAQ` e `medico-faq` — se não achar, o save não pegou.
5. Se o botão continuar branco: F12 → Elements → clique no link "Agendar consulta" → aba **Computed**: `background-color` deve ser `rgb(13, 110, 253)`. Me diga o que aparece lá que eu ajusto o seletor.

---

**Respondendo direto:**
- **Mexer no VPS? Não.** Nada disso exige servidor — tudo é pela interface (Module, MediaWiki:Common.css, MediaWiki:Minerva.css, MediaWiki:Common.js). VPS só seria necessário se quisesse links externos em nova aba (`$wgExternalLinkTarget`) ou o hardening — opcional.
- **FAQ cru com `</summary>`** → era o `<details>` que o Sanitizer não permite; v3.1 usa só `div`/`ul`/`li`, então some de vez.
- **Botão branco** → CSS estava no lugar errado (Minerva.css no Timeless) e/ou sem purge; agora está no Common.css (todos os skins) com `!important` e regra extra `.content .cta-agendar a.external`.
# BRIEFING — Projeto "Wikipédia de Médicos e Profissionais Liberais PMES" (determinar.ia.br)
**Uso:** colar em `Project:Handoff` na sua wiki (ou guardar fora) para outra IA retomar de onde paramos. **Atualizado em 2026-08-03** (v3.1 validado em produção).

---

## 1. Objetivo
"Wikipedia sem viés de editores minorias e leigos" premium: RDF SKOS como backend de dados estruturados (editado no desktop) + página MediaWiki renderizada com design elegante e responsivo no celular do médico. Entidade-modelo: **Q1** (Dr. Marco Tulio Souza, cardiologista, Campinas/SP).

## 2. Infraestrutura em produção
- VPS: `root@vmi3464040` · domínio: https://determinar.ia.br (HTTPS)
- MediaWiki 1.45.4 + Wikibase Suite 7.1.0 (Docker Compose: MW+Wikibase, MySQL/MariaDB, Elasticsearch 7.10.2, WDQS em query.determinar.ia.br)
- API: https://determinar.ia.br/w/api.php · Versão: Special:Version
- Skins: Timeless (ativa, responsiva) + MinervaNeue instalada
- Admin com `apihighlimits` (bot/sysop)
- **Nenhuma alteração de VPS foi necessária** para o estado atual — tudo foi feito pela interface gráfica (Module + MediaWiki: namespace)

## 3. Esquema de dados REAL (confirmado via wbgetentities + render do Q1)
| P-ID | Conteúdo |
|------|----------|
| P1 | Foto (localMedia: DrMarcoTulioSouza.jpg) |
| P2 | Nome · P4 | Registro CRM/RQE |
| P10/P11 | Especialidade / subespecialidade (wikibase-item) |
| P13,P14,P18,P19,P23 | Procedimentos/serviços (wikibase-item) |
| P28/P29/P36/P71/P72 | Logradouro/Bairro/Estado/Município/CEP |
| P30 | E-mail · P31 | Website · P32 | URL de agendamento (wa.link) |
| P33/P34 | Lat/Long · P35/P37 | Convênio/Operadora (Unimed, VIVEST...) |
| P38/P39/P40 | Qtd opiniões / Estrelas Doctoralia / Qtd dúvidas |
| **P56** | **FAQ — monolingualtext, formato `Q: ...?A: ...` (1 declaração = 1 par Q&A; várias declarações = mais perguntas)** |
| P57 | Perfil Doctoralia (url) · P62 | Google Scholar user |
| P73 | Bio (dentro do escopo) · P74 | Desambiguação (fora do escopo) |
| P75 | Google Maps CID · P76 | Horário · P77 | Publicações (urls) |
| P79/P80 | Qualificadores: Fonte / Data da coleta |
- Q106 Doctoralia · Q107 CFM · Q108 SIGTAP · Q109 CNES · Q110 curador Paulo C.P. Santos
- Q2–Q40 procedimentos; Q76/Q77 tratamentos
- **ANOMALIA:** P81, Q79, Q80 apagados mas ainda referenciados em qualificadores (limpar)

## 4. Camada de apresentação em produção (v3.1)
- **Module:MedicoInfobox** — Lua, lê Q via `mw.wikibase.*` (`getBestStatements`; statements são TABELAS: `st.mainsnak` / `st.qualifiers[P]`)
- **Template:Infobox Médico** — `{{#invoke:MedicoInfobox|main|ID={{{ID|{{{1|}}}}}}}}` (não muda)
- **MediaWiki:Common.css** (vale para TODOS os skins) + cópia em MediaWiki:Minerva.css — card, chips, estrelas, CTA azul fixo, dark mode, cards de desambiguação e FAQ
- **MediaWiki:Common.js** — accordion opcional (toggle classe `.fechado` em desambiguação/FAQ)
- Página do médico: `{{Infobox Médico|ID=Q1}}` → render real em `/w/index.php?title=Q1`

### Estado validado (2026-08-03)
- FAQ renderiza em cards (❓ pergunta / R: resposta), **sem** tags cruas
- Desambiguação em card âmbar com borda esquerda
- Botão "Agendar consulta" **azul** (não mais branco)
- Mapa do Google carrega **via CID** (P75) — sem mexer no VPS
- "57 opiniões" clicável · "Ver perfil" clicável · publicações clicáveis · "+N publicações no Google Scholar"

## 5. Lições aprendidas (IMPORTANTE para não regredir)
1. **Sanitizer do MediaWiki NÃO permite `<details>`/`<summary>`** — o parser vaza o markup cru (`</summary>` visível). Usar apenas `div`, `span`, `ul`, `li`, `b` no HTML gerado pelo módulo.
2. **`MediaWiki:Common.css` vale para todos os skins; `Minerva.css` só no Minerva.** CSS compartilhado vai no Common.css (e copiar para o Minerva.css).
3. **`action=purge` é obrigatório** após salvar Module/CSS/JS — sem purge, cache mascara o save e "parece que não salvou".
4. **Mapa:** o formato **CID funciona na wiki** (`https://maps.google.com/maps?cid=...&output=embed&hl=pt-BR`) — tentar CID primeiro, depois `q=endereço`, depois fallback em link estilizado. (URL `q=&cid=` com q vazio é rejeitada.)
5. **CTA branco** era especificidade de skin — resolve com `.content .cta-agendar a.external` + `!important` no Common.css.
6. **Links externos:** tag `<a>` crua não é permitida no wikitext; usar sintaxe `[URL texto]`. Para nova aba em todo o site: `$wgExternalLinkTarget = '_blank'` no LocalSettings.php (VPS — opcional).
7. **Chips lixo** (Unimed/VIVEST, "Clínica - Rua...") são **mascarados** pelo filtro `EXCLUIR_CHIPS` no módulo — a limpeza real dos dados no Q1 segue pendente.
8. **FAQ:** para adicionar mais perguntas, criar **novas declarações P56** no Q1 (cada uma = um par Q&A); o módulo também parseia vários `Q:...?A:...` numa mesma string.

## 6. Roadmap
- [x] Validar visual mobile real (FAQ, desambiguação, CTA, mapa OK no celular)
- [ ] **Importação em massa CSV → API `wbeditentity`** (script oferecido, não criado)
- [ ] **Modelo de cobrança/assinatura** (controle de acesso à página completa)
- [ ] **SEO:** JSON-LD Physician via hook `BeforePageDisplay` (LocalSettings.php)
- [ ] **Hardening:** CVE-2026-22710 (XSS Wikibase) — confirmar fix na Suite; portas ES 9200/9300 só internas; 2FA admin (OATHAuth); OAuth atualizado (CVE-2026-58029)
- [ ] **Limpar referências órfãs P81/Q79/Q80** no grafo
- [ ] **Limpar dados misturados no Q1** (remover operadoras e endereço de propriedades de serviço; remover máscara do filtro depois)
- [ ] **Decisão:** encurtar descrição longa do Q1 (renderiza como subtítulo)

## 7. Como retomar
1. Ler este briefing.
2. Ver `Module:MedicoInfobox` (v3.1) + `Template:Infobox Médico` + `MediaWiki:Common.css`/`Common.js`.
3. Abrir `https://determinar.ia.br/w/index.php?title=Q1` → conferir render.
4. Continuar pelo item [ ] da seção 6 que fizer mais sentido ao negócio.

---
