# Punishment-GUI-Full-control-over-toxic-players
Freeze &amp; Punish | Toxic handler menu


--[[
    Script: Punishment GUI - ENGLISH VERSION
    Fully translated script for Delta Executor
--]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- ========== FULL CLEANUP ==========
print("🔧 CLEANING UP...")

pcall(function()
    for _, conn in pairs(getconnections and getconnections(RunService.Stepped) or {}) do
        pcall(function() conn:Disconnect() end)
    end
end)

pcall(function()
    if CoreGui:FindFirstChild("PunishmentMenu") then CoreGui.PunishmentMenu:Destroy() end
end)

-- Unlock all players
pcall(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            local hum = player.Character:FindFirstChild("Humanoid")
            local hrp = player.Character:FindFirstChild("HumanoidRootPart")
            if hum then
                hum.WalkSpeed = 16
                hum.JumpPower = 50
                hum.PlatformStand = false
            end
            if hrp then
                hrp.Anchored = false
            end
        end
    end
end)

print("✅ CLEANUP COMPLETE")

-- ========== VARIABLES ==========
local currentVictim = nil
local loopConnection = nil
local isActive = false

-- SEND MESSAGE FUNCTION
local function sendMessage(message)
    local sayRequest = ReplicatedStorage:FindFirstChild("SayMessageRequest")
    if sayRequest then
        pcall(function() sayRequest:FireServer(message, "All") end)
    end
end

-- ========== STOP PUNISHMENT ==========
local function forceStop()
    print("🔴 STOPPING PUNISHMENT...")
    
    isActive = false
    
    if loopConnection then
        loopConnection:Disconnect()
        loopConnection = nil
    end
    
    if currentVictim then
        pcall(function()
            local char = currentVictim.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Anchored = false
                    end
                end
                local hum = char:FindFirstChild("Humanoid")
                if hum then
                    hum.WalkSpeed = 16
                    hum.JumpPower = 50
                    hum.PlatformStand = false
                end
            end
        end)
        print("✅ UNFROZEN:", currentVictim.DisplayName)
    end
    
    currentVictim = nil
end

-- ========== START PUNISHMENT ==========
local function blockPlayer(targetPlayer)
    if not targetPlayer then return end
    
    print("🚫 FREEZING:", targetPlayer.DisplayName)
    
    if isActive then
        forceStop()
        wait(0.2)
    end
    
    isActive = true
    currentVictim = targetPlayer
    
    pcall(function()
        if targetPlayer.Character then
            local hum = targetPlayer.Character:FindFirstChild("Humanoid")
            local hrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hum then
                hum.WalkSpeed = 0
                hum.JumpPower = 0
                hum.PlatformStand = true
            end
            if hrp then
                hrp.Anchored = true
            end
        end
    end)
    
    sendMessage("[SYSTEM] YOU SHOULDN'T HAVE DONE THAT...")
    wait(0.5)
    sendMessage("[SYSTEM] HOW PATHETIC")
    wait(0.5)
    sendMessage("[SYSTEM] YOU ARE FROZEN")
    
    loopConnection = RunService.Stepped:Connect(function()
        if not isActive or not currentVictim or currentVictim ~= targetPlayer then return end
        pcall(function()
            if targetPlayer and targetPlayer.Character then
                local hum = targetPlayer.Character:FindFirstChild("Humanoid")
                local hrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hum then
                    if hum.WalkSpeed ~= 0 then hum.WalkSpeed = 0 end
                    if hum.JumpPower ~= 0 then hum.JumpPower = 0 end
                    hum.PlatformStand = true
                end
                if hrp and not hrp.Anchored then
                    hrp.Anchored = true
                end
            end
        end)
    end)
end

-- ========== CREATE GUI MENU ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PunishmentMenu"
screenGui.Parent = CoreGui
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 450)
mainFrame.Position = UDim2.new(0.5, -140, 0.5, -225)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- TITLE BAR
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(1, 0, 1, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "⚡ PUNISHMENT MENU ⚡"
titleText.TextColor3 = Color3.fromRGB(255, 255, 255)
titleText.Font = Enum.Font.GothamBold
titleText.TextSize = 16
titleText.Parent = titleBar

-- DRAGGABLE WINDOW
local dragging = false
local dragStart, startPos

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)

titleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- PLAYER LIST
local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(1, -10, 0, 250)
playerList.Position = UDim2.new(0, 5, 0, 45)
playerList.BackgroundTransparency = 1
playerList.CanvasSize = UDim2.new(0, 0, 0, 0)
playerList.ScrollBarThickness = 5
playerList.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.SortOrder = Enum.SortOrder.Name
listLayout.Padding = UDim.new(0, 4)
listLayout.Parent = playerList

local function updateList()
    for _, child in pairs(playerList:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end
    
    local playersList = Players:GetPlayers()
    local height = 0
    
    for _, plr in pairs(playersList) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, -5, 0, 32)
        btn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
        btn.BorderSizePixel = 0
        btn.Text = plr.DisplayName
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 14
        btn.Parent = playerList
        
        if currentVictim == plr and isActive then
            btn.BackgroundColor3 = Color3.fromRGB(120, 30, 30)
            btn.Text = plr.DisplayName .. " 🔒"
        end
        
        btn.MouseButton1Click:Connect(function()
            blockPlayer(plr)
            updateList()
        end)
        
        height = height + 36
    end
    
    playerList.CanvasSize = UDim2.new(0, 0, 0, height + 5)
end

-- BUTTONS
local stopBtn = Instance.new("TextButton")
stopBtn.Size = UDim2.new(0, 130, 0, 40)
stopBtn.Position = UDim2.new(0, 5, 0, 305)
stopBtn.BackgroundColor3 = Color3.fromRGB(200, 40, 40)
stopBtn.Text = "🛑 STOP"
stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopBtn.Font = Enum.Font.GothamBold
stopBtn.TextSize = 14
stopBtn.Parent = mainFrame

stopBtn.MouseButton1Click:Connect(function()
    forceStop()
    updateList()
end)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(1, -10, 0, 35)
closeBtn.Position = UDim2.new(0, 5, 0, 355)
closeBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
closeBtn.Text = "❌ CLOSE MENU"
closeBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 13
closeBtn.Parent = mainFrame

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- AUTO UPDATE
Players.PlayerAdded:Connect(function() updateList() end)
Players.PlayerRemoving:Connect(function(plr)
    if currentVictim == plr then forceStop() end
    updateList()
end)

-- RESPAWN HANDLER
local function onRespawn(plr, char)
    if isActive and currentVictim == plr then
        wait(0.3)
        pcall(function()
            local hum = char:FindFirstChild("Humanoid")
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hum then
                hum.WalkSpeed = 0
                hum.JumpPower = 0
                hum.PlatformStand = true
            end
            if hrp then
                hrp.Anchored = true
            end
        end)
    end
end

for _, plr in pairs(Players:GetPlayers()) do
    plr.CharacterAdded:Connect(function(char) onRespawn(plr, char) end)
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char) onRespawn(plr, char) end)
end)

updateList()

print("========================================")
print("✅ SCRIPT LOADED")
print("📌 Displays player names above their head")
print("📌 Drag the window by the title bar")
print("📌 STOP button unfreezes the victim")
print("========================================")
