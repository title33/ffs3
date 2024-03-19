-- // Main Variables // --
local RunService, TweenService = game:GetService("RunService"), game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = {
    Parry = ReplicatedStorage.Remotes.ParryButtonPress
}

local Visual = false
local ParryEnabled = false

local HitboxPart = Instance.new('Part', workspace)
HitboxPart.Color = Color3.fromRGB(255, 29, 0)
HitboxPart.Anchored = true
HitboxPart.Material = Enum.Material.ForceField
HitboxPart.Shape = Enum.PartType.Ball
HitboxPart.CanCollide = false
HitboxPart.CastShadow = false
HitboxPart.Transparency = 0.75

local BallPart = Instance.new("Part")
BallPart.Size = Vector3.new(20, 20, 20)
BallPart.Shape = Enum.PartType.Ball
BallPart.Material = Enum.Material.ForceField
BallPart.CanQuery = false
BallPart.CanTouch = false
BallPart.CanCollide = false
BallPart.CastShadow = false
BallPart.Color = Color3.fromRGB(255, 255, 255)
BallPart.Parent = workspace

-- // Start of script // --
local function GetClosestLivingEntity()
    local closestEntity = nil
    local shortestDistance = math.huge

    for _, entity in ipairs(workspace:GetChildren()) do
        if entity:IsA("Model") and entity:FindFirstChild("Humanoid") then
            local distance = (BallPart.Position - entity.PrimaryPart.Position).Magnitude
            if distance < shortestDistance then
                shortestDistance = distance
                closestEntity = entity
            end
        end
    end

    return closestEntity
end

local function UpdateBallPartPosition()
    local closestEntity = GetClosestLivingEntity()
    if closestEntity and closestEntity:FindFirstChild("HumanoidRootPart") then
        BallPart.Position = closestEntity.HumanoidRootPart.Position
    end
end

RunService.Heartbeat:Connect(function()
    UpdateBallPartPosition()
end)

local function IsEntityInsideBallPart(entity)
    if entity and entity:FindFirstChild("HumanoidRootPart") then
        local distance = (BallPart.Position - entity.HumanoidRootPart.Position).Magnitude
        return distance <= BallPart.Size.X / 2
    end
    return false
end

local function ChangeBallPartColor(color)
    BallPart.Color = color
end

local function spamParryButtonPress()
    while true do
        Remotes.Parry:Fire()
        wait() 
    end
end

RunService.Heartbeat:Connect(function()
    if IsEntityInsideBallPart(GetClosestLivingEntity()) then
        local closestEntity = GetClosestLivingEntity()
        if closestEntity and closestEntity:FindFirstChild("Highlight") then
            spamParryButtonPress()
            ChangeBallPartColor(Color3.fromRGB(255, 0, 0)) -- เปลี่ยนสี BallPart เป็นแดง
        end
    else
        -- เมื่อไม่มีสิ่งที่มีชีวิตอยู่ใน BallPart ให้หยุด spamParryButtonPress() และเปลี่ยนสีของ BallPart เป็นขาว
        spamParryButtonPress = function() end
        ChangeBallPartColor(Color3.fromRGB(255, 255, 255)) -- เปลี่ยนสี BallPart เป็นขาว
    end
end)

local Library = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

local MainWindow = Library:MakeWindow({
    Name = 'Blade',
    HidePremium = true,
    SaveConfig = true,
    ConfigFolder = 'Blade'
})

-- // tabs // --
local Combat = MainWindow:MakeTab({
    Name = 'Combat',
    Icon = 'rbxassetid://11385161113',
    PremiumOnly = false
})

-- // toggles n shit // --

Combat:AddToggle({
    Name = 'Auto-parry',
    Default = false,
    Callback = function (Value)
        ParryEnabled = Value
    end
})

Combat:AddToggle({
    Name = 'Visual',
    Default = false,
    Callback = function (Value)
        Visual = Value
    end
})

-- // main stuff // --
local function PerformParry(OBJ, ParryEnabled)
    if ParryEnabled then
        Remotes.Parry:Fire()
    end
end

RunService.Heartbeat:Connect(function(Time, DeltaTime)
    local closestEntity = GetClosestLivingEntity()
    if closestEntity then
        for _, ball in pairs(workspace.Balls:GetChildren()) do
            if ball:GetAttribute('realBall') then
                local distance = (closestEntity.HumanoidRootPart.Position - ball.Position).Magnitude
                local ballVelocity = ball.Velocity
                local ballMagnitude = ballVelocity.Magnitude / 3
                local ballVolume = math.abs(ballVelocity.X + ballVelocity.Y + ballVelocity.Z)
                if Visual then
                    HitboxPart.Position = closestEntity.HumanoidRootPart.Position
                    HitboxPart.Size = Vector3.new(ballVolume, ballVolume, ballVolume)
                else
                    HitboxPart.Position = Vector3.new(0, 100000, 0)
                end
                if ball:GetAttribute('target') == closestEntity.Name then
                    if distance <= ballMagnitude or distance <= 15 then
                        PerformParry(ball, ParryEnabled)
                    end
                end
            end
        end
    end
end)

-- // extra d // --

Library:Init()
