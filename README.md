local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

local WHITELIST_URL = "https://gist.githubusercontent.com/magkaspro/5984ee38246ac0315677c2cd14d0c4bf/raw/whitelist.json"

local success, data = pcall(function()
    return game:HttpGet(WHITELIST_URL .. "?v=" .. tostring(os.time()))
end)

if not success then
    LocalPlayer:Kick("Whitelist server error")
    return
end

local whitelist
success, whitelist = pcall(function()
    return HttpService:JSONDecode(data)
end)

if not success then
    LocalPlayer:Kick("Invalid whitelist")
    return
end

if not table.find(whitelist, LocalPlayer.Name) then
    LocalPlayer:Kick("Not Whitelisted")
    return
end


-- SERVICES
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local ProximityPromptService = game:GetService("ProximityPromptService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

local LocalPlayer = Players.LocalPlayer

if _G.RyxxInstantStealLoaded then return end
_G.RyxxInstantStealLoaded = true

-- DESYNC FFLAGS

local autoGrabActive = false
local autoGrabConnection = nil

local FFlags = {
    GameNetPVHeaderRotationalVelocityZeroCutoffExponent = -5000,
    LargeReplicatorWrite5 = true,
    LargeReplicatorEnabled9 = true,
    AngularVelociryLimit = 360,
    TimestepArbiterVelocityCriteriaThresholdTwoDt = 2147483646,
    S2PhysicsSenderRate = 15000,
    DisableDPIScale = true,
    MaxDataPacketPerSend = 2147483647,
    PhysicsSenderMaxBandwidthBps = 20000,
    TimestepArbiterHumanoidLinearVelThreshold = 21,
    MaxMissedWorldStepsRemembered = -2147483648,
    PlayerHumanoidPropertyUpdateRestrict = true,
    SimDefaultHumanoidTimestepMultiplier = 0,
    StreamJobNOUVolumeLengthCap = 2147483647,
    DebugSendDistInSteps = -2147483648,
    GameNetDontSendRedundantNumTimes = 1,
    CheckPVLinearVelocityIntegrateVsDeltaPositionThresholdPercent = 1,
    CheckPVDifferencesForInterpolationMinVelThresholdStudsPerSecHundredth = 1,
    LargeReplicatorSerializeRead3 = true,
    ReplicationFocusNouExtentsSizeCutoffForPauseStuds = 2147483647,
    CheckPVCachedVelThresholdPercent = 10,
    CheckPVDifferencesForInterpolationMinRotVelThresholdRadsPerSecHundredth = 1,
    GameNetDontSendRedundantDeltaPositionMillionth = 1,
    InterpolationFrameVelocityThresholdMillionth = 5,
    StreamJobNOUVolumeCap = 2147483647,
    InterpolationFrameRotVelocityThresholdMillionth = 5,
    CheckPVCachedRotVelThresholdPercent = 10,
    WorldStepMax = 30,
    InterpolationFramePositionThresholdMillionth = 5,
    TimestepArbiterHumanoidTurningVelThreshold = 1,
    SimOwnedNOUCountThresholdMillionth = 2147483647,
    GameNetPVHeaderLinearVelocityZeroCutoffExponent = -5000,
    NextGenReplicatorEnabledWrite4 = true,
    TimestepArbiterOmegaThou = 1073741823,
    MaxAcceptableUpdateDelay = 1,
    LargeReplicatorSerializeWrite4 = true
}

local defaultFFlags = {
    GameNetPVHeaderRotationalVelocityZeroCutoffExponent = 8,
    LargeReplicatorWrite5 = false,
    LargeReplicatorEnabled9 = false,
    AngularVelociryLimit = 180,
    TimestepArbiterVelocityCriteriaThresholdTwoDt = 100,
    S2PhysicsSenderRate = 60,
    DisableDPIScale = false,
    MaxDataPacketPerSend = 1024,
    PhysicsSenderMaxBandwidthBps = 10000,
    TimestepArbiterHumanoidLinearVelThreshold = 10,
    MaxMissedWorldStepsRemembered = 10,
    PlayerHumanoidPropertyUpdateRestrict = false,
    SimDefaultHumanoidTimestepMultiplier = 1,
    StreamJobNOUVolumeLengthCap = 1000,
    DebugSendDistInSteps = 10,
    GameNetDontSendRedundantNumTimes = 10,
    CheckPVLinearVelocityIntegrateVsDeltaPositionThresholdPercent = 50,
    CheckPVDifferencesForInterpolationMinVelThresholdStudsPerSecHundredth = 100,
    LargeReplicatorSerializeRead3 = false,
    ReplicationFocusNouExtentsSizeCutoffForPauseStuds = 100,
    CheckPVCachedVelThresholdPercent = 50,
    CheckPVDifferencesForInterpolationMinRotVelThresholdRadsPerSecHundredth = 100,
    GameNetDontSendRedundantDeltaPositionMillionth = 100,
    InterpolationFrameVelocityThresholdMillionth = 100,
    StreamJobNOUVolumeCap = 1000,
    InterpolationFrameRotVelocityThresholdMillionth = 100,
    CheckPVCachedRotVelThresholdPercent = 50,
    WorldStepMax = 60,
    InterpolationFramePositionThresholdMillionth = 100,
    TimestepArbiterHumanoidTurningVelThreshold = 10,
    SimOwnedNOUCountThresholdMillionth = 1000,
    GameNetPVHeaderLinearVelocityZeroCutoffExponent = 8,
    NextGenReplicatorEnabledWrite4 = false,
    TimestepArbiterOmegaThou = 1000,
    MaxAcceptableUpdateDelay = 10,
    LargeReplicatorSerializeWrite4 = false
}

-- DESYNC CORE

local ESPFolder, ServerESP
local serverPosition
local positionConn

local function ApplyFFlags(flags)
    flags = flags or FFlags
    for name, value in pairs(flags) do
        pcall(function()
            setfflag(tostring(name), tostring(value))
        end)
    end
end

local function RespawnPlayer()
    local char = LocalPlayer.Character
    if not char then return end

    local hum = char:FindFirstChildWhichIsA("Humanoid")
    if hum then
        hum:ChangeState(Enum.HumanoidStateType.Dead)
    end

    char:ClearAllChildren()
    local temp = Instance.new("Model", Workspace)
    LocalPlayer.Character = temp
    task.wait()
    LocalPlayer.Character = char
    temp:Destroy()
end

local function ClearESP()
    if positionConn then
        positionConn:Disconnect()
        positionConn = nil
    end
    if ESPFolder then
        ESPFolder:Destroy()
        ESPFolder = nil
    end
    ServerESP = nil
end

local function CreateESPPart(name, color)
    local part = Instance.new("Part")
    part.Size = Vector3.new(2, 5, 2)
    part.Anchored = true
    part.CanCollide = false
    part.Material = Enum.Material.Neon
    part.Color = color
    part.Transparency = 0.25
    part.Parent = ESPFolder

    local highlight = Instance.new("Highlight", part)
    highlight.FillColor = color
    highlight.OutlineColor = color
    highlight.FillTransparency = 0.4

    local bb = Instance.new("BillboardGui", part)
    bb.Size = UDim2.new(0, 130, 0, 30)
    bb.AlwaysOnTop = true
    bb.Adornee = part

    local txt = Instance.new("TextLabel", bb)
    txt.Size = UDim2.new(1, 0, 1, 0)
    txt.BackgroundTransparency = 1
    txt.Text = name
    txt.TextScaled = true
    txt.Font = Enum.Font.GothamBold
    txt.TextColor3 = color

    return part
end

local function TrackServer(hrp)
    serverPosition = hrp.Position
    positionConn = hrp:GetPropertyChangedSignal("Position"):Connect(function()
        task.wait(0.15)
        if hrp then
            serverPosition = hrp.Position
            if ServerESP then
                ServerESP.CFrame = CFrame.new(serverPosition)
            end
        end
    end)
end

local function SetServerESP()
    ClearESP()
    ESPFolder = Instance.new("Folder", Workspace)
    ESPFolder.Name = "DesyncESP"

    ServerESP = CreateESPPart("SERVER POSITION", Color3.fromRGB(0, 200, 255))

    local char = LocalPlayer.Character
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            TrackServer(hrp)
            ServerESP.CFrame = CFrame.new(serverPosition)
        end
    end
end

-- TARGET POSITIONS
local targetPositions = {
    Vector3.new(-481.88, -3.79, 138.02),
    Vector3.new(-481.75, -3.79, 89.18),
    Vector3.new(-481.82, -3.79, 30.95),
    Vector3.new(-481.75, -3.79, -17.79),
    Vector3.new(-481.80, -3.79, -76.06),
    Vector3.new(-481.72, -3.79, -124.70),
    Vector3.new(-337.45, -3.85, -124.72),
    Vector3.new(-337.37, -3.85, -76.07),
    Vector3.new(-337.46, -3.79, -17.72),
    Vector3.new(-337.41, -3.79, 30.92),
    Vector3.new(-337.32, -3.79, 89.02),
    Vector3.new(-337.27, -3.79, 137.90),
    Vector3.new(-337.45, -3.79, 196.29),
    Vector3.new(-337.37, -3.79, 244.91),
    Vector3.new(-481.72, -3.79, 196.21),
    Vector3.new(-481.76, -3.79, 244.92)
}

-- GUI - SPLIT WINDOWS

local gui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
gui.Name = "RyxxInstantStealUI"
gui.ResetOnSpawn = false

-- HELPER FUNCTION TO CREATE BUTTONS
local function makeButton(parent, text, y)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, -24, 0, 36)
    b.Position = UDim2.fromOffset(12, y)
    b.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    b.BackgroundTransparency = 0.05
    b.Text = text
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.TextColor3 = Color3.fromRGB(230, 230, 230)
    b.AutoButtonColor = false
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)

    b.MouseEnter:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), {BackgroundTransparency = 0}):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), {BackgroundTransparency = 0.05}):Play()
    end)

    return b
end

-- WINDOW 1 - INSTANT STEAL
local frame1 = Instance.new("Frame", gui)
frame1.Size = UDim2.fromOffset(225, 130)
frame1.Position = UDim2.fromScale(0.25, 0.5)
frame1.AnchorPoint = Vector2.new(0.5, 0.5)
frame1.BackgroundColor3 = Color3.fromRGB(24, 23, 28)
frame1.BackgroundTransparency = 0.05
frame1.Active = true
frame1.Draggable = true

Instance.new("UICorner", frame1).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", frame1).Color = Color3.fromRGB(0, 200, 255)

local title1 = Instance.new("TextLabel", frame1)
title1.Size = UDim2.new(1, -10, 0, 30)
title1.Position = UDim2.fromOffset(5, 5)
title1.BackgroundTransparency = 1
title1.Text = "Insta Steal - Z"
title1.Font = Enum.Font.GothamBold
title1.TextSize = 14
title1.TextColor3 = Color3.fromRGB(0, 220, 255)
title1.TextXAlignment = Enum.TextXAlignment.Center

local instantStealBtn = makeButton(frame1, "INSTANT STEAL", 40)

-- WINDOW 2 - DESYNC & NO WALK
local frame2 = Instance.new("Frame", gui)
frame2.Size = UDim2.fromOffset(225, 210)
frame2.Position = UDim2.fromScale(0.75, 0.5)
frame2.AnchorPoint = Vector2.new(0.5, 0.5)
frame2.BackgroundColor3 = Color3.fromRGB(24, 23, 28)
frame2.BackgroundTransparency = 0.05
frame2.Active = true
frame2.Draggable = true

Instance.new("UICorner", frame2).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", frame2).Color = Color3.fromRGB(0, 200, 255)

local title2 = Instance.new("TextLabel", frame2)
title2.Size = UDim2.new(1, -10, 0, 30)
title2.Position = UDim2.fromOffset(5, 5)
title2.BackgroundTransparency = 1
title2.Text = "Shrek Hub Desync"
title2.Font = Enum.Font.GothamBold
title2.TextSize = 14
title2.TextColor3 = Color3.fromRGB(0, 220, 255)
title2.TextXAlignment = Enum.TextXAlignment.Center

local status = Instance.new("TextLabel", frame2)
status.Size = UDim2.new(1, -10, 0, 20)
status.Position = UDim2.fromOffset(5, 38)
status.BackgroundTransparency = 1
status.Text = "Status: Ready"
status.Font = Enum.Font.GothamMedium
status.TextSize = 11
status.TextColor3 = Color3.fromRGB(0, 200, 255)
status.TextXAlignment = Enum.TextXAlignment.Center

local noWalkBtn = makeButton(frame2, "No Walk: OFF", 62)
local activateBtn = makeButton(frame2, "Activate", 105)
local desyncBtn = makeButton(frame2, "Desync: OFF", 148)

-- BIG BEAMS

local pos1, pos2
local beam1, beam2
local part1, part2

local function createBeam(position, index)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    local part = Instance.new("Part", Workspace)
    part.Anchored = true
    part.CanCollide = false
    part.Transparency = 1
    part.CFrame = CFrame.new(position)

    local a0 = Instance.new("Attachment", part)
    local a1 = Instance.new("Attachment", char.HumanoidRootPart)

    local beam = Instance.new("Beam", Workspace)
    beam.Attachment0 = a0
    beam.Attachment1 = a1
    beam.Width0 = 0.7
    beam.Width1 = 0.7
    beam.FaceCamera = true
    beam.Color = ColorSequence.new(Color3.fromRGB(0, 200, 255))
    beam.LightEmission = 1

    if index == 1 then
        if beam1 then beam1:Destroy() end
        if part1 then part1:Destroy() end
        beam1, part1 = beam, part
    else
        if beam2 then beam2:Destroy() end
        if part2 then part2:Destroy() end
        beam2, part2 = beam, part
    end
end

-- AUTO SET POSITION BACKWARD WITH HORIZONTAL OFFSET

local visualPart = nil

local function AutoSetPosition()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        -- Destroy old visual part
        if visualPart then
            visualPart:Destroy()
            visualPart = nil
        end
        
        -- Find closest target position to determine base location (left/right)
        local closest, dist = nil, math.huge
        for _, v in ipairs(targetPositions) do
            local d = (hrp.Position - v).Magnitude
            if d < dist then
                dist = d
                closest = v
            end
        end
        
        -- Determine which base and set appropriate position
        local teleportPosition
        if closest then
            if closest.X < -409 then
                -- Base is on the left, use left position
                teleportPosition = Vector3.new(-363.99, -7.30, 39.13)
                status.Text = "Status: LEFT BASE POSITION"
            else
                -- Base is on the right, use right position
                teleportPosition = Vector3.new(-371.71, -7.30, 82.94)
                status.Text = "Status: RIGHT BASE POSITION"
            end
        end
        
        if teleportPosition then
            pos1 = CFrame.new(teleportPosition)
            createBeam(pos1.Position, 1)
            
            -- Create visual block at position
            visualPart = Instance.new("Part", Workspace)
            visualPart.Shape = Enum.PartType.Ball
            visualPart.Size = Vector3.new(3, 3, 3)
            visualPart.Color = Color3.fromRGB(0, 200, 255)
            visualPart.Material = Enum.Material.Neon
            visualPart.CanCollide = false
            visualPart.CFrame = CFrame.new(teleportPosition)
            visualPart.Transparency = 0.3
            visualPart.Anchored = true
            visualPart.TopSurface = Enum.SurfaceType.Smooth
            visualPart.BottomSurface = Enum.SurfaceType.Smooth
            
            -- Add label
            local billboard = Instance.new("BillboardGui", visualPart)
            billboard.Size = UDim2.new(0, 150, 0, 50)
            billboard.MaxDistance = 500
            billboard.Adornee = visualPart
            
            local textLabel = Instance.new("TextLabel", billboard)
            textLabel.Size = UDim2.new(1, 0, 1, 0)
            textLabel.BackgroundTransparency = 1
            textLabel.Text = "STEAL POSITION"
            textLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
            textLabel.Font = Enum.Font.GothamBold
            textLabel.TextSize = 14
        end
    end
end

task.delay(2, function()
    AutoSetPosition()
end)

-- AUTO E PRESS FUNCTIONALITY - SMOOTH HOLD
local function PressESmooth()
    pcall(function()
        keypress(0x45)
    end)
    task.wait(2)
    pcall(function()
        keyrelease(0x45)
    end)
    status.Text = "Status: E Pressed for 2s"
end

-- NO WALK FUNCTIONALITY
local noWalkEnabled = false
local desyncActive = false

local function EnableNoWalk()
    local char = LocalPlayer.Character
    if not char then return end
    
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    local animate = char:FindFirstChild("Animate")
    if animate then
        animate.Disabled = true
    end
    
    for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
        track:Stop()
    end
    
    noWalkBtn.Text = "No Walk: ON"
    noWalkBtn.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
end

local function DisableNoWalk()
    local char = LocalPlayer.Character
    if not char then return end
    
    local animate = char:FindFirstChild("Animate")
    if animate then
        animate.Disabled = false
    end
    
    noWalkBtn.Text = "No Walk: OFF"
    noWalkBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
end

noWalkBtn.MouseButton1Click:Connect(function()
    noWalkEnabled = not noWalkEnabled
    
    if noWalkEnabled then
        EnableNoWalk()
    else
        DisableNoWalk()
    end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    if noWalkEnabled then
        EnableNoWalk()
    end
    -- Recreate visual part on respawn
    AutoSetPosition()
end)

activateBtn.MouseButton1Click:Connect(function()
    activateBtn.Text = "Activating..."
    activateBtn.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
    RespawnPlayer()
    task.wait(2)
    activateBtn.Text = "Activate"
    activateBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
end)

-- BUTTON LOGIC

desyncBtn.MouseButton1Click:Connect(function()
    desyncActive = not desyncActive
    
    if desyncActive then
        ApplyFFlags(FFlags)
        desyncBtn.Text = "Desync: ON"
        desyncBtn.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
        status.Text = "Status: Desync Active"
    else
        ApplyFFlags(defaultFFlags)
        desyncBtn.Text = "Desync: OFF"
        desyncBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        status.Text = "Status: Ready"
    end
end)

instantStealBtn.MouseButton1Click:Connect(function()
    PressESmooth()
end)

-- AUTO TARGET + STEAL

task.spawn(function()
    while task.wait(1) do
        local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            local closest, dist = nil, math.huge
            for _, v in ipairs(targetPositions) do
                local d = (hrp.Position - v).Magnitude
                if d < dist then
                    dist = d
                    closest = v
                end
            end
            if closest then
                pos2 = CFrame.new(closest)
                createBeam(pos2.Position, 2)
            end
        end
    end
end)

-- SIMULATE E KEY PRESS FOR 2 SECONDS - SMOOTH HOLD

local function PressEKeySmooth()
    pcall(function()
        keypress(0x45)
    end)
    task.wait(2)
    pcall(function()
        keyrelease(0x45)
    end)
end

ProximityPromptService.PromptButtonHoldEnded:Connect(function(prompt, who)
    if who ~= LocalPlayer then return end
    if prompt.Name ~= "Steal" and prompt.ActionText ~= "Steal" then return end

    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    if pos1 then hrp.CFrame = pos1 end
    if pos2 then task.wait(0.05); hrp.CFrame = pos2 end

    task.wait(0.1)
    PressEKeySmooth()
    
    status.Text = "Status: Steal Executed"
end)


-- DISCORD LABELS

local discord1 = Instance.new("TextLabel", frame1)
discord1.Size = UDim2.new(1,0,0,14)
discord1.Position = UDim2.fromOffset(0, 112)
discord1.BackgroundTransparency = 1
discord1.Text = "discord.gg/nvuAR5waS9"
discord1.Font = Enum.Font.GothamBold
discord1.TextSize = 9
discord1.TextColor3 = Color3.fromRGB(150, 150, 150)
discord1.TextXAlignment = Enum.TextXAlignment.Center

local discord2 = Instance.new("TextLabel", frame2)
discord2.Size = UDim2.new(1,0,0,14)
discord2.Position = UDim2.fromOffset(0, 190)
discord2.BackgroundTransparency = 1
discord2.Text = "discord.gg/nvuAR5waS9"
discord2.Font = Enum.Font.GothamBold
discord2.TextSize = 9
discord2.TextColor3 = Color3.fromRGB(150, 150, 150)
discord2.TextXAlignment = Enum.TextXAlignment.Center
