-- ============================================
-- HACK LAB v10.0 - KILL AURA (VOID) + TELEPORT LOOP
-- ============================================
local Players=game:GetService("Players");local LP=Players.LocalPlayer;local RS=game:GetService("RunService");local plr=LP;local char=nil;local root=nil

-- ===== CONFIGS =====
local ESP_ON=false;local KILL_ON=false;local TELEPORT_LOOP=false
local COR=Color3.fromRGB(255,0,0)
local highlights={}
local loopIndex=1
local tempoUltimoTeleport=0

-- ===== FUNÇÕES AUXILIARES =====
local function getChar() char=plr.Character;if char then root=char:FindFirstChild("HumanoidRootPart") end;return char end

-- ===== ESP =====
local function criarESP(personagem) if not personagem then return nil end;local h=Instance.new("Highlight");h.Name="ESP";h.Adornee=personagem;h.OutlineColor=COR;h.FillColor=COR;h.FillTransparency=1;h.OutlineTransparency=0;h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop;h.Parent=personagem;return h end
local function removerESP() for _,h in pairs(highlights)do if h and h.Parent then h:Destroy()end end;highlights={} end
local function atualizarESP() if ESP_ON then for _,j in pairs(Players:GetPlayers())do if j~=plr then local c=j.Character;if c and not highlights[j]then local h=criarESP(c);if h then highlights[j]=h end end end end else removerESP() end end

-- ===== KILL AURA (VOID) =====
local function killAura()
    local c=getChar() if not c or not root then return end
    local origem=root.Position
    for _,j in pairs(Players:GetPlayers())do
        if j~=plr then
            local c2=j.Character
            if c2 then
                local r=c2:FindFirstChild("HumanoidRootPart")
                local h=c2:FindFirstChild("Humanoid")
                if r and h and h.Health>0 then
                    local dist=(r.Position-origem).Magnitude
                    if dist<20 then
                        -- JOGA DIRETO PRO VOID (morre instantaneamente)
                        pcall(function()
                            r.CFrame = CFrame.new(0, -5000, 0)
                            h.Health = 0
                            h:BreakJoints()
                        end)
                        print("💀 Mandou pro void: "..j.Name)
                    end
                end
            end
        end
    end
end

-- ===== TELEPORT LOOP (VAI TELEPORTANDO UM POR UM) =====
local function teleportLoop()
    local c=getChar() if not c or not root then return end
    
    -- Pega todos os jogadores (exceto você)
    local jogadores = {}
    for _,j in pairs(Players:GetPlayers()) do
        if j~=plr then
            local c2=j.Character
            if c2 and c2:FindFirstChild("HumanoidRootPart") then
                local h=c2:FindFirstChild("Humanoid")
                if h and h.Health>0 then
                    table.insert(jogadores, j)
                end
            end
        end
    end
    
    if #jogadores == 0 then return end
    
    -- Reseta o índice se passou do limite
    if loopIndex > #jogadores then
        loopIndex = 1
    end
    
    -- Pega o jogador atual
    local alvo = jogadores[loopIndex]
    if alvo then
        local alvoChar = alvo.Character
        if alvoChar then
            local alvoRoot = alvoChar:FindFirstChild("HumanoidRootPart")
            if alvoRoot then
                -- Teleporta para perto do jogador
                local posAlvo = alvoRoot.Position
                local lookVector = alvoRoot.CFrame.LookVector
                local posAtras = posAlvo - lookVector*3
                
                root.CFrame = CFrame.new(posAtras, posAlvo)
                print("🚀 Teleportou para: "..alvo.Name)
                
                -- Avança para o próximo jogador
                loopIndex = loopIndex + 1
            end
        end
    end
end

-- ===== LOOP PRINCIPAL =====
RS.RenderStepped:Connect(function()
    local c=getChar()
    
    -- ESP
    if ESP_ON then
        for _,j in pairs(Players:GetPlayers())do
            if j~=plr then
                local c2=j.Character
                if c2 then
                    if not highlights[j]then
                        local h=criarESP(c2);if h then highlights[j]=h end
                    elseif highlights[j].Adornee~=c2 then
                        highlights[j]:Destroy();highlights[j]=nil
                        local h=criarESP(c2);if h then highlights[j]=h end
                    end
                else
                    if highlights[j]then highlights[j]:Destroy();highlights[j]=nil end
                end
            end
        end
    end
    
    -- KILL AURA (VOID)
    if KILL_ON then
        killAura()
    end
    
    -- TELEPORT LOOP (teleporta um por um)
    if TELEPORT_LOOP then
        local tempoAtual = tick()
        if tempoAtual - tempoUltimoTeleport > 0.5 then -- A cada 0.5 segundos
            teleportLoop()
            tempoUltimoTeleport = tempoAtual
        end
    end
end)

-- ===== MENU =====
local menuAberto=true;local sg=nil;local f=nil;local btnReabrir=nil

local function criarMenu()
    sg=Instance.new("ScreenGui");sg.Name="HackLab";sg.ResetOnSpawn=false;sg.Parent=plr:WaitForChild("PlayerGui")
    
    -- FRAME PRINCIPAL
    f=Instance.new("Frame");f.Size=UDim2.new(0,280,0,280);f.Position=UDim2.new(0.5,-140,0.5,-140);f.BackgroundColor3=Color3.fromRGB(10,10,20);f.BackgroundTransparency=0.1;f.BorderSizePixel=0;f.ClipsDescendants=true;f.Parent=sg
    local c=Instance.new("UICorner");c.CornerRadius=UDim.new(0,12);c.Parent=f
    
    -- TÍTULO
    local t=Instance.new("TextLabel");t.Size=UDim2.new(1,0,0,35);t.Position=UDim2.new(0,0,0,5);t.BackgroundTransparency=1;t.Text="💀 HACK LAB";t.TextColor3=Color3.fromRGB(255,255,255);t.TextScaled=true;t.Font=Enum.Font.GothamBold;t.Parent=f
    
    -- BOTÃO FECHAR
    local btnX=Instance.new("TextButton");btnX.Size=UDim2.new(0,35,0,35);btnX.Position=UDim2.new(1,-40,0,5);btnX.BackgroundColor3=Color3.fromRGB(200,50,50);btnX.Text="✕";btnX.TextColor3=Color3.fromRGB(255,255,255);btnX.TextScaled=true;btnX.Font=Enum.Font.GothamBold;btnX.Parent=f
    local cx=Instance.new("UICorner");cx.CornerRadius=UDim.new(0,8);cx.Parent=btnX
    
    -- SUBTÍTULO
    local st=Instance.new("TextLabel");st.Size=UDim2.new(1,0,0,15);st.Position=UDim2.new(0,0,0,37);st.BackgroundTransparency=1;st.Text="KILL AURA (VOID) + TELEPORT LOOP";st.TextColor3=Color3.fromRGB(150,150,150);st.TextScaled=true;st.Font=Enum.Font.Gotham;st.Parent=f
    
    -- LINHA DIVISÓRIA
    local linha=Instance.new("Frame");linha.Size=UDim2.new(0.9,0,0,1);linha.Position=UDim2.new(0.05,0,0,55);linha.BackgroundColor3=Color3.fromRGB(255,255,255);linha.BackgroundTransparency=0.3;linha.Parent=f
    
    -- ===== FUNÇÃO PARA CRIAR BOTÕES =====
    local function criarBotao(nome,pos,cor)
        local b=Instance.new("TextButton");b.Size=UDim2.new(0.9,0,0,45);b.Position=UDim2.new(0.05,0,0,pos);b.BackgroundColor3=cor or Color3.fromRGB(40,40,50);b.BackgroundTransparency=0.2;b.Text=nome;b.TextColor3=Color3.fromRGB(255,255,255);b.TextScaled=true;b.Font=Enum.Font.GothamBold;b.Parent=f
        local cb=Instance.new("UICorner");cb.CornerRadius=UDim.new(0,6);cb.Parent=b
        return b
    end
    
    -- ===== BOTÕES =====
    local bESP=criarBotao("🔴 ESP: OFF",65)
    local bKILL=criarBotao("💀 KILL AURA (VOID): OFF",120,Color3.fromRGB(200,50,50))
    local bTELEPORT=criarBotao("🔄 TELEPORT LOOP: OFF",175,Color3.fromRGB(50,100,200))
    
    -- ===== FUNÇÕES DOS BOTÕES =====
    bESP.MouseButton1Click:Connect(function()
        ESP_ON=not ESP_ON;bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF";bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50);atualizarESP()
    end)
    bESP.TouchTap:Connect(function()
        ESP_ON=not ESP_ON;bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF";bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50);atualizarESP()
    end)
    
    bKILL.MouseButton1Click:Connect(function()
        KILL_ON=not KILL_ON;bKILL.Text=KILL_ON and"💀 KILL AURA (VOID): ON"or"💀 KILL AURA (VOID): OFF";bKILL.BackgroundColor3=KILL_ON and Color3.fromRGB(200,0,0)or Color3.fromRGB(40,40,50)
    end)
    bKILL.TouchTap:Connect(function()
        KILL_ON=not KILL_ON;bKILL.Text=KILL_ON and"💀 KILL AURA (VOID): ON"or"💀 KILL AURA (VOID): OFF";bKILL.BackgroundColor3=KILL_ON and Color3.fromRGB(200,0,0)or Color3.fromRGB(40,40,50)
    end)
    
    bTELEPORT.MouseButton1Click:Connect(function()
        TELEPORT_LOOP=not TELEPORT_LOOP;bTELEPORT.Text=TELEPORT_LOOP and"🔄 TELEPORT LOOP: ON"or"🔄 TELEPORT LOOP: OFF";bTELEPORT.BackgroundColor3=TELEPORT_LOOP and Color3.fromRGB(50,100,200)or Color3.fromRGB(40,40,50)
        if TELEPORT_LOOP then
            loopIndex=1 -- Reseta o índice quando ativa
            tempoUltimoTeleport=0
        end
    end)
    bTELEPORT.TouchTap:Connect(function()
        TELEPORT_LOOP=not TELEPORT_LOOP;bTELEPORT.Text=TELEPORT_LOOP and"🔄 TELEPORT LOOP: ON"or"🔄 TELEPORT LOOP: OFF";bTELEPORT.BackgroundColor3=TELEPORT_LOOP and Color3.fromRGB(50,100,200)or Color3.fromRGB(40,40,50)
        if TELEPORT_LOOP then
            loopIndex=1
            tempoUltimoTeleport=0
        end
    end)
    
    -- ===== BOTÃO MINIMIZAR =====
    btnX.MouseButton1Click:Connect(function()
        menuAberto=false;f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("TextButton");btnReabrir.Size=UDim2.new(0,55,0,55);btnReabrir.Position=UDim2.new(0.02,0,0.85,0);btnReabrir.BackgroundColor3=Color3.fromRGB(200,50,50);btnReabrir.Text="💀";btnReabrir.TextColor3=Color3.fromRGB(255,255,255);btnReabrir.TextScaled=true;btnReabrir.Font=Enum.Font.GothamBold;btnReabrir.Parent=sg
            local cr=Instance.new("UICorner");cr.CornerRadius=UDim.new(0,28);cr.Parent=btnReabrir
            btnReabrir.MouseButton1Click:Connect(function() menuAberto=true;f.Visible=true;btnReabrir:Destroy() end)
            btnReabrir.TouchTap:Connect(function() menuAberto=true;f.Visible=true;btnReabrir:Destroy() end)
        else
            btnReabrir.Visible=true
        end
    end)
    btnX.TouchTap:Connect(function()
        menuAberto=false;f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("TextButton");btnReabrir.Size=UDim2.new(0,55,0,55);btnReabrir.Position=UDim2.new(0.02,0,0.85,0);btnReabrir.BackgroundColor3=Color3.fromRGB(200,50,50);btnReabrir.Text="💀";btnReabrir.TextColor3=Color3.fromRGB(255,255,255);btnReabrir.TextScaled=true;btnReabrir.Font=Enum.Font.GothamBold;btnReabrir.Parent=sg
            local cr=Instance.new("UICorner");cr.CornerRadius=UDim.new(0,28);cr.Parent=btnReabrir
            btnReabrir.MouseButton1Click:Connect(function() menuAberto=true;f.Visible=true;btnReabrir:Destroy() end)
            btnReabrir.TouchTap:Connect(function() menuAberto=true;f.Visible=true;btnReabrir:Destroy() end)
        else
            btnReabrir.Visible=true
        end
    end)
end

criarMenu()
atualizarESP()
print("💀 Hack Lab v10.0 - KILL AURA (VOID) + TELEPORT LOOP")
