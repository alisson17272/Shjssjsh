-- ============================================
-- HACK LAB COMPACT v2.0 - <400 LINHAS
-- ============================================
local Players=game:GetService("Players");local LP=Players.LocalPlayer;local RS=game:GetService("RunService");local UIS=game:GetService("UserInputService");local Camera=workspace.CurrentCamera;local plr=LP;local char=nil;local root=nil

-- CONFIGS
local DIST=35;local COR=Color3.fromRGB(255,0,0);local ESP_ON=false;local AIM_ON=false;local SPD_ON=false;local FLY_ON=false;local NOCLIP_ON=false;local JUMP_ON=false
local flying=false;local flySpeed=50;local noclipEnabled=false;local jumpMultiplier=1
local highlights={};local alvoAtual=nil

-- FUNÇÕES AUXILIARES
local function getChar() char=plr.Character;if char then root=char:FindFirstChild("HumanoidRootPart") end;return char end
local function getHumanoid() local c=getChar();return c and c:FindFirstChild("Humanoid") end

-- ESP
local function criarESP(personagem) if not personagem then return nil end;local h=Instance.new("Highlight");h.Name="ESP";h.Adornee=personagem;h.OutlineColor=COR;h.FillColor=COR;h.FillTransparency=1;h.OutlineTransparency=0;h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop;h.Parent=personagem;return h end
local function removerESP() for _,h in pairs(highlights)do if h and h.Parent then h:Destroy()end end;highlights={} end
local function atualizarESP() if ESP_ON then for _,j in pairs(Players:GetPlayers())do if j~=plr then local c=j.Character;if c and not highlights[j]then local h=criarESP(c);if h then highlights[j]=h end end end end else removerESP() end end

-- AIMBOT
local function getAlvo() local c=getChar()if not c or not root then return nil end;local origem=root.Position;local alvo=nil;local menorDist=DIST;for _,j in pairs(Players:GetPlayers())do if j~=plr then local c2=j.Character;if c2 then local r=c2:FindFirstChild("HumanoidRootPart");local h=c2:FindFirstChild("Humanoid");if r and h and h.Health>0 then local dist=(r.Position-origem).Magnitude;if dist<menorDist then menorDist=dist;alvo=j end end end end end;return alvo end

-- SPEED
local function toggleSpeed() SPD_ON=not SPD_ON;local hum=getHumanoid();if hum then hum.WalkSpeed=SPD_ON and 32 or 16 end end

-- FLY
local function toggleFly() FLY_ON=not FLY_ON;flying=false;local hum=getHumanoid();if hum then hum.PlatformStand=FLY_ON end end
UIS.InputBegan:Connect(function(i,g)if g then return end;if FLY_ON and i.KeyCode==Enum.KeyCode.Space then flying=true end end)
UIS.InputEnded:Connect(function(i)if FLY_ON and i.KeyCode==Enum.KeyCode.Space then flying=false end end)

-- NOCLIP
local function toggleNoClip() NOCLIP_ON=not NOCLIP_ON;noclipEnabled=NOCLIP_ON end

-- JUMP
local function toggleJump() JUMP_ON=not JUMP_ON;local hum=getHumanoid();if hum then hum.JumpPower=JUMP_ON and 200 or 50 end end

-- LOOP PRINCIPAL
RS.RenderStepped:Connect(function()
    local c=getChar()
    -- Fly
    if FLY_ON and c and c:FindFirstChild("HumanoidRootPart") then
        local hrp=c.HumanoidRootPart;local vel=hrp.Velocity;local moveDir=Vector3.new(0,0,0);local f=UIS:IsKeyDown(Enum.KeyCode.W) and 1 or 0;local b=UIS:IsKeyDown(Enum.KeyCode.S) and -1 or 0;local a=UIS:IsKeyDown(Enum.KeyCode.A) and -1 or 0;local d=UIS:IsKeyDown(Enum.KeyCode.D) and 1 or 0;local up=UIS:IsKeyDown(Enum.KeyCode.Space) and 1 or 0;local down=UIS:IsKeyDown(Enum.KeyCode.LeftShift) and -1 or 0;moveDir=Vector3.new(a+d,up+down,f+b);if moveDir.Magnitude>0 then moveDir=moveDir.Unit*flySpeed else moveDir=Vector3.new(0,0,0) end;hrp.Velocity=Vector3.new(moveDir.X,vel.Y+moveDir.Y,moveDir.Z)
    end
    -- NoClip
    if NOCLIP_ON and c then for _,p in pairs(c:GetDescendants())do if p:IsA("BasePart")then p.CanCollide=false end end end
    -- ESP
    if ESP_ON then for _,j in pairs(Players:GetPlayers())do if j~=plr then local c2=j.Character;if c2 then if not highlights[j]then local h=criarESP(c2);if h then highlights[j]=h end elseif highlights[j].Adornee~=c2 then highlights[j]:Destroy();highlights[j]=nil;local h=criarESP(c2);if h then highlights[j]=h end end else if highlights[j]then highlights[j]:Destroy();highlights[j]=nil end end end end end
    -- Aimbot
    if AIM_ON then local alvo=getAlvo();if alvo then local c2=alvo.Character;if c2 then local r=c2:FindFirstChild("HumanoidRootPart");if r then Camera.CFrame=CFrame.new(Camera.CFrame.Position,r.Position)end end end end
end)

-- MENU
local function criarMenu()
    local sg=Instance.new("ScreenGui");sg.Name="HackLab";sg.ResetOnSpawn=false;sg.Parent=plr:WaitForChild("PlayerGui")
    local f=Instance.new("Frame");f.Size=UDim2.new(0,280,0,400);f.Position=UDim2.new(0.5,-140,0.5,-200);f.BackgroundColor3=Color3.fromRGB(10,10,20);f.BackgroundTransparency=0.1;f.BorderSizePixel=0;f.ClipsDescendants=true;f.Parent=sg
    local c=Instance.new("UICorner");c.CornerRadius=UDim.new(0,10);c.Parent=f
    local t=Instance.new("TextLabel");t.Size=UDim2.new(1,0,0,30);t.Position=UDim2.new(0,0,0,5);t.BackgroundTransparency=1;t.Text="🧪 HACK LAB";t.TextColor3=Color3.fromRGB(255,255,255);t.TextScaled=true;t.Font=Enum.Font.GothamBold;t.Parent=f
    local st=Instance.new("TextLabel");st.Size=UDim2.new(1,0,0,15);st.Position=UDim2.new(0,0,0,32);st.BackgroundTransparency=1;st.Text="LEGACY EDITION";st.TextColor3=Color3.fromRGB(150,150,150);st.TextScaled=true;st.Font=Enum.Font.Gotham;st.Parent=f
    -- Abas
    local af=Instance.new("Frame");af.Size=UDim2.new(1,0,0,30);af.Position=UDim2.new(0,0,0,50);af.BackgroundTransparency=1;af.Parent=f
    local function criarAba(nome,pos,cor)local b=Instance.new("TextButton");b.Size=UDim2.new(0.25,0,1,0);b.Position=UDim2.new(pos,0,0,0);b.BackgroundColor3=cor or Color3.fromRGB(40,40,50);b.Text=nome;b.TextColor3=Color3.fromRGB(255,255,255);b.TextScaled=true;b.Font=Enum.Font.GothamBold;b.Parent=af;local cc=Instance.new("UICorner");cc.CornerRadius=UDim.new(0,4);cc.Parent=b;return b end
    local aba1=criarAba("ESP",0,Color3.fromRGB(200,50,50));local aba2=criarAba("AIM",0.25,Color3.fromRGB(40,40,50));local aba3=criarAba("MOV",0.5,Color3.fromRGB(40,40,50));local aba4=criarAba("MISC",0.75,Color3.fromRGB(40,40,50))
    -- Conteúdo
    local cf=Instance.new("ScrollingFrame");cf.Size=UDim2.new(1,-10,0,290);cf.Position=UDim2.new(0,5,0,85);cf.BackgroundTransparency=1;cf.BorderSizePixel=0;cf.CanvasSize=UDim2.new(0,0,0,500);cf.ScrollBarThickness=5;cf.Parent=f
    local function btn(nome,desc,pos,cor)local b=Instance.new("TextButton");b.Size=UDim2.new(0.95,0,0,40);b.Position=UDim2.new(0.025,0,0,pos);b.BackgroundColor3=cor or Color3.fromRGB(40,40,50);b.BackgroundTransparency=0.2;b.Text=nome..": OFF";b.TextColor3=Color3.fromRGB(255,255,255);b.TextScaled=true;b.Font=Enum.Font.GothamBold;b.Parent=cf;local bc=Instance.new("UICorner");bc.CornerRadius=UDim.new(0,5);bc.Parent=b;local d=Instance.new("TextLabel");d.Size=UDim2.new(1,0,0,15);d.Position=UDim2.new(0,0,0,42);d.BackgroundTransparency=1;d.Text=desc;d.TextColor3=Color3.fromRGB(100,100,100);d.TextScaled=true;d.Font=Enum.Font.Gotham;d.Parent=b;return b end
    -- BOTÕES
    local bESP=btn("🔴 ESP","Contorno nos inimigos",0,Color3.fromRGB(200,50,50))
    local bAIM=btn("🎯 Aimbot","Mira automática 35s",50,Color3.fromRGB(50,200,50))
    local bSPD=btn("⚡ Speed","Velocidade x2",100,Color3.fromRGB(50,100,200))
    local bFLY=btn("🕊️ Fly","Voar (Espaço)",150,Color3.fromRGB(150,50,200))
    local bNOC=btn("🌀 NoClip","Atravessar paredes",200,Color3.fromRGB(200,150,50))
    local bJMP=btn("🦘 Inf. Jump","Pulo infinito",250,Color3.fromRGB(50,200,200))
    local botoes={bESP,bAIM,bSPD,bFLY,bNOC,bJMP}
    -- Sistema de abas
    local function mostrarAba(i)for x,b in ipairs(botoes)do b.Visible=(x>=i*2-1 and x<=i*2)end end
    local function resetAbas()aba1.BackgroundColor3=Color3.fromRGB(40,40,50);aba2.BackgroundColor3=Color3.fromRGB(40,40,50);aba3.BackgroundColor3=Color3.fromRGB(40,40,50);aba4.BackgroundColor3=Color3.fromRGB(40,40,50);aba1.TextColor3=Color3.fromRGB(200,200,200);aba2.TextColor3=Color3.fromRGB(200,200,200);aba3.TextColor3=Color3.fromRGB(200,200,200);aba4.TextColor3=Color3.fromRGB(200,200,200)end
    aba1.MouseButton1Click:Connect(function()resetAbas();aba1.BackgroundColor3=Color3.fromRGB(200,50,50);aba1.TextColor3=Color3.fromRGB(255,255,255);mostrarAba(1)end)
    aba2.MouseButton1Click:Connect(function()resetAbas();aba2.BackgroundColor3=Color3.fromRGB(50,200,50);aba2.TextColor3=Color3.fromRGB(255,255,255);mostrarAba(2)end)
    aba3.MouseButton1Click:Connect(function()resetAbas();aba3.BackgroundColor3=Color3.fromRGB(100,100,200);aba3.TextColor3=Color3.fromRGB(255,255,255);mostrarAba(3)end)
    aba4.MouseButton1Click:Connect(function()resetAbas();aba4.BackgroundColor3=Color3.fromRGB(200,150,50);aba4.TextColor3=Color3.fromRGB(255,255,255);mostrarAba(4)end)
    mostrarAba(1)
    -- Funções dos botões
    bESP.MouseButton1Click:Connect(function()ESP_ON=not ESP_ON;bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF";bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50);atualizarESP()end)
    bAIM.MouseButton1Click:Connect(function()AIM_ON=not AIM_ON;bAIM.Text=AIM_ON and"🎯 AIM: ON"or"🎯 AIM: OFF";bAIM.BackgroundColor3=AIM_ON and Color3.fromRGB(50,200,50)or Color3.fromRGB(40,40,50)end)
    bSPD.MouseButton1Click:Connect(function()toggleSpeed();bSPD.Text=SPD_ON and"⚡ Speed: ON"or"⚡ Speed: OFF";bSPD.BackgroundColor3=SPD_ON and Color3.fromRGB(50,100,200)or Color3.fromRGB(40,40,50)end)
    bFLY.MouseButton1Click:Connect(function()toggleFly();bFLY.Text=FLY_ON and"🕊️ Fly: ON"or"🕊️ Fly: OFF";bFLY.BackgroundColor3=FLY_ON and Color3.fromRGB(150,50,200)or Color3.fromRGB(40,40,50)end)
    bNOC.MouseButton1Click:Connect(function()toggleNoClip();bNOC.Text=NOCLIP_ON and"🌀 NoClip: ON"or"🌀 NoClip: OFF";bNOC.BackgroundColor3=NOCLIP_ON and Color3.fromRGB(200,150,50)or Color3.fromRGB(40,40,50)end)
    bJMP.MouseButton1Click:Connect(function()toggleJump();bJMP.Text=JUMP_ON and"🦘 Jump: ON"or"🦘 Jump: OFF";bJMP.BackgroundColor3=JUMP_ON and Color3.fromRGB(50,200,200)or Color3.fromRGB(40,40,50)end)
    -- Arrastar
    local drag=false;local startPos=nil;local framePos=nil
    f.InputBegan:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 then drag=true;startPos=i.Position;framePos=f.Position end end)
    f.InputChanged:Connect(function(i)if drag and i.UserInputType==Enum.UserInputType.MouseMovement then local d=i.Position-startPos;f.Position=UDim2.new(framePos.X.Scale,framePos.X.Offset+d.X,framePos.Y.Scale,framePos.Y.Offset+d.Y)end end)
    f.InputEnded:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 then drag=false end end)
end

-- INICIAR
criarMenu()
print("Hack Lab v2.0 carregado! (<400 linhas)")
