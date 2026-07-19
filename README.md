-- ============================================
-- SCRIPT UNIFICADO: MENU + ESP + AIMBOT (AUTOMÁTICO)
-- (Apenas para testes no seu próprio jogo)
-- ============================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ===== CONFIGURAÇÕES =====
local CONFIG = {
    DistanciaAimbot = 50,           -- Só mira se estiver a 50 studs ou menos
    OffsetAimbotY = 1.5,
    CorESP = Color3.fromRGB(255, 0, 0) -- Vermelho
}

-- ===== ESTADOS DO MENU =====
local ESP_Ativado = false
local Aimbot_Ativado = false

-- ===== GERENCIADOR DE HIGHLIGHTS (ESP) =====
local highlights = {}

local function criarHighlight(personagem)
    if not personagem then return end
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

local function atualizarESP()
    if ESP_Ativado then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer then
                local char = jogador.Character
                if char and not highlights[jogador] then
                    highlights[jogador] = criarHighlight(char)
                end
            end
        end
    else
        for jogador, highlight in pairs(highlights) do
            if highlight and highlight.Parent then
                highlight:Destroy()
            end
        end
        highlights = {}
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
            highlights[jogador] = criarHighlight(char)
        end
    end)
end

for _, jogador in ipairs(Players:GetPlayers()) do
    if jogador ~= LocalPlayer then
        conectarRespawn(jogador)
    end
end
Players.PlayerAdded:Connect(conectarRespawn)

-- ===== LÓGICA DO AIMBOT (AUTOMÁTICO) =====
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

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 220, 0, 160)
    frame.Position = UDim2.new(0, 20, 0, 20)
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    frame.BackgroundTransparency = 0.15
    frame.BorderSizePixel = 0
    frame.ClipsDescendants = true
    frame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = frame

    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 30)
    titulo.Position = UDim2.new(0, 0, 0, 5)
    titulo.BackgroundTransparency = 1
    titulo.Text = "🎯 MENU DE TESTES"
    titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
    titulo.TextScaled = true
    titulo.Font = Enum.Font.GothamBold
    titulo.Parent = frame

    local btnESP = Instance.new("TextButton")
    btnESP.Size = UDim2.new(0.9, 0, 0, 35)
    btnESP.Position = UDim2.new(0.05, 0, 0, 45)
    btnESP.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    btnESP.Text = "🔴 ESP: DESLIGADO"
    btnESP.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnESP.TextScaled = true
    btnESP.Font = Enum.Font.Gotham
    btnESP.Parent = frame
    local cornerBtn1 = Instance.new("UICorner")
    cornerBtn1.CornerRadius = UDim.new(0, 5)
    cornerBtn1.Parent = btnESP

    local btnAimbot = Instance.new("TextButton")
    btnAimbot.Size = UDim2.new(0.9, 0, 0, 35)
    btnAimbot.Position = UDim2.new(0.05, 0, 0, 90)
    btnAimbot.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    btnAimbot.Text = "🎯 AIMBOT: DESLIGADO"
    btnAimbot.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnAimbot.TextScaled = true
    btnAimbot.Font = Enum.Font.Gotham
    btnAimbot.Parent = frame
    local cornerBtn2 = Instance.new("UICorner")
    cornerBtn2.CornerRadius = UDim.new(0, 5)
    cornerBtn2.Parent = btnAimbot

    local rodape = Instance.new("TextLabel")
    rodape.Size = UDim2.new(1, 0, 0, 20)
    rodape.Position = UDim2.new(0, 0, 0, 135)
    rodape.BackgroundTransparency = 1
    rodape.Text = "Automático (50 studs)"
    rodape.TextColor3 = Color3.fromRGB(150, 150, 150)
    rodape.TextScaled = true
    rodape.Font = Enum.Font.Gotham
    rodape.Parent = frame

    btnESP.MouseButton1Click:Connect(function()
        ESP_Ativado = not ESP_Ativado
        btnESP.Text = ESP_Ativado and "🔴 ESP: LIGADO" or "🔴 ESP: DESLIGADO"
        btnESP.BackgroundColor3 = ESP_Ativado and Color3.fromRGB(200, 50, 50) or Color3.fromRGB(60, 60, 70)
        atualizarESP()
    end)

    btnAimbot.MouseButton1Click:Connect(function()
        Aimbot_Ativado = not Aimbot_Ativado
        btnAimbot.Text = Aimbot_Ativado and "🎯 AIMBOT: LIGADO (50 studs)" or "🎯 AIMBOT: DESLIGADO"
        btnAimbot.BackgroundColor3 = Aimbot_Ativado and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(60, 60, 70)
    end)

    -- Sistema de arrastar
    local dragging = false
    local dragStartPos = nil
    local dragFramePos = nil

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStartPos = input.Position
            dragFramePos = frame.Position
        end
    end)

    frame.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStartPos
            frame.Position = UDim2.new(dragFramePos.X.Scale, dragFramePos.X.Offset + delta.X,
                                      dragFramePos.Y.Scale, dragFramePos.Y.Offset + delta.Y)
        end
    end)

    frame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
end

-- ===== LOOP PRINCIPAL (Aimbot automático + ESP contínuo) =====
RunService.RenderStepped:Connect(function()
    -- AIMBOT AUTOMÁTICO (só se ativado no menu)
    if Aimbot_Ativado then
        local alvo = getJogadorMaisProximo()
        if alvo and alvo.Character and alvo.Character:FindFirstChild("HumanoidRootPart") then
            local posMirar = alvo.Character.HumanoidRootPart.Position + Vector3.new(0, CONFIG.OffsetAimbotY, 0)
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, posMirar)
        end
        -- Se não tiver ninguém a 50 studs, a câmera fica livre (não mexe)
    end

    -- ESP (mantém os highlights ativos)
    if ESP_Ativado then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer then
                local char = jogador.Character
                if char then
                    if not highlights[jogador] then
                        highlights[jogador] = criarHighlight(char)
                    elseif highlights[jogador].Parent ~= char then
                        highlights[jogador]:Destroy()
                        highlights[jogador] = nil
                        highlights[jogador] = criarHighlight(char)
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
print("Menu de testes carregado! Aimbot automático (50 studs).")
