-- ============================================
-- HACK LAB v7.0 - AIMBOT PARA JOGO DE TIRO
-- ESP (TODOS) + AIMBOT (VISÍVEIS + INIMIGOS)
-- ============================================
local Players=game:GetService("Players");local LP=Players.LocalPlayer;local RS=game:GetService("RunService");local Camera=workspace.CurrentCamera;local plr=LP;local char=nil;local root=nil

-- CONFIGS
local COR=Color3.fromRGB(255,0,0);local ESP_ON=false;local AIM_ON=false;local MODO_INIMIGOS=false
local DISTANCIA_MAXIMA=1000 -- Alcance infinito (pode aumentar)

local highlights={}
local alvoAtual=nil

-- FUNÇÕES AUXILIARES
local function getChar() char=plr.Character;if char then root=char:FindFirstChild("HumanoidRootPart") end;return char end
local function getHumanoid() local c=getChar();return c and c:FindFirstChild("Humanoid") end

-- ===== DETECÇÃO DE TIME (3 MÉTODOS) =====
local function isInimigo(jogador)
    if jogador == plr then return false end
    
    -- MÉTODO 1: Team Color (mais comum)
    local team1 = plr.Team
    local team2 = jogador.Team
    if team1 and team2 then
        return team1 ~= team2
    end
    
    -- MÉTODO 2: Atributo "Team" no jogador
    local teamAtrib1 = plr:GetAttribute("Team")
    local teamAtrib2 = jogador:GetAttribute("Team")
    if teamAtrib1 and teamAtrib2 then
        return teamAtrib1 ~= teamAtrib2
    end
    
    -- MÉTODO 3: Nome da equipe (ex: "Red", "Blue")
    local teamName1 = plr:GetAttribute("TeamName") or plr:GetAttribute("TeamColor")
    local teamName2 = jogador:GetAttribute("TeamName") or jogador:GetAttribute("TeamColor")
    if teamName1 and teamName2 then
        return teamName1 ~= teamName2
    end
    
    -- Se não conseguir detectar, considera todos inimigos (exceto você)
    return true
end

-- ===== DETECÇÃO DE VISIBILIDADE (Raycast) =====
local function isVisivel(origem, destino)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {plr.Character}
    
    local ray = workspace:Raycast(origem, destino - origem, raycastParams)
    
    -- Se não bateu em nada, está visível
    if not ray then return true end
    
    -- Se bateu no personagem do alvo, está visível
    local alvoChar = ray.Instance
    while alvoChar and alvoChar.Parent do
        if alvoChar:IsA("Model") and alvoChar:FindFirstChild("Humanoid") then
            return true
        end
        alvoChar = alvoChar.Parent
    end
    
    return false
end

-- ===== ESP =====
local function criarESP(personagem) if not personagem then return nil end;local h=Instance.new("Highlight");h.Name="ESP";h.Adornee=personagem;h.OutlineColor=COR;h.FillColor=COR;h.FillTransparency=1;h.OutlineTransparency=0;h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop;h.Parent=personagem;return h end
local function removerESP() for _,h in pairs(highlights)do if h and h.Parent then h:Destroy()end end;highlights={} end
local function atualizarESP() if ESP_ON then for _,j in pairs(Players:GetPlayers())do if j~=plr then local c=j.Character;if c and not highlights[j]then local h=criarESP(c);if h then highlights[j]=h end end end end else removerESP() end end

-- ===== AIMBOT (VISÍVEL + INIMIGO) =====
local function getMelhorAlvo()
    local c=getChar() if not c or not root then return nil end
    local origem = root.Position
    local melhorAlvo = nil
    local menorDist = DISTANCIA_MAXIMA
    
    for _, jogador in pairs(Players:GetPlayers()) do
        if jogador ~= plr then
            local c2 = jogador.Character
            if c2 then
                local r = c2:FindFirstChild("HumanoidRootPart")
                local h = c2:FindFirstChild("Humanoid")
                if r and h and h.Health > 0 then
                    -- VERIFICA SE É INIMIGO
                    if MODO_INIMIGOS and not isInimigo(jogador) then
                        continue -- Pula se for aliado
                    end
                    
                    -- VERIFICA DISTÂNCIA
                    local dist = (r.Position - origem).Magnitude
                    if dist > DISTANCIA_MAXIMA then continue end
                    
                    -- VERIFICA VISIBILIDADE (NÃO PODE ESTAR ATRÁS DA PAREDE)
                    if not isVisivel(origem, r.Position) then
                        continue -- Pula se estiver atrás da parede
                    end
                    
                    -- ESCOLHE O MAIS PRÓXIMO
                    if dist < menorDist then
                        menorDist = dist
                        melhorAlvo = jogador
                    end
                end
            end
        end
    end
    
    return melhorAlvo
end

-- ===== LOOP PRINCIPAL =====
RS.RenderStepped:Connect(function()
    local c = getChar()
    
    -- ESP (TODOS OS JOGADORES)
    if ESP_ON then
        for _,j in pairs(Players:GetPlayers()) do
            if j ~= plr then
                local c2 = j.Character
                if c2 then
                    if not highlights[j] then
                        local h = criarESP(c2); if h then highlights[j] = h end
                    elseif highlights[j].Adornee ~= c2 then
                        highlights[j]:Destroy(); highlights[j] = nil
                        local h = criarESP(c2); if h then highlights[j] = h end
                    end
                else
                    if highlights[j] then highlights[j]:Destroy(); highlights[j] = nil end
                end
            end
        end
    end
    
    -- AIMBOT (SÓ INIMIGOS VISÍVEIS)
    if AIM_ON then
        local alvo = getMelhorAlvo()
        if alvo then
            alvoAtual = alvo
            local c2 = alvo.Character
            if c2 then
                local r = c2:FindFirstChild("HumanoidRootPart")
                if r then
                    -- Mira no peito (centro do HumanoidRootPart)
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, r.Position)
                end
            end
        else
            alvoAtual = nil
        end
    end
end)

-- ===== MENU =====
local menuAberto=true; local sg=nil; local f=nil; local btnReabrir=nil

local function criarMenu()
    sg = Instance.new("ScreenGui"); sg.Name = "HackLab"; sg.ResetOnSpawn = false; sg.Parent = plr:WaitForChild("PlayerGui")
    f = Instance.new("Frame"); f.Size = UDim2.new(0, 280, 0, 250); f.Position = UDim2.new(0.5, -140, 0.5, -125); f.BackgroundColor3 = Color3.fromRGB(10,10,20); f.BackgroundTransparency = 0.1; f.BorderSizePixel = 0; f.ClipsDescendants = true; f.Parent = sg
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,10); c.Parent = f
    
    local t = Instance.new("TextLabel"); t.Size = UDim2.new(1,0,0,30); t.Position = UDim2.new(0,0,0,5); t.BackgroundTransparency = 1; t.Text = "🔫 HACK LAB"; t.TextColor3 = Color3.fromRGB(255,255,255); t.TextScaled = true; t.Font = Enum.Font.GothamBold; t.Parent = f
    
    local btnX = Instance.new("TextButton"); btnX.Size = UDim2.new(0,35,0,35); btnX.Position = UDim2.new(1,-40,0,5); btnX.BackgroundColor3 = Color3.fromRGB(200,50,50); btnX.Text = "✕"; btnX.TextColor3 = Color3.fromRGB(255,255,255); btnX.TextScaled = true; btnX.Font = Enum.Font.GothamBold; btnX.Parent = f
    local cx = Instance.new("UICorner"); cx.CornerRadius = UDim.new(0,8); cx.Parent = btnX
    
    local st = Instance.new("TextLabel"); st.Size = UDim2.new(1,0,0,15); st.Position = UDim2.new(0,0,0,32); st.BackgroundTransparency = 1; st.Text = "AIMBOT: VISÍVEIS + INIMIGOS"; st.TextColor3 = Color3.fromRGB(150,150,150); st.TextScaled = true; st.Font = Enum.Font.Gotham; st.Parent = f
    
    local function criarBotao(nome, pos)
        local b = Instance.new("TextButton"); b.Size = UDim2.new(0.9,0,0,38); b.Position = UDim2.new(0.05,0,0,pos); b.BackgroundColor3 = Color3.fromRGB(40,40,50); b.BackgroundTransparency = 0.2; b.Text = nome; b.TextColor3 = Color3.fromRGB(255,255,255); b.TextScaled = true; b.Font = Enum.Font.GothamBold; b.Parent = f
        local cb = Instance.new("UICorner"); cb.CornerRadius = UDim.new(0,6); cb.Parent = b
        return b
    end
    
    local bESP = criarBotao("🔴 ESP: OFF", 52)
    local bAIM = criarBotao("🎯 AIMBOT: OFF", 95)
    local bTEAM = criarBotao("👥 SÓ INIMIGOS: ON", 138) -- Começa ativado
    local bDIST = criarBotao("📏 DISTÂNCIA: 1000+", 181)
    
    -- FUNÇÕES DOS BOTÕES
    bESP.MouseButton1Click:Connect(function()
        ESP_ON = not ESP_ON; bESP.Text = ESP_ON and "🔴 ESP: ON" or "🔴 ESP: OFF"; bESP.BackgroundColor3 = ESP_ON and Color3.fromRGB(200,50,50) or Color3.fromRGB(40,40,50); atualizarESP()
    end)
    bESP.TouchTap:Connect(function()
        ESP_ON = not ESP_ON; bESP.Text = ESP_ON and "🔴 ESP: ON" or "🔴 ESP: OFF"; bESP.BackgroundColor3 = ESP_ON and Color3.fromRGB(200,50,50) or Color3.fromRGB(40,40,50); atualizarESP()
    end)
    
    bAIM.MouseButton1Click:Connect(function()
        AIM_ON = not AIM_ON; bAIM.Text = AIM_ON and "🎯 AIMBOT: ON" or "🎯 AIMBOT: OFF"; bAIM.BackgroundColor3 = AIM_ON and Color3.fromRGB(50,200,50) or Color3.fromRGB(40,40,50)
    end)
    bAIM.TouchTap:Connect(function()
        AIM_ON = not AIM_ON; bAIM.Text = AIM_ON and "🎯 AIMBOT: ON" or "🎯 AIMBOT: OFF"; bAIM.BackgroundColor3 = AIM_ON and Color3.fromRGB(50,200,50) or Color3.fromRGB(40,40,50)
    end)
    
    bTEAM.MouseButton1Click:Connect(function()
        MODO_INIMIGOS = not MODO_INIMIGOS; bTEAM.Text = MODO_INIMIGOS and "👥 SÓ INIMIGOS: ON" or "👥 SÓ INIMIGOS: OFF"; bTEAM.BackgroundColor3 = MODO_INIMIGOS and Color3.fromRGB(50,150,200) or Color3.fromRGB(40,40,50)
    end)
    bTEAM.TouchTap:Connect(function()
        MODO_INIMIGOS = not MODO_INIMIGOS; bTEAM.Text = MODO_INIMIGOS and "👥 SÓ INIMIGOS: ON" or "👥 SÓ INIMIGOS: OFF"; bTEAM.BackgroundColor3 = MODO_INIMIGOS and Color3.fromRGB(50,150,200) or Color3.fromRGB(40,40,50)
    end)
    
    bDIST.MouseButton1Click:Connect(function()
        if DISTANCIA_MAXIMA == 1000 then
            DISTANCIA_MAXIMA = 99999
            bDIST.Text = "📏 DISTÂNCIA: INFINITA"
        else
            DISTANCIA_MAXIMA = 1000
            bDIST.Text = "📏 DISTÂNCIA: 1000+"
        end
    end)
    bDIST.TouchTap:Connect(function()
        if DISTANCIA_MAXIMA == 1000 then
            DISTANCIA_MAXIMA = 99999
            bDIST.Text = "📏 DISTÂNCIA: INFINITA"
        else
            DISTANCIA_MAXIMA = 1000
            bDIST.Text = "📏 DISTÂNCIA: 1000+"
        end
    end)
    
    -- BOTÃO MINIMIZAR
    btnX.MouseButton1Click:Connect(function()
        menuAberto=false; f.Visible=false
        if not btnReabrir then
            btnReabrir = Instance.new("TextButton"); btnReabrir.Size = UDim2.new(0,55,0,55); btnReabrir.Position = UDim2.new(0.02,0,0.85,0); btnReabrir.BackgroundColor3 = Color3.fromRGB(200,50,50); btnReabrir.Text = "🔫"; btnReabrir.TextColor3 = Color3.fromRGB(255,255,255); btnReabrir.TextScaled = true; btnReabrir.Font = Enum.Font.GothamBold; btnReabrir.Parent = sg
            local cr = Instance.new("UICorner"); cr.CornerRadius = UDim.new(0,28); cr.Parent = btnReabrir
            btnReabrir.MouseButton1Click:Connect(function() menuAberto=true; f.Visible=true; btnReabrir:Destroy() end)
            btnReabrir.TouchTap:Connect(function() menuAberto=true; f.Visible=true; btnReabrir:Destroy() end)
        else
            btnReabrir.Visible = true
        end
    end)
    btnX.TouchTap:Connect(function()
        menuAberto=false; f.Visible=false
        if not btnReabrir then
            btnReabrir = Instance.new("TextButton"); btnReabrir.Size = UDim2.new(0,55,0,55); btnReabrir.Position = UDim2.new(0.02,0,0.85,0); btnReabrir.BackgroundColor3 = Color3.fromRGB(200,50,50); btnReabrir.Text = "🔫"; btnReabrir.TextColor3 = Color3.fromRGB(255,255,255); btnReabrir.TextScaled = true; btnReabrir.Font = Enum.Font.GothamBold; btnReabrir.Parent = sg
            local cr = Instance.new("UICorner"); cr.CornerRadius = UDim.new(0,28); cr.Parent = btnReabrir
            btnReabrir.MouseButton1Click:Connect(function() menuAberto=true; f.Visible=true; btnReabrir:Destroy() end)
            btnReabrir.TouchTap:Connect(function() menuAberto=true; f.Visible=true; btnReabrir:Destroy() end)
        else
            btnReabrir.Visible = true
        end
    end)
end

criarMenu()
atualizarESP()
print("🔫 Hack Lab v7.0 - Aimbot: Visíveis + Inimigos")
