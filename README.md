-- ============================================
-- AIMBOT PRO v2.0 - ESP + AIMBOT + FLY + WALKSPEED
-- ============================================
local Players=game:GetService("Players")
local RunService=game:GetService("RunService")
local UserInputService=game:GetService("UserInputService")
local LP=Players.LocalPlayer
local Camera=workspace.CurrentCamera
local plr=LP
local char=nil
local root=nil

-- ===== CONFIGS =====
local ESP_ON=false
local AIMBOT_ON=false
local FLY_ON=false
local WALKSPEED_ON=false
local DISTANCIA_MAXIMA=35
local COR=Color3.fromRGB(255,0,0)
local highlights={}
local alvoAtual=nil
local flying=false
local flySpeed=50
local walkSpeedValue=16
local walkSpeedStep=4

-- ===== FUNÇÕES AUXILIARES =====
local function getChar() 
    char=plr.Character
    if char then 
        root=char:FindFirstChild("HumanoidRootPart") 
    end
    return char 
end

local function getHumanoid()
    local c=getChar()
    return c and c:FindFirstChild("Humanoid")
end

-- ===== ESP =====
local function criarESP(personagem)
    if not personagem then return nil end
    local h=Instance.new("Highlight")
    h.Name="ESP"
    h.Adornee=personagem
    h.OutlineColor=COR
    h.FillColor=COR
    h.FillTransparency=1
    h.OutlineTransparency=0
    h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop
    h.Parent=personagem
    return h
end

local function removerESP()
    for _,h in pairs(highlights) do
        if h and h.Parent then h:Destroy() end
    end
    highlights={}
end

local function atualizarESP()
    if ESP_ON then
        for _,j in pairs(Players:GetPlayers()) do
            if j~=plr then
                local c=j.Character
                if c and not highlights[j] then
                    local h=criarESP(c)
                    if h then highlights[j]=h end
                end
            end
        end
    else
        removerESP()
    end
end

-- ===== FLY =====
local function toggleFly()
    FLY_ON=not FLY_ON
    flying=false
    local hum=getHumanoid()
    if hum then
        hum.PlatformStand=FLY_ON
    end
    if FLY_ON then
        print("🕊️ Fly ATIVADO - Pressione ESPAÇO para voar")
    else
        print("🕊️ Fly DESATIVADO")
    end
end

-- ===== WALKSPEED =====
local function ajustarWalkSpeed(valor)
    local hum=getHumanoid()
    if hum then
        walkSpeedValue=math.clamp(walkSpeedValue+valor, 4, 200)
        hum.WalkSpeed=walkSpeedValue
        print("🏃 WalkSpeed: "..walkSpeedValue)
    end
end

local function toggleWalkSpeed()
    WALKSPEED_ON=not WALKSPEED_ON
    local hum=getHumanoid()
    if hum then
        if WALKSPEED_ON then
            walkSpeedValue=32
            hum.WalkSpeed=32
            print("🏃 WalkSpeed MAX: 32")
        else
            walkSpeedValue=16
            hum.WalkSpeed=16
            print("🏃 WalkSpeed NORMAL: 16")
        end
    end
end

-- ===== AIMBOT PRECISO =====
local function getMelhorAlvo()
    local c=getChar()
    if not c or not root then return nil end
    
    local origem=root.Position
    local melhor=nil
    local menorDist=DISTANCIA_MAXIMA
    
    for _,j in pairs(Players:GetPlayers()) do
        if j~=plr then
            local c2=j.Character
            if c2 then
                local r=c2:FindFirstChild("HumanoidRootPart")
                local h=c2:FindFirstChild("Humanoid")
                if r and h and h.Health>0 then
                    local dist=(r.Position-origem).Magnitude
                    if dist<menorDist then
                        menorDist=dist
                        melhor=j
                    end
                end
            end
        end
    end
    return melhor
end

-- ===== CONTROLES DO FLY =====
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if FLY_ON and input.KeyCode==Enum.KeyCode.Space then
        flying=true
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if FLY_ON and input.KeyCode==Enum.KeyCode.Space then
        flying=false
    end
end)

-- ===== LOOP PRINCIPAL =====
RunService.RenderStepped:Connect(function()
    local c=getChar()
    
    -- ESP
    if ESP_ON then
        for _,j in pairs(Players:GetPlayers()) do
            if j~=plr then
                local c2=j.Character
                if c2 then
                    if not highlights[j] then
                        local h=criarESP(c2)
                        if h then highlights[j]=h end
                    elseif highlights[j].Adornee~=c2 then
                        highlights[j]:Destroy()
                        highlights[j]=nil
                        local h=criarESP(c2)
                        if h then highlights[j]=h end
                    end
                else
                    if highlights[j] then
                        highlights[j]:Destroy()
                        highlights[j]=nil
                    end
                end
            end
        end
    end
    
    -- AIMBOT PRECISO
    if AIMBOT_ON then
        local alvo=getMelhorAlvo()
        if alvo then
            alvoAtual=alvo
            local c2=alvo.Character
            if c2 then
                local r=c2:FindFirstChild("HumanoidRootPart")
                if r then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, r.Position)
                end
            end
        else
            alvoAtual=nil
        end
    end
    
    -- FLY
    if FLY_ON and c and root then
        local hrp=root
        local vel=hrp.Velocity
        local moveDir=Vector3.new(0,0,0)
        
        local f=UserInputService:IsKeyDown(Enum.KeyCode.W) and 1 or 0
        local b=UserInputService:IsKeyDown(Enum.KeyCode.S) and -1 or 0
        local a=UserInputService:IsKeyDown(Enum.KeyCode.A) and -1 or 0
        local d=UserInputService:IsKeyDown(Enum.KeyCode.D) and 1 or 0
        local up=flying and 1 or 0
        local down=UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) and -1 or 0
        
        moveDir=Vector3.new(a+d, up+down, f+b)
        if moveDir.Magnitude>0 then
            moveDir=moveDir.Unit*flySpeed
        else
            moveDir=Vector3.new(0,0,0)
        end
        
        hrp.Velocity=Vector3.new(moveDir.X, vel.Y+moveDir.Y, moveDir.Z)
    end
end)

-- ===== MENU =====
local menuAberto=true
local sg=nil
local f=nil
local btnReabrir=nil

local function criarMenu()
    sg=Instance.new("ScreenGui")
    sg.Name="AimbotPro"
    sg.ResetOnSpawn=false
    sg.Parent=plr:WaitForChild("PlayerGui")
    
    -- ===== FRAME PRINCIPAL =====
    f=Instance.new("Frame")
    f.Size=UDim2.new(0,260,0,310)
    f.Position=UDim2.new(0.5,-130,0.5,-155)
    f.BackgroundColor3=Color3.fromRGB(10,10,20)
    f.BackgroundTransparency=0.1
    f.BorderSizePixel=0
    f.ClipsDescendants=true
    f.Parent=sg
    
    local c=Instance.new("UICorner")
    c.CornerRadius=UDim.new(0,12)
    c.Parent=f
    
    -- ===== TÍTULO =====
    local t=Instance.new("TextLabel")
    t.Size=UDim2.new(1,0,0,35)
    t.Position=UDim2.new(0,0,0,5)
    t.BackgroundTransparency=1
    t.Text="🎯 AIMBOT PRO"
    t.TextColor3=Color3.fromRGB(255,255,255)
    t.TextScaled=true
    t.Font=Enum.Font.GothamBold
    t.Parent=f
    
    -- ===== BOTÃO FECHAR =====
    local btnX=Instance.new("TextButton")
    btnX.Size=UDim2.new(0,35,0,35)
    btnX.Position=UDim2.new(1,-40,0,5)
    btnX.BackgroundColor3=Color3.fromRGB(200,50,50)
    btnX.Text="✕"
    btnX.TextColor3=Color3.fromRGB(255,255,255)
    btnX.TextScaled=true
    btnX.Font=Enum.Font.GothamBold
    btnX.Parent=f
    
    local cx=Instance.new("UICorner")
    cx.CornerRadius=UDim.new(0,8)
    cx.Parent=btnX
    
    -- ===== SUBTÍTULO =====
    local st=Instance.new("TextLabel")
    st.Size=UDim2.new(1,0,0,15)
    st.Position=UDim2.new(0,0,0,37)
    st.BackgroundTransparency=1
    st.Text="ESP + AIMBOT + FLY + WALKSPEED"
    st.TextColor3=Color3.fromRGB(150,150,150)
    st.TextScaled=true
    st.Font=Enum.Font.Gotham
    st.Parent=f
    
    -- ===== LINHA DIVISÓRIA =====
    local linha=Instance.new("Frame")
    linha.Size=UDim2.new(0.9,0,0,1)
    linha.Position=UDim2.new(0.05,0,0,55)
    linha.BackgroundColor3=Color3.fromRGB(255,255,255)
    linha.BackgroundTransparency=0.3
    linha.Parent=f
    
    -- ===== FUNÇÃO PARA CRIAR BOTÕES =====
    local function criarBotao(nome,pos,cor)
        local b=Instance.new("TextButton")
        b.Size=UDim2.new(0.9,0,0,36)
        b.Position=UDim2.new(0.05,0,0,pos)
        b.BackgroundColor3=cor or Color3.fromRGB(40,40,50)
        b.BackgroundTransparency=0.2
        b.Text=nome
        b.TextColor3=Color3.fromRGB(255,255,255)
        b.TextScaled=true
        b.Font=Enum.Font.GothamBold
        b.Parent=f
        
        local cb=Instance.new("UICorner")
        cb.CornerRadius=UDim.new(0,6)
        cb.Parent=b
        return b
    end
    
    -- ===== BOTÕES =====
    local bESP=criarBotao("🔴 ESP: OFF",65)
    local bAIM=criarBotao("🎯 AIMBOT: OFF",108,Color3.fromRGB(50,200,50))
    local bFLY=criarBotao("🕊️ FLY: OFF",151,Color3.fromRGB(100,50,200))
    local bWALK=criarBotao("🏃 WALKSPEED: NORMAL",194,Color3.fromRGB(200,150,50))
    
    -- ===== BOTÃO WALKSPEED + (AUMENTAR) =====
    local bWalkPlus=Instance.new("TextButton")
    bWalkPlus.Size=UDim2.new(0.4,0,0,30)
    bWalkPlus.Position=UDim2.new(0.05,0,0,238)
    bWalkPlus.BackgroundColor3=Color3.fromRGB(50,200,50)
    bWalkPlus.BackgroundTransparency=0.2
    bWalkPlus.Text="➕ VEL +"
    bWalkPlus.TextColor3=Color3.fromRGB(255,255,255)
    bWalkPlus.TextScaled=true
    bWalkPlus.Font=Enum.Font.GothamBold
    bWalkPlus.Parent=f
    local cbPlus=Instance.new("UICorner")
    cbPlus.CornerRadius=UDim.new(0,6)
    cbPlus.Parent=bWalkPlus
    
    -- ===== BOTÃO WALKSPEED - (DIMINUIR) =====
    local bWalkMinus=Instance.new("TextButton")
    bWalkMinus.Size=UDim2.new(0.4,0,0,30)
    bWalkMinus.Position=UDim2.new(0.55,0,0,238)
    bWalkMinus.BackgroundColor3=Color3.fromRGB(200,50,50)
    bWalkMinus.BackgroundTransparency=0.2
    bWalkMinus.Text="➖ VEL -"
    bWalkMinus.TextColor3=Color3.fromRGB(255,255,255)
    bWalkMinus.TextScaled=true
    bWalkMinus.Font=Enum.Font.GothamBold
    bWalkMinus.Parent=f
    local cbMinus=Instance.new("UICorner")
    cbMinus.CornerRadius=UDim.new(0,6)
    cbMinus.Parent=bWalkMinus
    
    -- ===== FUNÇÕES DOS BOTÕES =====
    -- ESP
    bESP.MouseButton1Click:Connect(function()
        ESP_ON=not ESP_ON
        bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF"
        bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50)
        atualizarESP()
    end)
    bESP.TouchTap:Connect(function()
        ESP_ON=not ESP_ON
        bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF"
        bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50)
        atualizarESP()
    end)
    
    -- AIMBOT
    bAIM.MouseButton1Click:Connect(function()
        AIMBOT_ON=not AIMBOT_ON
        bAIM.Text=AIMBOT_ON and"🎯 AIMBOT: ON"or"🎯 AIMBOT: OFF"
        bAIM.BackgroundColor3=AIMBOT_ON and Color3.fromRGB(50,200,50)or Color3.fromRGB(40,40,50)
        if AIMBOT_ON then
            print("🎯 Aimbot ATIVADO - Mirando no peito (35 studs)")
        else
            print("🎯 Aimbot DESATIVADO")
        end
    end)
    bAIM.TouchTap:Connect(function()
        AIMBOT_ON=not AIMBOT_ON
        bAIM.Text=AIMBOT_ON and"🎯 AIMBOT: ON"or"🎯 AIMBOT: OFF"
        bAIM.BackgroundColor3=AIMBOT_ON and Color3.fromRGB(50,200,50)or Color3.fromRGB(40,40,50)
        if AIMBOT_ON then
            print("🎯 Aimbot ATIVADO - Mirando no peito (35 studs)")
        else
            print("🎯 Aimbot DESATIVADO")
        end
    end)
    
    -- FLY
    bFLY.MouseButton1Click:Connect(function()
        toggleFly()
        bFLY.Text=FLY_ON and"🕊️ FLY: ON"or"🕊️ FLY: OFF"
        bFLY.BackgroundColor3=FLY_ON and Color3.fromRGB(100,50,200)or Color3.fromRGB(40,40,50)
    end)
    bFLY.TouchTap:Connect(function()
        toggleFly()
        bFLY.Text=FLY_ON and"🕊️ FLY: ON"or"🕊️ FLY: OFF"
        bFLY.BackgroundColor3=FLY_ON and Color3.fromRGB(100,50,200)or Color3.fromRGB(40,40,50)
    end)
    
    -- WALKSPEED TOGGLE
    bWALK.MouseButton1Click:Connect(function()
        toggleWalkSpeed()
        bWALK.Text=WALKSPEED_ON and"🏃 WALKSPEED: RÁPIDO"or"🏃 WALKSPEED: NORMAL"
        bWALK.BackgroundColor3=WALKSPEED_ON and Color3.fromRGB(200,150,50)or Color3.fromRGB(40,40,50)
    end)
    bWALK.TouchTap:Connect(function()
        toggleWalkSpeed()
        bWALK.Text=WALKSPEED_ON and"🏃 WALKSPEED: RÁPIDO"or"🏃 WALKSPEED: NORMAL"
        bWALK.BackgroundColor3=WALKSPEED_ON and Color3.fromRGB(200,150,50)or Color3.fromRGB(40,40,50)
    end)
    
    -- WALKSPEED + (Aumentar)
    bWalkPlus.MouseButton1Click:Connect(function()
        ajustarWalkSpeed(walkSpeedStep)
    end)
    bWalkPlus.TouchTap:Connect(function()
        ajustarWalkSpeed(walkSpeedStep)
    end)
    
    -- WALKSPEED - (Diminuir)
    bWalkMinus.MouseButton1Click:Connect(function()
        ajustarWalkSpeed(-walkSpeedStep)
    end)
    bWalkMinus.TouchTap:Connect(function()
        ajustarWalkSpeed(-walkSpeedStep)
    end)
    
    -- ===== BOTÃO MINIMIZAR =====
    btnX.MouseButton1Click:Connect(function()
        menuAberto=false
        f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("TextButton")
            btnReabrir.Size=UDim2.new(0,50,0,50)
            btnReabrir.Position=UDim2.new(0.02,0,0.85,0)
            btnReabrir.BackgroundColor3=Color3.fromRGB(200,50,50)
            btnReabrir.Text="🎯"
            btnReabrir.TextColor3=Color3.fromRGB(255,255,255)
            btnReabrir.TextScaled=true
            btnReabrir.Font=Enum.Font.GothamBold
            btnReabrir.Parent=sg
            
            local cr=Instance.new("UICorner")
            cr.CornerRadius=UDim.new(0,25)
            cr.Parent=btnReabrir
            
            btnReabrir.MouseButton1Click:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
            btnReabrir.TouchTap:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
        else
            btnReabrir.Visible=true
        end
    end)
    btnX.TouchTap:Connect(function()
        menuAberto=false
        f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("TextButton")
            btnReabrir.Size=UDim2.new(0,50,0,50)
            btnReabrir.Position=UDim2.new(0.02,0,0.85,0)
            btnReabrir.BackgroundColor3=Color3.fromRGB(200,50,50)
            btnReabrir.Text="🎯"
            btnReabrir.TextColor3=Color3.fromRGB(255,255,255)
            btnReabrir.TextScaled=true
            btnReabrir.Font=Enum.Font.GothamBold
            btnReabrir.Parent=sg
            
            local cr=Instance.new("UICorner")
            cr.CornerRadius=UDim.new(0,25)
            cr.Parent=btnReabrir
            
            btnReabrir.MouseButton1Click:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
            btnReabrir.TouchTap:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
        else
            btnReabrir.Visible=true
        end
    end)
    
    -- ===== SISTEMA DE ARRASTAR =====
    local drag=false
    local startPos=nil
    local framePos=nil
    
    f.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            drag=true
            startPos=input.Position
            framePos=f.Position
        end
    end)
    f.InputChanged:Connect(function(input)
        if drag and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
            local delta=input.Position-startPos
            f.Position=UDim2.new(framePos.X.Scale,framePos.X.Offset+delta.X,framePos.Y.Scale,framePos.Y.Offset+delta.Y)
        end
    end)
    f.InputEnded:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            drag=false
        end
    end)
end

-- ===== INICIAR =====
criarMenu()
atualizarESP()
print("🎯 Aimbot Pro v2.0 carregado!")
print("📌 ESP: Contorno vermelho nos jogadores")
print("📌 Aimbot: Mira no peito a 35 studs")
print("📌 Fly: Pressione ESPAÇO para voar")
print("📌 WalkSpeed: Use + e - para ajustar")
