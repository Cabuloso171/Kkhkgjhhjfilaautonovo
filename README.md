local imgui = require 'mimgui'
local v = imgui.new.bool(false)
local spam_fila = imgui.new.bool(false)

local pontos = {}
for i = 1, 50 do
    table.insert(pontos, {
        x = math.random(0, 250),
        y = math.random(0, 220),
        vx = (math.random() - 0.5) * 1.5,
        vy = (math.random() - 0.5) * 1.5
    })
end

imgui.OnFrame(function() return v[0] end, function()
    imgui.SetNextWindowPos(imgui.ImVec2(600, 350), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(250, 220), imgui.Cond.FirstUseEver)
    imgui.Begin("AUTO FILA HORA by NukY", v, imgui.WindowFlags.NoCollapse + imgui.WindowFlags.NoResize)

    local draw = imgui.GetWindowDrawList()
    local pos = imgui.GetWindowPos()

    for i, p1 in ipairs(pontos) do
        p1.x = p1.x + p1.vx
        p1.y = p1.y + p1.vy

        if p1.x < 0 or p1.x > 250 then p1.vx = -p1.vx end
        if p1.y < 0 or p1.y > 220 then p1.vy = -p1.vy end

        draw:AddCircleFilled(imgui.ImVec2(pos.x + p1.x, pos.y + p1.y), 2, 0xFFFFA500)

        for j, p2 in ipairs(pontos) do
            local dist = math.sqrt((p1.x - p2.x)^2 + (p1.y - p2.y)^2)
            if dist < 70 then
                draw:AddLine(
                    imgui.ImVec2(pos.x + p1.x, pos.y + p1.y),
                    imgui.ImVec2(pos.x + p2.x, pos.y + p2.y),
                    0x33FFFFFF, -- Linhas quase transparentes
                    1
                )
            end
        end
    end

    if imgui.Button("AUTO", imgui.ImVec2(150, 30)) then
        spam_fila[0] = not spam_fila[0]
    end
    
    imgui.End()
end)

function main()
    while true do
        wait(0)
        
        if isSampAvailable() then
            sampRegisterChatCommand("nk", function()
                v[0] = not v[0]
            end)
            
            if spam_fila[0] then
                sampSendChat("/fila")
                wait(1)
            end
        end
    end
end
