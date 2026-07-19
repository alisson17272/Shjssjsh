-- ============================================
-- SCRIPT UNIFICADO: MENU + ESP + AIMBOT (AUTOMÁTICO - VERSÃO ESTÁVEL)
-- (Apenas para testes no seu próprio jogo)
-- ============================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ===== CONFIGURAÇÕES =====
local CONFIG = {
    DistanciaAimbot = 35,
    OffsetAimbotY = 0,
    CorESP = Color3.fromRGB(255, 0, 0)
}

-- ===== ESTADOS DO MENU =====
local ESP_Ativado = false
local Aimbot_Ativado = false

-- ===== GERENCIADOR DE HIGHLIGHTS (ESP) =====
local highlights = {}

local function criarHighlight(personagem)
    if not personagem then return nil end
    local highlight = Instance.new("Highlight")
    highlight.Name = "MeuESP_Menu"
    highlight.Adornee = personagem
    highlight.OutlineColor = CONFIG.CorESP
    highlight.FillColor = CONFIG.CorESP
    highlight.FillTransparency = 1
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = personagem
    return highlight
end

local function removerESPDeTodos()
    for jogador, highlight in pairs(highlights) do
        if highlight and highlight.Parent then
            highlight:Destroy()
        end
    end
    highlights = {}
end

local function atualizarESP()
    if ESP_Ativado then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer then
                local char = jogador.Character
                if char and not highlights[jogador] then
                    local novoHighlight = criarHighlight(char)
                    if novoHighlight then
                        highlights[jogador] = novoHighlight
                    end
                end
            end
        end
    else
        removerESPDeTodos()
    end
end

local function conectarRespawn(jogador)
    jogador.CharacterAdded:Connect(function(char)
        if ESP_Ativado and jogador ~= LocalPlayer then
            if highlights[jogador] then
                highlights[jogador]:Destroy()
                highlights[jogador] = nil
            end
            wait(0.1)
            local novoHighlight = criarHighlight(char)
            if novoHighlight then
                highlights[jogador] = novoHighlight
            end
        end
    end)
end

for _, jogador in ipairs(Players:GetPlayers()) do
    if jogador ~= LocalPlayer then
        conectarRespawn(jogador)
    end
end
Players.PlayerAdded:Connect(conectarRespawn)

Players.PlayerRemoving:Connect(function(jogador)
    if highlights[jogador] then
        highlights[jogador]:Destroy()
        highlights[jogador] = nil
    end
end)

-- ===== LÓGICA DO AIMBOT =====
local alvoAtual = nil

local function getJogadorMaisProximo()
    local personagem = LocalPlayer.Character
    if not personagem then return nil end
    local root = personagem:FindFirstChild("HumanoidRootPart")
    if not root then return nil end

    local origem = root.Position
    local maisProximo = nil
    local menorDist = CONFIG.DistanciaAimbot

    for _, jogador in ipairs(Players:GetPlayers()) do
        if jogador ~= LocalPlayer then
            local char = jogador.Character
            if char then
                local alvoRoot = char:FindFirstChild("HumanoidRootPart")
                local humanoide = char:FindFirstChild("Humanoid")
                if alvoRoot and humanoide and humanoide.Health > 0 then
                    local dist = (alvoRoot.Position - origem).Magnitude
                    if dist < menorDist then
                        menorDist = dist
                        maisProximo = jogador
                    end
                end
            end
        end
    end
    return maisProximo
end

-- ===== CRIAÇÃO DO MENU (UI) =====
local function criarMenu()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MenuHack_Tests"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    -- FRAME PRINCIPAL
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 240, 0, 250) -- Aumentei para caber mais opções
    frame.Position = UDim2.new(0.5, -120, 0.5, -125) -- Centralizado
    frame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
    frame.BackgroundTransparency = 0.1
    frame.BorderSizePixel = 0
    frame.ClipsDescendants = true
    frame.Parent = screenGui

    -- Borda brilhante (efeito neon)
    local borda = Instance.new("Frame")
    borda.Size = UDim2.new(1, 2, 1, 2)
    borda.Position = UDim2.new(0, -1, 0, -1)
    borda.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    borda.BackgroundTransparency = 0.3
    borda.BorderSizePixel = 0
    borda.Parent = frame
    local cornerBorda = Instance.new("UICorner")
    cornerBorda.CornerRadius = UDim.new(0, 10)
    cornerBorda.Parent = borda

    -- Arredondamento principal
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame

    -- Sombra (DropShadow)
    local sombra = Instance.new("Frame")
    sombra.Size = UDim2.new(1, 10, 1, 10)
    sombra.Position = UDim2.new(0, -5, 0, -5)
    sombra.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    sombra.BackgroundTransparency = 0.7
    sombra.BorderSizePixel = 0
    sombra.ZIndex = 0
    sombra.Parent = frame
    local cornerSombra = Instance.new("UICorner")
    cornerSombra.CornerRadius = UDim.new(0, 12)
    cornerSombra.Parent = sombra

    -- GRADIENTE DO TOPO
    local gradiente = Instance.new("Frame")
    gradiente.Size = UDim2.new(1, 0, 0, 45)
    gradiente.Position = UDim2.new(0, 0, 0, 0)
    gradiente.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    gradiente.BackgroundTransparency = 0.2
    gradiente.BorderSizePixel = 0
    gradiente.Parent = frame
    local cornerGrad = Instance.new("UICorner")
    cornerGrad.CornerRadius = UDim.new(0, 10)
    cornerGrad.Parent = gradiente

    -- TÍTULO "CREATOR: LEGACY"
    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 25)
    titulo.Position = UDim2.new(0, 0, 0, 8)
    titulo.BackgroundTransparency = 1
    titulo.Text = "CREATOR: LEGACY"
    titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
    titulo.TextScaled = true
    titulo.Font = Enum.Font.GothamBold
    titulo.TextStrokeColor3 = Color3.fromRGB(150, 0, 0)
    titulo.TextStrokeTransparency = 0.5
    titulo.Parent = frame

    -- SUBTÍTULO
    local subtitulo = Instance.new("TextLabel")
    subtitulo.Size = UDim2.new(1, 0, 0, 15)
    subtitulo.Position = UDim2.new(0, 0, 0, 30)
    subtitulo.BackgroundTransparency = 1
    subtitulo.Text = "⚡ TESTING TOOLS ⚡"
    subtitulo.TextColor3 = Color3.fromRGB(200, 200, 200)
    subtitulo.TextScaled = true
    subtitulo.Font = Enum.Font.Gotham
    subtitulo.Parent = frame

    -- LINHA DIVISÓRIA
    local linha = Instance.new("Frame")
    linha.Size = UDim2.new(0.9, 0, 0, 1)
    linha.Position = UDim2.new(0.05, 0, 0, 50)
    linha.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    linha.BackgroundTransparency = 0.3
    linha.BorderSizePixel = 0
    linha.Parent = frame

    -- BOTÃO ESP
    local btnESP = Instance.new("TextButton")
    btnESP.Size = UDim2.new(0.9, 0, 0, 40)
    btnESP.Position = UDim2.new(0.05, 0, 0, 60)
    btnESP.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    btnESP.BackgroundTransparency = 0.3
    btnESP.Text = "🔴 ESP: DESLIGADO"
    btnESP.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnESP.TextScaled = true
    btnESP.Font = Enum.Font.GothamBold
    btnESP.Parent = frame
    local cornerBtn1 = Instance.new("UICorner")
    cornerBtn1.CornerRadius = UDim.new(0, 6)
    cornerBtn1.Parent = btnESP

    -- Efeito hover (mouse em cima) - BOTÃO ESP
    btnESP.MouseEnter:Connect(function()
        btnESP.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    end)
    btnESP.MouseLeave:Connect(function()
        if ESP_Ativado then
            btnESP.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        else
            btnESP.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
        end
    end)

    -- BOTÃO AIMBOT
    local btnAimbot = Instance.new("TextButton")
    btnAimbot.Size = UDim2.new(0.9, 0, 0, 40)
    btnAimbot.Position = UDim2.new(0.05, 0, 0, 110)
    btnAimbot.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    btnAimbot.BackgroundTransparency = 0.3
    btnAimbot.Text = "🎯 AIMBOT: DESLIGADO"
    btnAimbot.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnAimbot.TextScaled = true
    btnAimbot.Font = Enum.Font.GothamBold
    btnAimbot.Parent = frame
    local cornerBtn2 = Instance.new("UICorner")
    cornerBtn2.CornerRadius = UDim.new(0, 6)
    cornerBtn2.Parent = btnAimbot

    -- Efeito hover (mouse em cima) - BOTÃO AIMBOT
    btnAimbot.MouseEnter:Connect(function()
        btnAimbot.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    end)
    btnAimbot.MouseLeave:Connect(function()
        if Aimbot_Ativado then
            btnAimbot.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
        else
            btnAimbot.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
        end
    end)

    -- BOTÃO FUTURO (placeholder para mais opções)
    local btnFuturo = Instance.new("TextButton")
    btnFuturo.Size = UDim2.new(0.9, 0, 0, 40)
    btnFuturo.Position = UDim2.new(0.05, 0, 0, 160)
    btnFuturo.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    btnFuturo.BackgroundTransparency = 0.3
    btnFuturo.Text = "⏳ MAIS OPÇÕES EM BREVE"
    btnFuturo.TextColor3 = Color3.fromRGB(150, 150, 150)
    btnFuturo.TextScaled = true
    btnFuturo.Font = Enum.Font.Gotham
    btnFuturo.Parent = frame
    local cornerBtn3 = Instance.new("UICorner")
    cornerBtn3.CornerRadius = UDim.new(0, 6)
    cornerBtn3.Parent = btnFuturo
    btnFuturo.MouseEnter:Connect(function()
        btnFuturo.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    end)
    btnFuturo.MouseLeave:Connect(function()
        btnFuturo.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    end)

    -- RODAPÉ
    local rodape = Instance.new("TextLabel")
    rodape.Size = UDim2.new(1, 0, 0, 20)
    rodape.Position = UDim2.new(0, 0, 0, 225)
    rodape.BackgroundTransparency = 1
    rodape.Text = "Aimbot: 35 studs | Mira: Peito"
    rodape.TextColor3 = Color3.fromRGB(100, 100, 100)
    rodape.TextScaled = true
    rodape.Font = Enum.Font.Gotham
    rodape.Parent = frame

    -- ===== FUNÇÕES DOS BOTÕES =====
    btnESP.MouseButton1Click:Connect(function()
        ESP_Ativado = not ESP_Ativado
        btnESP.Text = ESP_Ativado and "🔴 ESP: LIGADO" or "🔴 ESP: DESLIGADO"
        btnESP.BackgroundColor3 = ESP_Ativado and Color3.fromRGB(200, 50, 50) or Color3.fromRGB(40, 40, 50)
        atualizarESP()
    end)

    btnAimbot.MouseButton1Click:Connect(function()
        Aimbot_Ativado = not Aimbot_Ativado
        btnAimbot.Text = Aimbot_Ativado and "🎯 AIMBOT: LIGADO (35 studs)" or "🎯 AIMBOT: DESLIGADO"
        btnAimbot.BackgroundColor3 = Aimbot_Ativado and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(40, 40, 50)
        if not Aimbot_Ativado then
            alvoAtual = nil
        end
    end)

    -- ===== SISTEMA DE ARRASTAR (CORRIGIDO) =====
    local dragging = false
    local dragStartPos = nil
    local dragFramePos = nil
    local dragInput = nil

    -- Função para iniciar o arraste
    local function startDrag(input)
        dragging = true
        dragStartPos = input.Position
        dragFramePos = frame.Position
    end

    -- Função para atualizar o arraste
    local function updateDrag(input)
        if dragging then
            local delta = input.Position - dragStartPos
            frame.Position = UDim2.new(
                dragFramePos.X.Scale, 
                dragFramePos.X.Offset + delta.X,
                dragFramePos.Y.Scale, 
                dragFramePos.Y.Offset + delta.Y
            )
        end
    end

    -- Função para parar o arraste
    local function stopDrag()
        dragging = false
    end

    -- Eventos para arrastar (qualquer lugar do frame)
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            startDrag(input)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            updateDrag(input)
        end
    end)

    frame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            stopDrag()
        end
    end)

    -- Suporte a toque (dispositivos móveis)
    frame.TouchBegan:Connect(function(touch)
        startDrag(touch)
    end)

    frame.TouchMoved:Connect(function(touch)
        updateDrag(touch)
    end)

    frame.TouchEnded:Connect(function()
        stopDrag()
    end)
end

-- ===== LOOP PRINCIPAL =====
RunService.RenderStepped:Connect(function()
    -- AIMBOT AUTOMÁTICO
    if Aimbot_Ativado then
        local novoAlvo = getJogadorMaisProximo()
        
        if novoAlvo then
            alvoAtual = novoAlvo
            local char = alvoAtual.Character
            if char then
                local alvoRoot = char:FindFirstChild("HumanoidRootPart")
                if alvoRoot then
                    local posMirar = alvoRoot.Position + Vector3.new(0, CONFIG.OffsetAimbotY, 0)
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, posMirar)
                end
            end
        else
            alvoAtual = nil
        end
    else
        if alvoAtual then
            alvoAtual = nil
        end
    end

    -- ESP (mantém os highlights ativos)
    if ESP_Ativado then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer then
                local char = jogador.Character
                if char then
                    if not highlights[jogador] then
                        local novoHighlight = criarHighlight(char)
                        if novoHighlight then
                            highlights[jogador] = novoHighlight
                        end
                    elseif highlights[jogador].Adornee ~= char then
                        highlights[jogador]:Destroy()
                        highlights[jogador] = nil
                        local novoHighlight = criarHighlight(char)
                        if novoHighlight then
                            highlights[jogador] = novoHighlight
                        end
                    end
                else
                    if highlights[jogador] then
                        highlights[jogador]:Destroy()
                        highlights[jogador] = nil
                    end
                end
            end
        end
    end
end)

-- ===== INICIALIZAÇÃO =====
criarMenu()
atualizarESP()
print("Menu de testes carregado! Creator: Legacy - Aimbot automático (35 studs) - Mira no peito.")
