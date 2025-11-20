--[[
    WScripts Hub - StackSpot Flex
    ⚠️ Use por sua conta e risco!
--]]

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Funções de proteção básica (antiban/antikick)
local mt = getrawmetatable(game)
setreadonly(mt, false)
local oldNamecall = mt.__namecall

mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if method == "Kick" and self == LocalPlayer then
        return warn("[WScripts] Antikick ativado!")
    end
    return oldNamecall(self, ...)
end)

LocalPlayer.Kick = function() return warn("[WScripts] Antikick ativado!") end

-- Anticheat Detector (básico)
local function anticheatDetector()
    for _,v in pairs(getgc(true)) do
        if type(v) == "function" and islclosure(v) then
            local src = debug.getinfo(v).source
            if src and src:lower():find("anticheat") then
                warn("[WScripts] Anticheat detectado: "..src)
            end
        end
    end
end

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "WScriptsHub"
ScreenGui.Parent = game.CoreGui

-- Draggable Frame
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 350)
MainFrame.Position = UDim2.new(0.3, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30,30,40)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

-- Título
local Title = Instance.new("TextLabel")
Title.Text = "WScripts"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 28
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1,0,0,40)
Title.Parent = MainFrame

-- Botão de fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Text = "X"
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 20
CloseBtn.TextColor3 = Color3.fromRGB(255,80,80)
CloseBtn.Size = UDim2.new(0,40,0,40)
CloseBtn.Position = UDim2.new(1,-40,0,0)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Parent = MainFrame

-- Botão flutuante (bolinha)
local FloatBtn = Instance.new("TextButton")
FloatBtn.Text = "⚡"
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 30
FloatBtn.Size = UDim2.new(0,60,0,60)
FloatBtn.Position = UDim2.new(0.5,-30,0.5,-30)
FloatBtn.BackgroundColor3 = Color3.fromRGB(80,80,255)
FloatBtn.Visible = false
FloatBtn.Parent = ScreenGui

-- Função de abrir/fechar GUI
CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    FloatBtn.Visible = true
end)

FloatBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    FloatBtn.Visible = false
end)

-- Arrastar bolinha flutuante
local dragging, dragInput, dragStart, startPos

FloatBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = FloatBtn.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging=false end
        end)
    end
end)

FloatBtn.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        FloatBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
                                      startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Adicione aqui os botões das funções:
local function addButton(name, callback, y)
    local btn = Instance.new("TextButton")
    btn.Text = name
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.Size = UDim2.new(0.9,0,0,35)
    btn.Position = UDim2.new(0.05,0,0,(y*40)+50)
    btn.BackgroundColor3 = Color3.fromRGB(50+20*y,50+10*y,100+10*y)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Parent = MainFrame
    btn.MouseButton1Click:Connect(callback)
end

-- Funções dos hacks (exemplo simples):

-- Wallhack/ESP colorido por time:
local espEnabled = false
function toggleESP()
    espEnabled = not espEnabled
    if espEnabled then
        for _,plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
                local bill = Instance.new("BillboardGui", plr.Character.Head)
                bill.Name="WScriptsESP"
                bill.Size=UDim2.new(0,100,0,40)
                bill.AlwaysOnTop=true

                local txt=Instance.new("TextLabel",bill)
                txt.Size=UDim2.new(1,0,1,0)
                txt.BackgroundTransparency=1
                txt.Text=plr.Name

                -- Cores por time (exemplo genérico)
                if plr.Team and plr.Team ~= LocalPlayer.Team then
                    txt.TextColor3=Color3.fromRGB(255,80,80) -- inimigo vermelho
                else
                    txt.TextColor3=Color3.fromRGB(80,255,80) -- aliado verde
                end
            end
        end
    else -- desativa ESPs criados:
        for _,plr in pairs(Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("Head") then
                local esp=plr.Character.Head:FindFirstChild("WScriptsESP")
                if esp then esp:Destroy() end
            end
        end
    end
end

-- Walkspeed:
function toggleWalkspeed()
    LocalPlayer.Character.Humanoid.WalkSpeed=LocalPlayer.Character.Humanoid.WalkSpeed==16 and 50 or 16
end

-- Fly (simples):
local flying=false; local flyConn=nil;
function toggleFly()
    flying=not flying;
    if flying then
        flyConn=RunService.RenderStepped:Connect(function()
            LocalPlayer.Character.Humanoid:ChangeState(11) -- PlatformStand true (voando)
            LocalPlayer.Character.HumanoidRootPart.Velocity=Vector3.new(Mouse.X-400,10,(Mouse.Y-300)/10)
        end)
    else if flyConn then flyConn:Disconnect() end; LocalPlayer.Character.Humanoid:ChangeState(8) end -- PlatformStand false (parado)
end

-- Godmode:
function toggleGod()
    for _,v in pairs(LocalPlayer.Character:GetDescendants()) do
        if v:IsA("BasePart") then v.CanCollide=false; v.Anchored=false; v.Transparency=0.5 end -- exemplo visual de godmode fake!
    end
end

-- Noclip:
local noclip=false; local ncConn=nil;
function toggleNoclip()
    noclip=not noclip;
    if noclip then 
        ncConn=RunService.Stepped:Connect(function()
            for _,v in pairs(LocalPlayer.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide=false end end 
        end)
    else if ncConn then ncConn:Disconnect() end; end 
end

-- Aimbot (simples):
local aimbotEnabled=false;
function toggleAimbot()
    aimbotEnabled=not aimbotEnabled;
end

RunService.RenderStepped:Connect(function()
    if aimbotEnabled then 
        local closest=nil; local dist=math.huge;
        for _,plr in pairs(Players:GetPlayers()) do 
            if plr~=LocalPlayer and plr.Team~=LocalPlayer.Team and plr.Character and plr.Character:FindFirstChild("Head") then 
                local pos=(workspace.CurrentCamera:WorldToViewportPoint(plr.Character.Head.Position))
                local mag=(Vector2.new(pos.X,pos.Y)-Vector2.new(Mouse.X,Mouse.Y)).Magnitude;
                if mag<dist then dist=mag; closest=plr; end 
            end 
        end 
        if closest and closest.Character and closest.Character:FindFirstChild("Head") then 
            mousemoverel((workspace.CurrentCamera:WorldToViewportPoint(closest.Character.Head.Position).X-Mouse.X)/5,
                         (workspace.CurrentCamera:WorldToViewportPoint(closest.Character.Head.Position).Y-Mouse.Y)/5) 
        end 
    end 
end)

-- Configurações do Aimbot (exemplo):
function aimbotConfig()
    game.StarterGui:SetCore("SendNotification", {Title="Aimbot", Text="Configurações não implementadas neste exemplo!"})
end

-- Adicionando botões:
addButton("Wallhack/ESP", toggleESP, 1)
addButton("Walkspeed", toggleWalkspeed, 2)
addButton("Fly", toggleFly, 3)
addButton("Godmode", toggleGod, 4)
addButton("Noclip", toggleNoclip, 5)
addButton("Aimbot", toggleAimbot, 6)
addButton("Config Aimbot", aimbotConfig, 7)

-- Anticheat detector ao abrir o hub:
anticheatDetector()
