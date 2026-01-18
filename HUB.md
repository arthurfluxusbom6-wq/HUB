--[[
    SCRIPT OTIMIZADO PARA DELTA EXECUTOR
    FUNÇÃO: ESP OVERDRIVE APENAS PARA VOCÊ
    PAPÉIS: ASSASSINO (VERMELHO) | XERIFE (AZUL)
]]

-- Serviços Locais (Delta Compatível)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui") -- Garante carregamento

-- Configurações FINALIZADAS
local CONFIG = {
    Assassino = {
        Cor = Color3.new(1, 0, 0),
        CorBrilho = Color3.new(1, 0.2, 0.2),
        Espessura = 5,
        Transparencia = 0.25,
        Tamanho = Vector3.new(0.9, 0.9, 0.9),
        Texto = "[⚠️ ASSASSINO ⚠️] "
    },
    Xerife = {
        Cor = Color3.new(0, 0.5, 1),
        CorBrilho = Color3.new(0.2, 0.5, 1),
        Espessura = 4,
        Transparencia = 0.35,
        Tamanho = Vector3.new(0.8, 0.8, 0.8),
        Texto = "[🛡️ XERIFE 🛡️] "
    },
    Atualizacao = 60 -- FPS do ESP
}

-- Armazenamento Seguro (Delta Friendly)
local ESPs = {}
local Conexoes = {}

-- Função de Limpeza (Evita Vazamentos)
local function LimparESP(jogador)
    if ESPs[jogador.UserId] then
        for _, obj in pairs(ESPs[jogador.UserId]) do
            pcall(function() obj:Destroy() end)
        end
        ESPs[jogador.UserId] = nil
    end
    if Conexoes[jogador.UserId] then
        for _, conn in pairs(Conexoes[jogador.UserId]) do
            pcall(function() conn:Disconnect() end)
        end
        Conexoes[jogador.UserId] = nil
    end
end

-- Função: Criar Cubo Overdrive
local function CriarCubo(personagem, config)
    local root = personagem:WaitForChild("HumanoidRootPart")
    local tamanho = personagem:GetExtentsSize() + config.Tamanho

    -- Cubo Principal (Apenas no seu GUI)
    local cubo = Instance.new("BoxHandleAdornment")
    cubo.Name = "Delta_CuboESP"
    cubo.Adornee = root
    cubo.Size = tamanho
    cubo.Color3 = config.Cor
    cubo.Thickness = config.Espessura
    cubo.Transparency = config.Transparencia
    cubo.AlwaysOnTop = true
    cubo.ZIndex = 25
    cubo.Parent = PlayerGui

    -- Cubo de Brilho
    local brilho = Instance.new("BoxHandleAdornment")
    brilho.Name = "Delta_BrilhoESP"
    brilho.Adornee = root
    brilho.Size = tamanho + Vector3.new(0.3, 0.3, 0.3)
    brilho.Color3 = config.CorBrilho
    brilho.Thickness = 2
    brilho.Transparency = 0.6
    brilho.AlwaysOnTop = true
    brilho.ZIndex = 24
    brilho.Parent = PlayerGui

    -- Pulsar (Delta Otimizado)
    local pulsar = RunService.RenderStepped:Connect(function()
        if not root or not personagem.Parent then pulsar:Disconnect() return end
        brilho.Transparency = math.sin(tick() * 4) * 0.15 + 0.45
        brilho.Size = tamanho + Vector3.new(0.3 + math.cos(tick() * 3) * 0.12, 0.3 + math.cos(tick() * 3) * 0.12, 0.3 + math.cos(tick() * 3) * 0.12)
    end)

    return {cubo, brilho, pulsar}
end

-- Função: Criar Texto ESP
local function CriarTexto(personagem, config, nome)
    local root = personagem:WaitForChild("HumanoidRootPart")

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "Delta_TextoESP"
    billboard.Adornee = root
    billboard.Size = UDim2.new(0, 450, 0, 90)
    billboard.StudsOffset = Vector3.new(0, 5.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = PlayerGui

    local texto = Instance.new("TextLabel")
    texto.Size = UDim2.new(1, 0, 1, 0)
    texto.BackgroundTransparency = 1
    texto.Text = config.Texto .. nome
    texto.TextColor3 = config.Cor
    texto.Font = Enum.Font.Bold
    texto.TextSize = 36
    texto.TextScaled = true
    texto.Parent = billboard

    local sombra = Instance.new("TextLabel")
    sombra.Size = UDim2.new(1, 0, 1, 0)
    sombra.BackgroundTransparency = 1
    sombra.Text = texto.Text
    sombra.TextColor3 = Color3.new(0, 0, 0)
    sombra.Position = UDim2.new(0, 2, 0, 2)
    sombra.Parent = billboard

    return {billboard}
end

-- Função: Detectar Papel Correto
local function GetPapel(jogador)
    if jogador == LocalPlayer then return nil end
    local papel = jogador:GetAttribute("Role") or (jogador.Character and jogador.Character:FindFirstChild("Role") and jogador.Character.Role.Value) or ""
    return string.upper(papel) == "ASSASSIN" and CONFIG.Assassino or string.upper(papel) == "SHERIFF" and CONFIG.Xerife or nil
end

-- Função: Gerar ESP
local function GerarESP(jogador)
    if not jogador.Character then return end
    local config = GetPapel(jogador)
    if not config then LimparESP(jogador) return end

    LimparESP(jogador)
    local personagem = jogador.Character

    -- Cria Elementos
    local cubos = CriarCubo(personagem, config)
    local texto = CriarTexto(personagem, config, jogador.Name)

    -- Armazena
    ESPs[jogador.UserId] = {unpack(cubos), unpack(texto)}
    Conexoes[jogador.UserId] = {cubos[3]}

    -- Limpa se jogador morrer
    local morreu = personagem:WaitForChild("Humanoid").Died:Connect(function()
        LimparESP(jogador)
    end)
    table.insert(Conexoes[jogador.UserId], morreu)
end

-- Sistema Principal (Delta Compatível)
local function Iniciar()
    -- Jogadores Existentes
    for _, jogador in pairs(Players:GetPlayers()) do
        GerarESP(jogador)
    end

    -- Novos Jogadores
    Players.PlayerAdded:Connect(function(jogador)
        jogador.CharacterAdded:Connect(function()
            wait(1.5) -- Tempo seguro para carregamento
            GerarESP(jogador)
        end)
    end)

    -- Atualização Contínua
    Conexoes.Global = RunService.Heartbeat:Connect(function()
        for _, jogador in pairs(Players:GetPlayers()) do
            GerarESP(jogador)
        end
    end)
end

-- Inicia com Segurança (Delta Executor)
pcall(function()
    wait(2) -- Espera executor carregar completamente
    Iniciar()
    print("[DELTA ESP] Ativado com sucesso! Apenas você vê o ESP.")
end)
