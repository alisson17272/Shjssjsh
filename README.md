-- Menu Hacks Mobile - ShiftLock Toggle
local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local Players = game.Players

local ESP_ENABLED = false
local ANTIFOG_ENABLED = false
local SHIFTLOCK_ENABLED = false
local SHIFTLOCK_ACTIVE = false  -- Estado real do lock

local ESP_DRAWINGS = {}
local shiftlockConnection = nil
local shiftButton = nil

-- ======================= MENU =======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HackMenuMobile"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = player:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0.42, 0, 0.52, 0)
Frame.Position = UDim2.new(0.29, 0, 0.24, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Title.Text = "🔧 Menu Hacks Mobile"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextScaled = true
Title.Parent = Frame

local function CreateButton(text, yOffset, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 55)
    btn.Position = UDim2.new(0.05, 0, 0, yOffset)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    btn.Text = text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamBold
    btn.Parent = Frame
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- ShiftLock no Menu
CreateButton("🔒 ShiftLock (Mobile)", 60, function()
    SHIFTLOCK_ENABLED = not SHIFTLOCK_ENABLED
    
    if SHIFTLOCK_ENABLED then
        -- Cria botão flutuante
        if not shiftButton then
            shiftButton = Instance.new("TextButton")
            shiftButton.Size = UDim2.new(0, 85, 0, 85)
            shiftButton.Position = UDim2.new(0.78, 0, 0.65, 0)
            shiftButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            shiftButton.BackgroundTransparency = 0.2
            shiftButton.Text = "🔒\nOFF"
            shiftButton.TextScaled = true
            shiftButton.TextColor3 = Color3.new(1,1,1)
            shiftButton.Parent = ScreenGui
            shiftButton.ZIndex = 200
            
            shiftButton.MouseButton1Click:Connect(function()
                SHIFTLOCK_ACTIVE = not SHIFTLOCK_ACTIVE
                
                if SHIFTLOCK_ACTIVE then
                    shiftButton.Text = "🔒\nON"
                    shiftButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                    
                    shiftlockConnection = RunService.RenderStepped:Connect(function()
                        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                            local root = player.Character.HumanoidRootPart
                            local camLook = camera.CFrame.LookVector
                            root.CFrame = CFrame.new(root.Position, root.Position + Vector3.new(camLook.X, 0, camLook.Z))
                        end
                    end)
                else
                    shiftButton.Text = "🔒\nOFF"
                    shiftButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                    if shiftlockConnection then
                        shiftlockConnection:Disconnect()
                        shiftlockConnection = nil
                    end
                end
            end)
        end
        print("✅ ShiftLock habilitado - clique no botão vermelho")
    else
        -- Desliga tudo
        SHIFTLOCK_ACTIVE = false
        if shiftlockConnection then
            shiftlockConnection:Disconnect()
            shiftlockConnection = nil
        end
        if shiftButton then
            shiftButton:Destroy()
            shiftButton = nil
        end
        print("❌ ShiftLock desativado")
    end
end)

-- ESP
CreateButton("👁 ESP Círculo Vermelho", 125, function()
    ESP_ENABLED = not ESP_ENABLED
    if not ESP_ENABLED then
        for _, v in pairs(ESP_DRAWINGS) do v:Destroy() end
        ESP_DRAWINGS = {}
        print("❌ ESP DESATIVADO")
    else
        print("✅ ESP ATIVADO")
    end
end)

-- Anti Neblina
CreateButton("🌫 Anti-Neblina", 190, function()
    ANTIFOG_ENABLED = not ANTIFOG_ENABLED
    if ANTIFOG_ENABLED then
        lighting.FogEnd = 99999
        lighting.FogStart = 0
        lighting.Brightness = 3
        lighting.ClockTime = 14
        lighting.GlobalShadows = false
        print("✅ Anti-Neblina ATIVADO")
    else
        lighting.FogEnd = 1000
        lighting.Brightness = 1
        lighting.GlobalShadows = true
        print("❌ Anti-Neblina DESATIVADO")
    end
end)

-- Botão para abrir/fechar menu (flutuante)
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 65, 0, 65)
toggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
toggleBtn.Text = "☰"
toggleBtn.TextScaled = true
toggleBtn.Parent = ScreenGui

toggleBtn.MouseButton1Click:Connect(function()
    Frame.Visible = not Frame.Visible
end)

-- ==================== ESP LOOP ====================
RunService.RenderStepped:Connect(function()
    if not ESP_ENABLED then return end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr \~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local root = plr.Character.HumanoidRootPart
            local highlight = nil
            for _, ex in pairs(ESP_DRAWINGS) do
                if ex.Adornee == root then highlight = ex break end
            end
            if not highlight then
                highlight = Instance.new("Highlight")
                highlight.Adornee = root
                highlight.FillTransparency = 1
                highlight.OutlineColor = Color3.fromRGB(255, 0, 0)
                highlight.OutlineTransparency = 0
                highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                highlight.Parent = root
                table.insert(ESP_DRAWINGS, highlight)
            end
        end
    end
end)

print("✅ Menu Mobile carregado! Toque no botão azul para abrir.")
