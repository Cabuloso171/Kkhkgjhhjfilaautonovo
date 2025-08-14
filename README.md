local imgui = require "mimgui"
local v = imgui.new.bool(false)
local animation_pos = imgui.new.float(0)
local animation_speed = 2.0
local color_r = imgui.new.float(1.0)
local color_g = imgui.new.float(0.0)
local color_b = imgui.new.float(0.0)
local color_change_speed = 0.02

imgui.OnFrame(function() return v[0] end, function()
    imgui.SetNextWindowPos(imgui.ImVec2(600, 350), imgui.Cond.FirstUseEver)
    imgui.Begin("", v, imgui.WindowFlags.NoCollapse + imgui.WindowFlags.NoResize)
    
    animation_pos[0] = animation_pos[0] + animation_speed
    if animation_pos[0] > imgui.GetWindowWidth() then
        animation_pos[0] = -200
    end
    
    color_r[0], color_g[0], color_b[0] = updateColor(color_r[0], color_g[0], color_b[0], color_change_speed)
    
    local text = "EM DESENVOLVIMENTO by NukY"
    imgui.SetCursorPosX(animation_pos[0])
    imgui.SetCursorPosY(imgui.GetWindowHeight() / 2 - 10)
    imgui.TextColored(imgui.ImVec4(color_r[0], color_g[0], color_b[0], 1.0), text)
    
    imgui.End()
end)

function updateColor(r, g, b, speed)
    if r > 0 and b == 0 then
        g = g + speed
        if g >= 1.0 then
            g = 1.0
            r = r - speed
        end
    elseif g > 0 and r == 0 then
        b = b + speed
        if b >= 1.0 then
            b = 1.0
            g = g - speed
        end
    elseif b > 0 and g == 0 then
        r = r + speed
        if r >= 1.0 then
            r = 1.0
            b = b - speed
        end
    end
    return r, g, b
end

function main()
    while true do
        wait(0)
        if isSampAvailable() then
            sampRegisterChatCommand("nk", function()
                v[0] = not v[0]
            end)
        end
    end
end
