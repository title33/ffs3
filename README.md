-- // Main Variables // --

local RunService, UserInputService, TweenService = game.RunService, game.UserInputService, game.TweenService
local Player, ReplicatedStorage, Debris = game.Players.LocalPlayer, game.ReplicatedStorage, game.Debris

local Remotes = {
    ThunderDash = ReplicatedStorage.Remotes.ThunderDash,
    Emote = ReplicatedStorage.Remotes.CustomEmote,
    Parry = ReplicatedStorage.Remotes.ParryButtonPress
}

local Visual = false
local ParryEnabled = false -- เพิ่มตัวแปร ParryEnabled และกำหนดค่าเริ่มต้นเป็น false

local HitboxPart = Instance.new('Part', workspace)
HitboxPart.Color = Color3.fromHex('#f51d00')
HitboxPart.Anchored = true
HitboxPart.Material = Enum.Material.ForceField 
HitboxPart.Shape = Enum.PartType.Ball
HitboxPart.CanCollide = false
HitboxPart.CastShadow = false
HitboxPart.Transparency = 0.75

-- // Start of script // --
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
local function PerformParry(OBJ, ParryEnabled) -- แก้ไขการประกาศฟังก์ชัน PerformParry เพื่อรับ ParryEnabled
    if ParryEnabled then
        Remotes.Parry:Fire()
    end
end

RunService.Heartbeat:Connect(function(Time, DeltaTime)
    local Player = game.Players.LocalPlayer
for i, ball in pairs(workspace.Balls:GetChildren()) do
    if ball:GetAttribute('realBall') then
        local ballPosition = ball.Position
        local playerPosition = Player.Character.HumanoidRootPart.Position
        local distance = (playerPosition - ballPosition).Magnitude
        local ballVelocity = ball.Velocity.Magnitude
        local ballVolume = math.abs(ballVelocity.X + ballVelocity.Y + ballVelocity.Z)
        if Visual then
            HitboxPart.Position = playerPosition
            HitboxPart.Size = Vector3.new(ballVolume, ballVolume, ballVolume)
        else
            HitboxPart.Position = Vector3.new(0, 100000, 0)
        end
        if ball:GetAttribute('target') == Player.Name then
            local timeToReach = distance / ballVelocity -- คำนวณเวลาที่ลูกบอลจะถึงเหมือนกับคำนวณ distance/ballMagnitude
            if timeToReach <= 0.1 then -- ตั้งเวลาที่ลูกบอลจะถึงไปเป็น 0.1 วินาที
                PerformParry(ball, ParryEnabled)
            end
        end
    end
end


-- // extra // --

Library:Init()
