-- ============================================
-- HACK LAB v9.1 - KILL AURA (MATA DIRETO) + TELEPORT GRUDAR
-- ============================================
local Players=game:GetService("Players");local LP=Players.LocalPlayer;local RS=game:GetService("RunService");local plr=LP;local char=nil;local root=nil

-- ===== CONFIGS =====
local ESP_ON=false;local KILL_ON=false;local GRUDAR_ON=false
local COR=Color3.fromRGB(255,0,0)
local alvoTeleport=nil
local highlights={}

-- ===== FUNÇÕES AUXILIARES =====
local function getChar() char=plr.Character;if char then root=char:FindFirstChild("HumanoidRootPart") end;return char end

-- ===== ESP =====
local function criarESP(personagem) if not personagem then return nil end;local h=Instance.new("Highlight");h.Name="ESP";h.Adornee=personagem;h.OutlineColor=COR;h.FillColor=COR;h.FillTransparency=1;h.OutlineTransparency=0;h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop;h.Parent=personagem;return h end
local function removerESP() for _,h in pairs(highlights)do if h and h.Parent then h:Destroy()end end;highlights={} end
local function atualizarESP() if ESP_ON then for _,j in pairs(Players:GetPlayers())do if j~=plr then local c=j.Character;if c and not highlights[j]then local h=criarESP(c);if h then highlights[j]=h end end end end else removerESP() end end

-- ===== KILL AURA (MATA DIRETO) =====
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
                    if dist<20 then -- Raio de 20 studs
                        -- MATA DIRETO (sem jogar pra longe, sem destruir)
                        pcall(function() 
                            h.Health = 0 
                            h:BreakJoints() -- Força a morte imediata
                        end)
                        print("💀 Matou: "..j.Name)
                    end
                end
            end
        end
    end
end

-- ===== TELEPORT GRUDAR =====
local function teleportGrudar()
    if not alvoTeleport then return end
    local c=getChar() if not c or not root then return end
    local alvoChar=alvoTeleport.Character
    if not alvoChar then return end
    local alvoRoot=alvoChar:FindFirstChild("HumanoidRootPart")
    if not alvoRoot then return end
    
    -- Fica grudado atrás do personagem
    local posAlvo=alvoRoot.Position
    local lookVector=alvoRoot.CFrame.LookVector
    local posAtras=posAlvo - lookVector*3 -- 3 studs atrás
    
    -- Teleporta o jogador
    root.CFrame=CFrame.new(posAtras, posAlvo)
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
    
    -- KILL AURA (MATA DIRETO)
    if KILL_ON then
        killAura()
    end
    
    -- TELEPORT GRUDAR
    if GRUDAR_ON then
        teleportGrudar()
    end
end)

-- ===== MENU =====
local menuAberto=true;local sg=nil;local f=nil;local btnReabrir=nil
local dropDownAberto=false;local dropList=nil

local function criarMenu()
    sg=Instance.new("ScreenGui");sg.Name="HackLab";sg.ResetOnSpawn=false;sg.Parent=plr:WaitForChild("PlayerGui")
    
    -- FRAME PRINCIPAL
    f=Instance.new("Frame");f.Size=UDim2.new(0,300,0,380);f.Position=UDim2.new(0.5,-150,0.5,-190);f.BackgroundColor3=Color3.fromRGB(10,10,20);f.BackgroundTransparency=0.1;f.BorderSizePixel=0;f.ClipsDescendants=true;f.Parent=sg
    local c=Instance.new("UICorner");c.CornerRadius=UDim.new(0,12);c.Parent=f
    
    -- TÍTULO
    local t=Instance.new("TextLabel");t.Size=UDim2.new(1,0,0,35);t.Position=UDim2.new(0,0,0,5);t.BackgroundTransparency=1;t.Text="💀 HACK LAB";t.TextColor3=Color3.fromRGB(255,255,255);t.TextScaled=true;t.Font=Enum.Font.GothamBold;t.Parent=f
    
    -- BOTÃO FECHAR
    local btnX=Instance.new("TextButton");btnX.Size=UDim2.new(0,35,0,35);btnX.Position=UDim2.new(1,-40,0,5);btnX.BackgroundColor3=Color3.fromRGB(200,50,50);btnX.Text="✕";btnX.TextColor3=Color3.fromRGB(255,255,255);btnX.TextScaled=true;btnX.Font=Enum.Font.GothamBold;btnX.Parent=f
    local cx=Instance.new("UICorner");cx.CornerRadius=UDim.new(0,8);cx.Parent=btnX
    
    -- SUBTÍTULO
    local st=Instance.new("TextLabel");st.Size=UDim2.new(1,0,0,15);st.Position=UDim2.new(0,0,0,37);st.BackgroundTransparency=1;st.Text="KILL AURA + TELEPORT GRUDAR";st.TextColor3=Color3.fromRGB(150,150,150);st.TextScaled=true;st.Font=Enum.Font.Gotham;st.Parent=f
    
    -- LINHA DIVISÓRIA
    local linha=Instance.new("Frame");linha.Size=UDim2.new(0.9,0,0,1);linha.Position=UDim2.new(0.05,0,0,55);linha.BackgroundColor3=Color3.fromRGB(255,255,255);linha.BackgroundTransparency=0.3;linha.Parent=f
    
    -- ===== FUNÇÃO PARA CRIAR BOTÕES =====
    local function criarBotao(nome,pos,cor)
        local b=Instance.new("TextButton");b.Size=UDim2.new(0.9,0,0,40);b.Position=UDim2.new(0.05,0,0,pos);b.BackgroundColor3=cor or Color3.fromRGB(40,40,50);b.BackgroundTransparency=0.2;b.Text=nome;b.TextColor3=Color3.fromRGB(255,255,255);b.TextScaled=true;b.Font=Enum.Font.GothamBold;b.Parent=f
        local cb=Instance.new("UICorner");cb.CornerRadius=UDim.new(0,6);cb.Parent=b
        return b
    end
    
    -- ===== BOTÕES =====
    local bESP=criarBotao("🔴 ESP: OFF",65)
    local bKILL=criarBotao("💀 KILL AURA: OFF",115,Color3.fromRGB(200,50,50))
    local bTELEPORT=criarBotao("🚀 TELEPORT GRUDAR: OFF",165,Color3.fromRGB(50,100,200))
    
    -- ===== DROPDOWN PARA ESCOLHER JOGADOR =====
    local dropBtn=Instance.new("TextButton");dropBtn.Size=UDim2.new(0.9,0,0,40);dropBtn.Position=UDim2.new(0.05,0,0,215);dropBtn.BackgroundColor3=Color3.fromRGB(40,40,50);dropBtn.BackgroundTransparency=0.2;dropBtn.Text="👤 SELECIONAR JOGADOR";dropBtn.TextColor3=Color3.fromRGB(255,255,255);dropBtn.TextScaled=true;dropBtn.Font=Enum.Font.GothamBold;dropBtn.Parent=f
    local cbDrop=Instance.new("UICorner");cbDrop.CornerRadius=UDim.new(0,6);cbDrop.Parent=dropBtn
    
    -- NOME DO JOGADOR SELECIONADO
    local nomeLabel=Instance.new("TextLabel");nomeLabel.Size=UDim2.new(0.9,0,0,20);nomeLabel.Position=UDim2.new(0.05,0,0,258);nomeLabel.BackgroundTransparency=1;nomeLabel.Text="Nenhum jogador selecionado";nomeLabel.TextColor3=Color3.fromRGB(150,150,150);nomeLabel.TextScaled=true;nomeLabel.Font=Enum.Font.Gotham;nomeLabel.Parent=f
    
    -- ===== DROPDOWN (LISTA DE JOGADORES) =====
    dropList=Instance.new("ScrollingFrame");dropList.Size=UDim2.new(0.9,0,0,130);dropList.Position=UDim2.new(0.05,0,0,238);dropList.BackgroundColor3=Color3.fromRGB(20,20,30);dropList.BackgroundTransparency=0.2;dropList.BorderSizePixel=0;dropList.CanvasSize=UDim2.new(0,0,0,0);dropList.ScrollBarThickness=5;dropList.Visible=false;dropList.Parent=f
    local cDropList=Instance.new("UICorner");cDropList.CornerRadius=UDim.new(0,6);cDropList.Parent=dropList
    
    -- ===== FUNÇÃO PARA ATUALIZAR DROPDOWN =====
    local function atualizarDropDown()
        for _,child in pairs(dropList:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        
        local y=0
        for _,j in pairs(Players:GetPlayers()) do
            if j~=plr then
                local btn=Instance.new("TextButton");btn.Size=UDim2.new(1,0,0,30);btn.Position=UDim2.new(0,0,0,y);btn.BackgroundColor3=Color3.fromRGB(40,40,50);btn.BackgroundTransparency=0.5;btn.Text=j.Name;btn.TextColor3=Color3.fromRGB(255,255,255);btn.TextScaled=true;btn.Font=Enum.Font.Gotham;btn.Parent=dropList
                local cb=Instance.new("UICorner");cb.CornerRadius=UDim.new(0,4);cb.Parent=btn
                
                btn.MouseButton1Click:Connect(function()
                    alvoTeleport=j
                    nomeLabel.Text="🎯 "..j.Name
                    dropList.Visible=false
                    dropDownAberto=false
                end)
                btn.TouchTap:Connect(function()
                    alvoTeleport=j
                    nomeLabel.Text="🎯 "..j.Name
                    dropList.Visible=false
                    dropDownAberto=false
                end)
                
                y=y+32
            end
        end
        dropList.CanvasSize=UDim2.new(0,0,0,y)
    end
    
    -- ===== FUNÇÃO DO DROPDOWN =====
    dropBtn.MouseButton1Click:Connect(function()
        dropDownAberto=not dropDownAberto
        dropList.Visible=dropDownAberto
        if dropDownAberto then atualizarDropDown() end
    end)
    dropBtn.TouchTap:Connect(function()
        dropDownAberto=not dropDownAberto
        dropList.Visible=dropDownAberto
        if dropDownAberto then atualizarDropDown() end
    end)
    
    -- ===== FUNÇÕES DOS BOTÕES =====
    bESP.MouseButton1Click:Connect(function()
        ESP_ON=not ESP_ON;bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF";bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50);atualizarESP()
    end)
    bESP.TouchTap:Connect(function()
        ESP_ON=not ESP_ON;bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF";bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50);atualizarESP()
    end)
    
    bKILL.MouseButton1Click:Connect(function()
        KILL_ON=not KILL_ON;bKILL.Text=KILL_ON and"💀 KILL AURA: ON"or"💀 KILL AURA: OFF";bKILL.BackgroundColor3=KILL_ON and Color3.fromRGB(200,0,0)or Color3.fromRGB(40,40,50)
    end)
    bKILL.TouchTap:Connect(function()
        KILL_ON=not KILL_ON;bKILL.Text=KILL_ON and"💀 KILL AURA: ON"or"💀 KILL AURA: OFF";bKILL.BackgroundColor3=KILL_ON and Color3.fromRGB(200,0,0)or Color3.fromRGB(40,40,50)
    end)
    
    bTELEPORT.MouseButton1Click:Connect(function()
        GRUDAR_ON=not GRUDAR_ON;bTELEPORT.Text=GRUDAR_ON and"🚀 TELEPORT GRUDAR: ON"or"🚀 TELEPORT GRUDAR: OFF";bTELEPORT.BackgroundColor3=GRUDAR_ON and Color3.fromRGB(50,100,200)or Color3.fromRGB(40,40,50)
    end)
    bTELEPORT.TouchTap:Connect(function()
        GRUDAR_ON=not GRUDAR_ON;bTELEPORT.Text=GRUDAR_ON and"🚀 TELEPORT GRUDAR: ON"or"🚀 TELEPORT GRUDAR: OFF";bTELEPORT.BackgroundColor3=GRUDAR_ON and Color3.fromRGB(50,100,200)or Color3.fromRGB(40,40,50)
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
print("💀 Hack Lab v9.1 - KILL AURA (MATA DIRETO) + TELEPORT GRUDAR")
