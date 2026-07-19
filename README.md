-- ============================================
-- HACK LAB v6.0 - MOBILE (CLICK NO BOTÃO)
-- ESP + AIMBOT + AUTO-ATAQUE NO BOTÃO
-- ============================================
local Players=game:GetService("Players");local LP=Players.LocalPlayer;local RS=game:GetService("RunService");local UIS=game:GetService("UserInputService");local Camera=workspace.CurrentCamera;local VIM=game:GetService("VirtualInputManager");local plr=LP;local char=nil;local root=nil

-- CONFIGS
local DIST=35;local COR=Color3.fromRGB(255,0,0);local ESP_ON=false;local AIM_ON=false;local ATK_ON=false

local highlights={}

-- FUNÇÕES AUXILIARES
local function getChar() char=plr.Character;if char then root=char:FindFirstChild("HumanoidRootPart") end;return char end

-- ESP
local function criarESP(personagem) if not personagem then return nil end;local h=Instance.new("Highlight");h.Name="ESP";h.Adornee=personagem;h.OutlineColor=COR;h.FillColor=COR;h.FillTransparency=1;h.OutlineTransparency=0;h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop;h.Parent=personagem;return h end
local function removerESP() for _,h in pairs(highlights)do if h and h.Parent then h:Destroy()end end;highlights={} end
local function atualizarESP() if ESP_ON then for _,j in pairs(Players:GetPlayers())do if j~=plr then local c=j.Character;if c and not highlights[j]then local h=criarESP(c);if h then highlights[j]=h end end end end else removerESP() end end

-- AIMBOT
local function getAlvo() local c=getChar()if not c or not root then return nil end;local origem=root.Position;local alvo=nil;local menorDist=DIST;for _,j in pairs(Players:GetPlayers())do if j~=plr then local c2=j.Character;if c2 then local r=c2:FindFirstChild("HumanoidRootPart");local h=c2:FindFirstChild("Humanoid");if r and h and h.Health>0 then local dist=(r.Position-origem).Magnitude;if dist<menorDist then menorDist=dist;alvo=j end end end end end;return alvo end

-- ===== FUNÇÃO QUE CLICA NO BOTÃO DE ATAQUE =====
local botaoAtaque = nil

local function encontrarBotaoAtaque()
    -- Procura em todas as ScreenGuis da tela
    for _, gui in pairs(plr.PlayerGui:GetChildren()) do
        if gui:IsA("ScreenGui") then
            -- Procura por botões com nomes comuns de ataque
            for _, btn in pairs(gui:GetDescendants()) do
                if btn:IsA("ImageButton") or btn:IsA("TextButton") then
                    local nome = btn.Name:lower()
                    -- Lista de nomes comuns de botão de ataque
                    if nome:find("attack") or nome:find("soco") or nome:find("hit") or 
                       nome:find("fight") or nome:find("punch") or nome:find("ataque") or
                       nome:find("kill") or nome:find("damage") or nome:find("lutar") then
                        botaoAtaque = btn
                        print("🔍 Botão de ataque encontrado: " .. btn.Name)
                        return btn
                    end
                end
            end
        end
    end
    
    -- Se não encontrou, tenta achar pela posição (botões grandes no canto inferior direito)
    for _, gui in pairs(plr.PlayerGui:GetChildren()) do
        if gui:IsA("ScreenGui") then
            for _, btn in pairs(gui:GetDescendants()) do
                if (btn:IsA("ImageButton") or btn:IsA("TextButton")) and btn.Visible then
                    -- Verifica se está no canto inferior direito (posição comum de botão de ataque)
                    local pos = btn.AbsolutePosition
                    local size = btn.AbsoluteSize
                    local telaSize = Camera.ViewportSize
                    if pos.X + size.X > telaSize.X * 0.6 and pos.Y + size.Y > telaSize.Y * 0.6 then
                        botaoAtaque = btn
                        print("🔍 Botão de ataque encontrado pela posição: " .. btn.Name)
                        return btn
                    end
                end
            end
        end
    end
    
    print("❌ Nenhum botão de ataque encontrado!")
    return nil
end

-- Função para clicar no botão
local function clicarBotaoAtaque()
    if not botaoAtaque then
        encontrarBotaoAtaque()
        if not botaoAtaque then return end
    end
    
    if botaoAtaque and botaoAtaque.Visible then
        -- Método 1: Disparar evento de clique no botão
        pcall(function()
            botaoAtaque:Click()
        end)
        
        -- Método 2: Simular toque no botão (celular)
        pcall(function()
            local pos = botaoAtaque.AbsolutePosition
            local size = botaoAtaque.AbsoluteSize
            local x = pos.X + size.X/2
            local y = pos.Y + size.Y/2
            VIM:SendMouseButtonEvent(true, x, y, 0, false, Enum.UserInputType.Touch, 0)
            wait(0.01)
            VIM:SendMouseButtonEvent(false, x, y, 0, false, Enum.UserInputType.Touch, 0)
        end)
        
        -- Método 3: Disparar evento InputObject no botão
        pcall(function()
            local input = Instance.new("InputObject")
            input.UserInputType = Enum.UserInputType.Touch
            input.Position = Vector2.new(botaoAtaque.AbsolutePosition.X + botaoAtaque.AbsoluteSize.X/2, 
                                        botaoAtaque.AbsolutePosition.Y + botaoAtaque.AbsoluteSize.Y/2)
            botaoAtaque:Trigger(input)
        end)
    end
end

-- ===== REMOVER COOLDOWN =====
local function removerCooldown()
    local c=getChar()
    if not c then return end
    for _, obj in pairs(c:GetDescendants()) do
        if obj:IsA("NumberValue") and (obj.Name:lower():find("cooldown") or obj.Name:lower():find("cd") or obj.Name:lower():find("delay")) then
            obj.Value = 0
        end
        if obj:IsA("BoolValue") and (obj.Name:lower():find("canattack") or obj.Name:lower():find("ready")) then
            obj.Value = true
        end
        pcall(function()
            obj:SetAttribute("Cooldown", 0)
            obj:SetAttribute("AttackCooldown", 0)
            obj:SetAttribute("CanAttack", true)
        end)
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
    
    -- AIMBOT
    if AIM_ON then
        local alvo=getAlvo()
        if alvo then
            local c2=alvo.Character
            if c2 then
                local r=c2:FindFirstChild("HumanoidRootPart")
                if r then
                    Camera.CFrame=CFrame.new(Camera.CFrame.Position,r.Position)
                end
            end
        end
    end
    
    -- AUTO-ATAQUE (CLICA NO BOTÃO DA TELA)
    if ATK_ON then
        clicarBotaoAtaque()
        removerCooldown()
        wait(0.02)
    end
end)

-- ===== MENU =====
local menuAberto=true;local sg=nil;local f=nil;local btnReabrir=nil

local function criarMenu()
    sg=Instance.new("ScreenGui");sg.Name="HackLab";sg.ResetOnSpawn=false;sg.Parent=plr:WaitForChild("PlayerGui")
    f=Instance.new("Frame");f.Size=UDim2.new(0,260,0,220);f.Position=UDim2.new(0.5,-130,0.5,-110);f.BackgroundColor3=Color3.fromRGB(10,10,20);f.BackgroundTransparency=0.1;f.BorderSizePixel=0;f.ClipsDescendants=true;f.Parent=sg
    local c=Instance.new("UICorner");c.CornerRadius=UDim.new(0,10);c.Parent=f
    
    local t=Instance.new("TextLabel");t.Size=UDim2.new(1,0,0,30);t.Position=UDim2.new(0,0,0,5);t.BackgroundTransparency=1;t.Text="📱 HACK LAB";t.TextColor3=Color3.fromRGB(255,255,255);t.TextScaled=true;t.Font=Enum.Font.GothamBold;t.Parent=f
    
    local btnX=Instance.new("TextButton");btnX.Size=UDim2.new(0,35,0,35);btnX.Position=UDim2.new(1,-40,0,5);btnX.BackgroundColor3=Color3.fromRGB(200,50,50);btnX.Text="✕";btnX.TextColor3=Color3.fromRGB(255,255,255);btnX.TextScaled=true;btnX.Font=Enum.Font.GothamBold;btnX.Parent=f
    local cx=Instance.new("UICorner");cx.CornerRadius=UDim.new(0,8);cx.Parent=btnX
    
    local st=Instance.new("TextLabel");st.Size=UDim2.new(1,0,0,15);st.Position=UDim2.new(0,0,0,32);st.BackgroundTransparency=1;st.Text="CLICA NO BOTÃO DE ATAQUE";st.TextColor3=Color3.fromRGB(150,150,150);st.TextScaled=true;st.Font=Enum.Font.Gotham;st.Parent=f
    
    local function criarBotao(nome,pos)
        local b=Instance.new("TextButton");b.Size=UDim2.new(0.9,0,0,40);b.Position=UDim2.new(0.05,0,0,pos);b.BackgroundColor3=Color3.fromRGB(40,40,50);b.BackgroundTransparency=0.2;b.Text=nome;b.TextColor3=Color3.fromRGB(255,255,255);b.TextScaled=true;b.Font=Enum.Font.GothamBold;b.Parent=f
        local cb=Instance.new("UICorner");cb.CornerRadius=UDim.new(0,6);cb.Parent=b
        return b
    end
    
    local bESP=criarBotao("🔴 ESP: OFF",55)
    local bAIM=criarBotao("🎯 AIMBOT: OFF",100)
    local bATK=criarBotao("⚔️ AUTO-ATAQUE: OFF",145)
    
    bESP.MouseButton1Click:Connect(function()
        ESP_ON=not ESP_ON;bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF";bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50);atualizarESP()
    end)
    bESP.TouchTap:Connect(function()
        ESP_ON=not ESP_ON;bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF";bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50);atualizarESP()
    end)
    
    bAIM.MouseButton1Click:Connect(function()
        AIM_ON=not AIM_ON;bAIM.Text=AIM_ON and"🎯 AIMBOT: ON"or"🎯 AIMBOT: OFF";bAIM.BackgroundColor3=AIM_ON and Color3.fromRGB(50,200,50)or Color3.fromRGB(40,40,50)
    end)
    bAIM.TouchTap:Connect(function()
        AIM_ON=not AIM_ON;bAIM.Text=AIM_ON and"🎯 AIMBOT: ON"or"🎯 AIMBOT: OFF";bAIM.BackgroundColor3=AIM_ON and Color3.fromRGB(50,200,50)or Color3.fromRGB(40,40,50)
    end)
    
    bATK.MouseButton1Click:Connect(function()
        ATK_ON=not ATK_ON;bATK.Text=ATK_ON and"⚔️ AUTO-ATAQUE: ON"or"⚔️ AUTO-ATAQUE: OFF";bATK.BackgroundColor3=ATK_ON and Color3.fromRGB(200,150,50)or Color3.fromRGB(40,40,50)
        if ATK_ON then
            encontrarBotaoAtaque() -- Procura o botão quando ativa
        end
    end)
    bATK.TouchTap:Connect(function()
        ATK_ON=not ATK_ON;bATK.Text=ATK_ON and"⚔️ AUTO-ATAQUE: ON"or"⚔️ AUTO-ATAQUE: OFF";bATK.BackgroundColor3=ATK_ON and Color3.fromRGB(200,150,50)or Color3.fromRGB(40,40,50)
        if ATK_ON then
            encontrarBotaoAtaque()
        end
    end)
    
    btnX.MouseButton1Click:Connect(function()
        menuAberto=false;f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("TextButton");btnReabrir.Size=UDim2.new(0,55,0,55);btnReabrir.Position=UDim2.new(0.02,0,0.85,0);btnReabrir.BackgroundColor3=Color3.fromRGB(200,50,50);btnReabrir.Text="⚔️";btnReabrir.TextColor3=Color3.fromRGB(255,255,255);btnReabrir.TextScaled=true;btnReabrir.Font=Enum.Font.GothamBold;btnReabrir.Parent=sg
            local cr=Instance.new("UICorner");cr.CornerRadius=UDim.new(0,28);cr.Parent=btnReabrir
            btnReabrir.MouseButton1Click:Connect(function() menuAberto=true;f.Visible=true;btnReabrir.Visible=false end)
            btnReabrir.TouchTap:Connect(function() menuAberto=true;f.Visible=true;btnReabrir.Visible=false end)
        else
            btnReabrir.Visible=true
        end
    end)
    btnX.TouchTap:Connect(function()
        menuAberto=false;f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("ScreenGui");btnReabrir.Name="Reabrir";btnReabrir.ResetOnSpawn=false;btnReabrir.Parent=plr.PlayerGui
            local b=Instance.new("TextButton");b.Size=UDim2.new(0,55,0,55);b.Position=UDim2.new(0.02,0,0.85,0);b.BackgroundColor3=Color3.fromRGB(200,50,50);b.Text="⚔️";b.TextColor3=Color3.fromRGB(255,255,255);b.TextScaled=true;b.Font=Enum.Font.GothamBold;b.Parent=btnReabrir
            local cr=Instance.new("UICorner");cr.CornerRadius=UDim.new(0,28);cr.Parent=b
            b.MouseButton1Click:Connect(function() menuAberto=true;f.Visible=true;btnReabrir:Destroy() end)
            b.TouchTap:Connect(function() menuAberto=true;f.Visible=true;btnReabrir:Destroy() end)
        else
            btnReabrir.Visible=true
        end
    end)
end

criarMenu()
atualizarESP()
print("📱 Hack Lab v6.0 - Procura e clica no botão de ataque!")
