-- // Main Variables // --
local RunService, UserInputService, TweenService = game:GetService("RunService"), game:GetService("UserInputService"), game:GetService("TweenService")
local Player, ReplicatedStorage, Debris = game.Players.LocalPlayer, game.ReplicatedStorage, game.Debris

local Remotes = {
    ThunderDash = ReplicatedStorage.Remotes.ThunderDash,
    Emote = ReplicatedStorage.Remotes.CustomEmote,
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
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= Player and player.Character and player.Character.PrimaryPart then
            local distance = (Player.Character.PrimaryPart.Position - player.Character.PrimaryPart.Position).Magnitude
            if distance < shortestDistance then
                shortestDistance = distance
                closestPlayer = player
            end
        end
    end

    return closestPlayer
end

local function UpdateBallPartPosition()
    local closestPlayer = GetClosestPlayer()
    if closestPlayer and closestPlayer.Character and closestPlayer.Character.PrimaryPart then
        BallPart.Position = closestPlayer.Character.PrimaryPart.Position
    end
end

RunService.Heartbeat:Connect(function()
    UpdateBallPartPosition()
end)

local function IsPlayerInsideBallPart()
    local closestPlayer = GetClosestPlayer()
    if closestPlayer and closestPlayer.Character and closestPlayer.Character.PrimaryPart then
        local distance = (closestPlayer.Character.PrimaryPart.Position - BallPart.Position).Magnitude
        if distance <= BallPart.Size.X / 2 then
            return true
        end
    end
    return false
end

local function ChangeBallPartColor(color)
    BallPart.Color = color
end

local function spamParryButtonPress()
    while true do
        game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
        wait() 
    end
end

RunService.Heartbeat:Connect(function()
    if IsPlayerInsideBallPart() then
        if PrimaryPart:FindFirstChild("Highlight") then
            spamParryButtonPress()
            ChangeBallPartColor(Color3.fromRGB(255, 0, 0)) -- เปลี่ยนสี BallPart เป็นแดง
        end
    else
        -- เมื่อผู้เล่นไม่ได้อยู่ใน BallPart ของ PrimaryPart ให้หยุด spamParryButtonPress() และเปลี่ยนสีของ BallPart เป็นขาว
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
    local Player = game.Players.LocalPlayer
    for i, ball in pairs(workspace.Balls:GetChildren()) do
        if ball:GetAttribute('realBall') then
            local distance = (Player.Character.HumanoidRootPart.Position - ball.Position).Magnitude
            local ballVelocity = ball.Velocity
            local ballMagnitude = ballVelocity.Magnitude / 3
            local ballVolume = math.abs(ballVelocity.X + ballVelocity.Y + ballVelocity.Z)
            if Visual then
                HitboxPart.Position = Player.Character.HumanoidRootPart.Position
                HitboxPart.Size = Vector3.new(ballVolume, ballVolume, ballVolume)
            else
                HitboxPart.Position = Vector3.new(0, 100000, 0)
            end
            if ball:GetAttribute('target') == Player.Name then
                if distance <= ballMagnitude or distance <= 15 then
                    PerformParry(ball, ParryEnabled)
                end
            end
        end
    end
end)

-- // extra // --

Library:Init()
