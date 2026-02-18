-- TZ HUB V32 - AUTO-TRACKING STORM EDITION
if not game:IsLoaded() then game.Loaded:Wait() end

local p = game.Players.LocalPlayer
local pGui = p:WaitForChild("PlayerGui")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- CONFIGURAÇÕES
local ArcadePos = Vector3.new(901.6, -7.7, -15.8)
local DeepLevel, SkyLevel, TravelHeight = 25, 90, 150
local SpeedArcade, SpeedStorm = 0.8, 0.8

local AreaCoords = {
    ["legendary"] = Vector3.new(901.6, 70, -15.8), ["lendária"] = Vector3.new(901.6, 70, -15.8),
    ["mythical"] = Vector3.new(1500, 70, 200), ["mítica"] = Vector3.new(1500, 70, 200),
    ["epic"] = Vector3.new(500, 70, -100), ["épica"] = Vector3.new(500, 70, -100)
}

-- INTERFACE (TZ HUB V32)
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local ScrollingFrame = Instance.new("ScrollingFrame")
local Title = Instance.new("TextLabel")
local CloseBtn = Instance.new("TextButton")
local MinimizeBtn = Instance.new("TextButton")

local FrameSizeFull, FrameSizeMin = UDim2.new(0, 340, 0, 320), UDim2.new(0, 340, 0, 45)
pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end)
MainFrame.Parent = ScreenGui; MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20); MainFrame.Position = UDim2.new(0.35, 0, 0.2, 0); MainFrame.Size = FrameSizeFull; MainFrame.Active = true; MainFrame.Draggable = true; MainFrame.ClipsDescendants = true; Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 15)
local Stroke = Instance.new("UIStroke", MainFrame); Stroke.Thickness = 3; local StrokeGradient = Instance.new("UIGradient", Stroke); StrokeGradient.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 255, 255)), ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 100, 255))})
Title.Parent = MainFrame; Title.Size = UDim2.new(1, 0, 0, 45); Title.Text = "⚡ TZ HUB V32 ⚡"; Title.TextColor3 = Color3.new(1,1,1); Title.TextSize = 20; Title.Font = Enum.Font.LuckiestGuy; Title.BackgroundTransparency = 1
ScrollingFrame.Parent = MainFrame; ScrollingFrame.Size = UDim2.new(1, 0, 0, 240); ScrollingFrame.Position = UDim2.new(0, 0, 0, 50); ScrollingFrame.BackgroundTransparency = 1; ScrollingFrame.ScrollBarThickness = 0; Instance.new("UIListLayout", ScrollingFrame).Padding = UDim.new(0, 10); Instance.new("UIPadding", ScrollingFrame).PaddingLeft = UDim.new(0, 15)

local luckyActive, platActive, platPart, currentPlatY = false, false, nil, 0

local function EnsurePlat()
    if not platPart or not platPart.Parent then
        platPart = Instance.new("Part"); platPart.Name = "TZ_PLAT"; platPart.Size = Vector3.new(22, 1, 22); platPart.Transparency = 0.5; platPart.Color = Color3.fromRGB(0, 255, 255); platPart.Material = Enum.Material.Neon; platPart.Anchored = true; platPart.Parent = workspace
        platActive = true
    end
end

-- MOVIMENTO SINCRONIZADO
local function SmoothTravel(targetPos, isSky)
    local char = p.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    EnsurePlat()

    local travelY = isSky and (hrp.Position.Y + TravelHeight) or (hrp.Position.Y - DeepLevel)
    currentPlatY = travelY
    platPart.CFrame = CFrame.new(hrp.Position.X, travelY, hrp.Position.Z)
    hrp.CFrame = platPart.CFrame + Vector3.new(0, 3.5, 0)
    task.wait(0.05)

    local tween = TweenService:Create(platPart, TweenInfo.new(SpeedStorm, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(targetPos.X, travelY, targetPos.Z)})
    local connection = RunService.Heartbeat:Connect(function()
        if platPart and platPart.Parent then hrp.CFrame = platPart.CFrame + Vector3.new(0, 3.5, 0); hrp.Velocity = Vector3.new(0,0,0) end
    end)

    tween:Play(); tween.Completed:Wait(); connection:Disconnect()
    currentPlatY = isSky and SkyLevel or (targetPos.Y - 3.5)
    hrp.CFrame = CFrame.new(targetPos.X, currentPlatY + 3.5, targetPos.Z)
end

-- RASTREADOR DE EVENTO (TELA + CHAT)
local lastArea = ""
local function ProcessText(txt)
    if not luckyActive then return end
    local low = txt:lower()
    if low:find("lucky storm") or low:find("storm appeared") then
        for areaName, pos in pairs(AreaCoords) do
            if low:find(areaName) and areaName ~= lastArea then
                lastArea = areaName -- Salva a última área para evitar loops
                SmoothTravel(pos, true)
                break
            end
        end
    end
end

-- Monitorar Chat
p.Chatted:Connect(ProcessText)

-- Monitorar Interface (Vigia mudanças em tempo real)
pGui.DescendantAdded:Connect(function(obj)
    if obj:IsA("TextLabel") then
        ProcessText(obj.Text)
        obj:GetPropertyChangedSignal("Text"):Connect(function() ProcessText(obj.Text) end)
    end
end)

-- Botões
function AddBtn(txt, color, func)
    local Frame = Instance.new("Frame", ScrollingFrame); Frame.Size = UDim2.new(0, 310, 0, 45); Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30); Instance.new("UICorner", Frame)
    local Lbl = Instance.new("TextLabel", Frame); Lbl.Text = txt; Lbl.Size = UDim2.new(0.6, 0, 1, 0); Lbl.Position = UDim2.new(0.05, 0, 0, 0); Lbl.TextColor3 = Color3.new(1,1,1); Lbl.BackgroundTransparency = 1; Lbl.Font = Enum.Font.GothamBold; Lbl.TextXAlignment = 0
    local Btn = Instance.new("TextButton", Frame); Btn.Size = UDim2.new(0, 100, 0, 25); Btn.Position = UDim2.new(0.63, 0, 0.22, 0); Btn.BackgroundColor3 = color; Btn.Text = (txt:find("STORM") or txt:find("V32")) and "ATIVAR" or "IR"; Btn.TextColor3 = Color3.new(1,1,1); Btn.Font = Enum.Font.GothamBold; Instance.new("UICorner", Btn)
    Btn.MouseButton1Click:Connect(function() func(Btn) end)
end

AddBtn("🍀 LUCKY STORM (DETECTOR)", Color3.fromRGB(0, 100, 255), function(b)
    luckyActive = not luckyActive; b.Text = luckyActive and "✅ ON" or "ATIVAR"
    b.BackgroundColor3 = luckyActive and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(0, 100, 255)
    if not luckyActive then lastArea = "" end -- Reseta quando desliga
end)

AddBtn("🕹️ ARCADE (UNDER)", Color3.fromRGB(80, 0, 200), function() SmoothTravel(ArcadePos, false) end)

AddBtn("🛸 PLATAFORMA V32", Color3.fromRGB(0, 100, 255), function(b)
    platActive = not platActive
    if platActive then EnsurePlat(); b.Text = "ON"; b.BackgroundColor3 = Color3.fromRGB(0, 200, 180); currentPlatY = p.Character.HumanoidRootPart.Position.Y - 3.5
    else b.Text = "OFF"; b.BackgroundColor3 = Color3.fromRGB(0, 100, 255); if platPart then platPart:Destroy(); platPart = nil end end
end)

AddBtn("♻️ LIMPAR TUDO", Color3.fromRGB(200, 50, 50), function()
    for _, obj in pairs(workspace:GetChildren()) do if obj.Name == "TZ_PLAT" then obj:Destroy() end end
    platPart = nil; platActive = false; lastArea = ""
end)

-- Heartbeat e Jump
RunService.Heartbeat:Connect(function(dt)
    StrokeGradient.Rotation = (StrokeGradient.Rotation + 90 * dt) % 360
    if platActive and platPart and p.Character:FindFirstChild("HumanoidRootPart") then
        platPart.CFrame = CFrame.new(p.Character.HumanoidRootPart.Position.X, currentPlatY, p.Character.HumanoidRootPart.Position.Z)
    end
end)

UserInputService.JumpRequest:Connect(function()
    if platActive and p.Character:FindFirstChild("HumanoidRootPart") then task.wait(0.1); currentPlatY = p.Character.HumanoidRootPart.Position.Y - 3.5 end
end)

-- Minimizar/Fechar
CloseBtn.Parent = MainFrame; CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50); CloseBtn.Position = UDim2.new(0.88, 0, 0.03, 0); CloseBtn.Size = UDim2.new(0, 30, 0, 30); CloseBtn.Text = "X"; CloseBtn.TextColor3 = Color3.new(1, 1, 1); CloseBtn.ZIndex = 5; Instance.new("UICorner", CloseBtn); CloseBtn.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)
MinimizeBtn.Parent = MainFrame; MinimizeBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40); MinimizeBtn.Position = UDim2.new(0.77, 0, 0.03, 0); MinimizeBtn.Size = UDim2.new(0, 30, 0, 30); MinimizeBtn.Text = "-"; MinimizeBtn.TextColor3 = Color3.new(1, 1, 1); MinimizeBtn.ZIndex = 5; Instance.new("UICorner", MinimizeBtn)
MinimizeBtn.MouseButton1Click:Connect(function()
    local isMin = MainFrame.Size == FrameSizeMin
    TweenService:Create(MainFrame, TweenInfo.new(0.3), {Size = isMin and FrameSizeFull or FrameSizeMin}):Play()
    ScrollingFrame.Visible = isMin; MinimizeBtn.Text = isMin and "-" or "+"
end)
