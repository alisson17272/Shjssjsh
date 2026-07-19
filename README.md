local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Configuração do ESP
local ESP_CONFIG = {
    OutlineColor = Color3.fromRGB(255, 0, 0), -- Vermelho puro
    FillColor = Color3.fromRGB(255, 0, 0),   -- Vermelho (mas vamos deixar transparente)
    FillTransparency = 1,                    -- 1 = completamente transparente (só a borda)
    OutlineTransparency = 0,                 -- 0 = borda totalmente opaca
    DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- Mostra através das paredes
}

local function criarESP(personagem)
    -- Evita criar ESP em si mesmo ou em personagens nulos
    if not personagem or personagem == LocalPlayer.Character then return end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "MeuESP_Vermelho"
    highlight.Adornee = personagem
    highlight.OutlineColor = ESP_CONFIG.OutlineColor
    highlight.FillColor = ESP_CONFIG.FillColor
    highlight.FillTransparency = ESP_CONFIG.FillTransparency
    highlight.OutlineTransparency = ESP_CONFIG.OutlineTransparency
    highlight.DepthMode = ESP_CONFIG.DepthMode
    highlight.Parent = personagem -- Gruda no personagem
    
    return highlight
end

local function removerESP(personagem)
    if personagem then
        local highlight = personagem:FindFirstChild("MeuESP_Vermelho")
        if highlight then highlight:Destroy() end
    end
end

-- Escaneia jogadores já existentes no servidor
for _, jogador in ipairs(Players:GetPlayers()) do
    if jogador ~= LocalPlayer and jogador.Character then
        criarESP(jogador.Character)
    end
end

-- Dispara quando um novo jogador entra
Players.PlayerAdded:Connect(function(jogador)
    jogador.CharacterAdded:Connect(function(personagem)
        criarESP(personagem)
    end)
end)

-- Dispara quando o personagem do jogador respawna (morre e renasce)
Players.PlayerRemoving:Connect(function(jogador)
    -- Limpa o ESP se o jogador sair
    if jogador.Character then
        removerESP(jogador.Character)
    end
end)

-- Detecta personagens que entram no Workspace (útil para respawn)
RunService.Heartbeat:Connect(function()
    for _, jogador in ipairs(Players:GetPlayers()) do
        if jogador ~= LocalPlayer and jogador.Character then
            if not jogador.Character:FindFirstChild("MeuESP_Vermelho") then
                criarESP(jogador.Character)
            end
        end
    end
end)
