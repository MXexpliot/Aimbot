-- MobileAimbot.lua (Place in StarterPlayerScripts)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")

-- Player references
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera

-- Configuration
local AIMBOT_RANGE = 500 -- Very long range
local SNAP_STRENGTH = 0.95 -- Almost instant snap (0-1)
local LOCK_THROUGH_WALLS = true -- See through walls
local AUTO_SWITCH_TARGETS = true -- Switch to new better targets

-- Mobile settings
local IS_MOBILE = UserInputService.TouchEnabled
local AIMBOT_BUTTON_SIZE = UDim2.new(0, 100, 0, 100)
local AIMBOT_BUTTON_POSITION = UDim2.new(1, -120, 1, -120)

-- System variables
local lockedTarget = nil
local isAimbotActive = false
local espHighlights = {}
local aimbotButton = nil

-- Create mobile UI
function createMobileUI()
    if not IS_MOBILE then return end
    
    -- Aimbot toggle button
    aimbotButton = Instance.new("ImageButton")
    aimbotButton.Name = "AimbotButton"
    aimbotButton.Size = AIMBOT_BUTTON_SIZE
    aimbotButton.Position = AIMBOT_BUTTON_POSITION
    aimbotButton.AnchorPoint = Vector2.new(1, 1)
    aimbotButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    aimbotButton.BackgroundTransparency = 0.3
    aimbotButton.Image = "rbxassetid://3926305904"
    aimbotButton.ImageRectOffset = Vector2.new(524, 724)
    aimbotButton.ImageRectSize = Vector2.new(36, 36)
    aimbotButton.ImageColor3 = Color3.fromRGB(255, 0, 0)
    aimbotButton.ZIndex = 10
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = aimbotButton
    
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Color = Color3.fromRGB(255, 255, 255)
    uiStroke.Thickness = 3
    uiStroke.Parent = aimbotButton
    
    -- Status label
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "AimbotStatus"
    statusLabel.Size = UDim2.new(0, 200, 0, 30)
    statusLabel.Position = UDim2.new(0.5, -100, 0, 10)
    statusLabel.AnchorPoint = Vector2.new(0.5, 0)
    statusLabel.BackgroundTransparency = 0.8
    statusLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    statusLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    statusLabel.Text = "AIMBOT: OFF"
    statusLabel.TextSize = 16
    statusLabel.Font = Enum.Font.GothamBold
    statusLabel.ZIndex = 10
    
    local corner2 = Instance.new("UICorner")
    corner2.CornerRadius = UDim.new(0, 8)
    corner2.Parent = statusLabel
    
    -- Create screen GUI
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MobileAimbotUI"
    screenGui.Parent = playerGui
    screenGui.ResetOnSpawn = false
    
    aimbotButton.Parent = screenGui
    statusLabel.Parent = screenGui
    
    return aimbotButton, statusLabel
end

-- Find closest target regardless of walls or position
function findClosestTarget()
    local character = localPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return nil end
    
    local localRoot = character.HumanoidRootPart
    local closestTarget = nil
    local closestDistance = math.huge
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character then
            local targetChar = player.Character
            local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
            local humanoid = targetChar:FindFirstChild("Humanoid")
            
            if targetRoot and humanoid and humanoid.Health > 0 then
                -- Calculate distance (IGNORE WALLS)
                local distance = (targetRoot.Position - localRoot.Position).Magnitude
                
                if distance < closestDistance and distance <= AIMBOT_RANGE then
                    closestDistance = distance
                    closestTarget = targetChar
                end
            end
        end
    end
    
    return closestTarget
end

-- Smooth aimbot
function smoothAimToTarget(target)
    if not target or not target:FindFirstChild("HumanoidRootPart") then return end
    
    local targetRoot = target.HumanoidRootPart
    local targetPos = targetRoot.Position + Vector3.new(0, 1.5, 0) -- Head level
    
    -- Smooth but very strong aim
    local currentCFrame = camera.CFrame
    local currentLook = currentCFrame.LookVector
    local desiredLook = (targetPos - currentCFrame.Position).Unit
    
    -- Almost instant snap but slightly smoothed
    local smoothedLook = currentLook:Lerp(desiredLook, SNAP_STRENGTH)
    camera.CFrame = CFrame.lookAt(currentCFrame.Position, currentCFrame.Position + smoothedLook)
end

-- ESP through walls
function createWallhackESP(character)
    if not character or espHighlights[character] then return end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "AimbotESP"
    highlight.Adornee = character
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.FillTransparency = 0.9
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- Through walls
    highlight.Parent = character
    
    espHighlights[character] = highlight
end

-- Toggle aimbot
function toggleAimbot()
    isAimbotActive = not isAimbotActive
    
    if isAimbotActive then
        lockedTarget = findClosestTarget()
        if lockedTarget then
            print("Aimbot LOCKED ON to target")
            -- Update button appearance
            if aimbotButton then
                aimbotButton.ImageColor3 = Color3.fromRGB(0, 255, 0)
                aimbotButton.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
            end
            -- Update status
            if statusLabel then
                statusLabel.Text = "AIMBOT: LOCKED"
                statusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
            end
        end
    else
        print("Aimbot DISABLED")
        lockedTarget = nil
        -- Update button appearance
        if aimbotButton then
            aimbotButton.ImageColor3 = Color3.fromRGB(255, 0, 0)
            aimbotButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        end
        -- Update status
        if statusLabel then
            statusLabel.Text = "AIMBOT: OFF"
            statusLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
        end
    end
end

-- Setup mobile controls
local aimbotButton, statusLabel = createMobileUI()

-- Input handling
if IS_MOBILE then
    -- Mobile touch button
    aimbotButton.MouseButton1Click:Connect(toggleAimbot)
    
    -- Double-tap to toggle
    local lastTapTime = 0
    UserInputService.TouchStarted:Connect(function(input)
        local currentTime = tick()
        if currentTime - lastTapTime < 0.3 then -- Double tap
            toggleAimbot()
        end
        lastTapTime = currentTime
    end)
else
    -- PC keyboard input
    UserInputService.InputBegan:Connect(function(input, processed)
        if processed then return end
        if input.KeyCode == Enum.KeyCode.F then
            toggleAimbot()
        end
    end)
end

-- Main aimbot loop
RunService.RenderStepped:Connect(function()
    -- Create ESP for all players
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character then
            createWallhackESP(player.Character)
        end
    end
    
    -- Auto-switch to better targets
    if AUTO_SWITCH_TARGETS and isAimbotActive then
        local newTarget = findClosestTarget()
        if newTarget and newTarget ~= lockedTarget then
            lockedTarget = newTarget
        end
    end
    
    -- Apply aimbot
    if isAimbotActive and lockedTarget then
        if not lockedTarget.Parent then
            lockedTarget = findClosestTarget()
        else
            smoothAimToTarget(lockedTarget)
        end
    end
end)

-- Cleanup
localPlayer.CharacterAdded:Connect(function()
    lockedTarget = nil
    isAimbotActive = false
    -- Reset UI
    if aimbotButton then
        aimbotButton.ImageColor3 = Color3.fromRGB(255, 0, 0)
        aimbotButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    end
    if statusLabel then
        statusLabel.Text = "AIMBOT: OFF"
        statusLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

print("MOBILE AIMBOT LOADED!")
if IS_MOBILE then
    print("Tap the red button or double-tap screen to toggle aimbot")
else
    print("Press F to toggle aimbot")
end
print("Features:")
print("- 360 degree lock-on")
print("- Through walls targeting")
print("- Mobile touch controls")
print("- Visual status indicator")
